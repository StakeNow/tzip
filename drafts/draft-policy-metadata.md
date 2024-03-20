---
tzip: tbd
title: Policy Tokenization
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Roy Scheeren <roy.scheeren@vdl.digital>, Pierre Mai <pmai@pmsfit.de>
type: LA
created: 2023-06-19
date: 2024-02-29
requires: ["TZIP-32"]
---


## Abstract

This proposal defines the tokenization of policies - a process to represent policies as unique digital assets on the blockchain, written in the Open Digital Rights Language (ODRL). ODRL is a language used to express policies for information management and digital rights governance. The proposal extends a Tezos FA2-compliant smart contract for functionality and interoperability on the Tezos blockchain to express usage rights and obligations of resources in a trustless environment.

This proposal is built as an extension to existing Tezos Improvement Proposals (TZIPs):

1. TZIP-012: Establishing usage constraints and interfaces for the FA2 smart contract. This alignment ensures that the proposed tokenized policy mechanism integrates with the existing infrastructure.
2. TZIP-016: Describing the metadata schema and standards for tokenized ODRL-compliant policies, thereby standardizing the representation and interpretation of policy metadata.

Moreover, this document specifies a methodical procedure for minting ODRL policy tokens. This process guarantees the authenticity and correctness of the tokenized policies through a robust verification mechanism. By integrating cryptographic techniques and blockchain's inherent transparency, the proposal ensures that every policy token minted is verifiable and traceable.

Potential use cases of this proposal encompass various domains where granular access control and rights management are critical. Examples include digital content distribution, information access control in organizations, and rights management in open data ecosystems. It is expected that the implementation of this proposal will significantly contribute to the robustness, transparency, and flexibility of policy management on the Tezos blockchain.

## Table of Contents

- [General](##General)
- [Motivation & Goals](#Motivation-amp-Goals)
- [ODRL Information Model](#odrl-information-model)
- [Tokenization of Policies](#tokenization-of-policies)
- [Specification](#specification)
  - [Overview](#overview)
    - [Smart Contract Entry Points](#smart-contract-entry-points)
    - [Contract Storage](#contract-storage)
  - [Prerequisite: Authentication](#prerequisite-authentication)
  - [Verifier: Create, Verify & Sign Policies](#Verifier-Create-Verify-amp-Sign-Policies)
    - [Policy Message](#policy-message)
    - [Verifier API](#verifier-api)
    - [Message Signing](#message-signing)
      - [Failing_noop](#failing_noop)
      - [Authenticated Signing Request](#authenticated-signing-request)
  - [Step 1: Mint](#step-1-mint)
  - [Step 2: Update](#step-2-update)
  - [Administrative Interface](#administrative-interface)
  - [Metadata](#metadata)
    - [Token Metadata](#token-metadata)
    - [Contract Metadata (TZIP-016)](#contract-metadata-tzip-016)
  - [FA2.1](#fa21)
- [Use Cases](#use-cases)
  - [Digital Art Ownership & Licensing](#Digital-Art-Ownership-amp-Licensing)
  - [Media Content Distribution](#media-content-distribution)
  - [Academic Research Licensing](#academic-research-licensing)
  - [Traditional Organizations and DAOs](#traditional-organizations-and-daos)
  - [Game Assets Ownership and Trading](#game-assets-ownership-and-trading)
- [Implementations](#implementations)
- [Future Directions](#future-directions)
- [Copyright](#copyright)

## General

Many of the terms in this standard are derived from [The Dublin Core, RFC 2413][3].

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”,
“SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be
interpreted as described in [RFC 2119][27].

## Motivation & Goals

While significant strides have been made toward standardizing the format of Non-Fungible Tokens (NFTs), there's been a conspicuous lack of progress in formalizing the true essence of what a user is purchasing. So far, most rights pertaining to NFTs have been implicit, taken for granted rather than explicitly stated.

> Property rights exist only for (physical) objects and thus not for digital works. Only rights of use can be considered, but these must be granted individually.
>
> The most similar to ownership are "exclusive as well as temporally, spatially and content-wise unrestricted, transferable, sublicensable and rentable rights of use″. These could be supplemented by editing rights or various types of use, such as the right to print on paper.
>
> *translated from [CMS Deutschland Blog][14]*

Generally speaking, any form of digital art or data can be tokenized, for instance, as an NFT, effectively serving as a digital counterpart or "twin". However, the digital rights associated with these assets must be independently defined. Given the sheer breadth of use cases for these digital assets, there are bound to be circumstances where a transparent, tamper-proof policy, complete with its history, will be advantageous.

This proposal seeks to establish a robust framework for defining rights and obligations. As a potential application, it aims to delineate the rights of both providers and consumers when interacting with resources such as, for example, a web3 application.

## ODRL Information Model

ODRL is a W3C standard recommendation to describe policies that define rights and obligations and is widely used in the industry, e.g., in the [Mobility Data Space][15].

> The Open Digital Rights Language (ODRL) is a policy expression language that provides a flexible and interoperable information model, vocabulary, and encoding mechanisms for representing statements about the usage of content and services. The ODRL Information Model describes the underlying concepts, entities, and relationships that form the foundational basis for the semantics of the ODRL policies.
>
> Policies are used to represent permitted and prohibited actions over a particular asset, as well as the obligations required to be met by stakeholders. In addition, policies may be limited by constraints (e.g., temporal or spatial constraints) and duties (e.g., payments) may be imposed on permissions.
>
> from [ODRL Information Model 2.2][2]

ODRL implementations are, e.g., listed [here][4] and the best practices are collected [here][5] which are recommended to be followed in this specification as well.

## Tokenization of Policies

Tokens provide an immutable record of the permissions, prohibitions, and obligations dictated by the tokenized ODRL policy. This record can be referenced by the application to enforce access controls.

- **Role of Tokens:** Each token can be seen as a 'license' carrying the access policy (ODRL policy) for a particular resource, codified within it as a smart contract. The access policy specifies the permissions, obligations, and prohibitions associated with that digital asset.

- **Enforcing Access:** The actual enforcement of access control takes place in the application layer, not at the token level. When a user tries to interact with a resource through the application, the application refers to the token's policy (on-chain data) to ascertain the user's permissions regarding that resource.

- **Decentralized Documentation:** Each token serves as a decentralized, immutable proof of the policy. If a dispute arises, the token's smart contract can provide irrefutable evidence of the conditions set forth at the time of the user's interaction with the resource.

- **Claim State:** The immutability and transparency of the blockchain provide a solid ground for users to state a claim if they feel their access rights, as detailed in the token, have been violated.

## Specification

### Overview

#### Smart Contract Entry Points

The following actions MUST be implemented:

```jsligo
type Action =
 | ["Set_adminstrator": string, admin_uid : address]
 | ["Set_paused": string, is_paused : bool]
 | ["Mint": string, policy_params : PolicyParams]
 | ["Update": string, policy_params : PolicyParams]
 | ["Update_pricing_model": string, price_uid : nat, price : tez, is_valid : bool]
 | ["Payout": string, amount : tez]
 | ["Update_operators": string, update : FA2_UpdateOperators.Types.update_operators]
 | ["Balance_of": string, balance : FA2_BalanceOf.Types.balance_of]
 | ["Transfer": string, transfer : FA2_Transfer.Types.transfer]
```

#### Contract Storage

The policy token contract MUST contain:

- a reverse lookup table, named `%asset_table`. This table SHOULD be of type `(big_map (pair (address %asset_address, nat %token_id)) (nat %policy_token_id))`.
- a lookup table for the prices for policies, named `%pricing_table`. This table SHOULD be of type `(big_map (nat %pricing_id) pair((tez %price)(bool %is_valid))`. The table MUST contain an entry for `%MINT_PRICE`.

It is RECOMMENDED that the policy token contract contains:

- a lookup table for allowed NFT asset contract addresses, named `%allowed_assets`. This table SHOULD be of type `(big_map (address %asset_contract_address) bool %is_valid)`.

### Prerequisite: Authentication

The instantiation of an application tasked with minting policies MUST employ an authentication Software Development Kit (SDK), such as Sign in with Tezos [(SIWT)][9], to ascertain that the user possesses the necessary access to the private key corresponding to the utilized address.

To ensure secure communication between the frontend and backend components of the application, it is RECOMMENDED that JSON Web Tokens (JWT) be adopted as the preferred method of facilitating this interaction. This recommendation arises from JWT's proven reliability in maintaining information security, offering confidentiality and integrity during data exchanges.

However, developers SHOULD take note that this document only suggests SIWT and JWT as viable options for authentication and secure communication. The developer MUST ensure that whatever approach chosen aligns with the best practices for maintaining secure and robust communication within the application's architecture.

It is RECOMMENDED to evaluate whether the chosen tools comply with the specific privacy and data protection standards required by their jurisdiction or the standards stipulated by their organization. Furthermore, developers should consider the integration with existing systems and the potential need for future scalability when making these decisions.

Underlying Assumptions:

- Requests to the `Verifier` from a user are authenticated using the user's Tezos address.
- There is no requirement for individual message requests to be signed by the user for the purposes of authenticating their origin.

### Verifier: Create, Verify & Sign Policies

The user MUST create a ODRL policy and MAY be guided through a user interface in a front-end application. The `json-ld` policy MUST contain the following entries:

- `"uid": <policy_contract_address:policy_token_id>`: The address of the policy FA2 contract and unique token id. The `Mint` step allocates the `policy_token_id` and therefore REQUIRES the `policy_contract_address`. The `Update` step additionally REQUIRES the `policy_token_id` in order to contain a fully unique `uid`.
- `"assignee": <asset_contract_address:asset_token_id>`: The NFT contract address and token id of the target asset NFT to which the policy belongs.

All further entries to construct a valid policy are specified in the [ODRL specification][2].

User, who makes a call to the verifier, are identified by their Tezos address `tzXXXXXX` of type `address` as a result of the preceding authentication process. The policy is inherently linked to the NFT asset, and the caller of the smart contract is only permitted to update or mint a policy if the ownership status of the asset NFT is validated.

The caller MUST provide the following parameters:

- `"policy_message"`: The message of type `PolicyMessage` that will be utilized for the `Mint` and `Update` entry points of the policy FA2 contract.

#### Policy Message

```jsligo
type UpdateMessage = {
  timestamp : timestamp;
  policy_contract_address : address;
  asset_contract_address : address;
  asset_token_id : nat;
  pricing_id : nat;
  policy : bytes;
};
```

Defined as:

- `"timestamp"`: Contains the time this message was created of type `timestamp`.
- `"policy_contract_address"`: Target policy contract `ktXXXXXX` of type `address`.
- `"asset_contract_address"`: Respective NFT asset contract `ktXXXXXX` of type `address`.
- `"asset_token_id"`: Token id of the associated NFT used as constraint of type `nat`.
- `"pricing_id"`: id of the `%pricing_table` in the policy contract of type `nat`.
- `"policy"`: The ODRL [json-ld][16] policy.

This is an example for an ODRL file that allows the usage of <https://stakenow.fi> under the condition to be the owner of the [StakeNow Beta User NFT][26]:

```json
{
   "@context": "http://www.w3.org/ns/odrl.jsonld",
   "@type": "Policy",
   "uid": "policy_contract_address:policy_token_id", // known after contract deployment and policy token reservation with `Mint`
   "profile": "http://www.w3.org/ns/odrl/2/Agreement",
   "permission": [{
       "target": "http://stakenow.fi",
       "action": "use",
       "assigner": "http://vdl.digital",
       "assignee": "KT1G5v7LfnZKRQhifjhdmusEKcVmupEhZ4F3:0"
   }]
}
```

#### Verifier API

The verifier MUST provide the following function:

```jsligo
let policy_message_verifier = (policy_message : PolicyMessage) => {
  // This function should call out to an external service to perform the actual verification & signing
  // For the purpose of this example, we are just returning the input message as bytes
  // and a placeholder signature

  let signed_policy_message = Bytes.pack(policy_message);
  let verifier_signature : signature = "sigXyz123"; // Replace with actual signature

  let policy_params: PolicyParams = {
     signed_policy_message = signed_policy_message;
     verifier_signature = verifier_signature;
   };

  return policy_params;
}
```

The return value MUST be of type:

```jsligo
type PolicyParams = {
  policy_message : PolicyMessage;
  verifier_signature : signature;
};
```

The `policy_message_verifier` MUST at least check the following information:

1) Check if the caller of the update action is the owner of the asset NFT.
2) If the `policy_token_id` is missing then these fields MUST be `"pricing_id" = id of %MINT_PRICE` and `"policy" = ""`.

In addition, the function SHOULD check the following information before the `policy_message` can be used as parameter of `Update`:

1) Check if the `policy_token_id` is allocated in the contract `policy_contract_address`.
2) Check if the `asset_contract_address` and `asset_token_id` is pointing to the `policy_token_id` within the policy `%asset_table`.
3) Validate the content of the ODRL policy according to the specification given as a template.
4) Check if the template has a corresponding price entry in `%pricing_table`.

#### Message Signing

The encapsulation of the `policy_message` using [Of-Chain Message Signing][24] is REQUIRED. This precaution prevents the signed message from being utilized for on-chain transactions. The feasibility of this process is largely contingent upon the support provided by various libraries and wallets.

##### Off-chain message signing

```jsligo
type magic_string = bytes;
type interface = bytes;
type character_encoding;
type message = bytes; // serialized PolicyMessage with `PACK`

type my_message = {
  magic_string : magic_string;
  interface : interface;
  character_encoding : character_encoding
  message : message;
};

>> b"\x80tezos signed offchain messagedraft-policy-tokenization2706f6c6963795f6d657373616765"
```

Defined as:

- `"magic_bytes": <b"\x80tezos signed offchain message">`: Magic bytes (aka watermark) referring to `Off-Chain Message Signing`.
- `"interface": <b"draft-policy-tokenization">`: Reference to this TZIP.
- `"character_encoding": <2>`: Refer
- `"message": <706f6c6963795f6d657373616765>`: The serialized `PolicyMessage` in `bytes` using `PACK`. (This example uses the string `"policy_message"`).

### Step 1: `Mint`

The minting process reserves a policy token in the contract. This token MUST be used as the `"uid"<policy_contract_address:policy_token_id>` within the ODRL policy. Only the owner of the associated NFT asset is permitted to `Mint` the policy token.

```jsligo
type Action = ["Mint" : string, policy_params : PolicyParams]
```

A number of actions are RECOMMENDED:

- Setting a moderate price of `%MINT_PRICE` to discourage the reservation of multiple empty policy tokens.
- Initialize a list of allowed associated NFT asset contract addresses in `%allowed_assets` on origination.
- Check if an entry for the asset NFT already exists in the `%asset_table`.

Advantages:

- The linking between entry points is completed.
- Potential replay attacks are mitigated by checking if a big map item already exists.
- It provides a straightforward way to check if a token has a policy.
- It allows SIWT to combine token and policy requirements.

Disadvantages:

- The additional ledger introduces extra complexity.
- Each asset can only be associated with a single policy.

### Step 2: `Update`

The update process replaces the policy token metadata in the contract. Only the owner of the associated NFT asset is permitted to `Update` the policy token.

```
type Action = ["Update" : string, policy_params : PolicyParams]
```

To maintain the integrity of the `Update` process, a number of checks are REQUIRED:

1) Signature Verification: The ODRL json-ld policy MUST be signed by the `verifier` prior to the update of a policy token. The signature of the `policy_message` MUST be validated using the [check_signature][17] function in LIGO:

    ```jsligo
    let check_signature =
      (pk: key, verifier_signature : signature, policy_message : bytes) =>
      Crypto.check(pk, verifier_signature, policy_message);
    ```

3) Timestamp Verification: The timestamp MUST be within a valid range.
4) Contract Address Verification: The contract address MUST correspond with the `policy_contract_address` specified within the `policy_message`.
5) Policy ID Verification: The `policy_token_id` MUST exist within the `policy_contract_address` contract. This implies that the token MUST have been reserved using the `Mint` action beforehand.

These checks ensure that the Update action is executed under the proper permissions and conditions.

### Administrative Interface

The policy associated with a token is under the purview of the application's administrator. It is envisioned that the administrator will have the authority to modify, revoke, or update policies linked to a token as per their discretion. Upon minting, a list of permissible associated NFT contracts is generated.

The following administrative actions are defined:

```jsligo
type AdminActions =
 | ["Set_adminstrator": string, admin_uid : address]
 | ["Set_paused": string, is_paused : bool]
 | ["Update_pricing_model": string, price_uid : nat, price : tez, is_valid : bool]
```

1) `Set_administrator`: This action allows the replacement of the current administrator with a new one. The `admin_uid` parameter designates the address of the new administrator. As all policy tokens are owned by the administrator, upon setting a new administrator, all tokens MUST be transferred to the new administrator's address.
2) `Set_paused`: This action enables or disables interactions with the smart contract by non-administrative users. If `is_paused` is set to `true`, all interactions with the contract are suspended, excluding those issued by the administrator.
3) `Update_pricing_model`: This action allows the administrator to modify the pricing model associated with a particular `price_uid`. The `price` parameter, denominated in `tez`, represents the new price. `is_valid` indicated that this price model is still applicable.

These actions ensure that the administrator retains full control over the application, allowing them to manage policies and pricing models according to their requirements.

### Metadata

#### Token Metadata

Token-specific metadata is stored/presented as a Michelson value of type `(map string bytes)` with bytes as UTF-8 stream of `bytes` without NULL termination. A few of the keys are reserved and predefined by TZIP-012 and in the case of a TZIP-016 URI pointing to a JSON blob, the JSON preserves the same 3 reserved non-empty fields:

`{"name": <string>, "symbol": <string>, "decimals": <number>, ... }`

Providing a value for `"decimals"` is required for all token types. `"name”` and `"symbol"` are not required but it is highly recommended for most tokens to provide the values either in the map or the JSON found via the TZIP-016 URI.

A valid ODRL policy token MUST contain the following token metadata values:

- `"policy"`: The ODRL [json-ld][16] policy.
- `"signature"`: A signature of [type][20] serialized with [`PACK`][22].

**Additional metadata properties MAY be added to the policy token metadata** to support further authenticity, and integrity purposes from external vocabularies. The ODRL Information Model recommends the use of Dublin Core Metadata Terms [dcterms][21] for ODRL policies.

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

A reference implementation of a policy contract using the described metadata schema in combination with a [FA2][8] contract is implemented in the library [Sign in with Tezos][9].

## Future Directions

1. `Update Verifier Public Key`: The inclusion of a mechanism to update the public key of the verifier should be considered. This mechanism might involve extending the entry points with a method specifically for updating the verifier key. Extra caution must be taken to ensure that the validation of a policy (using the signature and the raw bytes) can still be performed with a changed key, or potentially with a list of valid keys. An event mechanism to document changes to the verifier key is RECOMMENDED, and the ability to set a status for each verifier key may provide useful context about their usage state.

   ```jsligo
   type action = ["Set_verifier": string, verifier_uid : address, status : bool]
   ```

2. `Multiple Policy Tokens for a Single NFT Asset`: Future work could explore the creation of a mechanism that allows for the association of multiple policy tokens with a single NFT asset.
3. `View Definitions`: Defining views to access a policy and its verification status, as well as to access the list of verifiers and their respective statuses, would improve the transparency and accessibility of policy information.
4. `Policy URI in Policy Token Metadata`: It is worth considering the addition of a `policy_uri` to the policy token metadata. This URI, which would comply with [TZIP-16][1], would point to the policy represented json-ld e.g. on IPFS.
5. `Conceptualize Policy Evolution`: It is important to consider how policies might evolve or change over time. Developing a concept for how policy updates, revocations, and additions could be managed while maintaining the integrity of the policy token and the associated NFT asset, could provide a valuable solution to the dynamic nature of policy management.
6. `Multiple Assets to one Policy`: It may be considered to allow multiple pairs of `asset_contract_address:asset_token_id` to point to a single policy.
7. `Update allowed NFT asset contract addresses`: Add an entry point to update the `%allowed_assets` analogously to updating  `%pricing_table`:

   ```jsligo
   type AdminAction = ["Update_allowed_assets": string, asset_uid : nat, is_valid : bool]
   ```

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
[19]: https://tezos.gitlab.io/developer/michelson_anti_patterns.html?highlight=signature#signatures-alone-do-not-prevent-replay-attacks
[20]: https://ligolang.org/docs/language-basics/tezos-specific?lang=jsligo#checking-signatures
[21]: https://www.w3.org/TR/odrl-model/#bib-dcterms
[22]: https://ligolang.org/docs/language-basics/tezos-specific?lang=jsligo#pack-and-unpack
[24]: tbd
[26]: https://objkt.com/asset/KT1G5v7LfnZKRQhifjhdmusEKcVmupEhZ4F3/0
[27]: https://www.ietf.org/rfc/rfc2119.txt
