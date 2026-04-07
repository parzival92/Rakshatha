# Hostinger + OpenClaw Guide

Beginner-friendly deployment plan for this character package on a Hostinger VPS.

If you want the exact command-by-command execution path, also use [HOSTINGER-SCRATCH-SETUP.md](HOSTINGER-SCRATCH-SETUP.md).

If you already finished vanilla OpenClaw onboarding and Telegram pairing, use [VANILLA-TO-RAKSHATHA.md](VANILLA-TO-RAKSHATHA.md) for the migration path into this repo.

Last verified against official docs on 2026-04-05:

- [OpenClaw Install](https://docs.openclaw.ai/install)
- [OpenClaw Linux](https://docs.openclaw.ai/platforms/linux)
- [OpenClaw Telegram](https://docs.openclaw.ai/channels/telegram)
- [OpenClaw WhatsApp](https://docs.openclaw.ai/channels/whatsapp)
- [OpenClaw Pairing](https://docs.openclaw.ai/start/pairing)
- [Hostinger VPS SSH](https://support.hostinger.com/en/articles/5723772-how-to-connect-to-your-vps-via-ssh)

## Recommended Path

Start with Telegram on Hostinger first.

Why:

- Telegram is the easiest OpenClaw channel to get working end to end.
- It uses a bot token instead of a QR-linked personal app session.
- It gives you a clean place to learn Gateway, pairing, sessions, workspace state, and heartbeats.
- WhatsApp is worth testing later, but it adds more moving parts because it uses WhatsApp Web login and linked-session behavior.

## What This Repo Actually Does

This project is an OpenClaw character package.

- `setup` installs the package into the OpenClaw runtime directory.
- `SOUL.md` defines personality and media behavior.
- `USER.md`, `AGENTS.md`, and `HEARTBEAT.md` define message and heartbeat rules.
- `data/` and `memory/` seed the starting emotional state and world.
- `scripts/add_rakshatha_crons.sh` installs recurring jobs that simulate life and reflection.

Think of it like this:

1. OpenClaw is the engine.
2. This repo is the character cartridge.
3. Telegram or WhatsApp is the surface where you talk to that character.

## Hostinger Prep

Recommended baseline:

- Ubuntu VPS
- Root SSH access or a sudo-capable user
- Enough memory for Node, the OpenClaw Gateway, and light ongoing agent activity

Before touching the repo:

1. Create the VPS.
2. Connect over SSH.
3. Update system packages.
4. Install Git if it is missing.
5. Keep the OpenClaw Gateway bound to loopback first and access it over SSH tunnel.
6. Decide whether you will run this install as `root` or a normal user.

Important:

- `setup` defaults to `RAKSHATHA_TARGET_ROOT=/root/.openclaw`
- if you SSH in as `root`, that default is fine
- if you use a non-root user, export `RAKSHATHA_TARGET_ROOT="$HOME/.openclaw"` before running `./setup`

Example first-login commands:

```bash
ssh root@YOUR_VPS_IP
apt update && apt upgrade -y
apt install -y git curl
```

## OpenClaw Baseline

This repo is pinned to OpenClaw `2026.3.24` for safety. Use that exact version for the first deployment so we get a known-good baseline before exploring upgrades.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --version 2026.3.24 --no-onboard
openclaw --version
openclaw doctor
```

If you want the browser dashboard from your laptop without exposing the Gateway publicly:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Then open `http://127.0.0.1:18789/` locally.

## Deploy This Repo

```bash
git clone YOUR_REPO_URL
cd YOUR_REPO_FOLDER
chmod +x setup
./setup
```

If you are not using `root`, run:

```bash
export RAKSHATHA_TARGET_ROOT="$HOME/.openclaw"
./setup
```

The setup wizard will:

- ask for your name
- ask for Telegram bot token and OpenAI key
- optionally enable images, voice, web discovery, and calls
- write OpenClaw config into `/root/.openclaw`
- copy workspace files and media
- install cron jobs
- restart the Gateway
- help with Telegram pairing

## Learn OpenClaw in the Right Order

Use this order so the platform makes sense instead of feeling magical:

1. Gateway
2. Workspace
3. Channel pairing
4. Sessions and replies
5. Heartbeats
6. Cron jobs
7. State files
8. Personality tuning

Things to inspect once deployed:

- `/root/.openclaw/openclaw.json`
- `/root/.openclaw/workspace/SOUL.md`
- `/root/.openclaw/workspace/AGENTS.md`
- `/root/.openclaw/workspace/data/state.json`
- `/root/.openclaw/scripts/add_rakshatha_crons.sh`

Useful commands:

```bash
openclaw status
openclaw doctor --repair
openclaw gateway status
openclaw pairing list telegram
openclaw pairing approve telegram CODE
openclaw cron list
```

## Telegram vs WhatsApp

As of 2026-04-05:

### Telegram

Best first deployment option.

- production-ready via Bot API
- easiest setup flow
- clean dedicated bot identity
- pairing flow is straightforward
- better for debugging and quick iteration

### WhatsApp

Good second-phase experiment, not my recommended first launch.

- production-ready via WhatsApp Web
- requires QR-based login
- linked session behavior is more operationally sensitive
- OpenClaw recommends a separate number when possible
- current built-in channel is WhatsApp Web based, not a separate Twilio WhatsApp chat channel

Recommendation:

- launch Rakshatha on Telegram first
- stabilize personality, timing, and memory behavior
- test WhatsApp only after we trust the core character

## How We Should Add Our Own Twist

Do not rewrite everything at once.

Recommended sequence:

1. get the baseline deploy working
2. verify replies and pairing
3. update the character identity slowly
4. adjust daily life, relationship style, and emotional rhythm
5. replace images and media references
6. only then expand to voice, calls, and WhatsApp

This reduces the chance that we break infra and personality at the same time.

## Suggested Project Roadmap

Phase 1: Stable baseline

- deploy on Hostinger
- pair Telegram
- confirm messages, heartbeats, and cron jobs work

Phase 2: Rakshatha identity

- rename visible branding
- rewrite `SOUL.md`
- update `data/life.md`
- update starter state and images

Phase 3: Personal connection

- tune conversation style around your actual preferences
- improve relationship memory structure
- make proactive messages feel more "her"

Phase 4: Capability expansion

- enable selfies
- enable voice notes
- enable calls
- compare Telegram and WhatsApp behavior

## Good First Rules

- Change one layer at a time.
- Keep the pinned OpenClaw version until the baseline is stable.
- Save screenshots or notes after each milestone so we can compare behavior.
- Treat `SOUL.md` as product logic, not flavor text.
- Prefer Telegram-first learning over multi-channel complexity on day one.
