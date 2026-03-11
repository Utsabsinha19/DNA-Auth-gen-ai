# GenAuth AI — Installation Guide

## Prerequisites
- Docker 24+ & Docker Compose v2
- Python 3.11+
- Node.js 20+
- NVIDIA GPU (A100/H100 recommended for face + generation services)
- NVIDIA Container Toolkit (for GPU passthrough)

---

## Quick Start (Docker Compose)

```bash
git clone https://github.com/your-org/genauth-ai
cd genauth-ai

# Copy environment file
cp .env.example .env
# Edit .env with your secrets

# Start all services
docker compose -f deployment/docker/docker-compose.yml up -d

# Check health
curl http://localhost:8080/health
```

The platform will be available at:
- **Frontend**: http://localhost:3000
- **API Gateway**: http://localhost:8080
- **Face Service**: http://localhost:8002/docs
- **DNA Service**: http://localhost:8003/docs

---

## Manual Backend Setup

```bash
cd backend/face_service
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8002 --workers 4

cd backend/dna_service
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8003 --workers 2

cd backend/auth_service
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8001 --workers 2
```

---

## Frontend Setup

```bash
cd frontend
npm install
npm run dev          # Development
npm run build        # Production build
npm start            # Production server
```

---

## Training Pipeline

```bash
cd training_pipeline
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt

# Train face recognition model
python train_face_model.py \
  --num_classes 100000 \
  --embedding_dim 512 \
  --batch_size 512 \
  --epochs 50 \
  --lr 0.1 \
  --output_dir ./checkpoints

# For distributed training (multi-GPU):
torchrun --nproc_per_node=8 train_face_model.py --batch_size 4096
```

---

## Production Models (Download separately)

| Model | Source | License |
|-------|--------|---------|
| ArcFace R100 | insightface-buffalo_l | MIT |
| Stable Diffusion v1.5 | runwayml/stable-diffusion-v1-5 | CreativeML |
| ControlNet OpenPose | lllyasviel/sd-controlnet-openpose | Apache 2.0 |

```bash
# InsightFace models (auto-download on first run):
python -c "from insightface.app import FaceAnalysis; FaceAnalysis(name='buffalo_l').prepare(ctx_id=0)"

# Diffusion models (HuggingFace):
python -c "from diffusers import StableDiffusionPipeline; StableDiffusionPipeline.from_pretrained('runwayml/stable-diffusion-v1-5')"
```

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| JWT_SECRET | JWT signing secret | (required) |
| DB_PASSWORD | PostgreSQL password | genauth |
| SECRET_KEY | App secret key | (required) |
| USE_GPU | Enable GPU inference | true |
| FACE_THRESHOLD | Face match threshold | 0.80 |
| DNA_THRESHOLD | DNA match threshold | 0.85 |

---

## Security Notes

- All biometric data is AES-256 encrypted at rest
- DNA vectors are stored as hashed + encrypted blobs
- Zero-knowledge proof layer prevents raw data exposure
- Enable rate limiting and WAF in production gateway
- Use TLS/mTLS for all inter-service communication
