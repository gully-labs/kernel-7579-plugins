# Audit remediation test matrix

**Date:** 2026-09-15

**Scope:** All 27 findings in the Trail of Bits Kernel 7579 Plugins security assessment, the
remediated plugin modules, and deployment against the existing Kernel module ABI. No Kernel source
patch is part of this remediation.

**Result labels:** “Pass” means the plugin implementation has direct test evidence.
“Deployment-mitigated” means the stated risk is blocked only when every executing permission uses
`DefaultSecurityHook`; the risk remains for unhooked permissions. “Accepted/documented” means an
exact generic fix requires account-host or signing-client support outside this plugin repository.

## Final results

| Check | Result |
| --- | --- |
| Finding status | 23 pass, 3 deployment-mitigated, 1 accepted/documented |
| Plugin suite, production compiler settings | 685 passed, 0 failed |
| Plugin suite, isolated test transactions | 685 passed, 0 failed |
| Fuzzing | 27 properties, 270,000 generated cases passed |
| Timelock focused unit, behavior-tree, and integration tests | 101 passed, 0 failed |
| Real EntryPoint v0.9 Timelock integration | 17 passed, 0 failed |
| Pristine Kernel `a836274` composition | 3 passed, 0 failed; Kernel tracked worktree clean |
| Halmos symbolic safety properties | 79 passed, 0 failed, 0 timeout/unknown |
| Instrumented source coverage | 685 passed; 84.85% all-source lines, 95.68% active-code lines |
| Build size, formatting, and diff checks | Passed |

The Halmos total counts all 79 load-bearing safety properties. The 61 separate functions whose names
contain `_reachable` are witnesses: they are expected to produce counterexamples because they
demonstrate that success and rejection branches are reachable, so those results are not safety
failures.

## Finding-by-finding evidence

| # | Finding | Primary validation | Result |
| ---: | --- | --- | --- |
| 1 | Zero-address stateless ECDSA owners | Zero-owner and every noncanonical-length fuzz properties for signer and validator; ECDSA auth-gate Halmos proofs | Pass |
| 2 | Finite token-selector denylist | All ten blocked selectors, allowlist behavior, and explicit ERC-1363 boundary regression | Pass; documented boundary |
| 3 | RateLimit `validAfter` timestamp | 10,000 timestamp/window/count combinations, reset boundaries, exhaustion, and symbolic count cap | Pass |
| 4 | Missing paymaster gas accounting | 10,000 five-field gas combinations, exact/over-budget boundaries, overflow regression, and symbolic no-undercharge proofs | Pass |
| 5 | Unsigned Timelock proposal data | Sponsored creation calldata is part of the signed UserOperation; calldata/nonce/wait fuzzing, unsigned and post-signature tampering rejection, canonical decoder tests, locally derived v0.9 hash equivalence, one-use transient EntryPoint binding in an ERC-7562 account-associated slot, and real EntryPoint flow | Pass |
| 6 | Hook decodes every callback as execution | Canonical execute/executor/uninstall paths; malformed, trailing, short, unknown, and wrong-offset fuzz cases | Pass |
| 7 | Duplicate hook targets make cleanup unbounded | Re-add/remove regression, 10,000 randomized allowlist operation sequences, uninstall cleanup, and symbolic removal properties | Pass |
| 8 | ERC-1271 bypasses budget policies | Arbitrary sender/hash/signature fuzzing for Gas, RateLimit, and Throttle; budgets verified unchanged | Pass |
| 9 | Legacy WebAuthn ABI broken | Six-field ABI on stateful/stateless signer and validator paths, including both legacy boolean values | Pass |
| 10 | Replayable weighted hashes are chain-agnostic | Random distinct-chain fuzzing for signer and both validator variants plus weighted hash symbolic proofs | Pass |
| 11 | Zero `allowedPaymaster` disables enforcement | 10,000 arbitrary nonzero sponsors plus empty, zero, short, malformed, match, and mismatch cases | Pass |
| 12 | Daimo backend removed on Aurora | Missing-Solady-backend simulation covers Daimo success, malformed assertion rejection, and high-S rejection | Pass |
| 13 | Proposal creation charges the account immediately | Real EntryPoint test proves paymaster deposit pays while account balance remains unchanged; unsponsored creation rejects | Pass |
| 14 | ECDSAValidator falsely advertises hook support | Unit and behavior-tree checks prove hook type is false and supported types remain true | Pass |
| 15 | Module/internal permission ID mismatch | `DefaultSecurityHook` rejects account self-calls and module install/uninstall callbacks other than its own uninstall; exact module/internal ID equality remains a Kernel-host responsibility | Deployment-mitigated; residual risk for unhooked permissions |
| 16 | Zero-target hook alias bypass | Deterministic and 10,000-case target/value/calldata fuzz checks plus symbolic self-call denial | Pass |
| 17 | Unbounded WebAuthn indices exhaust gas | 10,000 arbitrary out-of-bounds locations reject successfully under a 150,000-gas cap | Pass |
| 18 | Timelock falsely supports stateless validation | 10,000 arbitrary module types prove only policy type succeeds; every stateless/signature entrypoint rejects | Pass |
| 19 | Unusable nonzero P-256 keys | Arbitrary off-curve key fuzzing, field boundaries, valid zero-coordinate point, and P-256/WebAuthn lifecycle proofs | Pass |
| 20 | RecoveryAction conflicts with the security hook | Direct regression proves allowlisting cannot bypass delegatecall rejection; RecoveryAction order/revert suite passes | Pass; documented configuration constraint |
| 21 | One permission consumes another policy's state | `DefaultSecurityHook` rejects policy checks and account self-calls before ordinary allowlist handling; its sole Timelock creation exception requires the exact selector, zero value, and a matching live transient commitment | Deployment-mitigated; residual risk for unhooked permissions |
| 22 | Signer callbacks replace another permission's signer | `DefaultSecurityHook` rejects calls to module-typed targets and account module-management callbacks; signer lifecycle tests cover normal installation and removal | Deployment-mitigated; residual risk for unhooked permissions |
| 23 | Validator-route selection bypasses CallerPolicy | Documentation and module API explicitly limit `CallerPolicy` to its permission route; root, validator, and fallback route selection remains outside plugin control | Accepted/documented Low risk; exact route binding requires host/client support |
| 24 | Timelock reads timestamp during validation | Validation derives the v0.9 UserOperation hash locally and records a transient commitment without an EntryPoint call or time read; the commitment slot has the exact ERC-7562 `keccak(account || bytes32 subkey)` shape; execution confirms the active hash with EntryPoint before starting the delay; gap, before-delay, expiry, symbolic-window, and real EntryPoint tests | Pass |
| 25 | MultiOwner storage is not account-associated | Storage-access regression covers all 129 relevant slots; two-account isolation and symbolic separation proofs | Pass |
| 26 | MultiOwner validation data is unbounded | Exact 32-owner and 4,096-byte limits, over-limit rejection, update/remove/uninstall, and word-boundary fuzzing | Pass |
| 27 | Weighted approvals survive guardian replacement | One-to-eight reinstall-cycle unit/fuzz tests for signer and both validator variants, plus symbolic dedup/configuration proofs | Pass |

## Module coverage

Coverage used `forge coverage --ir-minimum --exclude-tests`; Foundry warns that IR-minimum source
mapping is approximate.

| Module | Lines | Functions |
| --- | ---: | ---: |
| RecoveryAction | 100.00% | 100.00% |
| DefaultSecurityHook | 96.75% | 95.00% |
| CallerPolicy | 96.97% | 100.00% |
| GasPolicy | 100.00% | 100.00% |
| RateLimitPolicy | 100.00% | 100.00% |
| ThrottlePolicy | 100.00% | 100.00% |
| TimelockPolicy | 92.31% | 100.00% |
| ECDSASigner | 100.00% | 100.00% |
| P256Signer | 100.00% | 100.00% |
| WebAuthnSigner | 90.00% | 90.91% |
| WeightedECDSASigner | 97.92% | 100.00% |
| ECDSAValidator | 100.00% | 100.00% |
| P256Validator | 100.00% | 100.00% |
| WebAuthnValidator | 94.59% | 90.91% |
| WeightedECDSAValidator | 98.31% | 100.00% |
| MultiOwnerValidator | 96.75% | 100.00% |

All `src` files total 84.85% line coverage (1,042/1,228). Excluding the standalone, unreferenced
`src/utils/P256Verifier.sol`, active module code is 95.68% line covered (1,042/1,089). The audited
P-256 modules use `P256Validation` and Solady; `P256Verifier.sol` has no production reference.

## Commands

```sh
forge test --fuzz-runs 10000 --summary
forge test --isolate --summary
forge coverage --ir-minimum --exclude-tests --report summary
forge build --sizes
forge fmt --check
git diff --check
```

Every Halmos `check_` safety property was run individually; only explicitly named reachability
witnesses were excluded from the pass count. Kontrol proof files compile as part of the Forge build,
but the Kontrol CLI is not installed on this machine, so KEVM proofs were not rerun.
