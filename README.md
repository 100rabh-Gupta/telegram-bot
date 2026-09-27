# backend-projects
# Telegram Learn Bot

Just a collection of projects that I made while learning backend development.
A modular Telegram bot built with **Node.js and Telegraf v4** that combines everyday utilities with interactive lessons on bot development. Explore middleware, sessions, callback queries, multi-step conversations, inline queries, and external API integrations in one project.

> **Project status:** Learning / portfolio project. State is stored in memory; this is not yet a production-ready service. See [limitations and roadmap](#limitations-and-roadmap).

## Features

| Command | What it does |
| --- | --- |
| `/start`, `/help` | Open the navigation menu and command reference |
| `/learn` | Browse nine bot-development topics using inline keyboards |
| `/quiz` | Answer six questions and receive a score |
| `/weather <city>` | Fetch current weather from wttr.in and update a loading message |
| `/note <text>`, `/notes`, `/clearnotes` | Add, view, and clear session-backed notes |
| `/form` | Collect name, experience, language, and goal, then confirm or restart |
| `/cancel` | Exit the registration form while inside it |
| `/sendphoto`, `/senddoc` | Demonstrate sending an image and an in-memory document |
| `/whoami` | Show Telegram user information and a profile photo when available |

With inline mode enabled, type these in any Telegram chat:

```text
@yourbotname weather Mumbai
@yourbotname joke
@yourbotname help
```

## Tech stack

- **Node.js / JavaScript (CommonJS)** — runtime and application code
- **Telegraf v4** — commands, middleware, sessions, scenes, and Telegram API access
- **Express** — HTTP endpoint for webhook mode
- **Axios** — weather and joke API requests
- **dotenv** — local environment configuration
- **nodemon** — development auto-reload
- **wttr.in and JokeAPI** — external integrations; no separate API keys required

## Architecture

```text
Telegram updates (long polling or POST /webhook)
  → Logger
  → Per-user rate limiter
  → In-memory session
  → Scene stage (multi-step form)
  → Command / callback / inline-query handlers
  → Telegram replies and external API requests

Handler errors → bot.catch(errorHandler)
```

Middleware order matters: sessions are registered before the scene stage so conversations can keep state across updates. The rate limiter allows **10 updates per user per 60-second fixed window**, including button taps; it is not a sliding-window limiter.

Notes, quiz state, and form profiles use Telegraf's default user-and-chat-scoped sessions. They are **lost on restart** and are not shared across server instances. Session scoping does not make replies private: messages sent in a group are visible to that group.

## Repository structure

```text
telegram-bot/
├── README.md
└── telegram-learn-bot/
    ├── .env.example
    ├── LICENSE
    ├── package.json
    ├── package-lock.json
    └── src/
        ├── bot.js             # Middleware registration, routing, startup
        ├── commands/          # Help, learning, weather, notes, quiz, media
        ├── handlers/          # Menu/quiz callbacks and inline queries
        ├── middleware/        # Logging, rate limiting, error handling
        ├── scenes/            # Registration WizardScene
        └── utils/             # Keyboards, learning content, quiz questions
```

## Getting started

### Prerequisites

- A maintained Node.js LTS release (Node.js 22 or newer recommended) and npm
- A Telegram account
- A bot token created through [@BotFather](https://t.me/BotFather) using `/newbot`

### Install and configure

```bash
git clone https://github.com/100rabh-Gupta/telegram-bot.git
cd telegram-bot/telegram-learn-bot
npm ci
cp .env.example .env
```

On Windows PowerShell, use `Copy-Item .env.example .env` instead of `cp`.

Edit `.env` and replace the placeholder with your bot token:

```dotenv
BOT_TOKEN=replace_with_your_bot_token
WEBHOOK=false
```

Keep the token private. `.env` is ignored by Git; never paste a real token into source code, screenshots, or issue reports. If it is exposed, revoke it through BotFather.

### Run locally

Run commands from **`telegram-learn-bot/`**, where `package.json` and `.env` live:

```bash
npm run dev
# Or run without auto-reload:
npm start
```

Open your bot in Telegram and send `/start`. Long polling does not require a public URL. Run only one polling process per bot token.

### Environment variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `BOT_TOKEN` | None | Required Telegram bot token |
| `WEBHOOK` | `false` | Use webhook mode only when set to `true` |
| `WEBHOOK_URL` | None | Public HTTPS base URL, required for webhook mode; omit the trailing slash and `/webhook` |
| `PORT` | `3000` | Express listening port in webhook mode |

### Webhook mode

Configure `.env`:

```dotenv
BOT_TOKEN=replace_with_your_bot_token
WEBHOOK=true
WEBHOOK_URL=https://your-domain.example
PORT=3000
```

Then run `npm start`. On POSIX shells, `npm run webhook` also forces webhook mode.

The application registers `https://your-domain.example/webhook` with Telegram and listens over HTTP on `PORT`. Your hosting platform or reverse proxy must provide HTTPS and forward requests to that port. Do not run polling and webhook deployments simultaneously with the same token. Webhook authentication and shutdown handling still need hardening before public production use.

### Enable inline mode

1. Send `/setinline` to BotFather.
2. Select your bot and set a placeholder such as `weather London`.
3. In a Telegram chat, type `@yourbotname help`.

## Manual verification

There is currently **no automated test suite or `npm test` script**. After configuring a token, use this smoke-test checklist:

- Send `/start` and `/help`; navigate the learning topics and back buttons.
- Try `/weather London` and `/weather` without an argument.
- Add a note, list it, clear it, and confirm the list is empty.
- Complete `/quiz` and check the score.
- Complete `/form`; also try its restart button and `/cancel`.
- Try the media commands and inline `help`, `weather`, and `joke` queries.
- Exceed ten updates in a minute to check rate limiting, then wait for the window to expire.
- Restart the process and confirm that notes and profiles do not persist.

Pace the checklist: button taps also count toward the rate limit. Use dummy notes and profile data, not sensitive information.

## Limitations and roadmap

The project demonstrates backend concepts, but the following work remains:

- **Persistence and scaling:** Add a database for durable notes/profiles and a shared session store such as Redis. The current rate-limit map is process-local and does not evict inactive users.
- **Automated quality checks:** Add unit and integration tests for middleware, form transitions, callback validation, and API failures, plus linting and CI.
- **Input safety:** Escape user/API text before inserting it into Telegram HTML, validate callback payloads, and enforce input and message-size limits.
- **Quiz correctness:** Reject duplicate or stale answers so replayed callbacks cannot inflate scores.
- **Webhook security and lifecycle:** Configure and verify Telegram's webhook secret token, expose a health endpoint, and explicitly close the HTTP server during shutdown.
- **Privacy and observability:** The logger currently records names, identifiers, and message text. Redact sensitive content and add structured logs and metrics before handling real user data.
- **Reliability:** Add response-schema validation, appropriate retries/backoff, caching for repeated external requests, and fail-fast configuration checks.

For an SDE portfolio, prioritise tested correctness, durable storage, and a documented deployment over adding more commands. Include measured performance or usage figures only after collecting them.

## Author

[Saurabh Gupta](https://github.com/100rabh-Gupta)

## License

[MIT](telegram-learn-bot/LICENSE)
