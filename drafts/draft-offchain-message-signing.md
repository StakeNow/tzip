---
tzip: 28
title: Off-Chain Message Signing
status: Draft
author: Klas Harrysson <klas@kukai.app>, Carlo van Driesten <carlo.van-driesten@vdl.digital>
type: Interface
created: 2024-02-21
date: To be determined
version: 0.1
---

## Abstract

There is no formal message signing standard for Tezos yet. Message signing have many use cases. It can for example be used for code signing, so that authenticity and integrity can be validated. Right now dapps and wallets are relying on an informal way of doing message signing by using Micheline expressions, something it was never intended for. This TZIP aims to solve that problem and define a standard that is simple, secure, future-proofed and compatible with hardware wallets.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

### Definitions

* **Printable ASCII:** ASCII characters between 32 and 126 (inclusive).

### Data model

| Name | Type |
| -------- | -------- |
| Magic prefix | String |
| Interfaces | String |
| Character encoding | Integer |
| Message | Bytes |

#### Magic prefix

```text
b"\x80tezos signed offchain message"
```

First byte is the magic byte used for domain separation within the Tezos domain. 0x80 was chosen to put it on the upper half of the range and far away from protocol specify magic bytes. The remaining part is to protect against cross-chain signature replay attacks (not all wallets have this protection from bip44/slip10). Chosen to be "long enough" and to be descriptive. Any user-agents performing or verifying signatures of messages with first byte 0x80, **MUST** comply with this standard.

#### Interfaces
The Interfaces act as a domain separator to tell a wallet or SDK developer how to distinguish different off-chain message types. This also includes the Version of the current TZIP. String **MUST** be encoded with the printable ASCII character set and can not be an empty string. This field **MAY** be displayed to the user.

#### Message
The message content that the user is requested to sign. **MUST** be displayed to the user.

## Validation
Wallet **MUST** enforce strict validation to ensure there is no deviation in any way from the expected data format.

## Interface
* Wallet implementers MUST display the public key hash associated to the private key used for signing the message.
* Wallet implementers displaying the message as plaintext to the user SHOULD require the user to scroll to the bottom of the text area prior to signing.

## Replay protection
Replay protection is out of scope for this tzip. Implementers **MUST** take necessary security precautions to include for example timestamp, dApp url, chainId, nounce etc. in the message or in the domain_seperator, as neccessary for their specific usecase.

## Encoding

| Name | Size | Contents |
| -------- | -------- | -------- |
| magic_prefix | 30 bytes | bytes |
| # Bytes in next field | 1 byte | unsigned 8-bit integer |
| interfaces | Variable | bytes |
| character_encoding | 1 byte | unsigned 8-bit integer |
| # Bytes in next field | 2 bytes | unsigned 16-bit integer |
| message | Variable | bytes |

#### Character encoding
| Id | Encoding | Hardware Wallet Support |
| -------- | -------- | -------- |
| 0 | None | - |
| 1 | Printable ASCII | Yes |
| 2 | UTF-8 | No |

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
