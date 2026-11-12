# Contributing

Keep this crate deterministic and side-effect-free. Do not add database, runtime, process, network, provider, or workflow-store access to the policy layer.

Before submitting:

```sh
cargo fmt --all -- --check
cargo test --locked
cargo clippy --all-targets --locked -- -D warnings
cargo doc --no-deps --locked
cargo package --locked --no-verify
```

New evidence types must preserve the distinction between authoritative observations and unknown/unavailable sources. Changes to `NoKnownOwner`, `Unknown`, or atomic-terminalization eligibility require focused fail-closed tests and explicit compatibility notes.
