<!-- gully-header:start -->
<a href="https://gullychain.xyz"><img src="https://gullychain.xyz/brand/repos/kernel-7579-plugins.svg" alt="GullyChain · Kernel 7579 Plugins" width="100%"></a>

<p align="center">
  <a href="https://gullychain.xyz"><img src="https://img.shields.io/badge/gullychain.xyz-01D9FA?style=for-the-badge&labelColor=000000&logo=googlechrome&logoColor=000000" alt="Website"></a>
  <a href="https://app.gullychain.xyz"><img src="https://img.shields.io/badge/App-0A0F1A?style=for-the-badge&labelColor=000000&logo=googlechrome&logoColor=01D9FA" alt="App"></a>
  <a href="https://docs.gullychain.xyz"><img src="https://img.shields.io/badge/Docs-0A0F1A?style=for-the-badge&labelColor=000000&logo=readthedocs&logoColor=01D9FA" alt="Docs"></a>
  <a href="https://x.com/Gully_Labs"><img src="https://img.shields.io/badge/@Gully__Labs-0A0F1A?style=for-the-badge&labelColor=000000&logo=x&logoColor=white" alt="X"></a>
  <a href="https://t.me/GullyLabsNews"><img src="https://img.shields.io/badge/Telegram-0A0F1A?style=for-the-badge&labelColor=000000&logo=telegram&logoColor=26A5E4" alt="Telegram"></a>
  <a href="https://github.com/gully-labs"><img src="https://img.shields.io/badge/GitHub-0A0F1A?style=for-the-badge&labelColor=000000&logo=github&logoColor=white" alt="GitHub"></a>
</p>

<p align="center"><b>Kernel 7579 Plugins: Validators, executors and hooks for Kernel smart accounts on GullyChain.</b></p>

<p align="center"><sub>Maintained by <a href="https://gullylabs.xyz">Gully Labs</a> as part of <a href="https://gullychain.xyz">GullyChain</a>. Fork of <a href="https://github.com/zerodevapp/kernel-7579-plugins">zerodevapp/kernel-7579-plugins</a>, used under its MIT licence. Not affiliated with or endorsed by ZeroDev.</sub></p>

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
