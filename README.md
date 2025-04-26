# 📖 AI Automation Stack

This project builds a complete local AI-powered automation platform, including:

- Ollama for running large language models (GPU-accelerated)
- Open WebUI for a web-based chat interface
- FastAPI proxy for simple API communication
- n8n for no-code workflow automation

All services are containerized using Docker Compose and fully integrated.

---

## 📦 Project Structure

```
ai-automation-stack/
👉🏼n8n/                # n8n workflows, credentials, configs
👉🏼ollama-docker/      # Ollama server, Open WebUI, FastAPI app
```

---

## 🚀 Components Overview

| Component        | Description                                    | Port                  |
|------------------|------------------------------------------------|------------------------|
| Ollama           | Local LLM backend with GPU acceleration        | `7869`                 |
| Open WebUI       | Web UI to interact with LLM models             | `8080`                 |
| FastAPI Proxy    | Simple API to forward prompts to Ollama        | `8000` (Debug: `5678`) |
| n8n              | Visual automation workflows                   | `5679`                 |

---

## 💠 How to Start

### 1. Clone this Repository

```bash
git clone https://github.com/yourusername/ai-automation-stack.git
cd ai-automation-stack
```

---

### 2. Build and Start All Containers

```bash
docker-compose -f ollama-docker/docker-compose-ollama-gpu.yaml up -d
```

---

### 3. Access the Services

| Service | URL |
|---------|-----|
| Open WebUI | [http://localhost:8080](http://localhost:8080) |
| FastAPI Proxy | [http://localhost:8000](http://localhost:8000) |
| n8n Automation | [http://localhost:5679](http://localhost:5679) |

---

## 🛁 API Usage Example

### Send a prompt to FastAPI Proxy:

```bash
curl -X POST http://localhost:8000/generate -H "Content-Type: application/json" -d '{
  "prompt": "Tell me about the capital of Greece."
}'
```

✅ FastAPI forwards the prompt to Ollama and returns the generated text.

---

## 📊 Monitoring Commands

- See all containers:

```bash
docker ps
```

- Only project containers:

```bash
docker-compose -f ollama-docker/docker-compose-ollama-gpu.yaml ps
```

- View live logs:

```bash
docker logs -f ollama

docker logs -f ollama-webui

docker logs -f ollama-fastapi-proxy

docker logs -f n8n
```

- GPU Monitoring:

```bash
watch -n 1 nvidia-smi
```

---

## 🧐 Advanced Tips

- Add authentication to the FastAPI `/generate` endpoint
- Schedule n8n workflows to trigger LLM generations
- Add model streaming for faster partial responses
- Expand Open WebUI to support more complex model switching

---

## ⚙️ Future Improvements

- Automatic backup of n8n workflows
- Auto-pull latest models for Ollama
- Real-time dashboard for GPU/CPU usage
- Full monitoring with Grafana + Prometheus (optional)

---

## 🏁 Credits

- [Ollama](https://ollama.ai) for local LLM runtime
- [Open WebUI](https://github.com/open-webui/open-webui) for chat interface
- [FastAPI](https://fastapi.tiangolo.com/) for backend API proxy
- [n8n](https://n8n.io/) for automation workflows

---

