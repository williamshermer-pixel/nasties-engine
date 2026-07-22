# NASTIES — what exists where (recovery notes, 2026-07-22)

Written after an audit of the whole GitHub account and this repo, prompted by
"where's all my stuff."

## What is permanently saved (GitHub)

- `serverless/Dockerfile` — the complete v2-LITE image definition. This is
  the ONLY file ever pushed. 4 commits, all on `main`
  (093ee11 → b106207 → fe5d66c → c1bf381). No other branches, PRs, stashes,
  or orphaned commits exist (checked via reflog + fsck + account-wide repo
  search).

## What was never committed (existed only in chat sessions)

Referenced by the Dockerfile but absent from the repo:

- `SERVERLESS-SPEC.md` — the spec (Dockerfile cites its §7: worker-comfyui +
  custom nodes + models baked in).
- `SERVERLESS.md` — the original deploy runbook (steps 1–4+). A
  reconstruction now lives at `serverless/SERVERLESS.md`.
- `relaunch.sh` and `pod-setup.sh` — pod-era staging scripts for the Wan 2.2
  5B model set (pre-serverless workflow, likely superseded by the Dockerfile
  bake anyway).
- The ComfyUI workflow JSONs (the Dockerfile references an `IMAGE_CKPT`
  default of `juggernautXL_ragnarok.safetensors`, so at least one image
  workflow existed).

## Where the old chats likely are

Claude Code **CLI** sessions are stored locally on the machine where they ran
(`~/.claude/projects/`) and never sync to claude.ai — check the original
computer with `claude --resume` in the project directory. Connecting a new
device does not delete account history.

## Rule going forward

Anything produced in a session gets committed and pushed the same day.
Cloud-session containers are wiped between runs; chat is not durable storage.
