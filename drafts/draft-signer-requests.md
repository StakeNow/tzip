---
tzip: tbd
title: Signer Requests
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Marigold <contact@marigold.dev>
type: I
created: 2022-09-20
date: 2024-02-21
requires: ["TZIP-10"]
version: 1
---


## Abstract

Signer applications and wallets use a **magic byte** prefix to distinguish the type of data to be signed.

This document serves as:

- a reference for used magic bytes, and
- a workflow definition to reserve bytes.

## General

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

## Motivation

Currently the definition of **magic bytes** is maintained by Octez in the context of the [octez-signer magic bytes][].

Wallets based on the [beacon-sdk][] implementation of [TZIP-10][] are using `0x03` and `0x05` in order to type payloads. Most wallets only accept to sign those two kinds of payloads. Currently only [AirGap Wallet][] accepts to sign raw data.

Incoming Layer 2 applications like [Smart Rollups][] or [Zk Rollups][] need to be able to reserve a byte in order to integrate into the tooling ecosystem. In addition, certain applications like the secure [Off-Chain Message Signing][] need a magic byte to signal its purpose to a wallet application e.g. to facilitate the signing of a [CAIP-122][] compliant sign-in message as implemented in the [Sign In With Tezos (SIWT)][] library.

## Specification

Remote signer applications MUST handle signing requests with the following format:

```xml
<magic_byte><data>
```

### Ranges

- Bytes between 0x00 and 0x7F SHALL be reserved for the Tezos protocol
- Bytes between 0x80 and 0xFF MAY be used by applications

### Table of reserved bytes

- **Message type**: A name that indicates the purpose of the message.
- **Reference**: [URI][] including a URI fragment stating the specification namespace like e.g.:

   `https://gitlab.com/tezos/tzip/-/blob/57c32be0e5d4bc6867cea83a12cf909894c42c41/proposals/tzip-10/tzip-10.md#TZIP-10`
- **Magic byte**: Currently and previously reserved magic bytes.
- **Status**: Indicates if a magic byte associated with a message type is actively used `approved` or deprecated `revoked`. The range of magic bytes reserved for the Tezos protocol has no status as it is governed by the Tezos governance.

| Message type                  | Reference                    | Magic byte | Status     |
|-------------------------------|------------------------------|------------|------------|
| Legacy block                  | [octez-signer magic bytes][] | 0x01       | -          |
| Legacy endorsement            | -                            | 0x02       | -          |
| Transfer                      | -                            | 0x03       | -          |
| Authenticated signing request | -                            | 0x04       | -          |
| Michelson data                | -                            | 0x05       | -          |
| Block                         | -                            | 0x11       | -          |
| Pre-attestation               | -                            | 0x12       | -          |
| Attestation                   | -                            | 0x13       | -          |
|-------------------------------|------------------------------|------------|------------|
| Off-Chain Message Signing     | [Off-Chain Message Signing][]| 0x80       | approved   |

### Workflow

In order to reserve a new byte the completion of the following workflow is REQUIRED:

- A reference specification MUST be defined e.g. as TZIP:
  - You MUST define a name for the `message type`
  - You MUST define the specific `magic_byte`
  - It it RECOMMENDED to including a rationale for reserving a magic byte
- A Merge Request (MR) to this TZIP MUST:
  - Reserve a magic byte by modifying the `table of reserved bytes`
  - Increment the major version number of this TZIP to indicate a change
- It is RECOMMENDED to publish the request on [Tezos Agora][] to gather community feedback
- After the MR is merged, the specific magic byte is reserved

## Backwards compatibility

The `table of reserved bytes` is only allowed to be extended. Previously used magic bytes MUST NOT be reused as it breaks backwards compatibility. If a message type is not used anymore it SHALL be marked with the Status `revoked`. The range reserved by the protocol MAY have breaking changes due to protocol upgrades governed by the Tezos community. Changes SHALL be documented in this TZIP as soon as possible. Normally a version number in the TZIP is only used for drafts. As this TZIP is designed to evolve the version SHALL be used to indicate an extension of the `table of reserved bytes`.

## References

- [octez-signer magic bytes][] - The list of currently maintained magic bytes by Octez.
- [beacon-sdk][] - A software develop kit to communicate between wallets and dapps.
- [Smart Rollups][] - Documentation to Smart Rollups in Octez.
- [Zk Rollups][] - Documentation to Zero Knowledge Rollups in Octez.
- [Off-Chain Message Signing][] - Specification to secure off-chain message signing TZIP.
- [Sign In With Tezos (SIWT)][] - A Tezos reference implementation of SIWx [CAIP-122][].

[RFC 2119]: https://www.ietf.org/rfc/rfc2119.txt
[octez-signer magic bytes]: https://tezos.gitlab.io/user/key-management.html?highlight=magic%20bytes#signer-requests
[beacon-sdk]: https://docs.walletbeacon.io/guides/sign-payload
[TZIP-10]: https://gitlab.com/tezos/tzip/-/blob/06024a22384139b328a63747cb7951c81e5b9cd7/proposals/tzip-10/tzip-10.md
[AirGap Wallet]: https://support.airgap.it/
[Smart Rollups]: https://tezos.gitlab.io/protocols/alpha.html#smart-rollups
[Zk Rollups]: https://tezos.gitlab.io/protocols/alpha.html#zero-knowledge-rollups-ongoing
[Off-Chain Message Signing]: URI
[CAIP-122]: https://chainagnostic.org/CAIPs/caip-122
[Sign In With Tezos (SIWT)]: https://siwt.xyz/
[URI]: https://datatracker.ietf.org/doc/html/rfc3986
[Tezos Agora]: https://forum.tezosagora.org/

## Copyright

This document is licensed under [MIT](https://spdx.org/licenses/MIT.html).
