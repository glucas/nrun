# nrun

`nrun` is a small standalone shell tool for running long-running commands and getting notified when they finish.

It’s designed to be:
- quiet for fast failures (fat-finger mistakes)
- reliable for long jobs
- persistent via macOS Notification Center
- smart: escalates to your phone when you're away

`nrun` is intentionally **opinionated**, but configurable where it matters.
See [Notification Behavior](#notification-behavior-defaults) for details on when and how notifications are sent.

> [!IMPORTANT]
> macOS-only: This tool relies on macOS Shortcuts and Notification Center.

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
- `jq` (used to safely construct notification payloads)
- **Optional:**
  - [Hammerspoon](https://www.hammerspoon.org) (HUD / idle / lock detection)
  - [Pushover](https://pushover.net) account (phone notifications)

> [!TIP]
> Hammerspoon is optional; if it is not installed, nrun will still work,
> but HUD/modal alerts and idle/lock detection will be disabled.


### Installing dependencies with Homebrew

If you use Homebrew, you can install the required and optional dependencies with:

```sh
brew install jq
brew install --cask hammerspoon
```

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
# or any directory already on your PATH
```

---

## macOS Shortcut Setup

`nrun` uses a macOS Shortcut to display notifications via Notification Center.
Shortcuts are used instead of AppleScript to ensure notifications persist and are
not dismissed immediately.

![Shell Notification Shortcut](docs/shortcut-v2.png)

The Shortcut is intentionally simple and designed to be recreated manually.

➡️ **See [`docs/shortcut.md`](docs/shortcut.md) for full setup instructions,
troubleshooting notes, and a sanity test.**

---

## Usage


Run `nrun --help` to see all available options.

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

These defaults are chosen to avoid noise while still surfacing meaningful events.

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
NRUN_SUPPRESS_FAIL_LT=15        # Suppress notifications for failures faster than this (seconds)
NRUN_HUD_IF_LONGER=45           # Show HUD / modal alerts if runtime exceeds this (seconds)
NRUN_MODAL_ON_FAIL=1            # Use modal dialog instead of HUD on failure
NRUN_PHONE_IF_LONGER=300        # Send phone notification if away and runtime exceeds this (seconds)
NRUN_PHONE_IF_IDLE=120          # Consider machine "away" after this many idle seconds
NRUN_PHONE_IF_LOCKED=1          # Consider machine "away" when screen is locked
NRUN_SHORTCUT_NAME="Shell Notification"  # Name of the macOS Shortcut to invoke
NRUN_SECRETS_FILE="$HOME/.config/nrun/secrets"  # Pushover credentials file
```

---

## Pushover Setup (Optional)

Pushover credentials are read from `~/.config/nrun/secrets`. Override NRUN_SECRETS_FILE if you prefer a different location. If the file is missing or unreadable, phone notifications are silently skipped.

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

The secrets file is sourced at runtime; ensure it is readable only by you.

---

## Philosophy

`nrun` is meant for **long-running or attention-worthy commands**, not every shell invocation.

If a command finishes instantly, you’re probably watching it already.

If it runs for minutes, you probably want to know when it finishes — even if you’ve walked away.

---

## Non-goals

`nrun` is intentionally small and focused. The following are explicit non-goals:

- Cross-platform support
- Replacing shell history or job control
- Fine-grained per-command notification rules

---

## FAQ

### Why not `terminal-notifier` or `noti`?

I started with that idea 🙂

In practice, I ran into a few issues:

- Notifications often only appeared reliably after opening Notification Center
- Alerts could be dismissed immediately when Notification Center was opened
- Behavior varied depending on Terminal app, shell, and notification settings
- AppleScript-based notifications were increasingly unreliable

Using a macOS Shortcut to deliver notifications via Notification Center turned out to be
more reliable and consistent, especially for long-running jobs where I might have walked
away from my machine.

Once I added:
- duration-aware suppression (fast failures)
- escalation logic (HUD → Notification Center → phone)
- idle / lock detection
- optional phone notifications

…it became clear this was more than a thin wrapper around an existing tool.

If `terminal-notifier` works well for your use case, it’s still a perfectly reasonable choice.
`nrun` exists to solve a slightly different set of problems.

---

## License

MIT
