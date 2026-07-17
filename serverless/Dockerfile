# NASTIES engine image ‚Äî worker-comfyui + our custom nodes + our models baked
# in, for RunPod Serverless (per SERVERLESS-SPEC.md ¬ß7). Build via RunPod Hub
# "Deploy from GitHub" ‚Äî no local Docker needed (see SERVERLESS.md step 3).
#
# Tag verified against Docker Hub (runpod/worker-comfyui) 2026-07-15: 5.8.6 is
# the current stable release; "-base" is the plain ComfyUI variant (no extra
# baked-in checkpoints like the -sdxl/-flux1-* tags) ‚Äî the right base for us
# since we bring our own model set below.
#
# SIZE ‚Äî v1 targeted <30GB (Wan 5B only). This is v2, "the one big bake": it
# adds Hunyuan 1.5, Chroma1-HD, an inpainting checkpoint, IP-Adapter, and the
# detailer/hand-refiner stack on top of v1. Expected total image size is now
# ~100-105GB, NOT <30GB ‚Äî that old target is superseded, see below. Before
# this lands: raise the RunPod endpoint's CONTAINER DISK to >=120GB (an
# endpoint settings knob, not this file) or new workers can't even pull the
# image. FlashBoot-paused workers keep whatever image+env they last booted
# with ‚Äî after this build lands, TERMINATE ALL WORKERS on the endpoint so the
# next cold start actually pulls v2 instead of resuming a stale v1 worker.
FROM runpod/worker-comfyui:5.8.6-base

# ===========================================================================
# MODELS ‚Äî ordered first. These layers are huge (~100GB) and stable across
# rebuilds; custom nodes (small, change often) are cloned LAST below, so a
# node tweak invalidates only a cheap layer instead of re-pulling 100GB.
# ===========================================================================

# Wan 2.2 5B (TI2V) ‚Äî the exact set relaunch.sh/pod-setup.sh stage for the 5B
# engine. The 14B t2v/i2v pair is INTENTIONALLY EXCLUDED: these are 24GB-class
# workers running the Wan 2.2 5B (TI2V) engine, not the 14B pair. (The old
# "keep this image lean, target <30GB total" note here is superseded ‚Äî see
# the SIZE comment at the top of this file.)
ARG HF=https://huggingface.co/Comfy-Org/Wan_2.2_ComfyUI_Repackaged/resolve/main/split_files
ARG MM=https://huggingface.co/Kijai/MMAudio_safetensors/resolve/main

RUN mkdir -p /comfyui/models/checkpoints /comfyui/models/diffusion_models \
             /comfyui/models/vae /comfyui/models/text_encoders /comfyui/models/mmaudio

RUN wget -q -O /comfyui/models/vae/wan_2.1_vae.safetensors "$HF/vae/wan_2.1_vae.safetensors?download=true" && \
    wget -q -O /comfyui/models/vae/wan2.2_vae.safetensors "$HF/vae/wan2.2_vae.safetensors?download=true" && \
    wget -q -O /comfyui/models/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors "$HF/text_encoders/umt5_xxl_fp8_e4m3fn_scaled.safetensors?download=true" && \
    wget -q -O /comfyui/models/diffusion_models/wan2.2_ti2v_5B_fp16.safetensors "$HF/diffusion_models/wan2.2_ti2v_5B_fp16.safetensors?download=true"

RUN wget -q -O /comfyui/models/mmaudio/mmaudio_large_44k_v2_fp16.safetensors "$MM/mmaudio_large_44k_v2_fp16.safetensors?download=true" && \
    wget -q -O /comfyui/models/mmaudio/mmaudio_vae_44k_fp16.safetensors "$MM/mmaudio_vae_44k_fp16.safetensors?download=true" && \
    wget -q -O /comfyui/models/mmaudio/mmaudio_synchformer_fp16.safetensors "$MM/mmaudio_synchformer_fp16.safetensors?download=true" && \
    wget -q -O /comfyui/models/mmaudio/apple_DFN5B-CLIP-ViT-H-14-384_fp16.safetensors "$MM/apple_DFN5B-CLIP-ViT-H-14-384_fp16.safetensors?download=true"

# Juggernaut XL Ragnarok ‚Äî pulled straight from a public HuggingFace mirror
# (verified 2026-07-15: 7.11GB full checkpoint, anonymous 302‚ÜíCDN, no token).
# This removes the old pod‚ÜíR2 hop entirely: the image now bakes 100% of its
# models from public URLs at build time, zero pod dependency. The source repo
# names the file ...ragnarokBy.safetensors; we save it under the name our
# workflow's IMAGE_CKPT default expects (juggernautXL_ragnarok.safetensors).
ARG JUGGERNAUT_URL=https://huggingface.co/Granddyser/Juggernaut_XL_Ragnarok/resolve/main/juggernautXL_ragnarokBy.safetensors
RUN wget -q -O /comfyui/models/checkpoints/juggernautXL_ragnarok.safetensors "$JUGGERNAUT_URL?download=true"

# HUNYUAN 1.5 ‚Äî 480p t2v/i2v cfg-distilled + 720p super-res pass, plus the
# text/glyph encoders, VAE, and clip_vision it needs. All from one repackaged
# split_files tree, so one ARG base for the whole family like Wan above.
ARG HY=https://huggingface.co/Comfy-Org/HunyuanVideo_1.5_repackaged/resolve/main/split_files

RUN mkdir -p /comfyui/models/latent_upscale_models /comfyui/models/clip_vision

RUN wget -q -O /comfyui/models/diffusion_models/hunyuanvideo1.5_480p_t2v_cfg_distilled_fp8_scaled.safetensors "$HY/diffusion_models/hunyuanvideo1.5_480p_t2v_cfg_distilled_fp8_scaled.safetensors?download=true" && \
    wget -q -O /comfyui/models/diffusion_models/hunyuanvideo1.5_480p_i2v_cfg_distilled_fp8_scaled.safetensors "$HY/diffusion_models/hunyuanvideo1.5_480p_i2v_cfg_distilled_fp8_scaled.safetensors?download=true" && \
    wget -q -O /comfyui/models/diffusion_models/hunyuanvideo1.5_720p_sr_distilled_fp8_scaled.safetensors "$HY/diffusion_models/hunyuanvideo1.5_720p_sr_distilled_fp8_scaled.safetensors?download=true"

RUN wget -q -O /comfyui/models/latent_upscale_models/hunyuanvideo15_latent_upsampler_720p.safetensors "$HY/latent_upscale_models/hunyuanvideo15_latent_upsampler_720p.safetensors?download=true"

RUN wget -q -O /comfyui/models/text_encoders/qwen_2.5_vl_7b_fp8_scaled.safetensors "$HY/text_encoders/qwen_2.5_vl_7b_fp8_scaled.safetensors?download=true" && \
    wget -q -O /comfyui/models/text_encoders/byt5_small_glyphxl_fp16.safetensors "$HY/text_encoders/byt5_small_glyphxl_fp16.safetensors?download=true"

RUN wget -q -O /comfyui/models/vae/hunyuanvideo15_vae_fp16.safetensors "$HY/vae/hunyuanvideo15_vae_fp16.safetensors?download=true"

RUN wget -q -O /comfyui/models/clip_vision/sigclip_vision_patch14_384.safetensors "$HY/clip_vision/sigclip_vision_patch14_384.safetensors?download=true"

# CHROMA1-HD ‚Äî Apache 2.0, the flagship-photoreal bench candidate. Unlike the
# Flux/SD3 family this one is actually clear to ship commercially. Text
# encoder and VAE come from separate public repos, not a shared split_files
# tree, so no shared ARG base here.
RUN wget -q -O /comfyui/models/diffusion_models/Chroma1-HD.safetensors "https://huggingface.co/lodestones/Chroma1-HD/resolve/main/Chroma1-HD.safetensors?download=true" && \
    wget -q -O /comfyui/models/text_encoders/t5xxl_fp8_e4m3fn_scaled.safetensors "https://huggingface.co/comfyanonymous/flux_text_encoders/resolve/main/t5xxl_fp8_e4m3fn_scaled.safetensors?download=true"

# Flux VAE via a public non-gated mirror ‚Äî the BFL original is login-gated
# and we don't want an auth hop in the build.
RUN wget -q -O /comfyui/models/vae/ae.safetensors "https://huggingface.co/ffxvs/vae-flux/resolve/main/ae.safetensors?download=true"

# JUGGERNAUT XL INPAINTING ‚Äî masked inpaint pass (WOUND IT stage). Unofficial
# community mirror, could vanish ‚Äî an R2 mirror is queued as a follow-up.
# Same RAIL-M license as the Ragnarok checkpoint above; William's licensing
# email covers both. This is a public-launch blocker only, not a build
# blocker.
ARG JUGGERNAUT_INPAINT_URL=https://huggingface.co/WaterKnight/diffusion-models/resolve/main/checkpoints/juggernautXL_versionXInpaint.safetensors
RUN wget -q -O /comfyui/models/checkpoints/juggernautXL_inpaint.safetensors "$JUGGERNAUT_INPAINT_URL?download=true"

# IP-ADAPTER SDXL ‚Äî reference-image conditioning for ComfyUI_IPAdapter_plus
# (node cloned below). Must be the ViT-H image encoder, NOT the bigG one ‚Äî
# the vit-h adapter here only pairs with a vit-h encoder; mismatching them
# doesn't error, it just silently degrades results.
ARG IPA=https://huggingface.co/h94/IP-Adapter/resolve/main
RUN mkdir -p /comfyui/models/ipadapter
RUN wget -q -O /comfyui/models/ipadapter/ip-adapter_sdxl_vit-h.safetensors "$IPA/sdxl_models/ip-adapter_sdxl_vit-h.safetensors?download=true" && \
    wget -q -O /comfyui/models/clip_vision/clip_vision_vit_h.safetensors "$IPA/models/image_encoder/model.safetensors?download=true"

# DETAILERS ‚Äî face/hand yolo bbox detectors for Impact-Pack's
# UltralyticsDetectorProvider, plus an upscaler.
RUN mkdir -p /comfyui/models/ultralytics/bbox /comfyui/models/upscale_models
RUN wget -q -O /comfyui/models/ultralytics/bbox/face_yolov8m.pt "https://huggingface.co/Bingsu/adetailer/resolve/main/face_yolov8m.pt?download=true" && \
    wget -q -O /comfyui/models/ultralytics/bbox/hand_yolov8s.pt "https://huggingface.co/Bingsu/adetailer/resolve/main/hand_yolov8s.pt?download=true"

RUN wget -q -O /comfyui/models/upscale_models/4x-UltraSharp.pth "https://huggingface.co/uwg/upscaler/resolve/main/ESRGAN/4x-UltraSharp.pth?download=true"

# ===========================================================================
# CUSTOM NODES ‚Äî cloned last. Small, change often; keeping them after the
# ~100GB of model layers above means a node tweak only invalidates this
# cheap tail instead of re-pulling everything. Pip installs stay adjacent to
# their clone so each node owns one layer end to end.
# ===========================================================================

# Wan 2.2 first/last-frame-to-video latent helper ‚Äî pure python, no
# requirements.txt.
RUN git clone --depth 1 https://github.com/stduhpf/ComfyUI--Wan22FirstLastFrameToVideoLatent \
      /comfyui/custom_nodes/ComfyUI--Wan22FirstLastFrameToVideoLatent && \
    test -d /comfyui/custom_nodes/ComfyUI--Wan22FirstLastFrameToVideoLatent

# IPAdapter_plus ‚Äî we're only using the base IP-Adapter models baked above,
# not the FaceID variants, so no insightface / extra requirements needed.
RUN git clone --depth 1 https://github.com/cubiq/ComfyUI_IPAdapter_plus \
      /comfyui/custom_nodes/ComfyUI_IPAdapter_plus && \
    test -d /comfyui/custom_nodes/ComfyUI_IPAdapter_plus

# Impact-Pack + Impact-Subpack ‚Äî Subpack ships UltralyticsDetectorProvider
# separately from the main pack; both are required for the yolo detailers
# baked above to actually load.
RUN git clone --depth 1 https://github.com/ltdrdata/ComfyUI-Impact-Pack \
      /comfyui/custom_nodes/ComfyUI-Impact-Pack && \
    pip install -r /comfyui/custom_nodes/ComfyUI-Impact-Pack/requirements.txt && \
    test -d /comfyui/custom_nodes/ComfyUI-Impact-Pack

RUN git clone --depth 1 https://github.com/ltdrdata/ComfyUI-Impact-Subpack \
      /comfyui/custom_nodes/ComfyUI-Impact-Subpack && \
    pip install -r /comfyui/custom_nodes/ComfyUI-Impact-Subpack/requirements.txt && \
    pip install ultralytics && \
    test -d /comfyui/custom_nodes/ComfyUI-Impact-Subpack

# controlnet_aux ‚Äî the MeshGraphormer hand refiner lives inside this node;
# its weights are baked immediately below, right after this clone, because
# the node auto-downloads them at import time otherwise, and serverless
# workers must never fetch models at cold-start.
RUN git clone --depth 1 https://github.com/Fannovel16/comfyui_controlnet_aux \
      /comfyui/custom_nodes/comfyui_controlnet_aux && \
    pip install -r /comfyui/custom_nodes/comfyui_controlnet_aux/requirements.txt && \
    test -d /comfyui/custom_nodes/comfyui_controlnet_aux

# MeshGraphormer hand-refiner weights ‚Äî path convention confirmed straight
# from the node's util.py (custom_hf_download ‚Üí {node_dir}/ckpts/<repo_id>/
# <filename>, repo_id here is hr16/ControlNet-HandRefiner-pruned). Baking a
# model file here, after a node clone instead of in the MODELS section above,
# is the one deliberate exception to the "models first" cache ordering ‚Äî the
# destination directory doesn't exist until this node is cloned.
RUN mkdir -p /comfyui/custom_nodes/comfyui_controlnet_aux/ckpts/hr16/ControlNet-HandRefiner-pruned && \
    wget -q -O /comfyui/custom_nodes/comfyui_controlnet_aux/ckpts/hr16/ControlNet-HandRefiner-pruned/graphormer_hand_state_dict.bin \
      "https://huggingface.co/hr16/ControlNet-HandRefiner-pruned/resolve/main/graphormer_hand_state_dict.bin?download=true" && \
    wget -q -O /comfyui/custom_nodes/comfyui_controlnet_aux/ckpts/hr16/ControlNet-HandRefiner-pruned/hrnetv2_w64_imagenet_pretrained.pth \
      "https://huggingface.co/hr16/ControlNet-HandRefiner-pruned/resolve/main/hrnetv2_w64_imagenet_pretrained.pth?download=true"

# MMAudio ‚Äî the node-install path is a NO-OP TRAP: `comfy node install
# ComfyUI-MMAudio` exits 0 on this base image WITHOUT installing anything
# (verified 2026-07-15 in production ‚Äî the node silently never landed in the
# shipped image, no error anywhere in the build log). Lesson: don't trust
# ANY install step's exit code here. Fix is two-fold ‚Äî clone unconditionally
# instead of racing a fallback behind `||`, and verify at build time that the
# node actually landed: the directory exists AND its first pinned
# requirement resolves via `pip show`, not just "the command didn't fail."
RUN git clone --depth 1 https://github.com/kijai/ComfyUI-MMAudio \
      /comfyui/custom_nodes/ComfyUI-MMAudio && \
    pip install -r /comfyui/custom_nodes/ComfyUI-MMAudio/requirements.txt && \
    test -d /comfyui/custom_nodes/ComfyUI-MMAudio && \
    pip show $(grep -vE '^\s*(#|$)' /comfyui/custom_nodes/ComfyUI-MMAudio/requirements.txt | head -1 | sed -E 's/[<>=!~ ].*//') >/dev/null

# ---------------------------------------------------------------------------
# DELIBERATELY EXCLUDED from this image:
#   - Qwen-Image (20.7GB) ‚Äî title-art only, not on the launch path. Its text
#     encoder (qwen_2.5_vl_7b_fp8_scaled.safetensors) is already baked above
#     via Hunyuan 1.5 under an identical filename ‚Äî adding Qwen-Image later
#     is one wget for the diffusion model, not a second text-encoder pull.
#     DO NOT double-bake that encoder under a Qwen-Image-specific path.
#   - All Civitai gore LoRAs ‚Äî need authenticated R2 mirrors we haven't built
#     yet, and the Flux-family ones are non-commercial-licensed, which is
#     dead on arrival for a paid platform regardless of mirror.
# ---------------------------------------------------------------------------

# BUCKET_ENDPOINT_URL / BUCKET_ACCESS_KEY_ID / BUCKET_SECRET_ACCESS_KEY /
# BUCKET_NAME are read by worker-comfyui itself at runtime (not build time) ‚Äî
# set them as RunPod endpoint env vars (SERVERLESS.md step 4), not here.
