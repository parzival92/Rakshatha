# Manual File Migration To Rakshatha

Use this guide if:

- vanilla OpenClaw already works on your VPS
- Telegram already works
- you want to understand the file flow by moving things manually
- you do not want to run `./setup` yet

This path is intentionally conservative.

We are changing the character workspace first.
We are not touching auth or Telegram config in the first pass.

## What We Are Changing First

We will manually copy these repo files into the live OpenClaw workspace:

- `AGENTS.md`
- `CODEX.md`
- `HEARTBEAT.md`
- `IDENTITY.md`
- `MEMORY.md`
- `SOUL.md`
- `TOOLS.md`
- `USER.md`
- `data/state.json`
- `data/life.md`
- `data/relationship.md`
- `data/knowledge.md`

We will also copy:

- `rakshatha-image/` into `~/.openclaw/media/rakshatha-image`
- `scripts/add_rakshatha_crons.sh`
- `scripts/call_user.sh`

We are NOT changing these yet:

- `~/.openclaw/openclaw.json`
- `~/.openclaw/agents/main/agent/auth-profiles.json`

That means your already-working provider, key, and Telegram bot stay as they are.

Important:

- `AGENTS.md` in the repo contains placeholders like `<user-name>` and `<telegram-chat-id>`
- `./setup` normally replaces those automatically
- in the manual path, you must fill them in yourself before testing

## Phase 1: Copy The Repo To The VPS

Run this from your local machine, not the VPS:

```bash
rsync -av \
  --exclude '.git' \
  /Users/parzival/Documents/Developer/openclaw-Rakshatha/Rakshatha/ \
  root@YOUR_VPS_IP:/root/projects/Rakshatha/
```

If `rsync` is not available, use:

```bash
scp -r /Users/parzival/Documents/Developer/openclaw-Rakshatha/Rakshatha root@YOUR_VPS_IP:/root/projects/
```

## Phase 2: Back Up The Live Workspace

Run these on the VPS:

```bash
mkdir -p /root/projects
cd /root
cp -R ~/.openclaw/workspace ~/.openclaw/workspace.manual-pre-rakshatha.$(date +%Y%m%d-%H%M%S)
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.manual-pre-rakshatha.bak
cp ~/.openclaw/agents/main/agent/auth-profiles.json ~/.openclaw/agents/main/agent/auth-profiles.json.manual-pre-rakshatha.bak
```

## Phase 3: Create The Target Directories

Run these on the VPS:

```bash
mkdir -p ~/.openclaw/workspace/data
mkdir -p ~/.openclaw/workspace/memory
mkdir -p ~/.openclaw/media/rakshatha-image
mkdir -p ~/.openclaw/scripts
```

## Phase 4: Copy Core Character Files

Run these on the VPS:

```bash
cp /root/projects/Rakshatha/AGENTS.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/CODEX.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/HEARTBEAT.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/IDENTITY.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/MEMORY.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/SOUL.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/TOOLS.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/USER.md ~/.openclaw/workspace/
cp /root/projects/Rakshatha/data/state.json ~/.openclaw/workspace/data/
cp /root/projects/Rakshatha/data/life.md ~/.openclaw/workspace/data/
cp /root/projects/Rakshatha/data/relationship.md ~/.openclaw/workspace/data/
cp /root/projects/Rakshatha/data/knowledge.md ~/.openclaw/workspace/data/
```

## Phase 5: Copy Media And Scripts

Run these on the VPS:

```bash
cp -R /root/projects/Rakshatha/rakshatha-image/. ~/.openclaw/media/rakshatha-image/
cp /root/projects/Rakshatha/scripts/add_rakshatha_crons.sh ~/.openclaw/scripts/
cp /root/projects/Rakshatha/scripts/call_user.sh ~/.openclaw/scripts/
chmod +x ~/.openclaw/scripts/add_rakshatha_crons.sh
chmod +x ~/.openclaw/scripts/call_user.sh
```

## Phase 6: Verify The Files Landed Correctly

Run these on the VPS:

```bash
ls ~/.openclaw/workspace
ls ~/.openclaw/workspace/data
ls ~/.openclaw/media/rakshatha-image
ls ~/.openclaw/scripts
sed -n '1,20p' ~/.openclaw/workspace/SOUL.md
sed -n '1,40p' ~/.openclaw/workspace/data/life.md
```

You should now see the Bangalore / Nvidia / DevOps version of Rakshatha in the live workspace.

## Phase 7: Fill In `AGENTS.md` Manually

Because this is the manual path, the live `AGENTS.md` still contains placeholders.

Check it:

```bash
sed -n '1,20p' ~/.openclaw/workspace/AGENTS.md
```

If you already had a working vanilla workspace, your backup may contain the correct user metadata:

```bash
sed -n '1,20p' ~/.openclaw/workspace.manual-pre-rakshatha.YYYYMMDD-HHMMSS/AGENTS.md
```

Now edit the live file:

```bash
nano ~/.openclaw/workspace/AGENTS.md
```

Replace:

- `<user-name>` with your real name
- `<telegram-username>` with your Telegram username
- `<telegram-chat-id>` with your real Telegram chat ID
- `<user-phone-number>` with your phone number if you plan to use calls

Save and exit.

## Phase 8: Restart And Test

Run these on the VPS:

```bash
openclaw gateway restart
openclaw status
```

Then send a simple Telegram message like:

```text
/reset
```

After `/reset`, send:

```text
hi
```

At this point, the bot should still use your existing Telegram config and model auth, but the personality should come from the Rakshatha workspace files you copied manually.

Why `/reset`:

- OpenClaw reuses chat sessions until they expire
- `/new` or `/reset` starts a fresh session for that chat and reloads the workspace context

## Phase 9: Install Rakshatha Cron Jobs Later

Do this only after one normal Telegram reply works.

Run these on the VPS:

```bash
RAKSHATHA_MAIN_MODEL=openai/gpt-5.4-mini \
RAKSHATHA_ENABLE_WEB_SEARCH=0 \
~/.openclaw/scripts/add_rakshatha_crons.sh
```

Then verify:

```bash
openclaw cron list
```

## Rollback

If something feels wrong, restore the previous workspace backup:

```bash
rm -rf ~/.openclaw/workspace
cp -R ~/.openclaw/workspace.manual-pre-rakshatha.YYYYMMDD-HHMMSS ~/.openclaw/workspace
cp ~/.openclaw/openclaw.json.manual-pre-rakshatha.bak ~/.openclaw/openclaw.json
cp ~/.openclaw/agents/main/agent/auth-profiles.json.manual-pre-rakshatha.bak ~/.openclaw/agents/main/agent/auth-profiles.json
openclaw gateway restart
```

Replace `YYYYMMDD-HHMMSS` with the real backup folder name.
