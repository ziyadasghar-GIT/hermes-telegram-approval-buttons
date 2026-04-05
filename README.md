# Hermes Telegram Approval Buttons

Adds **inline keyboard approval buttons** to the Hermes gateway's Telegram integration — matching the button-based exec approvals that Discord already has.

Instead of typing `/approve <uuid> allow-once`, you get one-tap buttons:

- ✅ **Allow Once** — run this command one time
- 🔏 **Allow Session** — approve this command pattern for the rest of the session
- 🔓 **Always Allow** — permanently whitelist this command
- ❌ **Deny** — cancel the command

When a button is clicked, the message updates to show the decision and the agent unblocks immediately.

---

## Prerequisites

- Hermes gateway (`hermes-agent`) installed
- Telegram bot configured and working in Hermes
- Python 3.11+ with `python-telegram-bot` library

## Installation

### Option 1: Replace the file (simplest)

Backup and replace your `telegram.py` with the version in this repo:

```bash
# Backup original
cp /path/to/hermes-agent/gateway/platforms/telegram.py \
   /path/to/hermes-agent/gateway/platforms/telegram.py.bak

# Replace with patched version
cp gateway/platforms/telegram.py \
   /path/to/hermes-agent/gateway/platforms/telegram.py

# Restart Hermes
hermes gateway restart
```

### Option 2: Apply the patch

```bash
cd /path/to/hermes-agent
git apply telegram-approval-buttons.patch
```

### Option 3: Merge the changes manually

If you already have custom modifications to `telegram.py`, manually apply the changes from the diff:

1. Add to imports (line ~20):
```python
from telegram import Update, Bot, Message, InlineKeyboardMarkup, InlineKeyboardButton
from telegram.ext import (
    ...
    CallbackQueryHandler,  # ADD THIS
    ...
)
```

2. Add to `__init__` (after `self._dm_topics_config`):
```python
self._exec_approval_cache: Dict[str, dict] = {}
```

3. Register handler in `connect()` (after the media handler):
```python
self._app.add_handler(
    CallbackQueryHandler(self._handle_callback_query)
)
```

4. Add `send_exec_approval()` and `_handle_callback_query()` methods to the `TelegramAdapter` class. See the full diff in `telegram-approval-buttons.diff`.

---

## How it works

When Hermes blocks a dangerous exec command, instead of sending a plain text message with instructions to type `/approve`, it now sends a message with 4 inline keyboard buttons.

```
⚠️ Command Approval Required

```
sleep 1
```
Reason: harmless test command

[✅ Allow Once] [🔏 Allow Session]
[🔓 Always Allow] [❌ Deny]
```

Clicking a button:
1. Calls `resolve_gateway_approval(session_key, choice)` to unblock the waiting agent thread
2. Updates the message to show the decision and removes the buttons
3. Shows an alert confirming the choice

---

## What counts as a "dangerous" command?

Hermes uses its own exec safety system. By default, `rm`, `dd`, `fdisk`, and similar destructive commands require approval. The button system works with Hermes's existing approval flow — this feature just adds a better UI on top.

---

## Contributing

PRs welcome! This was built as a community contribution to the [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) project.

**View on GitHub:** https://github.com/ziyadasghar-GIT/hermes-telegram-approval-buttons

## License

Same as [Hermes Agent](https://github.com/NousResearch/hermes-agent) — MIT
