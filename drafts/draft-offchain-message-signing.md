---
tzip: tbd
title: Off-Chain Message Signing
status: Draft
author: Klas Harrysson <klas@kukai.app>, Carlo van Driesten <carlo.van-driesten@vdl.digital>
type: I
created: 2024-02-21
date: To be determined
requires: ["draft-signer-requests"]
---

## Abstract

There is no formal message signing standard for Tezos yet. Message signing has many use cases. It can for example be used for code signing, so that authenticity and integrity can be validated. Right now decentralized applications (dApps) and wallets are relying on an informal way of doing message signing by using Micheline expressions, something it was never intended for. In addition the [failing_noop (tag 17)][] was not adopted for the use of off-chain messages due to the lack of documentation. This TZIP aims to solve that problem and define a standard that is simple, secure, future-proof and compatible with hardware wallets.

## General

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

## Specification

**Printable ASCII:** ASCII characters between 32 and 126 (inclusive).

### Data model

| Name               | Type     |
| -------------------|----------|
| Magic byte         | String   |
| Interface          | String   |
| Character encoding | Integer  |
| Message            | Bytes    |

#### Magic byte

```text
b"\x80tezos signed offchain message"
```

The first byte is the magic byte used for domain separation within the Tezos ecosystem. `0x80` was chosen and is registered in the [draft-signer-requests][] TZIP. The remaining part is to protect against cross-chain signature replay attacks (not all wallets have this protection from [bip44/slip10][]). Chosen to be "long enough" and to be descriptive. Any user-agents performing or verifying signatures of messages with first byte 0x80, MUST comply with this standard.

#### Interface

The `interface` acts as a domain separator to tell a wallet or SDK developer how to distinguish different off-chain message types. The String MUST be encoded with the printable ASCII character set. This field MAY be displayed to the user. The interface MUST contain a reference to the specification defining the `message` encoding as [URI][] and add the specifications `namespace` as uri fragment.

Example: `https://gitlab.com/tezos/tzip/-/blob/proposals/tzip-XY/tzip-XY.md#OFFCHAIN-MESSAGE-SIGNING`

#### Character encoding

There are different kinds of encodings available for a `message`. If encoding `Other` is used then the `Interface` specification MUST include the type of encoding. In certain applications e.g. a [PACK encoding][] might be useful but this TZIP does only provide the basic options as application use cases are not foreseeable.

| Id       | Encoding        | Hardware Wallet Support |
| ---------|-----------------|-------------------------|
| 0        | None            | -                       |
| 1        | Other           | -                       |
| 2        | Printable ASCII | Yes                     |
| 3        | UTF-8           | No                      |

#### Message

This field contains the message content that the user is requested to sign. It MUST be displayed to the user. The data model and message encoding MAY be further specified through the specification in the `interface`.

Replay protection is out of scope for this TZIP. Implementers MUST define an interface specification protecting from replay attacks if it is relevant to the application. [CAIP-122] includes suitable measures like e.g. the use of a nonce inside the message data structure.

### Message encoding

| Name                  | Size     | Contents                |
|-----------------------|----------|-------------------------|
| magic_byte            | 30 bytes | bytes                   |
| # Bytes in next field | 1 byte   | unsigned 8-bit integer  |
| interface             | variable | bytes                   |
| character_encoding    | 1 byte   | unsigned 8-bit integer  |
| # Bytes in next field | 2 bytes  | unsigned 16-bit integer |
| message               | Variable | bytes                   |

## Validation

* The wallet MUST enforce strict validation to ensure there is no deviation in any way from the expected data format.
* The wallet implementers MUST display the public key hash associated to the private key used for signing the message.
* Wallet implementers displaying the message as plaintext to the user SHOULD require the user to scroll to the bottom of the text area prior to signing.

## Test vectors

### Encoding & signing

```
message = "Hello world! @ block 3961750"

domain_seperator = "My Test dApp | Ghostnet"

=>

bytes = 0x8074657a6f73207369676e6564206f6666636861696e206d657373616765011774657a6f73207369676e6564206f6666636861696e206d657373616765011c48656c6c6f20776f726c6421204020626c6f636b2033393631373530

# Wallet
mnemonic = "all all all all all all all all all all all all"
derivation_path = m/44'/1729'/0'/0' (slip-10)
private_key = edskRgEboayXzSZHW5wK2beB4aZtfQtuc2ywwjPmSQYCg7unpVT2Sr1KUSzX9hNLJC25YcB4qZ1Wotu6EuDveWY

=>

signature = edsigu6UNY8NP7eXEyGbszvugiDWxZiRbk18Qdvt78eQMZxo2oSXHf1bmWT5Xrdvq7e8pw9tubUX1hWtBcphjk9yiJe4k3P3F4t
```

## References

- [failing_noop (tag 17)][] - A possible solution to encode a signed message for off-chain usage.
- [PACK encoding][] - An encoding used in LIGO.

[RFC 2119]: https://www.ietf.org/rfc/rfc2119.txt
[failing_noop (tag 17)]: http://doc.tzalpha.net/shell/p2p_api.html#failing-noop-tag-17
[bip44/slip10]: https://github.com/satoshilabs/slips/tree/master/slip-0010
[URI]: https://datatracker.ietf.org/doc/html/rfc3986
[CAIP-122]: https://chainagnostic.org/CAIPs/caip-122
[PACK encoding]: https://ligolang.org/docs/language-basics/tezos-specific?lang=jsligo#pack-and-unpack
[draft-signer-requests]: URI

## Copyright

This document is licensed under [MIT][https://spdx.org/licenses/MIT.html].
