# Pascal Feishu Relay

This is a small local bridge between Pascal's Feishu private chat with Kalpas
and one persistent Codex session. It is a transport layer, not a second Jarvis
runtime.

## Current contract

- Ingress is the Kalpas bot event stream, with a 10-second user-message poll as
  reconnect insurance.
- Catch-up scans the last 24 hours and deduplicates persisted message IDs.
- Access is restricted to Pascal's exact Feishu open_id and the configured P2P
  chat. `ALLOW_ALL` remains disabled.
- Direct messages need no `/codex` prefix.
- All turns resume the `Pascal` App Server session, preserving the original
  Jarvis context across messages and restarts.
- The per-turn model is `gpt-5.6-terra`; this preserves the session while adding
  reliable image understanding.
- Feishu calendar, docs, drive, wiki, sheets, tasks, mail, approvals, IM, and
  minutes are available through installed `lark-*` skills and `lark-cli --as user`.
- Files, images, and voice messages are downloaded and attached to the same turn.
- Consequential external writes produce a Card 2.0 confirmation with Confirm
  and Cancel. Only Pascal can trigger its callback.
- Progress cards are deliberately low-noise and hide commands, local paths,
  session IDs, and duplicate final results.

## Runtime

LaunchAgent label:

```text
com.pascal.lark-codex-relay.next
```

Status:

```bash
launchctl print gui/$(id -u)/com.pascal.lark-codex-relay.next
```

Restart:

```bash
launchctl kickstart -k gui/$(id -u)/com.pascal.lark-codex-relay.next
```

Configuration:

```text
<repo>/.env
```

Logs:

```text
<repo>/.lark-codex/launchd.out.log
<repo>/.lark-codex/launchd.err.log
```

Run receipts and confirmation state live under `.lark-codex/`.

## Rollback

The previous Python relay, its database, logs, and LaunchAgent plist were not
deleted. To roll back, stop the new relay and bootstrap the old one:

```bash
launchctl bootout gui/$(id -u)/com.pascal.lark-codex-relay.next
launchctl bootstrap gui/$(id -u) "$HOME/Library/LaunchAgents/com.pascal.lark-codex-relay.plist"
launchctl kickstart -k gui/$(id -u)/com.pascal.lark-codex-relay
```

Do not run both relays simultaneously because both consume the same bot chat.

## Operating limits

The Mac must be awake and online, and Codex plus Feishu user authentication must
remain valid. If the machine is offline for more than 24 hours, continue in the
Codex desktop app or resend the old Feishu message after reconnecting.
