# Evidence: firstmate stops reporting stale liveness for a dead crew (PR 2501, branch fm/fm-liveness)

All four artifacts here compare the SAME fixture on the base commit `4ad8cba`
and on the branch head `379b683`, so every difference is this change.

The fixture is not mocked at the boundary that matters: it runs a REAL tmux 3.2a
server on an isolated socket (`TMUX_TMPDIR`) with a real `FM_HOME`, and models
the four cases the change is about.

| Crew | tmux window | Foreground process | Status log |
| --- | --- | --- | --- |
| `live-crew` | exists | a real verified-harness process | `working:` |
| `dead-crew` | exists | husk shell only, harness process exited | `working:` (frozen) plus a frozen `busy` hook record |
| `done-crew` | exists | husk shell only, harness process exited | `done:` (terminal outcome) |
| `ghost-crew` | GONE | - | `working:` (frozen) |

| Artifact | Surface |
| --- | --- |
| `01-crew-state-before-after.txt` | `bin/fm-crew-state.sh <id>` - the one line a supervisor reads per crew |
| `02-fleet-view-before-after.md` | `bin/fm-fleet-view.sh` - the rendered fleet table |
| `03-session-start-endpoint-digest-before-after.txt` | `bin/fm-session-start.sh` FLEET STATE digest - the exact surface of upstream issue #3402, plus the live tmux behaviour that caused it |
| `04-herdr-retired-pane-push-before-after.txt` | the watcher's herdr push path: wake queue, triage log, and dedupe marker for a pane no task records |
| `05-session-start-suite-regression-and-fix.txt` | a branch-caused failure in `tests/fm-session-start.test.sh` that this run found behind an environment-dependent early abort, and the fixture fix applied for it |

Headline results:

- `dead-crew` moves from `state: working` (read from a frozen busy hook) to
  `state: unknown - source: agent-state`, so a dead worker no longer reads as
  working forever.
- `done-crew` keeps its recorded terminal outcome (`done / status-log`) even
  though its harness is confirmed gone - the terminal-record exemption.
- `live-crew` is byte-identical before and after: a genuinely live crew is never
  reported dead.
- `ghost-crew` (recorded tmux window no longer exists) moves from
  `endpoint: alive` to `endpoint: dead` in the session-start digest, which is
  upstream issue #3402 itself.
- A herdr push edge for a pane no task records no longer wakes the supervisor;
  it is absorbed with a triage-log line, and the dedupe marker still advances.
  The identical edge still wakes for a live recorded task.
