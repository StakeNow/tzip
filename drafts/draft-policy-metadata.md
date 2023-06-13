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
    - [`transfer`](#transfer)
    - [`balance_of`](#balance_of)
    - [`update_operators`](#operators)
  - [Metadata](#token-metadata)
    - [Token Metadata](#token-metadata)
    - [Contract Metadata (TZIP-016)](#contract-metadata-tzip-016)
  - [Token balance updates](#token-balance-updates)
  - [FA2 Transfer Permission Policies and Configuration](#fa2-transfer-permission-policies-and-configuration)
  - [Error Handling](#error-handling)
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

The following actions MUST be implemnted:

```
type action = 
 | ["Mint", policy_params]
 | ["Update", update_policy_params]
 | ["SetAdministrator", address]
 | ["SetVerifier", key]
 | ["SetPaused", bool]
 | ["UpdatePricingModel", nat, tez]
 | ["DeletePricingModel", nat]
 | ["Payout", tez]
 | ["Update_operators", FA2_UpdateOperators.Types.update_operators]
 | ["Balance_of", FA2_BalanceOf.Types.balance_of]
 | ["Transfer", FA2_Transfer.Types.transfer];
```

### Admin Interface
The policy is owned and managed by the administrator of the application. The administrator SHOULD be allowed to update/alter/revoke policies connected to a token as per their discretion.

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

Extending the entry points with a method to update the verifier key. It has to be made sure that the validation of a policy using the signature and the raw bytes can still be performed with a changed key or e.g. a list of valid keys. It is also RECOMMEDED to implement an Event to document changes of the verifier key.
```
type action = ["SetVerifier", key]
```

Define varies views to access a policy and its verification status as another contract.


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
