# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RunPod serverless worker that runs Automatic1111 Stable Diffusion WebUI and exposes its txt2img API endpoint. The worker is pre-packaged with the CyberRealisticPony model and serves as a containerized image generation service.

## Architecture

- **Docker containerization**: Single-stage build downloads model during build and sets up Python environment with Automatic1111 WebUI
- **RunPod serverless handler**: Python handler (`src/handler.py`) acts as proxy between RunPod requests and local WebUI API
- **Service initialization**: Startup script (`src/start.sh`) includes filesystem debugging and launches WebUI in API-only mode
- **Request flow**: Handler waits for WebUI readiness on port 3000, then forwards txt2img requests

## Core Components

- `src/handler.py`: RunPod serverless handler with retry logic and service health checking
- `src/start.sh`: WebUI startup with model path validation and extensive logging
- Model management: Downloads CyberRealisticPony_V11.0_FP16.safetensors, expects it at `/runpod-volume/ImageModel.safetensors`

## Development Commands

**Build Docker image:**
```bash
docker build -t worker-a1111 .
```

**Local development:**
```bash
python src/handler.py  # After WebUI is running locally
```

## WebUI Configuration

Runs with specific flags for serverless deployment:
- Port 3000 with `--api --nowebui` (API-only mode)
- `--ckpt /runpod-volume/ImageModel.safetensors` for model loading
- Performance flags: `--xformers --opt-sdp-attention`
- `--disable-safe-unpickle` for model compatibility

## API Interface

Accepts standard Automatic1111 `/sdapi/v1/txt2img` parameters. Input format matches the test_input.json structure with fields like prompt, negative_prompt, steps, cfg_scale, width, height, and sampler_name.