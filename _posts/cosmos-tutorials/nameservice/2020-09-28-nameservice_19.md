---
layout: post
title:  "Cosmos-SDK nameservice 19"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Genesis

AppModule 인터페이스에는 체인의 GenesisState를 초기화하고 내보내는데 사용할 수 있는 여러 함수가 포함되어 있습니다.
`ModuleBasicManager`는 체인을 시작, 중지 또는 export할 때 각 모듈에서 이러한 함수를 호출합니다.
다음은 확장가능한 매우 기본적인 구현체 입니다.

`x/nameservice/genesis.go`로 이동하면 몇 가지 누락된 것을 확인할 수 있습니다. 모듈의 필요에 따라 채워야 합니다. 아래 코드에서 누락된 항목을 확인할 수 있습니다.

[genesis.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/genesis.go)

다음으로 genesis 상태가 무엇인지, default genesis 및 유효성 검사 방법을 정의하여 기존 상태로 체인을 시작할 때 오류가 발생하지 않도록 합니다.

[genesis.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/types/genesis.go)

위 코드에 대한 설명:

- `ValidateGenesis()`는 제공된 genesis 상태를 확인하여 불변성이 유지되는지 확인합니다.
- `DefaultGenesisState()`는 주로 테스트에 사용됩니다. 최소한의 GenesisState를 제공합니다.
- 체인 시작시 `InitGenesis()`가 호출되며, 이 함수는 genesis 상태를 keeper로 가져옵니다.
- 체인을 중지한 후 `ExportGenesis()`를 호출하면 이 함수는 애플리케이션 상태를 GenesisState 구조체에 로드하며, 나중에 다른 모듈의 데이터와 함께 `genesis.json`으로 내보낼 수 있습니다.

## 이제 모듈에는 Cosmos SDK 애플리케이션에 통합하는데 필요한 모든 것을 구현하었습니다
