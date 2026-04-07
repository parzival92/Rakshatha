# Hostinger Scratch Setup

Step-by-step runbook for deploying Rakshatha from scratch on a fresh Hostinger VPS.

This guide is written so you can run each command yourself and understand what it does.

If you already have vanilla OpenClaw working, skip this file and use [VANILLA-TO-RAKSHATHA.md](VANILLA-TO-RAKSHATHA.md).

Last verified against official docs on 2026-04-06:

- [OpenClaw Install](https://docs.openclaw.ai/install)
- [OpenClaw Linux](https://docs.openclaw.ai/platforms/linux)
- [OpenClaw Telegram](https://docs.openclaw.ai/channels/telegram)
- [OpenClaw Pairing](https://docs.openclaw.ai/channels/pairing)
- [Hostinger SSH access](https://support.hostinger.com/en/articles/5723772-how-to-connect-to-your-vps-via-ssh)

## Assumptions

This runbook assumes:

- you bought a fresh Hostinger VPS
- the VPS is running Ubuntu
- you want Telegram first
- you will deploy this private Rakshatha repo
- you want to learn the flow, not just paste a giant script

Recommended starting point:

- Hostinger KVM 1
- Ubuntu LTS
- one private owner
- no WhatsApp for the first deployment

## What You Need Before Running Setup

Have these ready:

- VPS IP address
- VPS root password or SSH key
- private Git repo URL
- Telegram bot token from `@BotFather`
- OpenAI API key

Optional for later:

- Google API key for selfies and web discovery
- ElevenLabs API key and voice ID for voice notes
- ElevenLabs agent ID and phone number ID for calls

## Phase 1: Connect To The VPS

### Step 1: Open the VPS terminal

You can use either:

- Hostinger Browser Terminal
- your local terminal with SSH

SSH version:

```bash
ssh root@YOUR_VPS_IP
```

What this does:

- opens a shell session on your VPS as `root`

If you are not using `root`, you can still follow this guide, but later you should export `RAKSHATHA_TARGET_ROOT="$HOME/.openclaw"` before running `./setup`.

### Step 2: Confirm who and where you are

```bash
whoami
pwd
uname -a
```

What this does:

- confirms which user you are logged in as
- confirms your current directory
- shows Linux/kernel info so you know the machine is alive and responsive

Expected:

- `whoami` should usually say `root`

## Phase 2: Prepare The Server

### Step 3: Update package metadata and upgrade the server

```bash
apt update && apt upgrade -y
```

What this does:

- refreshes Ubuntu package lists
- installs available security and package updates

### Step 4: Install base tools

```bash
apt install -y git curl
```

What this does:

- installs `git` so you can clone the repo
- installs `curl` so you can use the OpenClaw installer

### Step 5: Check the installed tools

```bash
git --version
curl --version
```

What this does:

- confirms the basic CLI tools are available before we continue

## Phase 3: Install OpenClaw

### Step 6: Install the pinned OpenClaw version

This repo is currently pinned to OpenClaw `2026.3.24`, so use that first before experimenting with upgrades.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --version 2026.3.24 --no-onboard
```

What this does:

- downloads the official OpenClaw installer
- installs Node if needed
- installs OpenClaw
- skips the default onboarding because this repo has its own setup flow

### Step 7: Verify the install

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

What this does:

- confirms the CLI is installed
- checks for common install/config issues
- checks whether the Gateway is already running

Notes:

- `openclaw doctor` may suggest fixes; that is normal on a fresh machine
- if the Gateway is not running yet, that is okay for now

## Phase 4: Clone The Repo

### Step 8: Choose a working folder

```bash
cd /root
mkdir -p projects
cd projects
```

What this does:

- creates a simple home for the repo on the VPS

If you are not using `root`, pick a path under your home directory instead.

### Step 9: Clone the private repo

```bash
git clone YOUR_PRIVATE_REPO_URL
cd YOUR_REPO_FOLDER
```

What this does:

- downloads your Rakshatha repo to the VPS
- enters the repo directory

### Step 10: Inspect the repo before setup

```bash
pwd
ls -la
sed -n '1,40p' README.md
```

What this does:

- confirms you are inside the correct repo
- shows the top-level files
- lets you quickly verify you are on the Rakshatha project you expect

## Phase 5: Run Rakshatha Setup

### Step 11: Make setup executable

```bash
chmod +x setup
```

What this does:

- allows the `setup` script to run as a program

### Step 12: Non-root only

Skip this if you are logged in as `root`.

```bash
export RAKSHATHA_TARGET_ROOT="$HOME/.openclaw"
```

What this does:

- tells the repo to install OpenClaw runtime files into your user home instead of `/root/.openclaw`

### Step 13: Run the setup wizard

```bash
./setup
```

What this does:

- asks for your owner name
- asks for the Telegram bot token
- asks for the OpenAI API key
- optionally enables photos, voice, web discovery, and calls
- writes the live OpenClaw workspace and config
- installs cron jobs
- restarts the Gateway
- helps you pair Telegram

Recommended choices for the first pass:

- images: `No`
- voice: `No`
- web discovery: `No`
- calls: `No`

Why:

- first we want a stable text-only Telegram deployment
- fewer moving parts means easier debugging

## Phase 6: Telegram Pairing

### Step 14: Create the bot in Telegram first

Before or during setup:

1. Open Telegram
2. Chat with `@BotFather`
3. Run `/newbot`
4. Copy the bot token

### Step 15: Message your bot

After the Gateway starts and setup asks for pairing:

1. open your new bot in Telegram
2. send any message, even just `hi`

What happens next:

- OpenClaw will create a pairing request
- the bot should give or trigger a pairing code flow
- pairing approval grants DM access

### Step 16: Approve pairing if needed from the VPS

```bash
openclaw pairing list telegram
openclaw pairing approve telegram YOUR_CODE
```

What this does:

- shows pending Telegram DM pairing requests
- approves your Telegram account so the bot can actually respond

## Phase 7: Verify The Live System

### Step 17: Check OpenClaw health

```bash
openclaw status
openclaw gateway status
openclaw cron list
```

What this does:

- shows overall OpenClaw health
- confirms the Gateway is up
- confirms Rakshatha’s cron jobs were installed

You should expect to see cron names like:

- `rakshatha-wakeup`
- `rakshatha-life`
- `rakshatha-reflection`

If web discovery was enabled, also:

- `rakshatha-discovers`

### Step 18: Inspect the deployed workspace

If you used `root`:

```bash
ls -la /root/.openclaw
ls -la /root/.openclaw/workspace
sed -n '1,80p' /root/.openclaw/workspace/AGENTS.md
sed -n '1,120p' /root/.openclaw/workspace/data/state.json
```

If you used a non-root user, replace `/root/.openclaw` with `$HOME/.openclaw`.

What this does:

- confirms the setup wrote the runtime files
- lets you inspect the personalized user and state data

### Step 19: Send the first real Telegram test message

Open Telegram and send something like:

```text
hey
```

Then watch behavior:

- does the bot reply
- does the reply feel in-character
- does it wait until after pairing approval

## Phase 8: Learn The Runtime

These are the most useful files to inspect after setup:

```bash
sed -n '1,80p' /root/.openclaw/openclaw.json
sed -n '1,80p' /root/.openclaw/workspace/SOUL.md
sed -n '1,80p' /root/.openclaw/workspace/USER.md
sed -n '1,120p' /root/.openclaw/workspace/HEARTBEAT.md
sed -n '1,120p' /root/.openclaw/workspace/data/life.md
```

What to look for:

- `openclaw.json`: platform config
- `SOUL.md`: personality logic
- `USER.md`: message-response rules
- `HEARTBEAT.md`: proactive behavior
- `data/life.md`: world/background context

## Phase 9: Safe Commands To Memorize

These are the commands you will use the most:

```bash
openclaw status
openclaw doctor --repair
openclaw gateway status
openclaw gateway restart
openclaw pairing list telegram
openclaw cron list
```

## Phase 10: Optional Local Dashboard Access

If you want to open the Gateway locally in your own browser without exposing it publicly:

Run this on your laptop, not on the VPS:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Then open:

```text
http://127.0.0.1:18789/
```

## First-Day Goal

Do not try to finish everything on day one.

A perfect first milestone is simply:

1. VPS updated
2. OpenClaw installed
3. repo cloned
4. `./setup` completed
5. Telegram pairing approved
6. Rakshatha replies once successfully

That is enough to prove the stack works.

## After The First Successful Reply

Only then move to:

1. tune `SOUL.md`
2. tune `data/life.md`
3. enable web discovery
4. enable images
5. test WhatsApp later

## If Something Breaks

Run these in order:

```bash
openclaw status
openclaw doctor --repair
openclaw gateway status
openclaw cron list
```

Then inspect:

```bash
sed -n '1,120p' /root/.openclaw/openclaw.json
sed -n '1,120p' /root/.openclaw/workspace/AGENTS.md
sed -n '1,120p' /root/.openclaw/workspace/data/state.json
```

If you want, use this guide together with:

- [docs/HOSTINGER-OPENCLAW-GUIDE.md](HOSTINGER-OPENCLAW-GUIDE.md)

That file explains the bigger-picture architecture. This file is the hands-on execution path.
