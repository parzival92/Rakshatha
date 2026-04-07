# Vanilla OpenClaw To Rakshatha

Use this guide if you already have a working vanilla OpenClaw install on your VPS and want to layer this Rakshatha repo on top of it.

If you want to move files one by one instead of running `./setup`, use [docs/MANUAL-FILE-MIGRATION.md](docs/MANUAL-FILE-MIGRATION.md).

This is the right path if:

- OpenClaw is already installed
- the Gateway already works
- Telegram already works
- you want to move from the generic bot into the Rakshatha character package

## What Changes When You Run `./setup`

This repo does not just add a few files.

It rewrites the active OpenClaw workspace and config for this character package:

- updates `~/.openclaw/openclaw.json`
- updates `~/.openclaw/agents/main/agent/auth-profiles.json`
- writes Rakshatha workspace files into `~/.openclaw/workspace`
- copies media into `~/.openclaw/media/rakshatha-image`
- installs Rakshatha cron jobs

That is intentional.

## Safety First

The `setup` script already creates backups for:

- `~/.openclaw/openclaw.json`
- `~/.openclaw/agents/main/agent/auth-profiles.json`

It now also creates an automatic backup of an existing non-Rakshatha workspace before the first Rakshatha install.

Backup path format:

```text
~/.openclaw/workspace.pre-rakshatha.YYYYMMDD-HHMMSS
```

Even so, it is smart to make your own manual snapshot first.

## Before You Migrate

Recommended manual backup commands:

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.manual.bak
cp ~/.openclaw/agents/main/agent/auth-profiles.json ~/.openclaw/agents/main/agent/auth-profiles.json.manual.bak
cp -R ~/.openclaw/workspace ~/.openclaw/workspace.manual.bak
```

If your Telegram bot token was exposed during testing, rotate it in `@BotFather` before doing the final Rakshatha install.

## Migration Path

### Step 1: Go to your repo folder

```bash
cd /root/projects/YOUR_REPO_FOLDER
```

### Step 2: Pull the latest local changes if needed

If this repo lives on the VPS already and you updated it elsewhere:

```bash
git pull
```

If you are copying it fresh:

```bash
git clone YOUR_PRIVATE_REPO_URL
cd YOUR_REPO_FOLDER
```

### Step 3: Review the main character files before installing

```bash
sed -n '1,80p' SOUL.md
sed -n '1,80p' USER.md
sed -n '1,80p' AGENTS.md
sed -n '1,120p' data/life.md
```

Why:

- once setup runs, these become the live character behavior

### Step 4: Run the repo setup

```bash
chmod +x setup
./setup
```

Recommended first-pass choices:

- keep Telegram enabled
- use the working model provider you already validated
- disable images for now unless you already have the Google key ready
- disable voice and calls for now
- disable web discovery for the very first Rakshatha deployment unless you specifically want it immediately

### Step 5: Verify the installed Rakshatha workspace

```bash
openclaw status
openclaw cron list
sed -n '1,80p' ~/.openclaw/workspace/AGENTS.md
sed -n '1,120p' ~/.openclaw/workspace/data/state.json
```

Expected cron names:

- `rakshatha-wakeup`
- `rakshatha-life`
- `rakshatha-reflection`

And if web discovery is enabled:

- `rakshatha-discovers`

### Step 6: Test in Telegram

Send a simple message like:

```text
hey
```

You are now checking:

- the Gateway still works
- Telegram still works
- the character behavior is now coming from the Rakshatha workspace files

## What To Tweak First

Do not rewrite everything at once.

Best first edits:

1. `SOUL.md`
2. `data/life.md`
3. `data/relationship.md`
4. `USER.md`

Why this order:

- `SOUL.md` changes voice and personality most strongly
- `data/life.md` makes her world feel specific
- `data/relationship.md` shapes how the connection feels over time
- `USER.md` controls operational chat behavior like deferring, reply timing, and state updates

## What Not To Change First

Avoid touching these until the first Rakshatha reply works:

- `setup`
- `scripts/add_rakshatha_crons.sh`
- auth file formats
- gateway networking

First prove the migration works. Then personalize.

## Rollback Path

If you want to revert to the previous vanilla state:

```bash
cp ~/.openclaw/openclaw.json.bak ~/.openclaw/openclaw.json
cp ~/.openclaw/agents/main/agent/auth-profiles.json.bak ~/.openclaw/agents/main/agent/auth-profiles.json
rm -rf ~/.openclaw/workspace
cp -R ~/.openclaw/workspace.pre-rakshatha.TIMESTAMP ~/.openclaw/workspace
openclaw gateway restart
```

Replace `TIMESTAMP` with the real backup folder name.

## Practical Recommendation

Your safest next move is:

1. keep the now-working vanilla Telegram baseline as your mental reference
2. run `./setup` once
3. test one Rakshatha reply
4. only then start deeper personality tuning
