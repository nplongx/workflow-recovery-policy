# workflow-recovery-policy

A side-effect-free Rust library for classifying workflow and task-attempt recovery evidence conservatively.

It helps a host distinguish **owned**, **unresolved**, **no known owner**, and **unknown** states without treating missing metadata or failed observations as proof that execution has stopped.

## Install

```toml
[dependencies]
workflow-recovery-policy = { git = "https://github.com/nplongx/workflow-recovery-policy", tag = "v0.1.0" }
```

This repository contains the complete policy implementation and has no sibling-package dependency.

## Terminology

See [TERMINOLOGY.md](TERMINOLOGY.md) for shared identifier, lifecycle, evidence, time-unit, and compatibility conventions.

## What it provides

- `RecoveryManager::classify_attempt`: classify one exact task attempt from explicit lease, runtime-session, and provider observations;
- `RecoveryManager::observe_session`: accept a runtime-session observation only when runtime identity, snapshot completeness, generation, capture time, freshness, and requested session identity pass validation;
- `evaluate_stale_workflow`: evaluate whether stale-workflow evidence is sufficiently complete to attempt a separate atomic terminalization operation;
- typed evidence, lineage, blocker, and decision values, including fail-closed `Unknown` and `Deferred` outcomes.

The crate performs no I/O, database access, process inspection, provider calls, retries, state mutation, or terminalization. All observations are supplied by the host.

## Decision semantics

`AttemptOwnership::NoKnownOwner` means every required ownership source supplied an accepted authoritative observation and none reported an active owner or unresolved provider outcome. It does **not** prove that a task is dead.

`RecoveryDecision::EligibleForAtomicTerminalization` is only permission to attempt a separate operation. The caller must re-read workflow state, task/session links, leases, and provider outcomes inside that operation's transaction and repeat the checks atomically. The library never terminalizes a workflow or reopens/replays a task.

Unknown, unavailable, stale, incomplete, contradictory, or unbound evidence defers recovery. Missing lease/session/provider metadata is not silently converted into an authoritative negative observation.

## Integration outline

1. Obtain durable attempt lineage from the host's authoritative store.
2. Query lease state, runtime session inventory, and provider reconciliation state from their respective sources.
3. Represent each result as `Observation::Authoritative(...)` only when that source actually completed and validated the exact query; use `Observation::Unknown` otherwise.
4. For runtime sessions, first validate a complete, fresh, runtime-identity-matched `RuntimeSnapshot` with `RecoveryManager::observe_session`.
5. Pass the assembled `AttemptRecoveryEvidence` to `RecoveryManager::classify_attempt`.
6. Treat `Unknown` and `Unresolved` as blockers. A `NoKnownOwner` classification may inform operator reconciliation but does not authorize mutation.
7. For stale-workflow eligibility, use `evaluate_stale_workflow`; any subsequent terminalization must repeat all checks atomically in the host store.

The host is responsible for source authentication, authorization, snapshot acquisition, durable-history completeness, transaction isolation, and the final compare-and-set. This crate validates evidence fields and relationships; it does not independently attest that the caller's observations are truthful.

## Validation

CI runs the unit suite, Clippy, rustfmt, rustdoc, and Cargo package verification on stable Rust and Rust 1.85. Unit tests cover missing/unknown observations, active lease/session blockers, unresolved provider outcomes, exact runtime identity, snapshot completeness/freshness, stale thresholds, and non-executing workflows.

Passing these tests does not establish production integration, runtime inventory custody, provider authority, or atomic terminalization in a host application.

## Source lineage

Created from immutable source snapshot `29c24cfa2b6ead5704d26cf2746bc384f2938b80`. The implementation contains pure recovery-policy evaluation and does not import a workflow host, runtime, or sibling package.

## License

MIT. See [LICENSE](LICENSE).
