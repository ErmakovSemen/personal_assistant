# Handoff: Telegram Personal Coach on n8n

_Last updated: 2026-03-25 UTC_

## 1. What this project is

This project is a self-hosted Telegram-based personal coach built on top of n8n.

The intended product behavior is:

- user writes or sends voice messages to a Telegram bot;
- n8n receives the message;
- OpenRouter/Gemini generates a coaching reply;
- important context can be written into an Obsidian vault;
- morning/evening proactive messages are sent automatically;
- Obsidian vault is synced through Dropbox.

At the moment, the system is **working in a basic operational form**, but it is still **prototype-grade**, not production-grade.

## 2. Current infrastructure

### Server

- OS: Ubuntu 24.04.4 LTS
- Public IP: `91.222.236.150`
- n8n UI: `http://91.222.236.150:5678`

### Main runtime components

- Docker + docker compose plugin
- n8n in Docker
- headless Dropbox daemon
- Obsidian vault mounted into n8n container as `/obsidian`
- Telegram bot integration
- OpenRouter integration for LLM calls

### Important filesystem paths

- n8n compose: `/opt/n8n/docker-compose.yml`
- n8n persistent data: Docker volume mounted to `/home/node/.n8n`
- Obsidian vault on host: `/root/Dropbox/Приложения/remotely-save/ObsidianVault`
- Obsidian vault inside container: `/obsidian`
- Dropbox service file: `/etc/systemd/system/dropbox.service`
- Local workflow export: `/root/n8n_exports/all_workflows.json`

## 3. What was done

### Base server setup

- system updated
- core packages installed
- Docker installed
- n8n deployed through docker compose
- firewall rules opened for SSH and n8n
- Dropbox daemon and CLI installed

### Workflows discovered on server

Current workflow IDs:

- `KJBp4f0aPCZWAsjV` — **Telegram Coach - Main**
- `xOTJbyekKg2vtV9B` — **Evening Checkin**
- `k4tJPdAUOkdcqUTy` — **Morning Briefing**
- `hv1JQVHA3ZjoOh4G` — **Telegram Coach - Poller**

### Telegram webhook problem and workaround

Telegram webhook setup to `http://91.222.236.150:5678/...` failed because Telegram requires HTTPS for webhooks.

The system was switched to a **polling-based fallback architecture**:

- a dedicated poller workflow calls Telegram `getUpdates` every minute;
- new updates are forwarded internally to the main webhook endpoint in n8n.

### n8n Code node runtime fixes

The following runtime-level fixes were applied in n8n config:

- env access in nodes enabled;
- built-in modules allowed;
- external modules allowed;
- nodes exclusion relaxed.

### HTTP access / secure cookie fix

Temporary workaround applied: `N8N_SECURE_COOKIE=false`

**Important:** production should move to HTTPS and re-enable secure cookies.

### Obsidian vault folders ensured

The following folders were ensured to exist and be writable by n8n:

- `Daily Notes`
- `Ideas`
- `Tasks`
- `Weekly`
- `System`

## 4. How the system currently works

### 4.1 Message ingress

1. **Telegram Coach - Poller** runs every minute.
2. It reads the last processed Telegram update offset from `/obsidian/System/telegram_offset.txt`.
3. It calls Telegram `getUpdates`.
4. It updates the local offset file.
5. It forwards each new update into the main workflow through the local n8n webhook endpoint.

### 4.2 Main conversational workflow

1. Receives forwarded Telegram update through webhook.
2. Detects whether the message contains voice.
3. If voice: requests file metadata, downloads audio, sends to OpenRouter for transcription.
4. If text: extracts text directly.
5. Reads today/yesterday notes from Obsidian.
6. Builds LLM request with system prompt + user message + note context.
7. Calls OpenRouter chat completions API.
8. Parses expected JSON response (`reply`, `create_note`, `schedule`).
9. Sends reply back to Telegram.
10. If `create_note` exists, writes/appends note into Obsidian.

### 4.3 Morning briefing workflow

- runs on cron schedule;
- reads yesterday's note;
- asks model to generate 3 priorities + 1 focus question;
- sends result to Telegram.

### 4.4 Evening check-in workflow

- runs on cron schedule;
- reads yesterday note and recent ideas;
- asks model to generate a short evening check-in;
- sends the message to Telegram.

## 5. Current product quality assessment

### What works

- n8n is up and reachable.
- Telegram bot can send and receive messages through the current polling architecture.
- Obsidian vault is mounted from Dropbox and is writable by n8n.
- Dropbox sync is active.
- Morning/evening automations exist and are active.
- Main chat loop works in a basic form.

### What is weak right now

- the system prompt is too generic;
- the model choice prioritizes low cost over quality;
- memory/context usage is shallow;
- note extraction policy is underdefined;
- no real planning layer for proactive outreach;
- reminder scheduling is not yet a robust product feature.

## 6. Known limitations and technical debt

### 6.1 Telegram transport

Current ingress uses polling because there is no HTTPS domain.

**Recommended fix:** put n8n behind Caddy or Nginx, issue Let's Encrypt certificate, switch Telegram back to real webhook.

### 6.2 Cookie / security mode

Current UI access requires `N8N_SECURE_COOKIE=false`.

**Recommended fix:** add HTTPS, re-enable secure cookies.

### 6.3 Agentic scheduling is not productized

Needs proper engineering for: reliable one-off scheduling, cancellation/editing of reminders, multiple pending reminders, timezone normalization, deduplication, persistence.

### 6.4 Note writing logic is too weak

Currently note creation depends on the model deciding to return `create_note` — inconsistent behavior.

### 6.5 Prompt and model quality

Current prompt tends toward shallow encouragement instead of concrete prioritization, sharper reflection, structured follow-up, tactical planning.

### 6.6 Workflow versioning

The system currently lives primarily in n8n state on the server. No exported workflow JSON committed to version control.

## 7. Suggested architecture for the next developer

### Layer A — message ingestion
- Telegram webhook over HTTPS (replace poller once TLS exists)

### Layer B — context assembly
- pull recent chat turns
- pull today/yesterday notes
- pull open tasks
- pull active goals/projects
- optionally pull rolling memory summary

### Layer C — reasoning / policy
- classify message intent;
- decide whether to answer, save, remind, ask follow-up, or schedule;
- generate structured action plan;
- generate user-facing reply.

### Layer D — action execution
- send reply
- append/create note
- create one-off reminder
- update recurring routine
- log state change

### Layer E — scheduler subsystem
Dedicated reminder table/datastore + one scheduler workflow that scans due reminders every minute with explicit statuses: pending / sent / cancelled / failed.

## 8. Recommended immediate next steps

### P1 — stabilize production transport
1. Put n8n behind HTTPS.
2. Re-enable secure cookies.
3. Replace Telegram poller with real Telegram webhook.

### P2 — improve coaching quality
1. Replace current system prompt.
2. Upgrade model choice if budget allows.
3. Add stronger context assembly from Obsidian.

### P3 — build real reminder engine
1. Introduce explicit reminder storage.
2. Add one-off reminders.
3. Add reminder edit/cancel flows.

### P4 — harden note system
1. Define deterministic note types.
2. Add validators for note path/content.
3. Separate journal vs ideas vs tasks.

### P5 — version control everything
1. Export all workflows as JSON.
2. Commit workflows + compose + docs to repo.
3. Add README / handoff / operations guide.

## 9. Operational notes for the next developer

### Check container health
- inspect docker compose in `/opt/n8n`
- inspect `docker ps`
- inspect n8n logs

### Check Dropbox
- `dropbox status` — expected: `Up to date`

### Check n8n editor access
- URL: `http://91.222.236.150:5678`

### Check Telegram bot path
Current ingress depends on Poller success + Main workflow success.

If poller starts failing, first inspect:
- `/obsidian/System/telegram_offset.txt`
- file permissions inside `/obsidian/System`
- Telegram getUpdates response

## 10. Security note

This handoff intentionally does **not** duplicate live secrets/tokens in plaintext.

A developer with server access can inspect currently configured env vars in `/opt/n8n/docker-compose.yml`.

Before handing the project to any third party, rotate:
- SSH credentials
- Telegram bot token
- OpenRouter API key
- n8n login credentials / API key

## 11. Honest status summary

### Working now
- Dropbox linked and syncing
- n8n reachable over HTTP
- Telegram bot can send/receive
- Main workflow operational
- Poller operational
- Morning/evening automations active
- Obsidian vault mounted and writable

### Not yet "good product"
- coaching quality is weak
- prompting/model need redesign
- proactive behavior is not truly intelligent
- one-off autonomous scheduling needs proper engineering
- HTTPS + real webhook should replace current workaround stack

## 12. Best handoff message to a new developer

There is a working self-hosted n8n-based Telegram coach on Ubuntu. Telegram currently enters through a polling workflow because webhook over HTTP was rejected by Telegram. Replies are generated through OpenRouter, context is read/written to an Obsidian vault synced by Dropbox. The stack is functional, but the coaching quality is weak and the scheduling logic is still prototype-grade. First priorities are: move n8n behind HTTPS, replace polling with real webhook, redesign prompt/model strategy, and build a proper reminder subsystem instead of relying on loosely structured LLM output + wait nodes.

## 13. File location

This handoff file was generated as:
- `/mnt/user-data/outputs/n8n_telegram_coach_handoff_2026-03-25.md`
