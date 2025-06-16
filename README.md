# ComfyUI Docker Build & Run Guide

This guide shows how to build GPU-enabled Docker images for [ComfyUI](https://github.com/comfyanonymous/ComfyUI) and run them locally.

## Prerequisites
* NVIDIA GPU with up-to-date drivers (CUDA 12.x capable).
* Docker ≥ 24 with the [NVIDIA Container Runtime](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) installed.
* (Optional) A local HTTP proxy at `127.0.0.1:8118` to accelerate downloads behind a firewall.

---

## 1️⃣ Build the Torch base image

Skip if you prefer to pull a pre-built runtime image.

```bash
# Pull the official CUDA runtime (optional)
docker pull nvidia/cuda:12.8.1-runtime-ubuntu22.04

# Build a minimal PyTorch layer on top (tag: comfyui-torch-base:cu128)
docker build \
  -f Dockerfile.torch-base \
  -t comfyui-torch-base:cu128 \
  .
```

## 2️⃣ Build the ComfyUI image

```bash
# Pin the desired ComfyUI release
export COMFYUI_VERSION=v0.3.40

# Build the ComfyUI image
docker build \
  -f Dockerfile.comfyui \
  --build-arg COMFYUI_VERSION=${COMFYUI_VERSION} \
  -t comfyui:${COMFYUI_VERSION}-cu128 \
  .
```

### Building through a proxy (optional)
If you rely on a local proxy, append the following to any `docker build` command:

```bash
--network=host \
--build-arg all_proxy=http://127.0.0.1:8118
```

Example:

```bash
docker build --network=host \
  --build-arg all_proxy=http://127.0.0.1:8118 \
  -f Dockerfile.torch-base \
  -t comfyui-torch-base:cu128 \
  .
```

## 3️⃣ Run ComfyUI

```bash
export COMFYUI_VERSION=v0.3.40

docker run -d \
  --name comfyui \
  --gpus all \
  -p 8188:8188 \
  -v /data/openwebui/comfyui:/opt/comfyui/data \  # persist models & outputs
  comfyui:${COMFYUI_VERSION}-cu128
```

Access the UI at http://localhost:8188/.

---

## 4️⃣ Prepare models (optional)

Download checkpoints or LoRA files into the mounted `models` directory, e.g.:

```bash
cd /data/openwebui/comfyui/models/checkpoints
aria2c -R -s 8 -j 8 --continue=true \
  https://modelscope.cn/models/Comfy-Org/stable-diffusion-v1-5-archive/resolve/master/v1-5-pruned-emaonly-fp16.safetensors
```

---

## 5️⃣ Updating & Cleaning up

* Restart the container: `docker restart comfyui`
* Stop and remove: `docker rm -f comfyui`
* To rebuild with a newer ComfyUI version, change `COMFYUI_VERSION` and repeat step 2.

---

Enjoy your local ComfyUI setup! 🚀