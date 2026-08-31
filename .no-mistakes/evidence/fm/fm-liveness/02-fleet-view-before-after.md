# bin/fm-fleet-view.sh - the fleet table an operator actually reads

Same real-tmux fixture as 01-crew-state-before-after.txt.

## BEFORE (base 4ad8cba)

## Under Way
| ID | Current | Kind | Repo/Project | Backend | Endpoint | Artifact | Path | Watch / return channel |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| dead-crew | working / pane | ship | alpha | tmux | present | - | /tmp/fm-liveness-evidence/home/projects/dead-crew | bin/fm-peek.sh fm-dead-crew |
| done-crew | unknown / pane | ship | alpha | tmux | present | https://github.com/example/x/pull/7 | /tmp/fm-liveness-evidence/home/projects/done-crew | bin/fm-peek.sh fm-done-crew |
| ghost-crew | unknown / pane | ship | alpha | tmux | present | - | /tmp/fm-liveness-evidence/home/projects/ghost-crew | bin/fm-peek.sh fm-ghost-crew |
| live-crew | working / pane | ship | alpha | tmux | present | - | /tmp/fm-liveness-evidence/home/projects/live-crew | bin/fm-peek.sh fm-live-crew |

A dead harness reads `working`, and `ghost-crew` - whose tmux window does not exist -
reads `present`, because the endpoint probe accepted tmux's active-window fallback.
The Endpoint column carries no agent liveness at all for ordinary workers.

## AFTER (branch head 379b683)

## Under Way
| ID | Current | Kind | Repo/Project | Backend | Endpoint | Artifact | Path | Watch / return channel |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| dead-crew | unknown / agent-state | ship | alpha | tmux | present / dead | - | /tmp/fm-liveness-evidence/home/projects/dead-crew | bin/fm-peek.sh fm-dead-crew |
| done-crew | done / status-log | ship | alpha | tmux | present / dead | https://github.com/example/x/pull/7 | /tmp/fm-liveness-evidence/home/projects/done-crew | bin/fm-peek.sh fm-done-crew |
| ghost-crew | unknown / agent-state | ship | alpha | tmux | absent / dead | - | /tmp/fm-liveness-evidence/home/projects/ghost-crew | bin/fm-peek.sh fm-ghost-crew |
| live-crew | working / pane | ship | alpha | tmux | present / alive | - | /tmp/fm-liveness-evidence/home/projects/live-crew | bin/fm-peek.sh fm-live-crew |

`dead-crew` surfaces as `unknown / agent-state` + `present / dead`, `ghost-crew` as `absent / dead`,
the genuinely live crew is untouched at `working / pane` + `present / alive`, and `done-crew` keeps
its recorded terminal outcome (`done / status-log`) even though its harness is confirmed gone.
