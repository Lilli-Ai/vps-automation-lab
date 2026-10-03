# VPS Automation Lab

Self-hosted AI automation infrastructure on a Contabo VPS, replacing paid/trial-limited cloud services with self-hosted equivalents under full control.

## Services

- **n8n** — [n8n.lilia.am](https://n8n.lilia.am) — workflow automation engine
- **Flowise** — [flowise.lilia.am](https://flowise.lilia.am) — visual LLM agent builder
- **Open WebUI** — [chat.lilia.am](https://chat.lilia.am) — self-hosted chat interface, powering a bilingual (RU/EN) voice-in/voice-out AI English tutor (Whisper STT, Groq LLM, Kokoro TTS)

## Stack

- **OS**: Ubuntu 24.04.4 LTS
- **Containers**: Docker + Docker Compose, all services set to auto-restart on reboot
- **Reverse proxy**: Nginx
- **HTTPS**: Let's Encrypt / Certbot, auto-renewing certificates
- **Security**: ufw firewall (22, 80, 443 only, default deny incoming)
- **Session persistence**: tmux, to survive SSH disconnects during long-running setup

## Why

Built to move off cloud free-tier trials and subscription limits onto infrastructure I fully control, manage, and understand end to end — from provisioning through DNS, HTTPS, and container orchestration.
