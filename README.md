# Telert Run – GitHub Action

![Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Use%20this%20Action-blue?logo=github)
![License](https://img.shields.io/github/license/navig-me/telert-action.svg)

**Wrap any shell command in your workflow, time it, and get an instant notification via Telegram, Slack, Microsoft Teams, Pushover, Desktop OS, or Audio when it finishes — success *or* failure.** Powered by the [telert CLI](https://github.com/navig-me/telert).

---

## Why use Telert in CI?

| Pain point | What Telert Run gives you |
|-----------|--------------------------|
| Long builds finish while you’re away | Immediate ping on your phone or desktop |
| Silent test failures | Rich message with exit‑code & last 10 lines of output |
| Multiple channels to alert | Same workflow can broadcast to Telegram **and** Slack **and** Desktop |

Telert stays completely **open source & free**. This action just wires it into GitHub Actions.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Inputs](#inputs)
- [Environment variables (secrets)](#environment-variables)
- [Quick start](#quick-start)
- [Recipes](#recipes)
  - [Notify only on failure](#notify-only-on-failure)
  - [Multiple providers](#multiple-providers)
  - [Build matrix](#build-matrix)
  - [Labeling with commit info](#labeling-with-commit-info)
- [Versioning](#versioning)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Prerequisites

1. Python 3.8+ is installed automatically by the action (via `actions/setup-python`).
2. **Configure at least one Telert provider token** as a repository or organisation secret, e.g.
   - `TELERT_TELEGRAM_TOKEN` & `TELERT_TELEGRAM_CHAT_ID`
   - or `TELERT_SLACK_WEBHOOK`
   - full list in [Environment variables](#environment-variables).

> **Tip:** You can set secrets at org‑level so every repo may reuse them.

---

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `command` | **yes** | – | Shell command to execute. Use quotes if it contains spaces. |
| `label` | no | `GitHub Action` | Human‑readable identifier in the notification. |
| `provider` | no | – | Override default provider(s) for *this* run. Comma‑separated list, e.g. `slack,desktop`. |

---

## Environment variables

Put any Telert variable into `env:` or **preferably** `secrets:`. See [Telert docs](https://github.com/navig-me/telert/?tab=readme-ov-file#-environment-variables) for the full matrix. Below are some commonly used variables.

| Variable | Purpose |
|----------|---------|
| `TELERT_DEFAULT_PROVIDER` | Comma‑separated list of providers Telert should try in order. |
| `TELERT_TELEGRAM_TOKEN` / `TELERT_CHAT_ID` | Telegram bot token & chat to post to. |
| `TELERT_SLACK_WEBHOOK` | Incoming‑webhook URL. |
| `TELERT_TEAMS_WEBHOOK` | Power Automate / Teams webhook URL. |
| `TELERT_PUSHOVER_TOKEN` / `TELERT_PUSHOVER_USER` | Pushover credentials. |
| `TELERT_DESKTOP_APP_NAME` | Name shown in OS notification (self‑hosted runners). |

> :lock: **Never** commit secrets — use `Settings → Secrets → Actions`.

---

## Quick start

Add this job (or step) to any workflow:

```yaml
name: Build & Notify
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 1 ️ Do your build / tests
      - name: Build
        run: npm run build

      # 2 ️ Get a ping when it’s done
      - uses: navig-me/telert-action@v1
        with:
          command: "npm run build"
          label: "CI Build"
        env:
          TELERT_DEFAULT_PROVIDER: "telegram"
          TELERT_TELEGRAM_TOKEN: ${{ secrets.TELERT_TELEGRAM_TOKEN }}
          TELERT_TELEGRAM_CHAT_ID: ${{ secrets.TELERT_TELEGRAM_CHAT_ID }}
```

---

## Recipes

### Notify only on failure

```yaml
- uses: navig-me/telert-action@v1
  if: failure()   # <-- GitHub expression
  with:
    command: "npm test"           # rerun to capture exit‑code & logs
    label: "Tests failed"
  env:
    TELERT_DEFAULT_PROVIDER: "slack"
    TELERT_SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Multiple providers

```yaml
- uses: navig-me/telert-action@v1
  with:
    command: "pytest -q"
    provider: "telegram,desktop"  # overrides env default
  env:
    TELERT_TELEGRAM_TOKEN: ${{ secrets.TELERT_TELEGRAM_TOKEN }}
    TELERT_TELEGRAM_CHAT_ID: ${{ secrets.TELERT_TELEGRAM_CHAT_ID }}
    TELERT_DESKTOP_APP_NAME: "CI"
```

### Build matrix

```yaml
strategy:
  matrix:
    node: [18, 20]
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node }}

  - uses: navig-me/telert-action@v1
    with:
      command: "npm test"
      label: "Tests – Node ${{ matrix.node }}"
    env:
      TELERT_TELEGRAM_TOKEN: ${{ secrets.TELERT_TELEGRAM_TOKEN }}
      TELERT_CHAT_ID: ${{ secrets.TELERT_CHAT_ID }}
```

### Labeling with commit info

```yaml
- uses: navig-me/telert-action@v1
  with:
    command: "make deploy"
    label: "Deploy ${{ github.ref_name }} @ ${{ github.sha }}"
```

---

## Versioning

Use the **floating major tag** (`@v1`) for automatic minor updates without breaking changes. Pin to a full tag (`@v1.2.0`) to lock the exact bits if you have strict reproducibility needs.

| Tag | Meaning |
|-----|---------|
| `v1` | Latest backward‑compatible release of major 1 (recommended). |
| `v1.x.y` | Frozen point‑in‑time version – no surprises. |

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `command` not found | Remember the step runs in a *clean* shell; install / build earlier or provide a full path. |
| “No provider configured” error | Pass the relevant env secrets or set `TELERT_DEFAULT_PROVIDER`. |
| Desktop/audio doesn’t fire on GitHub‑hosted runners | Those providers need a self‑hosted runner with GUI / sound. |
| Unicode emojis garbled in Teams | Teams webhook sometimes strips UTF‑8; use plain characters. |

Enable debug logging with `ACTIONS_STEP_DEBUG=true` repo secret plus `TELERT_VERBOSE=1`.

---

## Contributing

- Issue? PR? → [navig-me/telert-action issues](https://github.com/navig-me/telert-action/issues)
- All code & docs under MIT.
- Before submitting a PR run `act -j test` to ensure the self‑test passes.

---

## License

This action is distributed under the terms of the MIT License. See `LICENSE` for details.

---

> :coffee: **Like it?** Support the parent project on [Buy Me a Coffee](https://buymeacoffee.com/mihirk) or become a [GitHub Sponsor](https://github.com/sponsors/mihir-khandekar). Every ping keeps the lights on!

