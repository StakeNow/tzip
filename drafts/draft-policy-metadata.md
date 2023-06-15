---
tzip: 0xx
title: Policy Tokenization
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Roy Scheeren <roy.scheeren@vdl.digital>, Pierre Mai <pmai@pmsfit.de>
type: Interface
created: 2023-06-13
date: tbd
version: 0.1
---

## Abstract

This proposal defines the tokenization of policies written in the Open Digital Rights Language (ODRL) as a Tezos FA2 compliant smart contract to express usage rights and obligations of resources in a trustless environment.

It is designed as an extension of the following Tezos Improvement Proposals (TZIPs):

1) Defining the usage and constraints on the interfaces of the [TZIP-012][8] FA2 smart contract.
2) An extension of [TZIP-016][1] describing the metadata schema and standards for tokenized [ODRL][2] compliant policies.

In addition we define a process that ensures the minting of a correct ODRL policy token.

## Table of Contents

- [General](#General)
- [Motivation & Goals](#Motivation-amp-Goals)
- [ODRL Information Model](#ODRL-Information-Model)
- [Tokenization of Policies](#Tokenization-of-Policies)
- [Specification](#Specification)
  - [Entrypoint Semantics](#entrypoint-semantics)
    - [Overview](#Overview)
    - []
  - [Metadata](#Metadata)
    - [Token Metadata](#Token-Metadata)
    - [Contract Metadata (TZIP-016)](#Contract-Metadata-TZIP-016)
  - [FA2.1](#FA21)
- [Use Cases](#Use-Cases)
    - [Digital Art Ownership & Licensing](#Digital-Art-Ownership-amp-Licensing)
    - [Media Content Distribution](#Media-Content-Distribution)
    - [Academic Research Licensing](#Academic-Research-Licensing)
    - [Traditional Organizations and DAOs](#Traditional-Organizations-and-DAOs)
    - [Game Assets Ownership and Trading](#Game-Assets-Ownership-and-Trading)
- [Future Directions](#Future-Directions)
- [Copyright](#copyright)

## General

Many of the terms in this standard are derived from [The Dublin Core, RCF 2413][3].

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”,
“SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be
interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Motivation & Goals

While significant strides have been made towards standardizing the format of Non-Fungible Tokens (NFTs), there's been a conspicuous lack of progress in formalizing the true essence of what a user is purchasing. So far, most rights pertaining to NFTs have been implicit, taken for granted rather than explicitly stated.

> Property rights exist only for (physical) objects and thus not for digital works. Only rights of use can be considered, but these must be granted individually.
> 
> The most similar to ownership are "exclusive as well as temporally, spatially and content-wise unrestricted, transferable, sublicensable and rentable rights of use″. These could be supplemented by editing rights or various types of use, such as the right to print on paper.
> 
> *translated from [CMS Deutschland Blog][14]*

Generally speaking, any form of digital art or data can be tokenized, for instance, as an NFT, effectively serving as a digital counterpart or "twin". However, the digital rights associated with these assets must be independently defined. Given the sheer breadth of use cases for these digital assets, there are bound to be circumstances where a transparent, tamper-proof policy, complete with its history, will be advantageous.

This proposal seeks to establish a robust framework for defining rights and obligations. As a potential application, it aims to delineate the rights of both providers and consumers when interacting with resources such as, for example, a web3 application.

## ODRL Information Model

ODRL is a W3C standard recommendation to describe policies that define rights and obligations and is widely used in the industry as e.g. in the [Mobility Data Space][15].

> The Open Digital Rights Language (ODRL) is a policy expression language that provides a flexible and interoperable information model, vocabulary, and encoding mechanisms for representing statements about the usage of content and services. The ODRL Information Model describes the underlying concepts, entities, and relationships that form the foundational basis for the semantics of the ODRL policies.
>
> Policies are used to represent permitted and prohibited actions over a certain asset, as well as the obligations required to be meet by stakeholders. In addition, policies may be limited by constraints (e.g., temporal or spatial constraints) and duties (e.g. payments) may be imposed on permissions.
>
> from [ODRL Information Model 2.2][2]

ODRL implementations are e.g. listed [here][4] and the best practices are collected [here][5] which are recommended to be followed in this specification as well.

## Tokenization of Policies

Tokens provide an immutable record of the permissions, prohibitions, and obligations dictated by the tokenized ODRL policy. This record can be referenced by the application to enforce access controls.

* **Role of Tokens:** Each token can be seen as a 'license' carrying the access policy (ODRL policy) for a particular resource, codified within it as a smart contract. The access policy specifies the permissions, obligations, and prohibitions associated with that digital asset.

* **Enforcing Access:** The actual enforcement of access control takes place in the application layer, not at the token level. When a user tries to interact with a resource through the application, the application refers to the token's policy (on-chain data) to ascertain the user's permissions regarding that resource.

* **Decentralized Documentation:** Each token serves as a decentralized, immutable proof of the policy. If a dispute arises, the token's smart contract can provide irrefutable evidence of the conditions set forth at the time of the user's interaction with the resource.

* **Claim State:** The immutability and transparency of the blockchain provide a solid ground for users to state a claim if they feel their access rights, as detailed in the token, have been violated.

## Specification

### Overview

The following actions MUST be implemented:

```
type action = 
 | ["Mint", nft_uri]
 | ["Update", update_policy_params]
 | ["Set_adminstrator", address]
 | ["Set_paused", bool]
 | ["Update_pricing_model", nat, tez]
 | ["Delete_pricing_model", nat]
 | ["Payout", tez]
 | ["Update_operators", FA2_UpdateOperators.Types.update_operators]
 | ["Balance_of", FA2_BalanceOf.Types.balance_of]
 | ["Transfer", FA2_Transfer.Types.transfer];
```

### Prerequisites and recommendations

#### Authentication
Implementation of the application MAY use an authentication service like [SIWT][9] to verify the user has access to the private key belonging to the address he is using and the verifier service can validate the ownership of the respective NFT asset. OPTIONALLY, a signed authentication message can be requested from the user on `Mint` and `Update` steps which REQUIRE OPTIONAL parameters.

#### Message signing
It is REQUIRED to encapsule the `mint_message` and the `update_message` as a [`failing_noop (tag 17)`][24] in order to avoid the use of the signed message for on-chain transactions. This depends widely on the support in libraries and wallets as e.g. requested [here][25]. OPTIONALLY use the `magic_byte: <0x04>` referring to an authenticated signing request according to [tzip-26 proposal][23]:

##### Failing_noop:

```
{
  branch: "BLockGenesisGenesisGenesisGenesisGenesisf79b5d1CoW2"
  contents: [{
    kind: 'failing_noop',
    arbitrary: 'Hello'
   }]
}
>> 0x038fcf233671b6a04fcf679d2a381c2544ea6c1ea29ba6157776ed8424c7ccd00b11000032346d696e745f6d657373616765
```

Defined as:
- `"magic_bytes": <0x03>`: Magic bytes (aka watermark) refering to a `Transfer` operation.
- `"branch": <8fcf233671b6a04fcf679d2a381c2544ea6c1ea29ba6157776ed8424c7ccd00b>`: Contains the branch (genesis block).
- `"tag": <11>`: Tag 17 referring to a `failing_noop` operation.
- `"length": <00003234>`: Length of the following message in `bytes`.
- `"message": <6d696e745f6d657373616765>`: The message in `bytes` in this example the string `mint_message`.

##### Authenticated signing request:

```
message = magic_bytes + mint_message
>> 
```

Defined as:
- `"magic_bytes": <0x04>`: Magic bytes (aka watermark) refering to an `Authenticated signing request`.
- `"message": <6d696e745f6d657373616765>`: The message in `bytes` in this example the string `mint_message`.


### Step 1: `Mint`

```
type action = ["Mint", mint_message]

mint_message = [
    timestamp,
    policy_contract_address,
    asset_address,
    asset_id
    ]
```

Defined as:
- `"timestamp"`: Contains the time this message was created of type `timestamp `.
- `"policy_contract_address"`: Target policy contract `ktXXXXXX` of type `address`.
- `"asset_address"`: Associated NFT contract `ktXXXXXX` used as constraint of type `address`.
- `"asset_id"`: Token id of the associated NFT used as constraint of type `nat`.

The owner of the asset NFT named as assignee in the ODRL policy is REQUIRED to reserve a policy token in the contract which MUST be used as the `uid` in side the policy.

The policy_token_id is returned to the user and can be used as UID of the ODRL policy.

A number of checks has to be conducted:
- The mint can be given a moderate price to discourage the reservation of multiple empty policy tokens.
- Check via a inter contract call if the caller of the mint action is the owner of the associated NFT.
- Create an array of allowed associated NFT asset contract addresses.

The policy token contract MUST contain a reverse_lookup table `%asset_map_table` of type `(big_map (pair (address %asset_address, nat %token_id)) (nat %policy_token_id))`

Advantages:
- Linking between entry points done
- Replay attack mitigated by checking if big map item already exists
- Easy view to check if token has a policy
- Allows SIWT to combine token and policy requirements
- Multiple keys can point to one policy_token_id

Disadvantages:
- Extra ledger = added complexity
- One asset can only have one policy

### Step 2: Create & verify the ODRL policy

The user will create a ODRL policy e.g. guided through a user interface of a front end. The policy MUST contain the following entries:
- `"uid": <policy_uid>`: The policy contract address and token id of the policy FA2 contract `policy_contract_address:policy_token_id`.
- `"assignee": <assignee_uid>`: The NFT contract address and token id of the target asset giving the constraint `asset_address:asset_id`.

The call to the verifier service MUST contain the following parameters:
- `"policy"`: The ODRL [json-ld][16] policy.
- `"caller_uid"`: The Tezos address of the owner of the asset_id and caller of the policy contract.
- `"signature"`: A signature of [type][20].

The verifier MUST at least check the following information:
1) Read the NFT contract address and token id from the policy token meta data and check if the caller of the update action is the owner of the respective NFT.

The verifier MUST return the following parameter:
- `"update_message"`: The message according to this specification.

```
update_message = [
    timestamp
    policy_address
    asset_address
    asset_id_length
    asset_id
    pricing_table_id
    policy
    ]
```

Defined as:
- `"timestamp"`: Contains the time this message was created of type `timestamp `.
- `"policy_address"`: Target policy contract `ktXXXXXX` of type `address`.
- `"asset_address"`: Respective NFT asset contract `ktXXXXXX` of type `address`.
- `"asset_id_length"`: Length of the following `asset_id` in `bytes`.
- `"asset_id"`: Token id of the associated NFT used as constraint of type `nat`.
- `"pricing_table_id"`: id of the pricing table in the policy contract of type `nat`.
- `"policy"`: The ODRL [json-ld][16] policy.

This is an example for a valid (refer to verifier) ODRL file that allows the usage of https://stakenow.fi under the condition to be the owner of the StakeNow Beta User NFT:

```
{
   "@context": "http://www.w3.org/ns/odrl.jsonld",
   "@type": "Policy",
   "uid": "policy_contract:policy_token_id",
   "permission": [{
       "target": {
           "@type": "AssetCollection",
           "uid":  "http://stakenow.fi" },
       "action": "use",
       "assigner": "http://vdl.digital",
       "assignee": "asset_contract:asset_token_id",
   }]
}
```

Considerations:
- Be aware of replay attacks as described [here][19].

### Step 3: `Update`

```
type action = ["Update", policy_params]

policy_params = [update_message, signature]
```

A number of checks has to be conducted:

1) Check the signature of the `update_message`.
2) Check if the time stamp is in the valid range.
3) Comparing the contract address with the `policy_contract` address inside the `update_message`.
4) Check if the `policy_token_id` exist inside the `policy_contract` contract. The token has been reserved using the mint action before.

As a precondition for updating a policy token the ODRL json-ld policy MUST be signed by the `verifier` and checked using e.g. the Ligo [`check_signature`][17]

```
let check_signature =
  (pk: key, signed: signature, msg: bytes) =>
  Crypto.check(pk, signed, msg);
```

### Admin Interface
The policy is owned and managed by the administrator of the application. The administrator SHOULD be allowed to update/alter/revoke policies connected to a token as per their discretion. Create a list of allowed associated NFT contracts.

### Metadata

#### Token Metadata

Token-specific metadata is stored/presented as a Michelson value of type `(map string bytes)` with bytes as UTF-8 stream of `bytes` without NULL termination. A few of the keys are reserved and predefined by TZIP-012 and in the case of a TZIP-016 URI pointing to a JSON blob, the JSON preserves the same 3 reserved non-empty fields:

`{"name": <string>, "symbol": <string>, "decimals": <number>, ... }`

Providing a value for `"decimals"` is required for all token types. `"name”` and `"symbol"` are not required but it is highly recommended for most tokens to provide the values either in the map or the JSON found via the TZIP-016 URI.

A valid ODRL policy token MUST contain the following token meta data values:
- `"policy"`: The ODRL [json-ld][16] policy.
- `"signature"`: A signature of [type][20] serialized with [`PACK`][22].

**Additional metadata properties MAY be added to the policy token meta data** to support further authenticity, and integrity purposes from external vocabularies. The ODRL Information Model recommends the use of Dublin Core Metadata Terms [dcterms][21] for ODRL policies.

**The following Dublin Core Metadata Terms [dcterms][21] properties SHOULD be used:**

- none, one, or many `dc:creator` property values - the individual, agent, or organisation that authored the Policy.
- none, one, or many `dc:description` property values - a human-readable representation or summary of the Policy.
- none or one `dc:issued` property values - the date (and time) the Policy was first issued.
- none or one `dc:modified` property values - the date (and time) the Policy was updated.
- none, one, or many `dc:coverage` property values - the jurisdiction under which the Policy is relevant.
- none or one `dc:replaces` property values (of type Policy) - the identifier of a Policy that this Policy supersedes.
- none or one `dc:isReplacedBy` property values (of type Policy) - the identifier of a Policy that supersedes this Policy.

**The ODRL validation requirements for policies with the above metadata properties include:**

- If a policy has the `dc:isReplacedBy` property, then a processor MUST consider the first policy void and MUST retrieve and process the identified policy.


#### Contract Metadata (TZIP-016)

### FA2.1
[FA2.1][11] specification provides a usefull extension to the concept by introducing the export of [Tickets][12] in order to maintain the policy directly in a user wallet. In addition the mandatory use of [Events][13] e.g. for updating the token metadata ensures consistent logging of changes to policies made by the contract administrator or any permitted operator. The usage of a FA2.1 compliant contract is RECOMMENDED.

## Use Cases
Those are example use cases in order to understand the mechanism.

### Digital Art Ownership & Licensing

**Problem/Need:** The provenance and rights associated with digital art can often be ambiguous and disputable.

**Target Audience:** Digital artists, art collectors, art platforms, and enthusiasts.

**Benefits/Outcomes:** The integration of tokenized ODRL policies can offer an immutable, transparent record of ownership and usage rights. Each digital artwork can be tokenized as an NFT with its associated rights policy, which could include permissions for displaying publicly, creating derivative works, reselling, etc. Artists have full control over their work, and collectors are aware of what they're actually purchasing.

### Media Content Distribution

**Problem/Need:** Fair compensation for media content creators is often a challenge due to piracy and unauthorized distribution.

**Target Audience:** Content creators, media platforms, consumers.

**Benefits/Outcomes:** By tokenizing each piece of content (music, video, podcast, etc.) with its ODRL policy, content creators can define the rules of consumption and sharing. Users can gain access based on the rights they purchased, providing a transparent, fair compensation system for creators.

### Academic Research Licensing

**Problem/Need:** Accessing and licensing academic research often involves complex procedures and opaque agreements.

**Target Audience:** Researchers, institutions, publishers, readers.

**Benefits/Outcomes:** Academic papers can be tokenized, and their distribution rights can be managed through the blockchain. This tokenization can simplify licensing, bring transparency to the process, and ensure that institutions or individual researchers are fairly compensated for their work.

### Traditional Organizations and DAOs

**Problem/Need:** Traditional organizations and DAOs require a transparent, immutable method to document their policies and enforce access control to resources.

**Target Audience:** members, DAO members, blockchain developers.

**Benefits/Outcomes:** Organizations and DAOs can use tokenized ODRL policies to manage their resources. For example, voting rights can be tokenized, and each token's policy defines member's permissions. In addtion NFTs can be used to attest the membership status of individuals and entities.

### Game Assets Ownership and Trading

**Problem/Need:** Ownership and trading of in-game assets are often restricted to the game's platform, limiting players' rights.

**Target Audience:** Game developers, players.

**Benefits/Outcomes:** In-game assets can be tokenized and associated with an ODRL policy defining the rights of the asset's owner. This setup facilitates a player-driven economy, enabling players to trade, sell, or rent their game assets securely.

## Implementations
A reference implementation of a policy contract using the described meta data schema in combination with a [FA2][8] contract is implemented in the library [Sign in with Tezos][9].

## Future Directions

- Update `verifier` public key:
  - Extending the entry points with a method to update the verifier key. It has to be made sure that the validation of a policy using the signature and the raw bytes can still be performed with a changed key or e.g. a list of valid keys.
    ```
    type action = ["Set_verifier", key]
    ```
  - It is RECOMMENDED to implement an Event to document changes of the verifier key.
  - Set a status for each verifier key.
- Create a mechanism to allow for multiple policy tokens for a asset NFT.
- Define views
  - to access a policy and its verification status.
  - to access the (list) of verifier and their status.
- Add `"policy_uri"` to policy token meta data: A [TZIP-16][1] meta data URI to the policy represented as deserialized json-ld.
- Think about a concept how to 

## Copyright
This document is licensed under [MIT][10].

[1]: https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-16/tzip-16.md
[2]: https://www.w3.org/TR/odrl-model/
[3]: https://www.ietf.org/rfc/rfc2413.txt
[4]: https://www.w3.org/community/odrl/implementations/
[5]: https://w3c.github.io/odrl/bp/
[8]: https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-12/tzip-12.md
[9]: https://github.com/StakeNow/SIWT
[10]: https://spdx.org/licenses/MIT.html
[11]: https://gitlab.com/tezos/tzip/-/blob/master/drafts/current/draft-fa2-update.md
[12]: https://www.marigold.dev/post/tickets-for-dummies
[13]: https://www.marigold.dev/post/how-to-send-events-from-contract-fast
[14]: https://www.cmshs-bloggt.de/tmc/rechtliche-herausforderungen-sog-non-fungible-token-nfts/
[15]: https://www.mobility-data-space.de/en.html
[16]: https://json-ld.org/
[17]: https://ligolang.org/docs/language-basics/tezos-specific/?lang=jsligo#checking-signatures
[18]: https://json-schema.org/understanding-json-schema/reference/string.html#resource-identifiers
[19]: https://tezos.gitlab.io/developer/michelson_anti_patterns.html?highlight=signature#signatures-alone-do-not-prevent-replay-attacks
[20]: https://ligolang.org/docs/language-basics/tezos-specific?lang=jsligo#checking-signatures
[21]: https://www.w3.org/TR/odrl-model/#bib-dcterms
[22]: https://ligolang.org/docs/language-basics/tezos-specific?lang=jsligo#pack-and-unpack
[23]: https://gitlab.com/tezos/tzip/-/merge_requests/195
[24]: http://doc.tzalpha.net/shell/p2p_api.html#failing-noop-tag-17
[25]: https://github.com/ecadlabs/taquito/issues/2460