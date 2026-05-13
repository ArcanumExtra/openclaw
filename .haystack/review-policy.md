# Review Policies

## Plugin runtime lifecycle and registry activation
- **Paths**: `src/plugins/runtime.ts`, `src/plugins/registry.ts`, `src/plugins/host-hook-runtime.ts`, `src/plugins/host-hook-state.ts`
- **Severity**: critical
- **Reason**: Small lifecycle changes can permit stale or never-activated registries to perform side effects, causing cross-session state corruption, orphaned jobs, or unauthorized event emission that tests may not fully simulate.

## Host hooks and trusted policy execution paths
- **Paths**: `src/plugins/host-hooks.ts`, `src/plugins/host-hook-workflow.ts`, `src/plugins/trusted-tool-policy.ts`, `src/plugins/hooks.before-agent-finalize.ts`
- **Severity**: high
- **Reason**: These paths govern policy enforcement and before-tool-call decisions; subtle typing/normalization/cache bugs can bypass blocks or apply malformed rewrites while still passing unit tests.

## Session attachment validation and delivery routing
- **Paths**: `src/plugins/host-hook-workflow.ts`, `src/plugins/contracts/session-attachments.contract.test.ts`, `src/plugins/registry.ts`
- **Severity**: high
- **Reason**: Attachment handling mixes local filesystem access, MIME checks, and channel routing. Incorrect guards can leak host-local paths to gateway adapters, allow unsafe files, or produce partial-send inconsistencies with user-visible impact.

## Scheduler and cron integration
- **Paths**: `src/plugins/host-hook-workflow.ts`, `src/plugins/contracts/scheduled-turns.contract.test.ts`
- **Severity**: high
- **Reason**: Scheduler bugs can silently orphan cron jobs, mis-handle cleanup on restart/reset, or drift from cron protocol requirements; automated checks may miss production timing and rollback edge cases.

## Plugin SDK API surface and public types
- **Paths**: `src/plugin-sdk/**`, `src/plugins/hook-types.ts`, `src/plugins/host-hooks.ts`
- **Severity**: medium
- **Reason**: API/type surface changes can break third-party plugins or unintentionally expose internal host config/security-sensitive fields even when core tests pass.

## Instructions
- If a change alters package/source-entry boundary validation behavior, require human review to confirm it remains fail-closed when filesystem safety checks are uncertain, rather than falling back to runtime metadata that could mask invalid package state.
- If a PR changes whether malformed plugin policy output is ignored, normalized, or hard-rejected, require human judgment on compatibility versus security hardening, because both behaviors may be intentional depending on plugin ecosystem impact.
- If schema validation strategy changes across JSON Schema drafts, require human review to evaluate the compatibility tradeoff between loud failures and silent keyword drops for unlabeled schemas.
