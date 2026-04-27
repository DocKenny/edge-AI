## Ollama Configuration

This project uses [Ollama](https://ollama.com) to serve local LLMs to OpenWebUI, 
exposed securely via Cloudflare Tunnel.

### Requirements
- AMD GPU with ROCm support (tested on RX 7900 XT)
- Docker + Docker Compose
- Cloudflare account with your domain configured

### Setup

#### 1. Clone and configure environment

```bash
cp .env.example .env
nano .env
```

Add your Cloudflare tunnel token (see below):
```bash
TUNNEL_TOKEN=your-tunnel-token-here
```

#### 2. Create the Cloudflare Tunnel

```bash
# Install cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Authenticate
cloudflared tunnel login

# Create tunnel and route DNS
cloudflared tunnel create ollama
cloudflared tunnel route dns ollama ollama.yourdomain.com

# Get your tunnel token
cloudflared tunnel token ollama
```

Copy the token into your `.env` file.

#### 3. Start everything

```bash
docker compose up -d
```

#### 4. Pull models

```bash
docker exec -it ollama-rocm ollama pull gemma3:4b
docker exec -it ollama-rocm ollama pull llama3.2
```

### Environment Variables

| Variable | Description | Example |
|---|---|---|
| `OLLAMA_BASE_URL` | URL of the Ollama instance | `https://ollama.yourdomain.com` |
| `TUNNEL_TOKEN` | Cloudflare tunnel token | `eyJ...` |

### Useful Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f ollama
docker compose logs -f cloudflared

# List running models
docker exec -it ollama-rocm ollama list

# Pull a model
docker exec -it ollama-rocm ollama pull 

# Remove a model
docker exec -it ollama-rocm ollama rm 
```

### Notes
- Models are persisted in a Docker named volume and survive container restarts
- Both services restart automatically on system boot via `restart: always`