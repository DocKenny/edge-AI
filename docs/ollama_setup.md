## Ollama Configuration

This project uses [Ollama](https://ollama.com) to serve local LLMs to OpenWebUI.

### Requirements
- AMD GPU with ROCm support (tested on RX 7900 XT)
- Docker
- Cloudflare Tunnel (for remote access)

### Running Ollama with ROCm (AMD GPU)

```bash
docker run -d \
  --device=/dev/kfd \
  --device=/dev/dri \
  -p 11434:11434 \
  -v ollama:/root/.ollama \
  --name ollama-rocm \
  ollama/ollama:rocm
```

### Pull Models

```bash
docker exec -it ollama-rocm ollama pull llama3.2
docker exec -it ollama-rocm ollama pull gemma3:4b
```

### Environment Variables

| Variable | Description | Example |
|---|---|---|
| `OLLAMA_BASE_URL` | URL of the Ollama instance | `https://ollama.yourdomain.com` |

### Remote Access via Cloudflare Tunnel

To expose Ollama securely without port forwarding:

```bash
# Install cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb

# Authenticate and create tunnel
cloudflared tunnel login
cloudflared tunnel create ollama
cloudflared tunnel route dns ollama ollama.yourdomain.com
cloudflared tunnel run ollama
```

### Stopping & Starting

```bash
# Stop
sudo docker stop ollama-rocm
pkill cloudflared

# Start
sudo docker start ollama-rocm
cloudflared tunnel run ollama
```

### Auto-start on boot (optional)

```bash
sudo cloudflared service install
sudo systemctl enable cloudflared
```