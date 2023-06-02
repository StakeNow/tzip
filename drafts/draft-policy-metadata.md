---
tzip: 0xx
title: Policy Metadata
status: Draft
author: Carlo van Driesten <carlo.van-driesten@vdl.digital>, Roy Scheeren <roy.scheeren@vdl.digital>, Pierre Mai <pmai@pmsfit.de>
type: Interface
created: 2023-05-10
date: tbd
version: 0.1
---

## Abstract
This proposal is an extension of [TZIP-016][1] and describes a metadata schema and standards for tokenized [ODRL][2] compliant policies.

The document is broken into two main sections:
1) The metadata schema, and
2) standards and recommendations for how to apply the schema to different token types.

Many of the terms in this standard are derived from [The Dublin Core, RCF 2413][3].

## Table of Contents

## Motivation & Goals
Defining usage rights and obligations of resources in a trustless environment.

SIWT started with providing an access control mechanism in this trustless environment. Permissions to use resources can be granted based on the current state of the blockchain. 

The next step is to formalize the rights and obligations both provider and consumer have when using resources.

ODRL is a standardized way of describing policies that define these rights and obligations.

A lot of effort has been made into formalizing and standardizing the way NFTs should be formatted but little has been realized on effectively formalizing "what" a user is "really" buying. Up until now, many of the rights of NFTs have been implied and assumed.

With the vast amount of use cases for NFTs there will be situations where an explicit, trustless policy and history thereof is beneficial.

### Policies and their representations
Policies are represented by a seperate token. This token can be any NFT. The separation describes the rights and obligations connected to the policy token and the representative token. 

The policy is owned and managed by the administrator of the application. The administrator SHOULD be allowed to update/alter/revoke policies connected to a token as per their discretion.

The owner of a representative token however does not necessarily care to make use of the connected policy, and should in all cases be allowed to exercise the rights connected to that NFT, like trading. 

By not connecting policies directly to the metadata of the representative NFT, the use case of NFTs does not get altered, only extended.


## Use Cases
Those are example use cases in order to understand the mechanism.

### Access to events (online / real events)
Events tickets

- 

### Access to a web(3) application
StakeNow.fi use case

- subscription
- periodic: yearly or monthly
- perpetual
- access to an application
- transferable
- constraints based: If A owns (NFT) Token then gets access to application

### Membership
ENVITED use case

- periodic: yearly
- access to whatever the Terms of Service define (link to T&S)
- non-transferable

### Supported NFT tokens
- Multi asset (TZIP-12 / ERC-1155)
  - marketplace contracts: hic et nunc, teia, objkt
  - transferable
- non-fungible token (TZIP-12 / ERC-721)
  - custom contract
  - non-transferable (soul bound)
  - transferable

## ODRL Information Model

> The Open Digital Rights Language (ODRL) is a policy expression language that provides a flexible and interoperable information model, vocabulary, and encoding mechanisms for representing statements about the usage of content and services. The ODRL Information Model describes the underlying concepts, entities, and relationships that form the foundational basis for the semantics of the ODRL policies.
>
> Policies are used to represent permitted and prohibited actions over a certain asset, as well as the obligations required to be meet by stakeholders. In addition, policies may be limited by constraints (e.g., temporal or spatial constraints) and duties (e.g. payments) may be imposed on permissions.
>
> from [ODRL Information Model 2.2][2]

ODRL implementations are e.g. listed [here][4] and the best practices are collected [here][5] which are recommended to be followed in this specification as well.

## Implementations
A reference implementation of a policy contract using the described meta data schema in combination with a [FA2][8] contract is implemented in the library [Sign in with Tezos][9]

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

