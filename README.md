# PRF Coprocessor

This repository packages the Rust allocation witness kernel and the SP1 guest,
host, and script crates used by the Protocol Robustness Framework (PRF).

It owns proof program implementation and proof execution tooling. It does not
own canonical encoding, the public pro-rata allocation API, effect compilation,
the EVM statement contracts, scenario definitions, or capital lifecycle
transitions. Those remain in their existing repositories and PRF integration
workspace.

## Dependencies

- `prf-canonical-codec` is pinned to
  `prf-canonical` revision
  `5e2b520feca812b049ba397b7b3681963c66a48c`.
- `prf-pro-rata-allocation` is pinned to
  `prf-pro-rata-allocation` revision
  `48e0b362748cd39d09994a8aa1496c668199aa74`.
- SP1 crates are pinned to version `6.3.1` in the Cargo manifests.

These are source pins. This repository has not received SP1 or Solidity proof
qualification, and its crates are not released through a package registry.

## Workspace

Run native kernel checks from the repository root:

```sh
cargo test --locked -p allocation-kernel
```

SP1 guest builds and proof generation require the SP1 toolchain and are
documented in `sp1-script/README.md`. Proof generation is not implied by native
kernel test results.

The source snapshot and original PRF revision are recorded in `PROVENANCE.md`.
