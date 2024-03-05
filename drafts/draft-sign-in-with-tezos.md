---
tzip: tbd
title: Sign-In with Tezos
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Klas Harrysson <klas@kukai.app>
type: I
created: 2024-03-25
date: To be determined
requires: ["draft-offchain-message-signing", "TEZOS-CAIP-122"]
---


## Abstract

*For context, see the [CAIP-122][] and the Chain Agnostic Namespace [CANs][] specifications for Tezos.*

Sign-In with Tezos describes how Tezos accounts authenticate with off-chain services by signing a standard message format parameterized by scope, session details, and security mechanisms (e.g., a nonce). The goals of this specification are to provide a self-custodied alternative to centralized identity providers, improve interoperability across off-chain services for Tezos-based authentication, and provide wallet vendors a consistent machine-readable message format to achieve improved user experiences and consent management. **Adopted from [EIP-4361][] Sign-In with Ethereum**

## Specification

### General

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

### Workflow

Sign-In with Tezos (SIWT) works as follows:

1. The relying party generates a SIWT Message and encodes the SIWT Message as a Tezos off-chain message as defined in TZIP [offchain-message-signing][].
2. The wallet presents the user with a structured plaintext message or equivalent interface for signing the SIWT Message with the [offchain-message-signing][] data format.
3. The signature is then presented to the relying party, which checks the signature’s validity and SIWT Message content.
4. The relying party might further fetch data associated with the Tezos account ID [CAIP-10][], such as from the Tezos blockchain (e.g. account balances, public key revelation, FA2 asset ownership like [Tezos Domains][]), or other data sources that might or might not be permissioned.

### CAIP-122

This TZIP is based on the Chain Agnostic Improvement Proposal [CAIP-122][] and the Chain Agnostic Namespace [CANs][] specifications for `tezos` which provide:

1. a signing algorithm, or a finite set of these, where multiple different signing interfaces might be used,
2. a `type` string(s) that designates each signing algorithm, for inclusion in the `signatureMeta.t` value of each signed response
3. a procedure for creating a signing input from the data model specified in this document for each signing algorithm

The signing algorithm covers:

1. how to sign the signing input,
2. how to verify the signature.

### Message format

#### CAIP-122 data model

The data model from CAIP-122 proposes the following string representation of its data model:

```text
${domain} wants you to sign in with your ${namespace(account-id)} account:
${account_address(account-id)}
${statement}
URI: ${uri}
Version: ${version}
Nonce: ${nonce}
Issued At: ${issued-at}
Expiration Time: ${expiration-time}
Not Before: ${not-before}
Request ID: ${request-id}
Chain ID: ${chain_id(account-id)}
Resources:
- ${resources[0]}
- ${resources[1]}
...
- ${resources[n]}
```

For the specification of the data model and its attributes please review the `tezos` namespace [CAIP-122][].

#### ABNF message format

In order to have a machine readable format we translate the data model into a Augmented Backus–Naur Form (ABNF, [RFC 5234][]) expression (note that %s denotes case sensitivity for a string term, as per [RFC 7405][]):

```abnf
sign-in-with-tezos =
    domain %s" wants you to sign in with your " namespace %s" account:" LF
    account-address LF
    LF
    [ statement LF ]
    LF
    %s"URI: " uri LF
    %s"Version: " version LF
    %s"Chain ID: " chain-id LF
    %s"Nonce: " nonce LF
    %s"Issued At: " issued-at
    [ LF %s"Expiration Time: " expiration-time ]
    [ LF %s"Not Before: " not-before ]
    [ LF %s"Request ID: " request-id ]
    [ LF %s"Resources:"
    resources ]

domain = authority
    ; From RFC 3986:
    ;     authority     = [ userinfo "@" ] host [ ":" port ]
    ; See RFC 3986 for the fully contextualized
    ; definition of "authority".

namespace = "tezos"
    ; See README in CANs for Tezos

account-address = "tz" 34*34HEXDIG
    ; Must also conform to capitalization
    ; See CAIP-10 for valid 
    ; where applicable (EOAs).

statement = *( reserved / unreserved / " " )
    ; See RFC 3986 for the definition
    ; of "reserved" and "unreserved".
    ; The purpose is to exclude LF (line break).

uri = URI
    ; See RFC 3986 for the definition of "URI".

version = "1"

chain-id = 15*( ALPHA / DIGIT )
    ; See CAIP-2 for valid CHAIN_IDs.

nonce = 8*( ALPHA / DIGIT )
    ; See RFC 5234 for the definition
    ; of "ALPHA" and "DIGIT".

issued-at = date-time
expiration-time = date-time
not-before = date-time
    ; See RFC 3339 (ISO 8601) for the
    ; definition of "date-time".

request-id = *pchar
    ; See RFC 3986 for the definition of "pchar".

resources = *( LF resource )

resource = "- " URI
```

#### Message fields

This specification defines the following SIWT Message fields that can be parsed from a SIWT Message by following the rules in [ABNF Message Format](#abnf-message-format):

- `domain` REQUIRED. The domain that is requesting the signing. Its value MUST be an RFC 3986 authority. The authority includes an OPTIONAL port. If the port is not specified, the default port is assumed (e.g., 443 for HTTPS).
- `account-address` REQUIRED. The Tezos account address from the account ID performing the signing according to [CAIP-10][].
- `statement` OPTIONAL. A human-readable ASCII assertion that the user will sign which MUST NOT include `'\n'` (the byte `0x0a`).
- `uri` REQUIRED. An RFC 3986 URI referring to the resource that is the subject of the signing (as in the _subject of a claim_).
- `version` REQUIRED. The current version of the SIWT Message, which MUST be `1` for this specification.
- `chain-id` REQUIRED. The [CAIP-2][] chain ID to which the session is bound, and the network where Contract Accounts MUST be resolved. The wallet provider MAY display the chain ID alias instead.
- `nonce` REQUIRED. A random string typically chosen by the relying party and used to prevent replay attacks, at least 8 alphanumeric characters.
- `issued-at` REQUIRED. The time when the message was generated, typically the current time. Its value MUST be an [ISO 8601][] datetime string.
- `expiration-time` OPTIONAL. The time when the signed authentication message is no longer valid. Its value MUST be an [ISO 8601][] datetime string.
- `not-before` OPTIONAL. The time when the signed authentication message will become valid. Its value MUST be an [ISO 8601][] datetime string.
- `request-id` OPTIONAL. A system-specific identifier that MAY be used to uniquely refer to the sign-in request.
- `resources` OPTIONAL. A list of information or references to information the user wishes to have resolved as part of authentication by the relying party. Every resource MUST be an RFC 3986 URI separated by `"\n- "` where `\n` is the byte `0x0a`.

## Backwards compatibility

The data model of the message MUST be derived from the `interface` referencing to the SIWT specification in the [offchain-message-signing][] TZIP. Different versions MAY have breaking changes.

## Reference implementation

A reference implementation can be found in the [SIWT library][].

## Validation

- The wallet MUST enforce strict validation to ensure there is no deviation in any way from the expected data format.
- The wallet implementers MUST display the public key hash associated to the private key used for signing the message.
- Wallet implementers displaying the message as plaintext to the user SHOULD require the user to scroll to the bottom of the text area prior to signing.

## Test vectors

### Encoding & signing

```text
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

- [CANs][] - Chain Agnostic Namespaces.
- [CAIP-10][] - Account ID Specification.
- [CAIP-122][] - Sign in With X (SIWx).
- [EIP-4361][] - Sign-In with Ethereum.
- [SIWT library][] - A reference implementation adhering to the CAIP specifications.

[EIP-4361]: https://eips.ethereum.org/EIPS/eip-4361
[RFC 2119]: https://www.ietf.org/rfc/rfc2119.txt
[RFC 5234]: https://www.rfc-editor.org/rfc/rfc5234
[RFC 7405]: https://www.rfc-editor.org/rfc/rfc7405
[CAIP-2]: https://chainagnostic.org/CAIPs/caip-2
[CAIP-10]: https://chainagnostic.org/CAIPs/caip-10
[CAIP-122]: https://chainagnostic.org/CAIPs/caip-122
[CANs]: https://github.com/ChainAgnostic/namespaces
[ISO 8601]: https://www.iso.org/iso-8601-date-and-time-format.html
[Tezos Domains]: https://tezos.domains/
[SIWT library]: https://siwt.xyz/
[offchain-message-signing]: tbd

## Copyright

This document is licensed under [MIT](https://spdx.org/licenses/MIT.html).
