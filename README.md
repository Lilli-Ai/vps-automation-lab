# VPS Automation Lab

Self-hosted AI automation infrastructure on a Contabo VPS, replacing paid/trial-limited cloud services with self-hosted equivalents under full control.

## Services

- **n8n** — [n8n.lilia.am](https://n8n.lilia.am) — workflow automation engine
- **Flowise** — [flowise.lilia.am](https://flowise.lilia.am) — visual LLM agent builder
- **Open WebUI** — [chat.lilia.am](https://chat.lilia.am) — self-hosted chat interface, powering a bilingual (RU/EN) voice-in/voice-out AI English tutor (Whisper STT, Groq LLM, Kokoro TTS)
- **Kokoro TTS** — self-hosted text-to-speech (kokoro-fastapi), used by Open WebUI over an internal Docker network; test UI behind Basic Auth
- **Chatwoot** — [omni.lilia.am](https://omni.lilia.am) — self-hosted omnichannel inbox powering *AI Travel Concierge: From Message to Qualified Lead*, a project I'm currently building

## AI Travel Concierge (in progress)

- Chatwoot deployed from scratch with Docker Compose (PostgreSQL with pgvector, Redis, Sidekiq), behind Nginx + HTTPS
- Channels working: website chat widget on a demo travel-agency page and a Telegram bot
- Agent Bot → n8n webhook: every client message is verified (HMAC-SHA256 signature + timestamp check against replay), then routed by an LLM (Groq, qwen3.8-27b, JSON mode) to one of four actions: answer, collect details, create lead, hand off to a manager
- A validator checks every AI reply before it is sent (no invented prices, no facts without a knowledge base, safe fallback to a human); each decision is logged as a private note (decision trace)
- Replies in the client's language (Armenian, Armenian in Latin script, Russian, English)
- Qualified leads saved to a separate PostgreSQL database with duplicate protection (one open lead per conversation)
- Next: manager notifications, lead scoring, NocoDB kanban, RAG knowledge base, voice

## Stack

- **OS**: Ubuntu 24.04.4 LTS
- **Containers**: Docker (`docker run` with named volumes; Docker Compose for Chatwoot), all services set to auto-restart on reboot
- **Reverse proxy**: Nginx
- **HTTPS**: Let's Encrypt / Certbot, auto-renewing certificates
- **Databases**: PostgreSQL 16 (pgvector) — separate databases and users per project
- **Security**: ufw firewall (22, 80, 443 only, default deny incoming); all container ports bound to `127.0.0.1` and exposed only through Nginx + HTTPS; Basic Auth on the Kokoro TTS test UI
- **Session persistence**: tmux, to survive SSH disconnects during long-running setup

## Security fix (Oct 2026)

A later audit found that Docker publishes container ports through its own iptables rules, bypassing ufw: n8n, Flowise, Open WebUI and Kokoro were reachable directly by IP over plain HTTP. Fixed by recreating each container with `-p 127.0.0.1:<port>:<port>` (same volumes and settings), moving container-to-container traffic onto named Docker networks, and pointing Nginx at `127.0.0.1`. Lesson: after enabling a firewall, verify from outside by IP, not only through the domains.

## Why

Built to move off cloud free-tier trials and subscription limits onto infrastructure I fully control, manage, and understand end to end — from provisioning through DNS, HTTPS, and container orchestration.

## Note

This repo documents the infrastructure and setup process rather than hosting code — the actual configs (run commands, environment variables) live on the server itself and aren't published here, since they'd include credentials and internal networking details.
