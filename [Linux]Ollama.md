#### Install Ollama

- `curl -fsSL https://ollama.com/install.sh | sh`
- `screen -S ollama-session`
- `ollama serve`
- `CTRL-A+d`

#### Downloading LLM Model

`ollama pull llama3.1:405b`

#### Install Docker

- `sudo emerge app-containers/docker docker-cli`
- See [https://wiki.gentoo.org/wiki/Docker](https://wiki.gentoo.org/wiki/Docker) for more details.

##### Running Docker Image

- `docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main`
- Visit: [127.0.0.1:8080](http://127.0.0.1:8080)

#### Check Status of NVIDIA GPU

`watch -n 1 nvidia-smi`

#### TODO

- [ ] Figure out PiHole configuration / reverse proxy
- [ ] Port-Forwarding on new network
- [ ] Figure out reverse proxy setup for ollama/openwebui
- [ ] Figure out hosting openwebui @ ivanbond.tech/gpt ?
