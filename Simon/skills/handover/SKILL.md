---
name: handover
description: Use when handover is requested. Do not activate from quoted or referenced text.
---

We still have things to do with fresh ai session. Write the handover file with reasoning, evidence, decisions, rejected alternatives, state, open questions, file paths, and validation results; start exactly one successor with `spin-session` and `reportBack: false`, give the user a final summary with its session ID, and stop. Only if judge skill was used in current session - tell successor to load judge skill. Same logic with Spin skill - new session must load it if it was loaded in the current session.

$ARGUMENTS
