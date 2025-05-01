# 🧠 How to Add and Use a New Model in the Ollama Stack (e.g., `qwen3:8b`)

This guide documents the steps to integrate a new local LLM model (like `qwen3:8b`) into the AI automation stack using Docker, Ollama, Open WebUI, FastAPI, and n8n.

---

## 📦 1. Pull and Load the New Model in Ollama

1. **Enter the Ollama container:**

   ```bash
   docker exec -it ollama bash
   ```

2. **Run the new model (this triggers a pull if needed):**

   ```bash
   ollama run qwen3:8b
   ```

   ⚠️ If you see an error like:

   ```
   The model you are attempting to pull requires a newer version of Ollama.
   ```

   → You need to update the Ollama Docker image:

3. **Update Ollama in Docker:**

   ```bash
   docker pull ollama/ollama:latest
   docker-compose -f ollama-docker/docker-compose-ollama-gpu.yaml up -d --force-recreate --build ollama
   ```

---

## 🛠 2. Update the FastAPI Proxy

### Edit `src/main.py`

Update the FastAPI `generate` endpoint to properly stream results from the new model:

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import StreamingResponse
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import httpx
import debugpy
import json

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

debugpy.listen(("0.0.0.0", 5678))

class PromptRequest(BaseModel):
    prompt: str
    model: str = "qwen3:8b"  # Default is the new model

OLLAMA_API_URL = "http://ollama:11434/api/generate"

@app.post("/generate")
async def generate_prompt(request: PromptRequest):
    try:
        payload = {
            "model": request.model,
            "prompt": request.prompt
        }

        async with httpx.AsyncClient(timeout=None) as client:
            response = await client.post(OLLAMA_API_URL, json=payload)
            response.raise_for_status()

            async def stream_chunks():
                async for line in response.aiter_lines():
                    if line.strip():
                        try:
                            data = json.loads(line)
                            yield data.get("response", "") + " "
                        except json.JSONDecodeError:
                            continue

            return StreamingResponse(stream_chunks(), media_type="text/plain")

    except httpx.RequestError as e:
        raise HTTPException(status_code=500, detail=f"Error connecting to Ollama: {e}")
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Unexpected error: {e}")
```

---

## 🐳 3. Rebuild and Restart the App Container

```bash
docker-compose -f ollama-docker/docker-compose-ollama-gpu.yaml build app
docker-compose -f ollama-docker/docker-compose-ollama-gpu.yaml up -d app
```

---

## ✅ 4. Test the New Model Locally

```bash
curl -X POST http://localhost:8000/generate \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen3:8b", "prompt": "What is the capital of Greece?"}'
```

You should receive a streamed response such as:

```
The capital of Greece is Athens (Αθήνα in Greek).
```

---

## 🔄 5. Make It Work in Open WebUI

1. Open WebUI at: `http://localhost:8080`
2. The new model (`qwen3:8b`) should now appear in the dropdown list.
3. Test the model interactively from the browser.

---

## 🧪 6. Test via n8n HTTP Node

- Method: `POST`
- URL: `http://ollama-fastapi-proxy:8000/generate`
- Body Content Type: `JSON`
- Example body:

```json
{
  "prompt": "Explain quantum entanglement.",
  "model": "qwen3:8b"
}
```

---

## 💾 7. Commit the Changes

```bash
git add src/main.py
git commit -m "✨ Integrated qwen3:8b model and updated FastAPI streaming"
git push
```

---

**Done! 🎉 Your new model is fully integrated into the AI stack.**
