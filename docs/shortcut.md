# macOS Shortcut Setup for nrun

## Overview

`nrun` uses a macOS Shortcut to display notifications via Notification Center.
Shortcuts are used instead of AppleScript to ensure notifications behave correctly:

- Banners (or Alerts) are shown immediately, without opening Notification Center
- Notifications persist and are not dismissed automatically

> [!NOTE]
> The Shortcut instructions and screenshots were created on macOS Sequoia.
> The Shortcuts UI may differ slightly on other macOS versions, but the overall
> configuration should be the same.


---

## Shortcut Configuration

![Shell Notification Shortcut](shortcut-v2.png)

You can manually recreate this Shortcut using the steps below.
A downloadable `.shortcut` file is **not** required or included.

Create a Shortcut named (by default):

```

Shell Notification

```

### Shortcut actions (top to bottom)

1. **Receive Text and Rich Text**
   - From: **Nowhere**
   - If there’s no input: **Continue** (or **Get Clipboard**, if you prefer)

2. **Get Text from Shortcut Input**

3. **Get Dictionary from Input**

4. **Show Notification**
   - Title → dictionary value for key `title`
   - Body → dictionary value for key `body`
   - Attachment → empty

You can override the shortcut name via:

```sh
export NRUN_SHORTCUT_NAME="Your Shortcut Name"
```

---

## If you don’t see the “Receive Text and Rich Text” block

On some macOS versions, the “Receive … input” block does not appear until the Shortcut
is temporarily enabled as a Quick Action.

When first enabled, the block may appear as something like:

> **Receive Images and 18 more input from Quick Actions**

This is normal.

If you can’t find or add the Receive block:

1. Open the Shortcut’s info panel (ⓘ).
2. Enable **Use as Quick Action** (and enable **Finder** if prompted).
3. Return to the editor; a **Receive … input** block should now be present.
4. Edit that block to:
   - limit input types to **Text and Rich Text**
   - set **From** to **Nowhere**
   - keep **Continue** when there’s no input
5. After configuring the block, you may disable Finder again if desired.

`nrun` sends input to the Shortcut via **stdin**, so it works reliably even when the Shortcut
receives input from “Nowhere”.

---

## Sanity test

Before using `nrun`, you can verify the Shortcut works on its own:

```sh
shortcuts run "Shell Notification" <<< '{"title":"Test","body":"Hello"}'
```

You should see a Notification Center banner with title **Test** and body **Hello**.
