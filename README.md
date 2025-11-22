# telegram-n8n-ai-agent
---
## Overview

`telegram-n8n-ai-agent` is a reproducible n8n workflow that lets you interact with an AI agent over Telegram to schedule urgent meetings, create Google Calendar events, send Gmail notifications, and store lightweight memory. The project demonstrates a practical automation that combines Telegram, an LLM (Google Gemini Chat model in this example), Google Calendar, Gmail, and simple storage within n8n.

---

## Features

* Receive scheduling commands from Telegram.
* Natural-language parsing and orchestration using an AI agent (Google Gemini Chat model integrated in n8n).
* Create Google Calendar events (with title, description, date/time, duration).
* Send an email notification via Gmail to the invitee with meeting details.
* Save and recall basic conversation memory (Simple Memory node).
* Debug logs and example screenshots included.

---

## Architecture

```
Telegram Trigger -> AI Agent (Gemini) -> Create Google Calendar Event
                                 |- Send Gmail
                                 |- Save memory
                                 |- Reply to Telegram
```

## Screenshots

Below are the visual references included in the repository:

### Google Calendar Event Created Automatically

![Google Calendar Event](https://github.com/nagasivaninandam/telegram-n8n-ai-agent/blob/master/Google%20Calender.png)

### Gmail Notification Sent via n8n

![Gmail Notification](https://github.com/nagasivaninandam/telegram-n8n-ai-agent/blob/master/Gmail-n8n.jpeg)

### AI Agent Output Log

![AI Agent Output Log](https://github.com/nagasivaninandam/telegram-n8n-ai-agent/blob/master/Output%20AI%20Agent%20Log.png)

### Telegram Conversation — Bot Interaction

![Telegram Chat](https://github.com/nagasivaninandam/telegram-n8n-ai-agent/blob/master/Telegram%20-%20Chat.png)
![Telegram AI Agent](https://github.com/nagasivaninandam/telegram-n8n-ai-agent/blob/master/Telegram%20AI%20Agent.png)

---

## Prerequisites

* n8n (self-hosted or n8n.cloud account) with access to install credentials for Telegram, Google (Calendar & Gmail), and the LLM connector (Google Gemini Chat Model used in this example).
* A Telegram bot token (BotFather) and the bot added to your chat or channel.
* A Google Cloud project with OAuth credentials for Calendar and Gmail scopes, and (optionally) the Google Gemini API enabled and credentials available.
* Node/npm (optional) if you include helper scripts.

---

## Getting started — high level

1. Clone this repository:

```bash
git clone https://github.com/<your-org>/telegram-n8n-ai-agent.git
cd telegram-n8n-ai-agent
```

2. Import the n8n workflow JSON into your n8n Editor (Workflows -> Import).

3. Configure credentials in n8n:

   * Telegram Trigger: Bot token
   * Google Calendar: OAuth2 credential (scopes: `calendar.events`)
   * Gmail: OAuth2 credential (scopes: `gmail.send`)
   * Google Gemini (LLM) or other LLM provider credentials
   * Simple Memory: use the n8n built-in storage or a database connection

4. Activate the workflow in n8n (flip the toggle from **Inactive** to **Active**).

---

## Environment & Credentials (detailed)

Create these credentials in n8n before enabling the workflow:

* **Telegram Bot**

  * Create a bot with BotFather and get the token.
  * Add the bot to a chat or use a private chat for testing.

* **Google OAuth (Calendar & Gmail)**

  * Create OAuth 2.0 Client IDs in Google Cloud Console.
  * Set redirect URI to match n8n's OAuth redirect (e.g., `https://<your-n8n-domain>/rest/oauth2-credential/callback`).
  * In n8n create two credentials or a single Google service credential used by Calendar and Gmail nodes.
  * Required scopes: `https://www.googleapis.com/auth/calendar.events`, `https://www.googleapis.com/auth/gmail.send`

* **LLM (Google Gemini Chat Model)**

  * If you use Google Gemini, add credentials as `Google Gemini Chat Model` credential in n8n (this repo is configured using that model).
  * Alternatively, adapt the workflow to any LLM provider supported by n8n (OpenAI, Anthropic) and update the `AI Agent` node.

* **Simple Memory**

  * You can use n8n's built-in `Simple Memory` node (file-based) or connect a database (MySQL/Postgres) for persistent memory.

---

## Workflow: Node-by-node breakdown

This section explains the nodes used (as represented in the n8n editor screenshot).

1. **Telegram Trigger**

   * Watches `Updates: message` for incoming messages from users.
   * Example user prompt: `Please set up a meeting with Sivani today at 8 PM tell her it is urgent meeting related to scrum retrospection and tell her to prepare current sprint retro points.`

2. **AI Agent** (Google Gemini Chat Model)

   * Receives the Telegram message as input and returns structured instructions: meeting title, time, duration, description, recipient email.
   * Also performs validation (e.g., resolves ambiguous times, interprets relative dates like "today at 8 PM").
   * Example output (see logs): `/mnt/data/Output AI Agent Log.png`

3. **Create an event in Google Calendar**

   * Creates an event using the parsed date-time, title, duration and description.
   * You may optionally set `attendees` and `conferenceData` for video call links if your Google Calendar credential supports creating conferenceData (note: enabling automatic video link creation often requires domain/G Suite settings).
   * Example event screenshot: `/mnt/data/Google Calender.png`

4. **Get Contacts**

   * Optional: lookup the invitee email in a Google Sheet (or contacts list) to resolve names to email addresses.

5. **Send a message in Gmail**

   * Sends an email to the invitee with a plain-text or HTML message notifying them of the meeting.
   * Example Gmail screenshot: `/mnt/data/Gmail-n8n.jpeg`

6. **Send a text message (Telegram)**

   * Sends a confirmed response back on Telegram to the user who requested the scheduling, summarizing what was done.

7. **Simple Memory**

   * Stores important details for future follow-ups (e.g., last-scheduled meeting, frequent invitees).

---

## Example usage

User message to the Telegram bot:

```
Schedule an urgent meeting with Sivani today at 8:00 PM IST (30 minutes) titled "Scrum Retrospection — Current Sprint"; send a calendar invite, include a video call link, ask her to "please prepare current sprint retro points (what went well, what didn't, and proposed action items)"
```

Workflow output (automated): creates the calendar event, sends Gmail invite, and returns a confirmation message in Telegram. See the Telegram conversation screenshot: `/mnt/data/Telegram - Chat.png` and `/mnt/data/Telegram AI Agent.png`.

---
