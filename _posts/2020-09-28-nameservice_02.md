---
layout: post
title:  "Cosmos-SDK nameservice 02"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# Application Goals

애플리케이션의 목표는 사용자가 이름을 구매할 수 있도록하고 값을 설정하는 것 입니다. 주어진 이름의 소유자가 현재 최고 입찰자가 됩니다.
이 색션에서는 이러한 간단한 요구사항이 애플리케이션 설계로 어떻게 변환되는지 알아봅니다.

블록체인 애플리케이션은 [replicated deterministic state machine](https://en.wikipedia.org/wiki/State_machine_replication) 입니다.
개발자는 state machine을 정의하기만 하고(상태, 시작 상태 및 상태 전환을 트리거 하는 메시지 등), [_Tendermint_](https://docs.tendermint.com/master/introduction/)가 네트워크를 통해 복제를 처리합니다.

> Tendermint는 블록체인의 _networking_ 및 _consensus_ 레이어를 처리하는 엔진이며, 애플리케이션에 구애받지 않습니다. 실제로 이것은 Tendermint가 트랜잭션 바이트를 전파하고 주문하는 책임이 있음을 의미합니다. Tendermint Code는 트랜잭션 순서에 대한 합의에 도달하기위해 BFT(Byzantine-Fault-Tolerant) 알고리즘을 사용합니다.
Tendermint에 대한 보다 상세한 내용은 [여기](https://tendermint.com/docs/introduction/introduction.html)를 참조하세요.

[Cosmos SDK](https://github.com/cosmos/cosmos-sdk/)는 state machines을 구축할 수 있도록 설계되어있습니다.
SDK는 **modular framework** 입니다. 즉 상호 운용이 가능한 모듈 모음을 통합하여 애플리케이션이 빌드됩니다.
각 모듈에는 자체 메시지 / 트랜잭션 프로세서가 포함되어 있으며, SDK는 각 메시지를 해당 모듈로 라우팅하는 역할을 합니다.

다음은 nameservice 애플리케이션에 필요한 모듈입니다:

- `auth` : 이 모듈은 계정과 수수료를 정의하고 나머지 애플리케이션에 이러한 기능에 대한 액세스를 제공합니다.
- `bank` : 이 모듈을 사용하면 애플리케이션이 token 및 token balance를 만들고 관리할 수 있습니다.
- `staking` : 이 모듈을 사용하면 애플리케이션에서 사람들이 위임할 수있는 validators를 가질 수 있습니다.
- `distribution` : 이 모듈은 validator와 delegator간에 보상을 수동적으로 분배할 수 있는 기능을 제공합니다.
- `slashing` : 이 모듈은 네트워크에 validator와 같이 네트워크 value staked 사용자들에 대해 인센티브를 제공하지 않기 위해 동작합니다.
- `supply` : 이 모듈은 체인의 total supply 정보를 보유합니다.
- `nameservice` : 이 모듈은 아직 존재하지 않는 우리가 개발해야할 모듈입니다. `nameservice` 애플리케이션의 핵심 로직을 처리하며, 애플리케이션을 구축하기위해 작업해야하는 메인 애플리케이션 입니다.

이제 애플리케이션의 두 가지 주요 부분인 상태(state)와 메시지 유형(message types)를 살펴보겠습니다.

## State

state는 그 시점의 애플리케이션의 상태를 나타냅니다. 각 계정이 보유한 token의 수량, 각 이름의 수용자 및 가격, 각 이름이 어떤 가치로 해석되는지 알려줍니다.

계정의 상태와 token은 `auth`와 `bank` 모듈에 의해 정의됩니다.
그러므로 개발자는 `nameservice` 모듈과 관련된 state를 정의하는 부분만 생각하면 됩니다.

SDK에서는 모든것인 `multistore`라는 store에 저장됩니다.
이 multi store에서 원하는 수의 key/value store(Cosmos SDK에서는 [`KVStores`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#KVStore)라고 부릅니다.)를 만들 수 있습니다. 이 애플리케이션의 경우 하나의 store를 사용하여 `name`에 대한 값과 소유자, 각격을 보유하는 `whois` 구조체를 매핑할 수 있습니다.

## Messages

메시지는 트랜잭션을 포함합니다. 상태 전환을 트리거하며, 각 모듈은 메시지 목록과 처리 방법을 정의합니다.
다음은 nameservice 애플리케이션에 대해 원하는 기능을 구현하는데 필요한 메시지입니다.

- `MsgSetName`: 이 메시지는 name 소유자가 주어진 name에 대한 값을 설정할 수 있도록 합니다.
- `MsgBuyName`: 이 메시지를 통해 계정은 이름을 구매하고 소유자가 될 수 있습니다.
  - 누군가 이름을 사면 이전 소유자가 지불한 가격보다 더 높은 가격을 name의 이전 소유자에게 지불해야 합니다. name에 이전 소유자가 없는경우 `MinPrice` 금액을 소각합니다.

트랜잭션(블록 포함)이 Tendermint 노드에 도달하면 [ABCI](https://github.com/tendermint/tendermint/tree/master/abci)를 통해 애플리케이션으로 전달되고 decode되어 메시지를 받습니다.
그런 다음 메시지는 적절한 모듈로 라우팅되고 `Handler`에 정의된 로직에 따라 처리됩니다.
state를 업데이트 해야하는 경우 `Handler`는 `Keepr`를 호출하여 업데이트를 수행합니다.
이 튜토리얼의 다음 단계에서는 이러한 개념에 대해 자세히 알아봅니다.

### high-level 관점에서 애플리케이션의 동작 방식을 결정했으므로 이제 구현을 할 차례입니다
