# SP1 host script

This crate is the SP1 host script for the allocation kernel. It builds the
guest program, executes and proves it, verifies the proof, and checks the
three-way equality:

```
PRF public result == native Rust public result == SP1 guest public values
```

## Commands

```bash
# Execute the guest (no proof)
cargo run --locked --release --manifest-path sp1-script/Cargo.toml \
  --bin allocation-prove -- --execute --input <input.json>

# Prove the realized-allocation statement, locally verify it, compare its exact
# public JSON bytes to native Rust, and persist an unsigned proof artifact.
cargo run --locked --release --manifest-path sp1-script/Cargo.toml \
  --bin realized-statement-prove -- --prove \
  --input <realized-statement-input.json> \
  --artifact <sp1-proof-artifact.json>

# Generate and verify a core proof
cargo run --locked --release --manifest-path sp1-script/Cargo.toml \
  --bin allocation-prove -- --prove --input <input.json>

# Execute one source-bound entitlement conformance vector against its golden
cargo run --locked --manifest-path sp1-script/Cargo.toml --bin source-bound-entitlement-prove -- \
  --execute --vectors ../../vectors/source-bound-entitlement-v1.json \
  --vector A-minimum-realistic

# Generate and verify an EVM-compatible Groth16 proof for that vector; the
# proof fixture includes vector provenance and Groth16 on-chain proof bytes
cargo run --locked --manifest-path sp1-script/Cargo.toml --bin source-bound-entitlement-prove -- \
  --prove --vectors ../../vectors/source-bound-entitlement-v1.json \
  --vector A-minimum-realistic --fixture <proof-fixture.json>

# The legacy --input form accepts exactly one raw successor witness JSON object.
# For example, extract the identity vector's base witness from the bundle:
jq -c '."base-input"' ../../vectors/source-bound-entitlement-v1.json \
  > /tmp/source-bound-entitlement-base-input.json
cargo run --locked --manifest-path sp1-script/Cargo.toml --bin source-bound-entitlement-prove -- \
  --prove --input /tmp/source-bound-entitlement-base-input.json

# Print the program verification key
cargo run --locked --release --manifest-path sp1-script/Cargo.toml \
  --bin allocation-vkey

# Generate an EVM-compatible (groth16/plonk) proof + Solidity fixture
cargo run --locked --release --manifest-path sp1-script/Cargo.toml \
  --bin allocation-evm -- --input <input.json> --system groth16 --fixture <fixture.json>
```

The guest program lives in `../sp1-program` and depends on the shared kernel in
`../core`. There is one source of kernel logic: `../core`.

## Reproducible realized-proof target

For a reproducible realized-proof run, pass caller-owned input and output paths:

```bash
cargo run --locked --release --manifest-path sp1-script/Cargo.toml \
  --bin realized-statement-prove -- --prove \
  --input <realized-statement-input.json> --artifact <sp1-proof-artifact.json>
```

It uses ignored project-local `.sp1-proof-cache/` Cargo, target, and Go module
cache directories. Those paths affect build caching only; the persisted proof
artifact records the proof profile, Rust-visible guest ELF digest, VK, exact
public bytes/digest, and a content-addressed sibling `sp1-bincode.v1` Core-proof
envelope, plus `rustc --version` and the SHA-256 of the locked
root `Cargo.lock`. A fresh cache requires the normal Cargo package
acquisition available in the supported CI/proving environment; subsequent runs
use `--locked` and must not resolve a different dependency graph.

## Proof mode

`cargo run --locked --manifest-path sp1-script/Cargo.toml --bin allocation-prove -- --prove`
uses the prover configured by the environment. Set `SP1_PROVER=cpu` for local
proof generation, or omit it to use
the default prover. Proofs are verified locally with the SP1 SDK before any
comparison is asserted.

## Environment

The core CPU prover may build SP1's native executor binary. Its Cargo-registry
source/cache must be writable for that build (a read-only dependency cache will
fail while creating `sp1-native-bins/.../.cargo-build-lock`). This is a build
cache requirement and does not change the guest ELF, verification key, or proof
identity.

The gnark Go build (for groth16/plonk) requires a filesystem with a generous
`NAME_MAX`. On systems where `$HOME` is on ecryptfs (NAME_MAX 143), point the Go
module cache elsewhere, e.g.:

```bash
GOPATH=/tmp/opencode/gopath GOMODCACHE=/tmp/opencode/gopath/pkg/mod
```
