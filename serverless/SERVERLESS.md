# NASTIES — RunPod Serverless deploy runbook (RECONSTRUCTED)

> **Status: reconstructed 2026-07-22.** The original SERVERLESS.md was written in a
> Claude session and never committed; this version is rebuilt from the knowledge
> embedded in `serverless/Dockerfile` (which survived). Sections marked
> `[GAP]` are known to have existed but their exact contents are lost —
> re-verify against RunPod's console before relying on them.

## Overview

The NASTIES engine runs as a **RunPod Serverless** endpoint using the
`runpod/worker-comfyui` base image (tag `5.8.6-base`, verified current stable
against Docker Hub 2026-07-15). Our image bakes in:

- **Wan 2.2 5B (TI2V)** — the video engine (5B, for 24GB-class workers; the
  14B t2v/i2v pair is intentionally excluded).
- **MMAudio** (large 44k v2) — audio generation, with all four model files and
  the ComfyUI-MMAudio custom node.
- **Juggernaut XL Ragnarok** — image checkpoint (7.11GB, public HF mirror).
- **Juggernaut XL Inpainting** — masked inpaint pass ("WOUND IT" stage).
- **IP-Adapter SDXL (ViT-H)** + matching ViT-H CLIP vision encoder —
  reference-image conditioning.
- **Detailer stack** — face/hand YOLOv8 bbox detectors (Impact-Pack +
  Impact-Subpack), MeshGraphormer hand refiner (controlnet_aux), and
  4x-UltraSharp upscaler.
- **FLF2V custom node** — Wan 2.2 first/last-frame-to-video ("AIM THE KILL").

Total image size for this v2-LITE bake: **~42GB**.

## Deploy steps

1. **[GAP]** — (original steps 1–2 covered repo/RunPod account prep; exact
   contents lost.)
2. **[GAP]**
3. **Build via RunPod Hub "Deploy from GitHub"** — point RunPod at this repo;
   no local Docker needed. **HARD CONSTRAINT:** RunPod's GitHub builder has a
   **30-minute build cap** (hit 2026-07-16 at ~100GB). The v2-LITE image
   (~42GB) fits; the full bake with Hunyuan 1.5 + Chroma does NOT — those
   ship later via a **pre-built registry image** (RunPod's own recommended
   path for big images).
4. **Set endpoint env vars** (read by worker-comfyui at runtime, NOT build
   time — never bake them into the image):
   - `BUCKET_ENDPOINT_URL`
   - `BUCKET_ACCESS_KEY_ID`
   - `BUCKET_SECRET_ACCESS_KEY`
   - `BUCKET_NAME`

## Post-build checklist

- **TERMINATE ALL WORKERS after every image update.** FlashBoot keeps stale
  images + env on warm workers; a new build silently doesn't take effect
  until the old workers are killed.
- **Container disk: 120GB** (already set on the endpoint; fine for the ~42GB
  v2-LITE image).
- **Verify installs, don't trust exit codes.** `comfy node install
  ComfyUI-MMAudio` exits 0 on this base image WITHOUT installing anything
  (verified in production 2026-07-15). The Dockerfile now clones nodes
  unconditionally and verifies at build time (`test -d` + `pip show` on the
  first pinned requirement).

## Deliberately excluded from the v2-LITE image

| Item | Size | Why excluded | Ship path |
|---|---|---|---|
| Hunyuan 1.5 family | ~30GB | 30-min build cap can't fit it | Pre-built registry image |
| Chroma1-HD | ~23GB | Same | Pre-built registry image |
| Their text encoders (qwen_2.5_vl, t5xxl, flux ae) | — | Excluded with their engines — do not re-add | With registry image |
| Qwen-Image | 20.7GB | Title-art only, not on launch path | Registry image if ever needed |
| Civitai gore LoRAs | — | Need authenticated R2 mirrors (not built); Flux-family ones are non-commercial-licensed — dead on arrival for a paid platform | Blocked |

## Known follow-ups / launch blockers

- **R2 mirror for the Juggernaut inpaint checkpoint** — it currently pulls
  from an unofficial community HF mirror that could vanish. Public-launch
  blocker, not a build blocker.
- **Licensing** — both Juggernaut checkpoints are RAIL-M; William's licensing
  email covers both.
- **IP-Adapter pairing** — the ViT-H adapter must pair with the ViT-H
  encoder. Mismatching (e.g. with bigG) does NOT error; it silently degrades
  results.
