# NASTIES engine image — worker-comfyui + our custom node + our models baked
# in, for RunPod Serverless. Built from GitHub — no local Docker needed.
FROM runpod/worker-comfyui:5.8.6-base

# MMAudio custom node — registry install, with git-clone fallback.
RUN comfy node install ComfyUI-MMAudio || \
    (git clone --depth 1 https://github.com/kijai/ComfyUI-MMAudio \
      /comfyui/custom_nodes/ComfyUI-MMAudio && \
     pip install -r /comfyui/custom_nodes/ComfyUI-MMAudio/requirements.txt)

# Models — the Wan 2.2 5B engine set. 14B pair intentionally excluded
# (24GB-class workers). Keep the image lean enough to pull fast.
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

# Juggernaut XL Ragnarok — public HuggingFace mirror (7.11GB full checkpoint,
# anonymous download). Saved under the name our workflow expects.
ARG JUGGERNAUT_URL=https://huggingface.co/Granddyser/Juggernaut_XL_Ragnarok/resolve/main/juggernautXL_ragnarokBy.safetensors
RUN wget -q -O /comfyui/models/checkpoints/juggernautXL_ragnarok.safetensors "$JUGGERNAUT_URL?download=true"

# BUCKET_* credentials are read by worker-comfyui at RUNTIME — set them as
# RunPod endpoint env vars, never in this file.
