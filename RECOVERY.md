# NanoClaw Recovery Guide

Full restore from scratch on a new or rebuilt machine.

## What Is Backed Up (GitHub)

- All source code and customizations
- `groups/*/CLAUDE.md` — agent memory and instructions per group
- This recovery guide

## What Is NOT on GitHub (keep separately)

- `.env` — all credentials and tokens. Back this up to a password manager or secure note.
- `store/` — SQLite message history (optional, can start fresh)
- `data/` — runtime state (recreated automatically)

---

## Recovery Steps

### 1. Install prerequisites

```bash
# Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs

# Docker (if not already installed)
curl -fsSL https://get.docker.com | sh
```

### 2. Clone your fork

```bash
git clone https://github.com/Nitrousbob/nanoclaw.git
cd nanoclaw
git remote add upstream https://github.com/qwibitai/nanoclaw.git
```

### 3. Restore credentials

Create `.env` with your saved credentials:

```
CLAUDE_CODE_OAUTH_TOKEN=<your token>
TELEGRAM_BOT_TOKEN=<Squiz314bot token>
TELEGRAM_BOT_POOL=<pool bot 1 token>,<pool bot 2 token>
```

Sync to container environment:

```bash
mkdir -p data/env && cp .env data/env/env
```

### 4. Install dependencies and build

```bash
npm install
npm run build
./container/build.sh
```

### 5. Configure mounts (no external access)

```bash
npx tsx setup/index.ts --step mounts -- --empty
```

### 6. Start the service

```bash
npx tsx setup/index.ts --step service
```

### 7. Re-register your Telegram chat

The SQLite DB is not backed up, so registered groups need to be re-added.

Start the service, open @Squiz314bot in Telegram, send `/chatid` to confirm it responds, then:

```bash
npx tsx setup/index.ts --step register -- \
  --jid "tg:8101452674" \
  --name "maxchat" \
  --folder "telegram_main" \
  --trigger "@Squiz314bot" \
  --channel telegram \
  --no-trigger-required \
  --is-main
```

Restart the service:

```bash
systemctl restart nanoclaw
```

### 8. Verify

```bash
npx tsx setup/index.ts --step verify
```

All green = fully restored. Send a test message to @Squiz314bot.

---

## Key Reference

| Thing | Value |
|-------|-------|
| Telegram main bot | @Squiz314bot |
| Telegram chat JID | tg:8101452674 |
| Group folder | telegram_main |
| Group name | maxchat |
| GitHub fork | github.com/Nitrousbob/nanoclaw |
| Service | `systemctl restart nanoclaw` |
| Logs | `tail -f logs/nanoclaw.log` |

## Pulling Upstream Updates

```bash
git fetch upstream
git merge upstream/main
npm install && npm run build
systemctl restart nanoclaw
```
