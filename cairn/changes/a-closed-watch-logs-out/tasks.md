---
cairn: tasks
change: a-closed-watch-logs-out
---

# Tasks

- [x] src/client.rs: the `watch_mailbox` worker resumes once more after shutdown and sends `LOGOUT` when the watch completed (`client::tests::closing_a_watch_stream_ends_its_idle_and_logs_out`, `closing_a_watch_stream_whose_server_ignores_done_skips_the_logout`)
- [x] Against Dovecot 2.4.5 (preauth `imap`): a closed watch logs `Disconnected: Logged out`, not `Connection closed (IDLE running …)`
- [x] CHANGELOG under [Unreleased]
- [ ] Fold the delta into cairn/spec, log, land
