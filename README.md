# QuantumBench / runner

QuantumBench is an open-source signature conformance and measurement instrument for Stellar developers preparing for CAP-87. It keeps three evidence channels separate: Node/OpenSSL verification latency, local Protocol 30 SDK host-budget observations, and an observed testnet Ed25519 baseline. It does not blend their units or infer network capacity from local measurements.

The interface is an instrument panel with selectable evidence channels, latency distributions, binary sizes, a conformance matrix and an observed network capability gate. It is not a wallet or migration service. Contract-level ML-DSA verification does not replace Stellar network transaction signatures, which remain Ed25519.

## This repository

Strict raw-byte input validation, Node crypto verification, deterministic negative mutations, bounded repeated timings, sample statistics and protocol eligibility.

## Run the instrument

Use Node.js 26 or later (the initial pilot used 26.0.0). Clone the repositories as siblings:

```sh
for part in runner backend frontend contracts vectors; do
  git clone "https://github.com/quantumbench-labs/quantumbench-$part.git"
done
(cd quantumbench-backend && npm ci)
node quantumbench-backend/server.mjs
# In another terminal:
node quantumbench-frontend/server.mjs
```

Open http://127.0.0.1:4600. Load the recorded experiment, run a fresh bounded local measurement, switch evidence channels, inspect conformance cases, check the current testnet protocol, or simulate the fixed Ed25519 specimen. Reports can be exported/imported. Only the backend has npm dependencies; the runner, vector generator and frontend use Node built-ins.

Run JavaScript tests from the common parent: `node --test quantumbench-*/test.mjs`.

## Contract feature matrix

```sh
# Protocol 29 baseline: 2 host tests
cargo test --locked --manifest-path quantumbench-contracts/Cargo.toml
# Protocol 30 local host: 5 tests, including 18 ML-DSA cases
cargo test --locked --manifest-path quantumbench-contracts/Cargo.toml --no-default-features --features p30
# Reproduce local native-host observations
cargo run --quiet --locked --manifest-path quantumbench-contracts/Cargo.toml \
  --example host-probe --no-default-features --features p30
# Build only the P29 baseline with a compatible CLI
stellar contract build --locked --manifest-path quantumbench-contracts/Cargo.toml
```

Do not enable p29 and p30 together. P30 uses a pinned SDK release candidate. For a future P30 Wasm build, use CLI 30+ and `--no-default-features --features p30`, verify exported functions/artifact metadata, and first establish network support. `host-results.json` is a retained local observation, not chain data.

## Evidence and limits

| Channel | What it measures | What it does not establish |
| --- | --- | --- |
| OFFLINE_OPENSSL | Verification plus public-key import, microseconds, five warmups and retained timed samples | Soroban resources, network fees, validator throughput or certification |
| LOCAL_SOROBAN_HOST | Native test-contract calls under SDK 30.0.0-rc.1 / host 30.0.0 default calibration | Wasm guest metering, network calibration, latency or fees |
| TESTNET_SIMULATION | Ed25519 preflight resources or an explicit rejected changed-message call | Confirmed execution of that simulation or testnet ML-DSA support |

Four generated positive vectors are stored publicly; 22 deterministic cases cover valid input, message/signature mutation, truncated keys and ML-DSA context mutation/overlength. They are not authoritative NIST vectors or a certification suite. The local P30 Rust host checks accept the same generated signatures and reject changed message, signature, context, key and a 256-byte context for all three ML-DSA parameter sets.

Testnet reported protocol 29 during the pilot. Native ML-DSA eligibility is gated at protocol 30, and eligibility alone is not a successful native execution. The P30 verifier is implemented and tested in the local SDK host; **no P30 Wasm build or deployment was performed in this pilot**. Only the P29 Ed25519 baseline Wasm was built/deployed. A matching Stellar CLI 30+ is required before building the P30 feature for Wasm; never treat the P29 deployment as an ML-DSA verifier.

Snapshots and imported reports are point-in-time observations. Replaying retained XDR or a report does not authenticate historical RPC responses. Timings are noisy, include key import, and record runtime/platform metadata; sample counts are bounded at 1–200. The fixed API has no signing, arbitrary contract invocation, key custody or arbitrary RPC endpoint. Do not send private keys or confidential payloads to it.

## Public pilot

P29 baseline contract: `CDX54N62Q34SAVIWGQWDUDHEHKQWWU2MG5SLCA5JAA425RB2X2JSNPVZ`. Optimized Wasm SHA-256: `15971188b3a19ddbcb40ebf197c1c1e4f9063544169c62ee1c84a49d9c62c66b`. Three confirmed transactions (upload, deploy, verify Ed25519), one successful and one rejected Ed25519 simulation. Public resources and reproduction instructions are in the [backend](https://github.com/quantumbench-labs/quantumbench-backend), `testnet-evidence.json` and `TESTNET.md`.

## References

- [CAP-0087](https://github.com/stellar/stellar-protocol/blob/master/core/cap-0087.md)
- [Protocol 30 upgrade guide](https://stellar.org/blog/developers/bell-protocol-30-upgrade-guide)
- [Official Soroban SDK releases](https://github.com/stellar/rs-soroban-sdk/releases)
- [Node crypto verification API](https://nodejs.org/api/crypto.html#cryptoverifyalgorithm-data-key-signature-callback)

MIT licensed. See CONTRIBUTING.md and SECURITY.md.
