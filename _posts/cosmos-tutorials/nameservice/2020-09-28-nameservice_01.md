---
layout: post
title:  "Cosmos-SDK nameservice 01"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Getting Started

[Cosmos SDK](https://github.com/cosmos/cosmos-sdk/)를 이용해 블록체인을 처음부터 끝까지 개발하고 그 과정을 통해 SDK의 기본 개념과 구조를 학습합니다.

이 가이드를 통해 `nameservice` 애플리케이션을 개발하게 됩니다.
[Namecoin](https://namecoin.org/), [ENS](https://ens.domains/), or [Handshake](https://handshake.org/)와 유사한 서비스 이며
기존의 DNS시스템(`map[domain]zonefile`)을 모델링하는 문자열 매핑(`map[string]string`) 구조를 갖게 됩니다.

사용자는 사용하지 않는 이름을 구매하거나 이름을 판매/거래 할 수 있습니다.

## Requirements

- [`golang` >1.13.0](https://golang.org/doc/install)
- [scaffold tool](https://github.com/cosmos/scaffold)을 이용해 튜토리얼을 진행합니다.
  - 다음 명령어를 통해 설치할 수 있습니다. `git clone git@github.com:cosmos/scaffold.git && cd scaffold && make`.

## Tutorial

애플리케이션을 구성한는 기본 구조를 생성합니다.:

```text
./nameservice
├── Makefile
├── Makefile.ledger
├── app.go
├── cmd
│   ├── nscli
│   │   └── main.go
│   └── nsd
│       └── main.go
├── go.mod
├── go.sum
└── x
    └── nameservice
        ├── alias.go
        ├── client
        │   ├── cli
        │   │   ├── query.go
        │   │   └── tx.go
        │   └── rest
        │       ├── query.go
        │       ├── rest.go
        │       └── tx.go
        ├── genesis.go
        ├── handler.go
        ├── keeper
        │   ├── keeper.go
        │   └── querier.go
        ├── types
        │   ├── codec.go
        │   ├── errors.go
        │   ├── expected_keepers.go
        │   ├── key.go
        │   ├── msgs.go
        │   ├── querier.go
        │   └── types.go
        └── module.go
```

이제 [애필리케이션의 디자인](./01-app-design.md)에 대해 알아볼 차례 입니다.
