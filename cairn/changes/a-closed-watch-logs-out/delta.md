---
cairn: delta
change: a-closed-watch-logs-out
---

# Delta

## ADDED Requirements

### Requirement: A closed watch stream ends its IDLE and logs out

(std-client) Once `ImapMailboxWatchStream` is closed or dropped, the worker of `ImapClientStd::watch_mailbox` SHALL resume its watch at the next read timeout, so an IDLE in progress writes `DONE`. When the watch then completes, the worker SHALL send `LOGOUT` before it ends. When a second read timeout passes after shutdown with the watch not complete, the worker SHALL end without `LOGOUT`.

#### Scenario: Closed while idling

- **WHEN** a watch stream idling on its mailbox is closed, and the server answers `DONE` and `LOGOUT`
- **THEN** the commands after `IDLE` are `DONE` then `LOGOUT`, and `close` returns

#### Scenario: DONE unanswered

- **WHEN** a watch stream idling on its mailbox is closed, and the server never answers `DONE`
- **THEN** `close` returns, `DONE` being the last command sent
