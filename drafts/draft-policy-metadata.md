---
tzip: To be determined
title: Policy Tokenization
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Roy Scheeren <roy.scheeren@vdl.digital>, Pierre Mai <pmai@pmsfit.de>
type: Interface
created: 2023-06-13
date: To be determined
version: 0.1
---


## Abstract

This proposal defines the tokenization of policies - a process to represent policies as unique digital assets on the blockchain, written in the Open Digital Rights Language (ODRL). ODRL is a language used to express policies for information management and digital rights governance. The proposal extends a Tezos FA2 compliant smart contract for functionality and interoperability on the Tezos blockchain to express usage rights and obligations of resources in a trustless environment.

This proposal is designed as an extension of the following Tezos Improvement Proposals (TZIPs):

1. Defining the usage and constraints on the interfaces of the TZIP-012 FA2 smart contract. This ensures that the tokenized policy is compatible with the existing infrastructure.
2. An extension of TZIP-016 describing the metadata schema and standards for tokenized ODRL compliant policies.

In addition, a stepwise process is defined that ensures the minting of ODRL policy tokens to guarantee the authenticity and correctness of the tokenized policies.


## Table of Contents

- [General](#General)
- [Motivation & Goals](#Motivation-amp-Goals)
- [ODRL Information Model](#ODRL-Information-Model)
- [Tokenization of Policies](#Tokenization-of-Policies)
- [Specification](#Specification)
  - [Overview](#Overview)
  - [Prerequisite: Authentication](#Prerequisite-Authentication)
  - [Verifier: Create, Verify & Sign Policies](#Verifier-Create-Verify-amp-Sign-Policies)
    - [Mint Message](#Mint-Message)
    - [Update Message](#Update-Message)
    - [Verifier API](#Verifier-API)
    - [Message Signing](#Message-Signing)
      - [Failing_noop](#Failing_noop)
      - [Authenticated Signing Request](#Authenticated-Signing-Request)
  - [Step 1: Mint](#Step-1-Mint)
  - [Step 2: Update](#Step-2-Update)
  - [Administrative Interface](#Administrative-Interface)
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
- [Implementations](#Implementations)
- [Future Directions](#Future-Directions)
- [Copyright](#Copyright)

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

```jsligo
type Action =
 | {"Mint": string, mint_params : MintParams};
 | {"Update": string, update_params : UpdateParams};
 | {"Set_adminstrator": string, admin_uid : address};
 | {"Set_paused": string, is_paused : bool};
 | {"Update_pricing_model": string, price_uid : nat, price : tez};
 | {"Delete_pricing_model": string, price_uid : nat};
 | {"Payout": string, amount : tez};
 | {"Update_operators": string, update : FA2_UpdateOperators.Types.update_operators};
 | {"Balance_of": string, balance : FA2_BalanceOf.Types.balance_of};
 | {"Transfer": string, transfer : FA2_Transfer.Types.transfer};
```

### Prerequisite: Authentication

The instantiation of an application tasked with minting policies MUST employ an authentication Software Development Kit (SDK), such as Sign in with Tezos [(SIWT)][9], to ascertain that the user possesses the necessary access to the private key corresponding to the utilized address.

In order to ensure secure communication between the frontend and backend components of the application, it is RECOMMENDED that JSON Web Tokens (JWT) be adopted as the preferred method of facilitating this interaction. This recommendation arises from JWT's proven reliability in maintaining information security, offering both confidentiality and integrity during data exchanges.

However, developers SHOULD take note that this document only suggests SIWT and JWT as viable options for authentication and secure communication. The developer MUST ensure that whatever approach chosen aligns with the best practices for maintaining secure and robust communication within the application's architecture.

Developers are also encouraged to evaluate whether the chosen tools are compliant with the specific privacy and data protection standards required by their jurisdiction or the standards stipulated by their organization. Furthermore, they should consider the integration with existing systems and the potential need for future scalability when making these decisions.

Assumption:
- Requests from a user to the `Verifier` are now authenticated by the users Tezos address.
- Individual message requests do not need to be signed by the user in order to prove authenticity.

### Verifier: Create, Verify & Sign Policies

The user MAY create a ODRL policy e.g. guided through a user interface in a front end application. The `json-ld` policy MUST contain the following entries:
- `"uid": <policy_contract_address:policy_token_id>`: The address of the policy FA2 contract and unique token id. The `Mint` step allocates the `policy_token_id` and therefore REQUIRES the `policy_contract_address`. The `Update` step additionally REQUIRES the `policy_token_id` in order to contain a fully uniqure `uid`.
- `"assignee": <asset_address:asset_id>`: The NFT contract address and token id of the target asset NFT to which the policy belongs.

All further entires to construct a valid policy are specified in the [ODRL specification][2].

The user calling the verifier is known with its Tezos address `tzXXXXXX` of type `address` though the authentication procedure conducted before. The policy itself is always tied to the NFT asset and the caller of the smart contract is only entitled to update or mint a policy if the ownership status of the asset NFT is confirmed.

The caller MUST provide the following parameters:
- `"mint_message"`: The message of type `MintMessage` to be used for the `Mint` entry point of the policy FA2 contract.
- `"update_message"`: The message of type `UpdateMessage` to be used for the `Update` entry point of the policy FA2 contract.

#### Mint Message

```jsligo
type MintMessage = {
  timestamp : timestamp;
  policy_address : address;
  asset_address : address;
  asset_id : nat;
};

```

Defined as:
- `"timestamp"`: Contains the time this message was created of type `timestamp`.
- `"policy_address"`: Target policy contract `ktXXXXXX` of type `address`.
- `"asset_address"`: Associated NFT contract `ktXXXXXX` used as constraint of type `address`.
- `"asset_id"`: Token id of the associated NFT used as constraint of type `nat`.

#### Update Message

```jsligo
type UpdateMessage = {
  timestamp : timestamp;
  policy_address : address;
  asset_address : address;
  asset_id_length : nat;
  asset_id : nat;
  pricing_table_id : nat;
  policy : bytes;
};
```

Defined as:
- `"timestamp"`: Contains the time this message was created of type `timestamp`.
- `"policy_address"`: Target policy contract `ktXXXXXX` of type `address`.
- `"asset_address"`: Respective NFT asset contract `ktXXXXXX` of type `address`.
- `"asset_id_length"`: Length of the following `asset_id` as `nat` with `4 bytes`.
- `"asset_id"`: Token id of the associated NFT used as constraint of type `nat`.
- `"pricing_table_id"`: id of the pricing table in the policy contract of type `nat` with `4 bytes`.
- `"policy"`: The ODRL [json-ld][16] policy.

This is an example for an ODRL file that allows the usage of https://stakenow.fi under the condition to be the owner of the [StakeNow Beta User NFT][26]:

```
{
   "@context": "http://www.w3.org/ns/odrl.jsonld",
   "@type": "Policy",
   "uid": "policy_address:policy_id",
   "profile": "http://www.w3.org/ns/odrl/2/Agreement",
   "permission": [{
       "target": {
           "@type": "AssetCollection",
           "uid":  "http://stakenow.fi" },
       "action": {
           "@type": "@id",
           "uid": "http://www.w3.org/ns/odrl/2/use"
       },
       "assigner": {
           "@type": "@id",
           "uid": "http://vdl.digital"
       },
       "assignee": {
           "@type": "@id",
           "uid": "asset_address:asset_id"
       }
   }]
}

```

#### Verifier API

The verifier MUST provide the following functions:

```jsligo
let mint_message_verifier = (mint_message : MintMessage) : (bytes * signature) => {
  // This function should call out to an external service to perform the actual signing
  // For the purpose of this example, we are just returning the input message as bytes
  // and a placeholder signature

  let signed_mint_message = Bytes.pack(mint_message);
  let signature : signature = "sigXyz123"; // Replace with actual signature

  return (signed_mint_message, signature);
}
```

`mint_message_verifier` MUST at least check the following information:
1) Check if the caller of the update action is the owner of the asset NFT.

```jsligo
let update_message_verifier = (update_message : UpdateMessage) : (bytes * signature) => {
  // This function should call out to an external service to perform the actual signing
  // For the purpose of this example, we are just returning the input message as bytes
  // and a placeholder signature

  let signed_update_message = Bytes.pack(update_message);
  let signature : signature = "sigXyz123"; // Replace with actual signature

  return (signed_update_message, signature);
}
```

`update_message_verifier` MUST at least check the following information:
1) Check if the `policy_id` is allocated in the contract `policy_address`.
2) Check if the `asset_address` and `asset_id` is pointing to the `policy_id` within the policy contract reverse lookup registry.
3) Validate the content of the ODRL policy according to the specification given as a template.

#### Message Signing

The encapsulation of both `mint_message` and `update_message` as a [`failing_noop (tag 17)`][24] is REQUIRED. This precaution prevents the signed message from being utilized for on-chain transactions. The feasibility of this process is largely contingent upon the support provided by various libraries and wallets, as exemplified [here][25].

OPTIONALLY, developers may choose to use `magic_byte: <0x04>`, which refers to an authenticated signing request in accordance with the [TZIP-26 proposal][23].

##### Failing_noop:

```json
{
  branch: "BLockGenesisGenesisGenesisGenesisGenesisf79b5d1CoW2"
  contents: [{
    kind: 'failing_noop',
    arbitrary: 'Hello'
   }]
}
>> 0x038fcf233671b6a04fcf679d2a381c2544ea6c1ea29ba6157776ed8424c7ccd00b11000032346d696e745f6d657373616765
```

The `failing_noop` operation is defined as follows:
- `"magic_bytes": <0x03>`: Magic bytes (aka watermark) refering to a `Transfer` operation.
- `"branch": <8fcf233671b6a04fcf679d2a381c2544ea6c1ea29ba6157776ed8424c7ccd00b>`: Contains the branch (genesis block).
- `"tag": <11>`: Tag 17 referring to a `failing_noop` operation.
- `"length": <00003234>`: Length of the following message in `bytes`.
- `"message": <6d696e745f6d657373616765>`: The message in `bytes` in this example the string `mint_message`.


##### Authenticated Signing Request:

```jsligo
type magic_bytes = bytes;
type message = bytes;

type my_message = {
  magic_bytes : magic_bytes;
  message : message;
};

>> 0x046d696e745f6d657373616765
```

Defined as:
- `"magic_bytes": <0x04>`: Magic bytes (aka watermark) refering to an `Authenticated signing request`.
- `"message": <6d696e745f6d657373616765>`: The message in bytes in this example the string `mint_message`.
- 
Considerations:
- Be aware of replay attacks as described [here][19].


### Step 1: `Mint`

The minting process reserves a policy token in the contract. This token MUST be used as the `"uid"<policy_contract_address:policy_token_id>` within the ODRL policy. Only the owner of the associated NFT asset is permitted to `Mint` the policy token.

```jsligo
type MintParams = {
  mint_message : MintMessage;
  signature : bytes;
};

type Action = {
  tag : string; // This will always be "Mint" for this type
  mint_params : MintParams;
};
```

A number of checks are RECOMMENDED:
- The mint can be given a moderate price to discourage the reservation of multiple empty policy tokens.
- Create an array of allowed associated NFT asset contract addresses.
- Check if an entry for the asset NFT already exists in the `%asset_map_table`.

The policy token contract MUST contain a reverse lookup table, named `%asset_map_table`. This table should be of type `(big_map (pair (address %asset_address, nat %token_id)) (nat %policy_token_id))`.

Advantages:
- The linking between entry points is completed.
- Potential replay attacks are mitigated by checking if a big map item already exists.
- It provides a straightforward way to check if a token has a policy.
- It allows SIWT to combine token and policy requirements.
- Multiple keys can point to one `policy_id`.

Disadvantages:
- The additional ledger introduces extra complexity.
- Each asset can only be associated with a single policy.

### Step 2: `Update`

The update process replaces the policy token meta data in the contract. Only the owner of the associated NFT asset is permitted to `Update` the policy token.

```jsligo
type UpdateParams = {
  update_message : UpdateMessage;
  signature : bytes;
};

type Action = {
  tag : string; // This will always be "Update" for this type
  update_params : UpdateParams;
};
```

In order to maintain the integrity of the `Update` process, a number of checks are REQUIRED:

1) Signature Verification: The ODRL json-ld policy MUST be signed by the `verifier` prior to the update of a policy token. The signature of the `update_message` MUST be validated using the [check_signature][17] function in LIGO:
    ```jsligo
    let check_signature =
      (pk: key, signed: signature, msg: bytes) =>
      Crypto.check(pk, signed, msg);
    ```
3) Timestamp Verification: The timestamp MUST be within a valid range.
4) Contract Address Verification: The contract address MUST correspond with the `policy_address` specified within the `update_message`.
5) Policy ID Verification: The `policy_id` MUST exist within the `policy_address` contract. This implies that the token MUST have been reserved using the `Mint` action beforehand.

These checks ensure that the Update action is executed under the proper permissions and conditions.

### Administrative Interface

The policy associated with a token is under the purview of the application's administrator. It is envisioned that the administrator will have the authority to modify, revoke, or update policies linked to a token as per their discretion. Upon minting, a list of permissible associated NFT contracts is generated.

The following administrative actions are defined:

```jsligo
type AdminAction =
 | {"Set_adminstrator": string, admin_uid : address};
 | {"Set_paused": string, is_paused : bool};
 | {"Update_pricing_model": string, price_uid : nat, price : tez};
 | {"Delete_pricing_model": string, price_uid : nat};
```

1) `Set_administrator`: This action allows the replacement of the current administrator with a new one. The `admin_uid` parameter designates the address of the new administrator. As all policy tokens are owned by the administrator, upon setting a new administrator, all tokens MUST be transferred to the new administrator's address.
2) `Set_paused`: This action enables or disables interactions with the smart contract by non-administrative users. If `is_paused` is set to `true`, all interactions with the contract are suspended, excluding those issued by the administrator.
3) `Update_pricing_model`: This action allows the administrator to modify the pricing model associated with a particular `price_uid`. The `price` parameter, denominated in `tez`, represents the new price.
4) `Delete_pricing_model`: This action enables the administrator to remove a pricing model associated with a particular `price_uid`.

These actions ensure that the administrator retains full control over the application, allowing them to manage policies and pricing models according to their requirements.

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
The general recommendations of TZIP-16 MUST be followed.

### FA2.1

The [FA2.1][11] specification offers a significant extension to the policy token concept by incorporating the use of [Tickets][12]. This enables the direct maintenance of policies within a user's wallet, thereby promoting user-centric policy management. Additionally, the FA2.1 specification mandates the use of [Events][13] for tasks such as updating token metadata. This ensures a consistent audit trail of any policy changes made by the contract administrator or any authorized operator. Leveraging a FA2.1 compliant contract, which facilitates these advanced features, is RECOMMENDED for robust and user-friendly policy management.


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

1. `Update Verifier Public Key`: The inclusion of a mechanism to update the public key of the verifier should be considered. This mechanism might involve extending the entry points with a method specifically for updating the verifier key. Extra caution must be taken to ensure that the validation of a policy (using the signature and the raw bytes) can still be performed with a changed key, or potentially with a list of valid keys. An event mechanism to document changes to the verifier key is RECOMMENDED, and the ability to set a status for each verifier key may provide useful context about their usage state.

   ```jsligo
   type action = {"Set_verifier": string, verifier_uid : address, status : bool};
   ```
2) `Multiple Policy Tokens for a Single NFT Asset`: Future work could explore the creation of a mechanism that allows for the association of multiple policy tokens with a single NFT asset.
3) `View Definitions`: Defining views to access a policy and its verification status, as well as to access the list of verifiers and their respective statuses, would improve the transparency and accessibility of policy information.
4) `Policy URI in Policy Token Metadata`: It is worth considering the addition of a `policy_uri` to the policy token metadata. This URI, which would comply with [TZIP-16][1], would point to the policy represented json-ld e.g. on IPFS.
5) `Conceptualize Policy Evolution`: It is important to consider how policies might evolve or change over time. Developing a concept for how policy updates, revocations, and additions could be managed while maintaining the integrity of the policy token and the associated NFT asset, could provide a valuable solution to the dynamic nature of policy management.

The adoption of these future directions could significantly enhance the functionality of the policy token system, providing more flexibility and control over policy management and verification.

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
[26]: https://objkt.com/asset/KT1G5v7LfnZKRQhifjhdmusEKcVmupEhZ4F3/0