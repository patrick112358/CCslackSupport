# IT Support Slack Bot

Monitors `#it-support`. Any message → Claude classifies it → either answers directly in-thread or creates a ClickUp task and pings the assignee.

Works with plain text, Arabic/French, and image/file pastes (images are noted in the ClickUp task with a link back to the Slack thread).

---

## Step 1 — Create the Slack App (Patrick, ~20 min)

1. Go to https://api.slack.com/apps → **Create New App** → **From scratch**
2. Name: `IT Support Bot` | Workspace: LAL
3. **Socket Mode** (left sidebar) → Enable Socket Mode → Generate App-Level Token
   - Token name: `socket-token` | Scope: `connections:write`
   - Copy the `xapp-...` token → this is `SLACK_APP_TOKEN`
4. **OAuth & Permissions** → Bot Token Scopes → Add:
   - `channels:history`, `channels:read`, `chat:write`, `reactions:write`
5. **Event Subscriptions** → Enable Events: ON
   - Subscribe to bot events: `message.channels`
6. **Install to Workspace** → copy the `xoxb-...` Bot Token → this is `SLACK_BOT_TOKEN`
7. In `#it-support` Slack channel: `/invite @IT Support Bot`

---

## Step 2 — Deploy on Linode (Rami, ~20 min)

SSH into the Linode server, then:

```bash
# Clone the repo
cd /opt
git clone https://github.com/patrick112358/CCslackSupport.git cc-slack-support
cd cc-slack-support

# Install dependencies
npm install

# Create .env from example
cp .env.example .env
nano .env
# Paste the four tokens Patrick gave you, save

# Install and start the systemd service
sudo cp it-support-bot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable it-support-bot
sudo systemctl start it-support-bot

# Check it started OK
sudo systemctl status it-support-bot
```

The bot is now live. No restart needed after server reboots — systemd handles it.

---

## Step 3 — Test

Post these in `#it-support` and check the thread response:

| Message | Expected |
|---|---|
| `Adobe CC expired again` | TickUp ticket → Rami, 📋 reaction |
| `Tabshoura is completely down` | TickUp ticket → urgent, 📋 reaction |
| `How do I clear my cache?` | Direct answer in thread, ✅ reaction |
| `What time is the standup?` | No response (ignored) |
| *(paste a screenshot with no text)* | Ticket created → note says "see Slack thread for image" |

---

## Step 4 — Go live

1. Announce in `#it-support`:
   > "Suptask has been replaced. Just describe your issue here — the bot responds automatically. No `/support` command needed."
2. `/remove @Suptask` from the channel
3. Patrick cancels Suptask subscription

---

## Updating the bot

To update routing rules or self-serve answers, edit `system-prompt.js`, then:

```bash
cd /opt/cc-slack-support
git pull
sudo systemctl restart it-support-bot
```
