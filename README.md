# nrun

`nrun` is a small standalone shell tool for running long-running commands and getting notified when they finish.

It’s designed to be:
- quiet for fast failures (fat-finger mistakes)
- reliable for long jobs
- persistent via macOS Notification Center
- optionally escalated to your phone (Pushover)

This is intentionally **opinionated**, but configurable.

macOS-only. This tool relies on macOS Shortcuts and Notification Center.

---

## Features

- macOS Notification Center notifications (via Shortcuts)
- Optional phone notifications via Pushover
- Suppresses noisy fast failures
- Duration-aware escalation
- Optional HUD / modal alerts via Hammerspoon
- Idle / screen-lock aware escalation (optional)
- Safe handling of multiline command output
- No AppleScript
- No temp files

---

## Requirements

- macOS
- `bash` or `zsh`
- `jq`
- **Optional:**
  - Hammerspoon (HUD / idle / lock detection)
  - Pushover account (phone notifications)


### Installing dependencies with Homebrew

If you use Homebrew, you can install the required and optional dependencies with:

```sh
brew install jq
brew install --cask hammerspoon
```

> [!TIP]
> >Hammerspoon is optional; if it is not installed, nrun will still work,
> but HUD/modal alerts and idle/lock detection will be disabled.

> [!NOTE]
> Hammerspoon may prompt for Accessibility permissions on first run.
> These are required for idle time and screen lock detection.

---

## Installation

Clone the repo and put `nrun` somewhere on your `PATH`:

```sh
git clone https://github.com/glucas/nrun.git
cd nrun
install -m 0755 nrun ~/.local/bin/nrun
```

Make sure `~/.local/bin` (or your preferred location) is on your `PATH`.

---

## macOS Shortcut Setup

`nrun` uses a macOS Shortcut to display notifications via Notification Center.
Shortcuts are used instead of AppleScript to ensure notifications persist and are
not dismissed immediately.

![Shell Notification Shortcut](docs/shortcut-v2.png)

The Shortcut is intentionally simple and can be recreated manually.

➡️ **See [`docs/shortcut.md`](docs/shortcut.md) for full setup instructions,
troubleshooting notes, and a sanity test.**

---

## Usage


Show help and available options:

```sh
nrun --help

```

Basic usage:

```sh
nrun sleep 20
nrun terraform apply
nrun make deploy
```

Multiple commands:

```sh
nrun sh -c 'terraform init; terraform apply'
```

Custom title:

```sh
nrun --title "Deploy" make deploy
```

Force phone notification:

```sh
nrun --phone sleep 600
```

Disable local HUD / modal escalation:

```sh
nrun --no-hud terraform apply
```

---

## Notification Behavior (Defaults)

* **Fast failures** (default < 15s) are suppressed
* **Successful or long-running jobs** trigger a macOS notification
* **HUD / modal alerts** trigger after a configurable duration
* **Phone notifications** trigger only if:

  * the job ran long enough
  * and you appear to be away (idle / screen locked)
  * or `--phone` is explicitly set

All thresholds are configurable via environment variables.

---

## Configuration

Defaults (override via environment variables):

```sh
NRUN_SUPPRESS_FAIL_LT=15
NRUN_HUD_IF_LONGER=45
NRUN_MODAL_ON_FAIL=1
NRUN_PHONE_IF_LONGER=300
NRUN_PHONE_IF_IDLE=120
NRUN_PHONE_IF_LOCKED=1
NRUN_SHORTCUT_NAME="Shell Notification"
NRUN_SECRETS_FILE="$HOME/.config/nrun/secrets"
```

---

## Pushover Setup (Optional)

Pushover credentials are read `~/.config/nrun/secrets`. Override NRUN_SECRETS_FILE if you prefer a
different location.

Create a secrets file:

```sh
~/.config/nrun/secrets
```

Permissions:

```sh
chmod 600 ~/.config/nrun/secrets
```

Contents:

```sh
NRUN_PUSHOVER_USER_KEY="..."
NRUN_PUSHOVER_APP_TOKEN="..."
```

This file is sourced when `nrun` runs.

---

## Philosophy

`nrun` is meant for **long-running or attention-worthy commands**, not every shell invocation.

If a command finishes instantly, you’re probably watching it already.

If it runs for minutes, you probably want to know when it finishes — even if you’ve walked away.

---

## License

MIT
