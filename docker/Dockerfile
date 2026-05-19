# ghcr.io/selimhehe1/pattaia-comfyui:v1
# Pre-built image with ComfyUI + custom nodes + pip deps + small models baked.
# Volume holds only big SDXL checkpoints (cyberrealistic_xl, IPAdapter, CLIP, ControlNet).
# Goal : pod cold-start 1-2 min (vs 10 min from scratch bootstrap).
#
# Pinned to commits validated 2026-05-19 on pod L4 obllggk660613a.
#
# Build : docker buildx build -t ghcr.io/selimhehe1/pattaia-comfyui:v1 .
# Push  : docker push ghcr.io/selimhehe1/pattaia-comfyui:v1

FROM runpod/pytorch:1.0.3-cu1281-torch260-ubuntu2204

ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    COMFYUI_PATH=/opt/comfyui

# === System deps ===
RUN apt-get update -qq && \
    apt-get install -qq -y \
        git git-lfs aria2 ffmpeg libgl1-mesa-glx libglib2.0-0 wget unzip curl \
        ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# === ComfyUI (pinned commit 2026-05-19) ===
WORKDIR /opt
RUN git clone https://github.com/comfyanonymous/ComfyUI.git comfyui && \
    cd comfyui && \
    git checkout d0328b442dd2ecc27bdc112bf6452b2e96aed4f8 && \
    pip install -r requirements.txt

# === Custom nodes (pinned commits) ===
WORKDIR /opt/comfyui/custom_nodes

# Impact-Pack
RUN git clone https://github.com/ltdrdata/ComfyUI-Impact-Pack && \
    cd ComfyUI-Impact-Pack && \
    git checkout 429d0159ad429e64d2b3916e6e7be9c22d025c3c && \
    pip install -r requirements.txt && \
    git clone --depth 1 https://github.com/ltdrdata/ComfyUI-Impact-Subpack impact_subpack

# IPAdapter_plus
RUN git clone https://github.com/cubiq/ComfyUI_IPAdapter_plus && \
    cd ComfyUI_IPAdapter_plus && \
    git checkout a0f451a5113cf9becb0847b92884cb10cbdec0ef && \
    if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

# controlnet_aux
RUN git clone https://github.com/Fannovel16/comfyui_controlnet_aux && \
    cd comfyui_controlnet_aux && \
    git checkout e8b689a513c3e6b63edc44066560ca5919c0576e && \
    pip install -r requirements.txt

# ReActor (pip non-fatal — insightface compile may warn but node loads)
RUN git clone https://github.com/Gourieff/ComfyUI-ReActor && \
    cd ComfyUI-ReActor && \
    git checkout 6ad6b35a4df250d14cb2abf0808c9ffedf59f747 && \
    (pip install -r requirements.txt || echo "⚠ ReActor pip warnings ignored")

# === Pre-create model dir tree ===
RUN mkdir -p /opt/comfyui/models/{checkpoints,loras,ipadapter,clip_vision,controlnet,embeddings,ultralytics/bbox,insightface/models,facedetection}

# === Small/medium models baked (~1.5 GB total) ===
# InsightFace antelopev2 (~280 MB) + buffalo_l (~280 MB)
RUN cd /tmp && \
    aria2c -x 8 -s 8 -o antelopev2.zip "https://github.com/deepinsight/insightface/releases/download/v0.7/antelopev2.zip" && \
    unzip -qo antelopev2.zip -d /opt/comfyui/models/insightface/models/ && \
    rm -f antelopev2.zip && \
    aria2c -x 8 -s 8 -o buffalo_l.zip "https://github.com/deepinsight/insightface/releases/download/v0.7/buffalo_l.zip" && \
    mkdir -p /opt/comfyui/models/insightface/models/buffalo_l && \
    unzip -qo buffalo_l.zip -d /opt/comfyui/models/insightface/models/buffalo_l/ && \
    rm -f buffalo_l.zip

# ReActor inswapper (~530 MB)
RUN aria2c -x 8 -s 8 \
    -o inswapper_128.onnx \
    -d /opt/comfyui/models/insightface \
    "https://huggingface.co/datasets/Gourieff/ReActor/resolve/main/models/inswapper_128.onnx"

# YOLO bbox detectors (~70 MB total) — small, bake for convenience
RUN aria2c -x 8 -s 8 \
    -o hand_yolov8s.pt -d /opt/comfyui/models/ultralytics/bbox \
    "https://huggingface.co/Bingsu/adetailer/resolve/main/hand_yolov8s.pt" && \
    aria2c -x 8 -s 8 \
    -o face_yolov8m.pt -d /opt/comfyui/models/ultralytics/bbox \
    "https://huggingface.co/Bingsu/adetailer/resolve/main/face_yolov8m.pt"

# === Working dir + entry ===
WORKDIR /opt/comfyui
EXPOSE 8188

# Default command (overridden by RunPod start_ssh anyway)
CMD ["python", "main.py", "--listen", "0.0.0.0", "--port", "8188"]
