---
tzip: tbd
title: Sign-In with Tezos
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Klas Harrysson <klas@kukai.app>
type: I
created: 2024-03-05
date: 2024-03-13
requires: ["TZIP-32", "TEZOS-CAIP-122"]
---


## Abstract

*For context, see the [CAIP-122][] and the Chain Agnostic Namespace [CANs][] specifications for Tezos.*

Sign-In with Tezos describes how Tezos accounts authenticate with off-chain services by signing a standard message format parameterized by scope, session details, and security mechanisms (e.g., a nonce). The goals of this specification are to provide a self-custodied alternative to centralized identity providers, improve interoperability across off-chain services for Tezos-based authentication, and provide wallet vendors a consistent machine-readable message format to achieve improved user experiences and consent management. **Adopted from [EIP-4361][] Sign-In with Ethereum**

## Specification

### General

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][].

**Printable ASCII:** ASCII characters between 32 and 126 (inclusive).

### Workflow

Sign-In with Tezos (SIWT) works as follows:

1. The relying party SHALL generate a SIWT Message. It is RECOMMENDED to encode the SIWT Message as a Tezos off-chain message as defined in TZIP [offchain-message-signing][]. You MAY use the [taquito][] convention of signing messages with beacon.
2. The user SHALL present the public key to the relying party. It is RECOMMENDED to use a [TZIP-10][] compliant handshake procedure or OPTIONALLY to use Tezos on-chain data to retrieve the public key corresponding to the public key hash.
3. The wallet SHALL present the user with a structured plaintext message or OPTIONALLY an equivalent interface for signing the encoded SIWT Message.
4. The signature is presented to the relying party, which MUST check the signature’s validity and SIWT Message content.
5. The relying party MAY further fetch data associated with the Tezos account ID [CAIP-10][], such as from the Tezos blockchain (e.g. account balances, public key revelation, FA2 asset ownership like [Tezos Domains][]), or other data sources that might or might not be permissioned.

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
    %s"Uri: " URI LF
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

namespace = "tezos" / "Tezos"
  ; See README in CANs for Tezos

account-address = "tz" 34*34ALPHADIGIT
  ; Must also conform to capitalization
  ; See CAIP-10 for valid 
  ; where applicable (EOAs).

statement = *( reserved / unreserved / " " )
  ; See RFC 3986 for the definition
  ; of "reserved" and "unreserved".
  ; The purpose is to exclude LF (line break).

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

; ------------------------------------------------------------------------------
; RFC 3986

URI           = scheme ":" hier-part [ "?" query ] [ "#" fragment ]

hier-part     = "//" authority path-abempty
              / path-absolute
              / path-rootless
              / path-empty

scheme        = ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )

authority     = [ userinfo "@" ] host [ ":" port ]
userinfo      = *( unreserved / pct-encoded / sub-delims / ":" )
host          = IP-literal / IPv4address / reg-name
port          = *DIGIT

IP-literal    = "[" ( IPv6address / IPvFuture  ) "]"

IPvFuture     = "v" 1*HEXDIG "." 1*( unreserved / sub-delims / ":" )

IPv6address   =                            6( h16 ":" ) ls32
              /                       "::" 5( h16 ":" ) ls32
              / [               h16 ] "::" 4( h16 ":" ) ls32
              / [ *1( h16 ":" ) h16 ] "::" 3( h16 ":" ) ls32
              / [ *2( h16 ":" ) h16 ] "::" 2( h16 ":" ) ls32
              / [ *3( h16 ":" ) h16 ] "::"    h16 ":"   ls32
              / [ *4( h16 ":" ) h16 ] "::"              ls32
              / [ *5( h16 ":" ) h16 ] "::"              h16
              / [ *6( h16 ":" ) h16 ] "::"

h16           = 1*4HEXDIG
ls32          = ( h16 ":" h16 ) / IPv4address
IPv4address   = dec-octet "." dec-octet "." dec-octet "." dec-octet
dec-octet     = DIGIT                 ; 0-9
                 / %x31-39 DIGIT         ; 10-99
                 / "1" 2DIGIT            ; 100-199
                 / "2" %x30-34 DIGIT     ; 200-249
                 / "25" %x30-35          ; 250-255

reg-name      = *( unreserved / pct-encoded / sub-delims )

path-abempty  = *( "/" segment )
path-absolute = "/" [ segment-nz *( "/" segment ) ]
path-rootless = segment-nz *( "/" segment )
path-empty    = 0pchar

segment       = *pchar
segment-nz    = 1*pchar

pchar         = unreserved / pct-encoded / sub-delims / ":" / "@"

query         = *( pchar / "/" / "?" )

fragment      = *( pchar / "/" / "?" )

pct-encoded   = "%" HEXDIG HEXDIG

unreserved    = ALPHA / DIGIT / "-" / "." / "_" / "~"
reserved      = gen-delims / sub-delims
gen-delims    = ":" / "/" / "?" / "#" / "[" / "]" / "@"
sub-delims    = "!" / "$" / "&" / "'" / "(" / ")"
              / "*" / "+" / "," / ";" / "="

; ------------------------------------------------------------------------------
; RFC 3339

date-fullyear   = 4DIGIT
date-month      = 2DIGIT  ; 01-12
date-mday       = 2DIGIT  ; 01-28, 01-29, 01-30, 01-31 based on
                          ; month/year
time-hour       = 2DIGIT  ; 00-23
time-minute     = 2DIGIT  ; 00-59
time-second     = 2DIGIT  ; 00-58, 00-59, 00-60 based on leap second
                          ; rules
time-secfrac    = "." 1*DIGIT
time-numoffset  = ("+" / "-") time-hour ":" time-minute
time-offset     = "Z" / time-numoffset

partial-time    = time-hour ":" time-minute ":" time-second
                  [time-secfrac]
full-date       = date-fullyear "-" date-month "-" date-mday
full-time       = partial-time time-offset

date-time       = full-date "T" full-time

; ------------------------------------------------------------------------------
; RFC 5234

ALPHA          =  %x41-5A / %x61-7A   ; A-Z / a-z
LF             =  %x0A
                  ; linefeed
DIGIT          =  %x30-39
                  ; 0-9
ALPHADIGIT     =  ALPHA / DIGIT
HEXDIG         =  DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
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

The data model of the message MUST be derived from the SIWT specification referenced in the `interface` according to the [offchain-message-signing][] encoding. Different versions of SIWT MAY have breaking changes.

## Reference implementation

A reference implementation can be found in the [SIWT library][].

## Validation

- The wallet MUST enforce strict validation to ensure there is no deviation in any way from the expected data model defined as abnf.
- The wallet MUST check if all required fields are provided.
- The wallet implementers MUST display the public key hash associated to the private key used for signing the message.
- Wallet implementers displaying the message as plaintext to the user SHOULD require the user to scroll to the bottom of the text area prior to signing.

## Test vectors

### Valid message examples

1) Example message string including all optional fields:

```text
'SIWT wants you to sign in with your Tezos account:\ntz1TzrmTBSuiVHV2VfMnGRMYvTEPCP42oSM8\n\nadsf\n\nUri: https://siwt.xyz\nVersion: 1\nChain ID: NetXdQprcVkpaWU\nNonce: 12345678\nIssued At: 2021-08-25T12:34:56Z\nExpiration Time: 2021-08-25T12:34:56Z\nNot Before: 2021-08-25T12:34:56Z\nRequest ID: 123456789\nResources:\n- https://a.com\n- https://b.com'
```

2) Example message string with only mandatory fields:

```text
SIWT wants you to sign in with your Tezos account:\ntz1TzrmTBSuiVHV2VfMnGRMYvTEPCP42oSM8\n\n\n\nUri: https://siwt.xyz\nVersion: 1\nChain ID: NetXdQprcVkpaWU\nNonce: 12345678\nIssued At: 2021-08-25T12:34:56Z
```

### Encoding & signing

| Message substring  | Value                         | Hex value                                                  |
| -------------------|-------------------------------|------------------------------------------------------------|
| Prefix             | Hexadecimal                   | 0x                                                         |
| Magic byte         | 128                           | 80                                                         |
| Magic string       | tezos signed offchain message | 74657A6F73207369676E6564206F6666636861696E206D657373616765 |
| Interface length   | 9                             | 09                                                         |
| Interface          | tzip://33                     | 747A69703A2F2F3333                                         |
| Encoding           | ASCII                         | 00                                                         |
| Message length     | 206                           | 00ce                                                       |
| Message            | Example message string 2)     | 534957542077616E747320796F7520746F207369676E20696E207769746820796F75722054657A6F73206163636F756E743A5C6E747A31547A726D54425375695648563256664D6E47524D5976544550435034326F534D385C6E5C6E5C6E5C6E5572693A2068747470733A2F2F736977742E78797A5C6E56657273696F6E3A20315C6E436861696E2049443A204E6574586451707263566B706157555C6E4E6F6E63653A2031323334353637385C6E4973737565642041743A20323032312D30382D32355431323A33343A35365A |

```text
# Wallet
mnemonic = "all all all all all all all all all all all all"
derivation_path = m/44'/1729'/0'/0' (slip-10)
private_key = edskRgEboayXzSZHW5wK2beB4aZtfQtuc2ywwjPmSQYCg7unpVT2Sr1KUSzX9hNLJC25YcB4qZ1Wotu6EuDveWY4jkiKQr9H3k

bytes     = 0x8074657A6F73207369676E6564206F6666636861696E206D65737361676509747A69703A2F2F33330000ce534957542077616E747320796F7520746F207369676E20696E207769746820796F75722054657A6F73206163636F756E743A5C6E747A31547A726D54425375695648563256664D6E47524D5976544550435034326F534D385C6E5C6E5C6E5C6E5572693A2068747470733A2F2F736977742E78797A5C6E56657273696F6E3A20315C6E436861696E2049443A204E6574586451707263566B706157555C6E4E6F6E63653A2031323334353637385C6E4973737565642041743A20323032312D30382D32355431323A33343A35365A',
sig       = 'sigdiZ7CJsFQDqaXJ54rm4HiiarRigeFL1FStqkmiNgD86Sgj7aSQo4neF1SnhjHzwc3KU5QfdT9ToKz7yivvhn7GaqBuPL1',
prefixSig = 'edsigtoY2EFXmXLQsdiHNCtcFF216dFocctCbd8eaV7iMgZ93kWfi5LKt9qtA43QVpmBJ3ao5zvSzHmRa4nABP5GfyALghfHY16',
sbytes    = '0x8074657A6F73207369676E6564206F6666636861696E206D65737361676509747A69703A2F2F33330000ce534957542077616E747320796F7520746F207369676E20696E207769746820796F75722054657A6F73206163636F756E743A5C6E747A31547A726D54425375695648563256664D6E47524D5976544550435034326F534D385C6E5C6E5C6E5C6E5572693A2068747470733A2F2F736977742E78797A5C6E56657273696F6E3A20315C6E436861696E2049443A204E6574586451707263566B706157555C6E4E6F6E63653A2031323334353637385C6E4973737565642041743A20323032312D30382D32355431323A33343A35365A783920d411a73ad4c493ab98070a904d237b3e79fb2277a1dae82cc6e992d968c7e56c2e80865c333f940bcbb93572a86f5ff0aa3f12c8fe3a08682f5660c902'

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
[TZIP-10]: https://gitlab.com/tezos/tzip/-/blob/57c32be0e5d4bc6867cea83a12cf909894c42c41/proposals/tzip-10/tzip-10.md
[taquito]: https://tezostaquito.io/docs/signing/#generating-a-signature-with-beacon-sdk

## Copyright

This document is licensed under [MIT](https://spdx.org/licenses/MIT.html).
