# Security and integration overview

**Scope:** `src/actions`, `src/hooks`, `src/policies`, `src/signers`, and `src/validators`

**Revision:** 1.4

**Updated:** 2026-09-15

See [AUDIT_TEST_MATRIX.md](AUDIT_TEST_MATRIX.md) for the finding-by-finding regression, fuzz,
integration, formal-property, and coverage results.

## ERC-7579 module types

| ID | Type |
| ---: | --- |
| 1 | Validator |
| 2 | Executor |
| 3 | Fallback |
| 4 | Hook |
| 5 | Policy |
| 6 | Signer |
| 7 | Stateless validator |
| 10 | Stateless validator with sender |

Modules claim only the interfaces they implement. In particular, `ECDSAValidator` is not a hook and
`TimelockPolicy` is only a policy.

## Module capabilities

| Module | Types | Main configuration or restriction |
| --- | --- | --- |
| `CallerPolicy` | 5, 10 | Restricts ERC-1271 requests on its permission route by caller. |
| `GasPolicy` | 5 | UserOperation-only cumulative gas budget; optionally requires a specific paymaster. |
| `RateLimitPolicy` | 5 | UserOperation-only fixed-window operation budget. |
| `ThrottlePolicy` | 5 | UserOperation-only finite budget with a minimum interval. |
| `TimelockPolicy` | 5 | Sponsored, signed proposal creation followed by delayed execution. |
| `SudoPolicy` | 5 | Applies no restriction and must be selected intentionally. |
| `ECDSASigner` | 6, 7, 10 | One nonzero ECDSA signer per permission. |
| `P256Signer` | 6, 7, 10 | One P-256 signer per permission. |
| `WebAuthnSigner` | 6, 7, 10 | Legacy WebAuthn signature ABI with P-256 backend fallback. |
| `WeightedECDSASigner` | 6, 7, 10 | Sorted weighted guardians and an epoch-bound threshold. |
| `ECDSAValidator` | 1, 7, 10 | One nonzero ECDSA root owner. |
| `P256Validator` | 1, 7, 10 | One P-256 root owner. |
| `WebAuthnValidator` | 1, 7, 10 | WebAuthn root owner with curve-checked keys. |
| `WeightedECDSAValidator` | 1 | Sorted weighted root guardians and an epoch-bound threshold. |
| `MultiOwnerValidator` | 1 | Up to 32 stateless owner configurations and 4096 aggregate data bytes. |
| `DefaultSecurityHook` | 4 | Restricts account execution callbacks as described below. |

## Required deployment choices

### Existing Kernel compatibility and permission isolation

The plugins keep the existing Kernel module ABI and do not require a Kernel source patch.
`TimelockPolicy` binds proposal creation to the active UserOperation through EntryPoint v0.9. During
validation it derives the UserOperation hash locally with the v0.9 EIP-712 domain, so it does not call
EntryPoint or read `block.timestamp`. During execution it reads `getCurrentUserOpHash()` from the
EntryPoint configured for that policy installation and consumes the matching transient commitment.
This needs no new Kernel callback or storage protocol.

`DefaultSecurityHook` is the plugin-side deployment mitigation for cross-permission module-management
and callback attacks. It rejects account self-calls, the zero-target self-call alias, module install
or uninstall callbacks other than its own uninstall, calls to contracts that return the canonical
`isModuleType` result, and all delegatecalls. Every permission that can execute account calls must use
this hook for that mitigation to apply.

The sole module-call exception is an authenticated `TimelockPolicy.createProposal` call. The account
must explicitly allowlist that exact selector for a trusted Timelock target; a wildcard target
allowlist is insufficient. Before allowing the call, the hook asks the target, with a 150,000-gas cap,
to prove a live transient commitment for the same account, policy ID, proposal, nonce, and current
EntryPoint UserOperation hash. It also requires zero call value. This lets a Timelock permission use
the isolation hook without opening policy checks or signer/module lifecycle callbacks. The target is
a deployment trust boundary: a malicious replacement could lie through the authorizer interface,
while an over-budget response fails closed.

This is a deployment boundary rather than complete host enforcement. An unhooked permission, an
alternative hook, or a host path that does not invoke the hook retains the cross-permission risk.
Exact global binding between Kernel's internal permission ID and module callbacks must be enforced by
the account host. Findings 15, 21, and 22 are therefore deployment-mitigated and retain residual risk;
they are not technically closed by plugin code alone.

`CallerPolicy` restricts only the permission route on which it is installed. It does not govern root,
validator, or fallback ERC-1271 routes. Use distinct signing authority where route separation is a
security requirement. Generic cryptographic binding of a signature to Kernel's selected route needs
coordinated host and signing-client support; finding 23 is accepted and documented as residual Low
risk rather than treated as a plugin-only fix.

### Stateful budget policies

`GasPolicy`, `RateLimitPolicy`, and `ThrottlePolicy` reject ERC-1271 signature validation because a
signature request cannot safely consume or enforce their UserOperation state. Use `CallerPolicy` or
another signature-specific policy if a permission must support ERC-1271.

`GasPolicy` includes account verification gas, call gas, pre-verification gas, paymaster validation
gas, and paymaster post-operation gas in its maximum-cost calculation. When paymaster enforcement is
enabled with the zero-address wildcard, the UserOperation must still contain a real paymaster.

### Timelock proposals

Build proposal creation calldata with `TimelockPolicy.getProposalCreationCallData`. Submit that
calldata as a sponsored UserOperation. The permission signer authenticates the proposed calldata and
nonce because both are inside `userOp.callData`; validation writes a one-use commitment bound to the
current EntryPoint UserOperation hash and account execution consumes it after confirming the same
hash directly with the configured EntryPoint. The delay starts during execution, after a paymaster
has accepted the operation. The transient commitment disappears at the end of the transaction, so a
failed or unexecuted creation operation cannot leave reusable authorization.

When the permission uses `DefaultSecurityHook`, explicitly allowlist only the Timelock target's
`createProposal(bytes32,bytes,uint256)` selector. The hook rejects a wildcard allowlist and confirms
the live commitment before permitting the module call.

The deployment chain must support EIP-1153 `TSTORE` and `TLOAD`. The commitment exists only from
validation through execution in the same EntryPoint `handleOps` transaction, which is the intended
lifecycle. Proposal creation fails on chains without transient-storage opcode support; those chains
need a network upgrade or a different timelock design.

Proposal execution is one-shot: the policy marks the proposal executed during UserOperation
validation. If the account's later execution call fails, EntryPoint still consumes the account nonce
and the proposal remains consumed, so submit a new proposal before retrying the action.

Legacy 96-byte install data, `(delay, expirationPeriod, guardian)`, selects the official EntryPoint
v0.9 deployment at `0x433709009B8330FDa32311DF1C2AFA402eD8D009`. Extended 128-byte install data,
`(delay, expirationPeriod, guardian, entryPoint)`, supports a custom or local v0.9-compatible
EntryPoint. Proposal creation requires a nonzero paymaster in at least 52 bytes of paymaster data and
an empty `initCode`.

Do not encode proposal data in a policy signature or use an empty/no-op UserOperation. Those legacy
forms are rejected. Proposals are bound to `(account, policy id, calldata hash, nonce, configuration
epoch)`, and uninstall/reinstall invalidates proposals from the previous epoch.

### Weighted signatures

Weighted proposal and final UserOperation signatures include the current configuration epoch. A
guardian approval from an older installation cannot authorize a later guardian set. Replayable
Kernel validation modes use the chain-agnostic EIP-712 domain; other modes include the chain ID.
Signing clients must use the updated EIP-712 versions and include the configuration epoch in both
digests.

### WebAuthn and P-256

WebAuthn install and stateless data uses the 96-byte encoding
`abi.encode(WebAuthnSignerData(pubKeyX, pubKeyY), bytes32 authenticatorIdHash)` (or the validator
equivalent). Public keys must be valid P-256 curve points. Signatures retain the six-field legacy ABI:
`abi.encode(authenticatorData, clientDataJSON, responseTypeLocation, r, s, usePrecompiled)`.
`usePrecompiled` remains present for compatibility and is ignored; verification selects an available
Solady backend and falls back to the Daimo verifier when neither Solady backend exists.

### Multi-owner validation

Each `MultiOwnerValidator` owner selects a stateless validator and supplies its complete validation
data. Installation proves only that the target contract advertises stateless validation. Test every
owner configuration before deployment and keep a separately tested recovery authority. The owner
count and aggregate validation-data limits bound validation and uninstall costs.

The account-associated owner layout differs from the previous storage layout. Any in-place proxy
upgrade must migrate existing owner records before enabling validation with the new implementation.

## DefaultSecurityHook boundary

The hook accepts canonical `execute` and `executeFromExecutor` callbacks, plus the canonical callback
that uninstalls this hook itself. It rejects malformed or unknown callbacks, delegatecalls, account
self-calls (including a zero-target alias), detected module calls except the authenticated Timelock
creation path described above, non-allowlisted ETH transfers, and the ten token selectors listed in
[the hook specification](specs/DefaultSecurityHook.spec.md).

The token-selector list is finite. It does not cover ERC-777, ERC-1363, EIP-3009, permit-based pulls,
custom token functions, or future standards. Treat the target and selector allowlist as an explicit
deployment decision instead of relying on the denylist as a complete asset-transfer guarantee.

`RecoveryAction` requires delegatecall so validator lifecycle calls observe the account as
`msg.sender`. `DefaultSecurityHook` rejects every delegatecall before allowlist checks. A deployment
that uses both must provide and test a separate account-authorized recovery route.
