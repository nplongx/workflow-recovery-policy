# Security policy

`workflow-recovery-policy` is a pure decision library, not a trust root or an authorization boundary.

- The host supplies all observations. The crate cannot prove that a lease query, runtime snapshot, or provider response came from an authoritative source.
- Runtime identity, completeness, generation, and freshness fields are checked as supplied values; they are not cryptographically attested by this crate.
- `NoKnownOwner` and `EligibleForAtomicTerminalization` are classifications, not proof of worker death or permission to mutate durable state without a separate atomic recheck.
- Never convert missing, stale, partial, or failed evidence into an authoritative negative observation.
- The crate does not perform terminalization, retries, task replay, database writes, or provider dispatch.

Report security issues privately to the repository owner rather than opening a public issue with exploit details.
