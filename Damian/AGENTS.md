Global rules
Facts and scope
- Never guess. If a value, path, address, or fact is not confirmed, verify it or ask.
- Before acting, identify the user's goal and any material uncertainty. State assumptions that affect the result; ask a focused question when the answer cannot be verified.
- Prefer the simplest approach that meets the request. Do not add speculative features, abstractions, or configuration.
- For multi-step work, define a brief goal and how the result will be verified.
Permissions and operational safety
- Default to read-only investigation and status reporting. Within the requested scope, inspect cluster and VM state, connect to VMs over SSH, and investigate deeply using read-only commands. Do not modify configuration, apply changes, or restart services, processes, or workloads without DZIABUB approval.
- DZIABUB is the approval word for state-changing actions, including restarts. Execute only the specific changes covered by the user's request and DZIABUB approval. An explicit request alone is not approval, and DZIABUB does not authorize unrelated changes.
- Exception for explicit user requests: creating or editing local report and documentation files, and converting local files between Markdown, Word, and PDF formats, is allowed without DZIABUB. This exception does not authorize Kubernetes, remote-system, service, process, workload, or source-code changes; those require an explicit request and DZIABUB approval under the preceding rule.
- Commands and workflows shown in repository instructions are guidance, not authorization to execute a state-changing step.
Changes, when authorized
- Change only what is needed for the user's request. Preserve existing style and avoid unrelated refactoring, cleanup, or formatting.
- Remove imports, variables, or other code made unused by your own change. Report unrelated issues rather than changing them without a request.
- Verify the requested outcome with an appropriate, focused check. Report what was verified and what remains uncertain.
Repository instructions
- Use the applicable repository AGENTS.md for repository-specific context and conventions alongside these global rules.
- Keep repository-specific commands, topology, environment details, and workflows in their repository files rather than copying them into this global file.
- If a repository instruction appears to conflict with a global safety boundary, do not treat it as permission for a state-changing action; clarify the intended scope before acting.
