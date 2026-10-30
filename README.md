<!-- gully-header:start -->
<p align="center">
  <a href="https://gullychain.xyz">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset=".github/assets/gullychain-badge-dark.png">
      <img alt="Powered by GullyChain" src=".github/assets/gullychain-badge-light.png" width="360" height="96">
    </picture>
  </a>
</p>

<p align="center"><b>The social chain where your profile is your wallet. Arbitrum Orbit L3, testnet chain 485590.</b></p>

<p align="center">Validators, executors and hooks for Kernel smart accounts on Gully Chain.</p>

<p align="center">
  <a href="https://gullychain.xyz"><img src="https://img.shields.io/badge/gullychain.xyz-01D9FA?style=for-the-badge&labelColor=000000&logo=googlechrome&logoColor=000000" alt="Website"></a>
  <a href="https://docs.gullychain.xyz"><img src="https://img.shields.io/badge/Docs-0A0F1A?style=for-the-badge&labelColor=000000&logo=readthedocs&logoColor=01D9FA" alt="Docs"></a>
  <a href="https://t.me/GullyChain"><img src="https://img.shields.io/badge/Telegram-0A0F1A?style=for-the-badge&labelColor=000000&logo=telegram&logoColor=26A5E4" alt="Telegram"></a>
  <a href="https://discord.com/invite/tY2ErsNNBp"><img src="https://img.shields.io/badge/Discord-0A0F1A?style=for-the-badge&labelColor=000000&logo=discord&logoColor=5865F2" alt="Discord"></a>
  <a href="https://x.com/Gully_Labs"><img src="https://img.shields.io/badge/@Gully__Labs-0A0F1A?style=for-the-badge&labelColor=000000&logo=x&logoColor=white" alt="X"></a>
</p>

<p align="center"><sub>A Gully Chain fork of <a href="https://github.com/zerodevapp/kernel-7579-plugins">zerodevapp/kernel-7579-plugins</a>, maintained by <a href="https://gullylabs.xyz">Gully Labs</a>. It is used under its MIT licence; the upstream LICENSE, copyright and attribution notices are kept unchanged. The upstream README follows below as published. Not affiliated with or endorsed by zerodevapp.</sub></p>

---
<!-- gully-header:end -->

# Kernel 7579 Plugins

ERC-7579 modules for [Kernel](https://github.com/zerodevapp/kernel).

## Modules

| Module | Type(s) | Purpose |
|---|---|---|
| [`ECDSAValidator`](src/validators/ECDSAValidator.sol) | 1 / 4 / 7 / 10 | Single-owner ECDSA validator with owner-only `preCheck` |
| [`WebAuthnValidator`](src/validators/WebAuthnValidator.sol) | 1 | P256/WebAuthn validator |
| [`CallerPolicy`](src/policies/CallerPolicy.sol) | 5 / 10 | Per-`(id, wallet)` caller allowlist |
| [`TimelockPolicy`](src/policies/TimelockPolicy.sol) | 5 / 7 / 10 | Proposal → delay → execute flow with expiry |
| [`ECDSASigner`](src/signers/ECDSASigner.sol) | 6 / 7 / 10 | Single ECDSA signer per `(id, wallet)` |
| [`WeightedECDSASigner`](src/signers/WeightedECDSASigner.sol) | 6 / 7 / 10 | Weighted guardians with threshold |
| [`WebAuthnSigner`](src/signers/WebAuthnSigner.sol) | 6 | P256/WebAuthn signer |

Module type IDs follow ERC-7579: `1` validator, `4` hook, `5` policy, `6` signer, `7` stateless validator, `10` stateless validator with sender.

## Build & Test

```bash
forge build
forge test
```

## License

MIT


<!-- gully-footer:start -->
---

<p align="center">
  <a href="https://gullylabs.xyz">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset=".github/assets/powered-by-gully-labs-dark-2x.png">
      <img alt="Powered by Gully Labs" src=".github/assets/powered-by-gully-labs-light-2x.png" width="254" height="64">
    </picture>
  </a>
</p>
<!-- gully-footer:end -->
