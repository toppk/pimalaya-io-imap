---
cairn: change
id: a-closed-watch-logs-out
status: proposed
created: 2026-11-16
---

# A closed watch logs out

## Why

Closing an `ImapMailboxWatchStream` sets the shutdown flag, and the coroutine is written to answer it: `ImapMailboxWatch::resume` then sets the IDLE's done flag, the IDLE writes `DONE`, and the watch completes. But the worker of `ImapClientStd::watch_mailbox` never resumes it to find out: at the read timeout that would carry the flag in, it sees shutdown and returns, with the IDLE still running. The connection is then dropped mid-IDLE, without `DONE` or `LOGOUT`. Dovecot 2.4 logs every such close as `Disconnected: Connection closed (IDLE running for …)`, not `Logged out`, so a client that restarts its watch now and then fills the server log with what reads as broken connections.

## What

- At the first read timeout after shutdown, the worker resumes the coroutine once more instead of returning, so the IDLE writes `DONE` and the watch completes on the server's tagged `OK`.
- A watch that completed so, or a polling watch shut down between its waits, is between commands: the worker then sends `LOGOUT` on the connection before it ends.
- A second read timeout after shutdown means the server never answered `DONE`; the worker returns then without `LOGOUT`, as it did before. Closing takes at most two `shutdown_poll` plus the LOGOUT's own read, instead of one.
- A watch that ended in an error, or whose receiver was dropped while an event was pending, is not logged out: the connection may be mid-command.

## Scope / non-goals

- The NOTIFY watch of pimalaya/io-imap#7 has the same driver loop and gets the same change once both land.
- `ImapMailboxWatchStream::into_state` still returns the watch's state; the LOGOUT does not change it.
