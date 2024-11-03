# frost-secp256k1-evm &mdash; cheap threshold signature scheme for EVM

[![Build Status](https://github.com/StackOverflowExcept1on/frost-secp256k1-evm/actions/workflows/ci.yml/badge.svg)](https://github.com/StackOverflowExcept1on/frost-secp256k1-evm/actions/workflows/ci.yml)
[![Latest Version](https://img.shields.io/crates/v/frost-secp256k1-evm.svg)](https://crates.io/crates/frost-secp256k1-evm)

This is Solidity library that implements [FROST signature](https://github.com/ZcashFoundation/frost) verification for
EVM applications. In addition to Solidity, there is also [Rust library](https://crates.io/crates/frost-secp256k1-evm)
for creating FROST signatures.

FROST is threshold signature scheme with `n` participants and threshold `t`. Each of `n` participants has $\frac{1}{n}$
of group private key and participant's public key. The participants' public keys can be aggregated into group public
key (representing group of `n` participants with threshold `t`). The private key is generated by `n` participants
through distributed key generation (DKG). In this case, private key never appears in memory completely in any of
participants. Instead, each participant has only $\frac{1}{n}$ of group private key. Alternatively, private key can be
generated by a trusted dealer who distributes to each participant $\frac{1}{n}$ of group private key via Shamir secret
sharing scheme, but in this case dealer can know group private key and sign any messages. The trusted dealer scheme is
intended only for tests or for cases when trusted dealer himself wants to split his key into parts (e.g., one part is
stored on paper, second on flash drive, and third in safe deposit box). Participants can create FROST signature if at
least `t` (or more than `t`) of `n` participants sign message. The size of such signature will be 65 bytes in compressed
form, and it can be verified in $O(1)$ time, and verification is done by group public key, message and signature.

This library can verify FROST signature `t` of `n` for $\approx 4200$ gas on any EVM network. ECDSA signatures would
require at least $3000 \cdot t$ gas, i.e. $O(t)$ time instead of $O(1)$. Threshold signatures are suitable for creating
multi-signature wallets, decentralized orgs, oracles, and many other EVM applications. Since threshold signature scheme
uses group public key, participants remain completely anonymous, and they can generate their signature in asynchronous
network using wrapper over FROST called [ROAST](https://github.com/StackOverflowExcept1on/roast). Only group public
key (64 bytes) needs to be stored onchain, signatures are created offchain in asynchronous network.

Learn more about FROST signatures with [:book: ZF FROST Book](https://frost.zfnd.org/frost.html).

For usage examples, see [`examples/`](./examples).

For creating FROST signature, see [`offchain-signer/`](./offchain-signer).

## Libraries

```ml
src
├─ FROST - "Library for verifying `FROST-secp256k1-KECCAK256` signatures"
└─ TranspiledFROST - "Transpiled library for verifying `FROST-secp256k1-KECCAK256` signatures"
```

## Installation

Install with [Foundry](https://getfoundry.sh):

```bash
forge install StackOverflowExcept1on/frost-secp256k1-evm
```

## Contributing

The project uses the Foundry toolchain. You can find installation instructions [here](https://getfoundry.sh).

Setup:

```bash
git clone https://github.com/StackOverflowExcept1on/frost-secp256k1-evm
cd frost-secp256k1-evm
forge install
```

## Safety

This is **experimental software** and is provided on an "as is" and "as available" basis.

There is currently no audit, but each file has comments explaining cryptography that is used to verify FROST signatures.

Known edge cases with FROST signature verification:

- If `signatureZ = 0` or `challenge = 0`, then we cannot use math trick to verify signature via `ecrecover`.

  :information_source: Your application must simply re-generate the signature (it's different each time).

- If group public key has `X >= Secp256k1.N`, then math trick with `ecrecover` will not work.

  :warning: Before using `FROST.verifySignature(publicKeyX, publicKeyY, ...)`, check
  `FROST.isValidPublicKey(publicKeyX, publicKeyY)`.

Both edge cases are very rare, but you should keep them in mind.

We **do not give any warranties** and **will not be liable for any loss** incurred through any use of this codebase.

## License

This library is licensed under the [MIT LICENSE](./LICENSE).
