---
tzip: tbd
title: Off-Chain Message Signing
status: Draft
author: Klas Harrysson <klas@kukai.app>, Carlo van Driesten <carlo.van-driesten@vdl.digital>
type: I
created: 2024-02-21
date: To be determined
---

## Abstract

There is no formal message signing standard for Tezos yet. Message signing has many use cases. It can for example be used for code signing, so that authenticity and integrity can be validated. Right now decentralized applications (dApps) and wallets are relying on an informal way of doing message signing by using Michelson data (magic byte `0x05`), something it was never intended for. In addition the [failing_noop (tag 17)][] was not adopted for the use of off-chain messages due to the lack of documentation. This TZIP aims to solve that problem and defines a standard that is simple, secure, extendable and compatible with hardware wallets.

## General

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

## Specification

**Printable ASCII:** ASCII characters between 32 and 126 (inclusive).

### Data model

| Name               | Type     |
| -------------------|----------|
| Magic string       | String   |
| Interface          | String   |
| Character encoding | Integer  |
| Message            | Bytes    |

#### Magic string

```text
b"\x80tezos signed offchain message"
```

The first byte is the `magic byte` used for domain separation within the Tezos ecosystem. `0x80` was chosen and is registered in the [draft-signer-requests][] TZIP. The remaining part is to protect against cross-chain signature replay attacks (not all wallets have this protection from [bip44/slip10][]). It is chosen to be "long enough" and descriptive in order to make it very unlikely to have a meaning outside the context of the Tezos off-chain message signing. Any user-agents performing or verifying signatures of messages with the first byte `0x80` MUST comply with this standard.

#### Interface

The `interface` acts as a domain separator to tell a wallet or SDK developer how to distinguish different off-chain message types. The string MUST be encoded with the printable ASCII character set. This field MAY be displayed to the user. The interface MUST contain a reference to the specification defining the `message`. It is a [URI][] and RECOMMENDED to be suffixed with the specifications `namespace` as URI fragment or OPTIONALLY with the name and number of the specification like e.g. `#TZIP-10`.
It is RECOMMENDED to host the specifications under a dedicated domain in order to facilitate easily human readable URIs as it is done e.g. for the Chain Agnostic Improvement Proposals (CAIPs). Tezos Improvement Proposals MAY use the following abbreviation `tzip://` as prefix with the TZIP number as suffix whereas it is REQUIRED to maintain the TZIP specification at `https://gitlab.com/tezos/tzip`. The `interface` string MUST NOT be empty. The maximum length of the URI is `255` characters.

```text
Examples:
https://gitlab.com/tezos/tzip/-/blob/proposals/tzip-XY/tzip-XY.md#OFFCHAIN-MESSAGE-SIGNING
https://chainagnostic.org/CAIPs/caip-122#CAIP-122
https://chainagnostic.org/CAIPs/caip-122#SIGN-IN-WITH-X
tzip://78
```

#### Character encoding

There are different kinds of encodings available for a `message`. If encoding `Custom` is used then the `Interface` specification MUST include the type of encoding. In certain applications e.g. a [PACK encoding][] might be useful but this TZIP does only provide the basic options as application use cases are not foreseeable. If the encoding `Custom` is not supported then the dApp or wallet MUST display it as hexadecimal `string`.

| Id       | Encoding        | Hardware Wallet Support |
| ---------|-----------------|-------------------------|
| 0        | Printable ASCII | Yes                     |
| 1        | UTF-8           | No                      |
| 2        | Custom          | -                       |

#### Message

This field contains the message content that the user is requested to sign. It MUST be displayed to the user. The data model and message encoding MAY be further specified through the specification in the `interface`.

Replay protection is out of scope for this TZIP. Implementers MUST define an interface specification protecting from replay attacks if it is relevant to the application. [CAIP-122] includes suitable measures like e.g. the use of a nonce inside the message data structure.

### Message encoding

| Name                  | Size     | Contents                |
|-----------------------|----------|-------------------------|
| magic_string          | 30 bytes | bytes                   |
| # Bytes in next field | 1 byte   | unsigned 8-bit integer  |
| interface             | variable | bytes                   |
| character_encoding    | 1 byte   | unsigned 8-bit integer  |
| # Bytes in next field | 2 bytes  | unsigned 16-bit integer |
| message               | Variable | bytes                   |

## Backwards compatibility

A dApp or wallet provider MUST fall back to the default `interface: tzip://tbd` if it does not support the provided interface specification and display the message including a warning.

## Validation

* The wallet MUST enforce strict validation to ensure there is no deviation in any way from the expected data format.
* The wallet implementers MUST display the public key hash associated to the private key used for signing the message.
* Wallet implementers displaying the message as plaintext to the user SHOULD require the user to scroll to the bottom of the text area prior to signing.

## Test vectors

### Encoding & signing

```
message = "Hello world!"

interface = "tzip://tbd"

=>

bytes = 0x8074657a6f73207369676e6564206f6666636861696e206d6573736167650a747a69703a2f2f74626400000c48656c6c6f20776f726c6421

# Wallet
mnemonic = "all all all all all all all all all all all all"
derivation_path = m/44'/1729'/0'/0' (slip-10)
private_key = edskRgEboayXzSZHW5wK2beB4aZtfQtuc2ywwjPmSQYCg7unpVT2Sr1KUSzX9hNLJC25YcB4qZ1Wotu6EuDveWY4jkiKQr9H3k

=>

signature = edsigtvazvxVHsofbakqvqHtQGiYZBxNg8hfY45escmFpLTYeBjjBFUTt254UARm93qHpbQugGU5fmJWdf3Cm5FNMcP7oYPsa7c
```

## References

* [failing_noop (tag 17)][] - A possible solution to encode a signed message for off-chain usage.
* [PACK encoding][] - A Michelson instruction for encoding and decoding.

[RFC 2119]: https://www.ietf.org/rfc/rfc2119.txt
[failing_noop (tag 17)]: http://doc.tzalpha.net/shell/p2p_api.html#failing-noop-tag-17
[bip44/slip10]: https://github.com/satoshilabs/slips/tree/master/slip-0010
[URI]: https://datatracker.ietf.org/doc/html/rfc3986
[CAIP-122]: https://chainagnostic.org/CAIPs/caip-122
[PACK encoding]: https://ligolang.org/docs/language-basics/tezos-specific?lang=jsligo#pack-and-unpack
[draft-signer-requests]: tbd

## Copyright

This document is licensed under [MIT](https://spdx.org/licenses/MIT.html).
