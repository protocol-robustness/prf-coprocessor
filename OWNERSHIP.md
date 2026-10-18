# Artifact ownership

Repository boundaries follow artifact function and authoritative semantics.
Version labels alone do not determine ownership.

| Artifact | Owner | Boundary |
|---|---|---|
| Generic canonical encoding and SHA-256 domain hashing | `prf-canonical` | This workspace consumes the immutable canonical crate pin. `core/canonical.rs` is a compatibility facade. |
| Public allocation API, allocation arithmetic, allocation result roots and allocation-result V1 | `prf-pro-rata-allocation` | This workspace consumes the immutable allocator crate pin. `core/kernel.rs` adapts PRF witness input and public-value assembly. |
| Native witness execution, SP1 guest programs, SP1 hosts and proof scripts | `prf-coprocessor` | Owned here, including allocation, realized statement/execution, observed run, accounting arithmetic and source-bound guest/host code. |
| Source-bound assurance proof computation | `prf-coprocessor` | Rust witness computation and SP1 guest/host code are here. This is not Solidity admission or production qualification. |
| Source-bound admission, verifier adapters, controllers and Solidity commitments | `prf-evm` | Solidity contracts and their EVM verification tests stay there. |
| Effect compilation V2 and V3 | PRF integration workspace | `src/resolver_sim/pro_rata/effect_compilation_v2.clj` and `effect_compilation_v3.clj` stay in PRF. They are downstream economic effects, not coprocessor proof execution. |
| EVM statement V2 and admission ABI | `prf-evm` | Solidity statement verification and admission remain with the EVM contracts. SP1 host scripts may produce caller-owned proof fixtures. |
| Capital lifecycle transitions | `prf-capital-lifecycle` | Execution transitions stay in the capital lifecycle repository. `core/lifecycle.rs` only mirrors the PRF round cancellation-window projection used in a proof statement. |
| Scenario definitions and integration evidence | PRF integration workspace | Scenarios and broader evidence remain with PRF. `vectors/source-bound-entitlement-v1.json` is an immutable test input copy whose source revision and digest are recorded in `PROVENANCE.md`. |

## Dependency pins

```text
prf-coprocessor (this repository)
├── prf-canonical-codec @ prf-canonical 5e2b520feca812b049ba397b7b3681963c66a48c
├── prf-pro-rata-allocation @ prf-pro-rata-allocation 48e0b362748cd39d09994a8aa1496c668199aa74
└── SP1 Rust crates @ 6.3.1
```

The PRF integration repository and this repository must use these same
canonical and allocation revisions. Changing either pin requires re-running
cross-repository canonical-root and public-value compatibility checks.
