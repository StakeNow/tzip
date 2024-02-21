---
tzip: 27
title: Signer Requests
status: Draft
author: contact@marigold.dev, Carlo van Driesten <carlo.van-driesten@vdl.digital>
type: LA
created: 2022-09-20
date: 2024-02-21
requires: TZIP-10
version: 1
---


## Summary

A reference of Magic Bytes used by signer applications and the workflow to reserve a new byte.

## Abstract

Signer applications and wallet use a **magic byte** prefix to distinguish the type of data to be signed.

This document serves as:

- a reference for used bytes, and
- a workflow definition to reserve bytes.

## Motivation

Currently Magic Bytes definition is maintain by Octez for `tezos-signer`.

**INSERT REFERENCE TO THE DOCUMENTATION OF MAGIC BYTES IN OCTEZ**

Wallets based on [beacon-sdk][] implementation of [TZIP-10][] are using `0x03` and `0x05` in order to type payloads. Most wallet only accept to sign those 2 kinds of payloads. Currently only Airgap Wallet accept to sign raw data.

Incoming Layer 2 applications like [SCORUs][] or [Zk-rollups][] need to be able to reserve a byte in order to integrate the tooling ecosystem. In addition certain applications like the signing of a secure off-chain message TZIP-28 need a magic byte to signal its purpose to a wallet application.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

Remote signer applications MUST handle signing requests with the following format:

```xml
<magic_byte><data>
```

### Ranges

- Bytes between 0x00 and 0x7F SHALL be reserved for the Tezos protocol
- Bytes between 0x80 and 0xFF MAY be used by applications

### Table of reserved bytes

| Message type                  | Reference   | Magic byte |
|-------------------------------|-------------|------------|
| Legacy block                  | URI         | 0x01       |
| Legacy endorsement            | URI         | 0x02       |
| Transfer                      | URI         | 0x03       |
| Authenticated signing request | URI         | 0x04       |
| Michelson data                | URI         | 0x05       |
| Block                         | URI         | 0x11       |
| Pre-endorsement               | URI         | 0x12       |
| Endorsement                   | URI         | 0x13       |
|-------------------------------|-------------|------------|
| Off-Chain Message Signing     | URI#TZIP-28 | 0x80       |

### Workflow

In order to reserve a new byte the completion of the following workflow is REQUIRED:

- A reference specification/documentation for the `message type` MUST be defined
- It it RECOMMENDED to including a rationale why this specific magic byte is needed
- A Merge Request to this TZIP MUST:
  - Reserve a magic byte by modifying the `table of reserved bytes`
  - Add the specification URI including a URI fragment
  - Increment the major version number of this TZIP
- It is RECOMMENDED to publish the request to gather community feedback
- When the MR is merged, the byte is reserved

## Backwards compatibility

The `table of reserved bytes` is only allowed to be extended and previously used magic bytes MUST NOT be reused as it breaks backwards compatibility.

## References
- [beacon-sdk][] - A software develop kit to communicate between wallets and dapps.
- [SCORUs][] - Start building with SCORUs on Mondaynet.
- [Zk-rollups][] - Zero Knowledge Rollups.

[beacon-sdk]: https://docs.walletbeacon.io/guides/sign-payload
[TZIP-10]: https://gitlab.com/tezos/tzip/-/blob/06024a22384139b328a63747cb7951c81e5b9cd7/proposals/tzip-10/tzip-10.md
[SCORUs]: https://research-development.nomadic-labs.com/from-jakarta-to-kathmandu-non-stop.html#start-building-with-scorus-on-mondaynet
[Zk-rollups]: https://tezos.gitlab.io/protocols/alpha.html#zero-knowledge-rollups-ongoing
[RFC 2119]: https://www.ietf.org/rfc/rfc2119.txt

## Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
