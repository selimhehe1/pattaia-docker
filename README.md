# PattaIA ComfyUI Docker Image

Pre-built Docker image for PattaIA cloud pod RunPod : `ghcr.io/selimhehe1/pattaia-comfyui:v1`

Goal : cold-start pod 1-2 min (vs 10 min from-scratch bootstrap).

## Build via GitHub Actions

Actions tab → "Build PattaIA ComfyUI Image" → Run workflow → ~30-45 min.

## Pinned versions (validated 2026-05-19)

- Base : runpod/pytorch:1.0.3-cu1281-torch260-ubuntu2204
- ComfyUI : commit d0328b442d
- ComfyUI-Impact-Pack : 429d0159a
- ComfyUI_IPAdapter_plus : a0f451a51
- comfyui_controlnet_aux : e8b689a51
- ComfyUI-ReActor : 6ad6b35a4
