---
layout: post
title:  "Cosmos-SDK nameservice 18"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# AppModule Interface

Cosmos SDK는 모듈에 대한 표준 인터페이스를 제공합니다.
이 [`AppModule`](https://github.com/cosmos/cosmos-sdk/blob/master/types/module.go) 인터페이스에는 `ModuleBasicsManager`가 애플리케이션에 통합하는데 사용하는 메소드 세트를 제공하는 모듈이 필요합니다. 먼저 인터페이스를 scaffold하고 그 메소드 중 **일부**를 구현합니다. 그런 다음 `auth`, `bank`와 함께 nameservice 모듈을 앱에 통합합니다.

두 개의 새로운 파일 `module.go`, `genesis.go`을 열어 시작합니다. `module.go`에 AppModule 인터페이스를 구현하고 `genesis.go`에 genesis 상태 관리 관련 기능을 구현합니다. AppModule 구조체의 genesis-specific 메서드는 `genesis.go`에 정의된 호출을 통과합니다.

`module.go`에 다음 코드를 추가하는 것으로 시작합니다. scaffold 도구는 이미 필요한 데이터로 모든 기능을 채웠지만 몇 가지 해야할 일이 남았습니다. 이 모듈이 의존하는 모듈을 `AppModule` type에 추가해야 합니다.

[module.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/module.go)

AppModule 구현의 더 많은 예제를 보려면 [x/staking](https://github.com/cosmos/cosmos-sdk/blob/master/x/staking/genesis.go)과 같은 SDK의 다른 모듈의 구현 예제를 참조할 수 있습니다.

## 다음으로 위에서 호출한 genesis-specific 메서드를 구현할 차례 입니다
