# nrun

`nrun` is a small standalone shell tool for running long-running commands and getting notified when they finish.

It’s designed to be:
- quiet for fast failures (fat-finger mistakes)
- reliable for long jobs
- persistent via macOS Notification Center
- optionally escalated to your phone (Pushover)

This is intentionally **opinionated**, but configurable.

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

Hammerspoon is optional; if it is not installed, nrun will still work,
but HUD/modal alerts and idle/lock detection will be disabled.

> Note: Hammerspoon may prompt for Accessibility permissions on first run.
> These are required for idle time and screen lock detection.

---

## Installation

Clone the repo and put `nrun` somewhere on your `PATH`:

```sh
git clone https://github.com/glucas/nrun.git
cd nrun
chmod +x nrun
cp nrun ~/.local/bin/
````

Make sure `~/.local/bin` is on your `PATH`.

---

## macOS Shortcut Setup

`nrun` sends notifications via a macOS Shortcut.

![Shell Notification Shortcut](docs/shortcut-v2.png)

Create a Shortcut named (by default):

```
Shell Notification
```

Shortcut actions (top to bottom):

1. **Receive Text and Rich Text**

   * If there’s no input: Continue (or Get Clipboard)

2. **Get Text from Shortcut Input**

3. **Get Dictionary from Input**

4. **Show Notification**

   * Title → dictionary value for key `title`
   * Body → dictionary value for key `body`
   * Attachment → empty

You can override the shortcut name via:

```sh
export NRUN_SHORTCUT_NAME="Your Shortcut Name"
```

---

## Usage

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
```

---

## Pushover Setup (Optional)

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

```
