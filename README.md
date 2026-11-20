# VPS Automation Lab

Self-hosted AI automation infrastructure on a Contabo VPS, replacing paid and trial-limited cloud services with self-hosted equivalents under full control.

## Services

- n8n: [n8n.lilia.am](https://n8n.lilia.am/), workflow automation engine
- Flowise: [flowise.lilia.am](https://flowise.lilia.am/), visual LLM agent builder
- Open WebUI: [chat.lilia.am](https://chat.lilia.am/), self-hosted chat interface powering a bilingual (RU/EN) voice-in/voice-out AI English tutor (Whisper STT, Groq LLM, Kokoro TTS)
- Kokoro TTS: self-hosted text-to-speech (kokoro-fastapi), used by Open WebUI over an internal Docker network; test UI behind Basic Auth
- Chatwoot: [omni.lilia.am](https://omni.lilia.am/), self-hosted omnichannel inbox powering Omnichannel AI Lead Automation
- Startup Idea Validator: agentic RAG app on a local LLM (Ollama in Docker), Gradio UI running as a systemd service, reachable only through an SSH tunnel

## Omnichannel AI Lead Automation

A multilingual AI system for travel agencies that answers clients and turns each conversation into a qualified lead. Now being rolled out in a travel agency in Armenia.

- Chatwoot deployed from scratch with Docker Compose (PostgreSQL with pgvector, Redis, Sidekiq), behind Nginx and HTTPS
- Channels: website chat widget, Telegram bot and email, all in one inbox
- Voice messages in Armenian, Russian and English are transcribed (ElevenLabs Scribe, fallback Groq Whisper)
- Agent Bot to n8n webhook: every client message is verified (HMAC-SHA256 signature and timestamp check against replay), then an LLM (Groq, qwen3.8-27b, JSON mode) chooses one of four actions: answer, collect missing details, create a lead, hand off to a manager
- A validator checks every AI reply before it is sent: no invented prices, no facts outside the knowledge base, safe handoff to a human. If the AI mentions a price, the reply is blocked and a manager takes over
- Date parsing with typos (common in Armenian messages) is checked in code, not left to the prompt
- Every AI decision is saved in a decision log as a private note the manager can read
- Replies in the client's language: Armenian, Armenian in Latin script, Russian, English
- Leads are scored HOT, WARM or COLD, saved to a separate PostgreSQL database with duplicate protection and shown in NocoDB
- A lead card goes to the managers' Telegram group with an "Assign to me" button
- LLM fallback: OpenRouter. Both fallbacks were tested by deliberately breaking the primary service

Results in testing: 1.8 s median reply time, 11 of 11 test leads complete, 0 duplicates, $0 per conversation.

## Stack

- OS: Ubuntu 24.04.4 LTS
- Containers: Docker (`docker run` with named volumes, Docker Compose for Chatwoot), all services set to auto-restart on reboot
- Reverse proxy: Nginx
- HTTPS: Let's Encrypt / Certbot, auto-renewing certificates
- Databases: PostgreSQL 16 (pgvector), separate databases and users per project
- Monitoring: UptimeRobot with email alerts
- Security: ufw firewall (22, 80, 443 only, default deny incoming), SSH key-only login, all container ports bound to `127.0.0.1` and exposed only through Nginx and HTTPS, Basic Auth on the Kokoro TTS test UI
- Session persistence: tmux, to survive SSH disconnects during long-running setup

## Security fixes

**Oct 2026: Docker bypassing ufw.** An audit found that Docker publishes container ports through its own iptables rules, bypassing ufw: n8n, Flowise, Open WebUI and Kokoro were reachable directly by IP over plain HTTP. Fixed by recreating each container with `-p 127.0.0.1:<port>:<port>` (same volumes and settings), moving container-to-container traffic onto named Docker networks, and pointing Nginx at `127.0.0.1`. Lesson: after enabling a firewall, verify from outside by IP, not only through the domains.

**Nov 2026: nginx2root.** After the disclosure of the nginx2root vulnerabilities (CVE-2026-123007 and others), checked the installed package version
