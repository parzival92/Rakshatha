# Workspace Notes

This repository is a character package for OpenClaw, not a traditional web app or backend service.

## What Matters Most

- `SOUL.md` defines the character's identity, tone, emotional patterns, and media behavior.
- `USER.md`, `AGENTS.md`, and `HEARTBEAT.md` define runtime behavior and session rules.
- `data/` and `memory/` contain starter state that becomes live continuity after deployment.
- `setup` is the operational entry point. It writes the real OpenClaw workspace, config, scripts, and media into the target machine.

## Safe Editing Guidance

- Treat persona files as product logic. Small wording changes can change runtime behavior a lot.
- Keep instructions consistent across `SOUL.md`, `USER.md`, `AGENTS.md`, and `HEARTBEAT.md`.
- Do not casually wipe or simplify `data/` and `memory/`; those files are part of continuity.
- Prefer additive documentation and narrow setup fixes before large personality rewrites.

## Deployment Mental Model

- Repo files are source templates.
- `./setup` copies and personalizes them into the OpenClaw runtime directory.
- The deployed workspace then evolves independently as conversations and cron jobs update state.

## Current Direction

- Short-term goal: make deployment smooth for a beginner OpenClaw workflow on Hostinger.
- Recommended first channel: Telegram.
- Explore WhatsApp only after the Telegram path is stable and understood.
