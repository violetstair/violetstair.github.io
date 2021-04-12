---
layout: post
title:  "Cosmos-SDK nameservice 07"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Expected Keepers

`types` 디렉토리 내에 `expected_keepers.go` 라는 파일을 만들고, 다른 모듈들에서 가져올 기능을 정의할것 입니다.

예를 들어 nameservice 모듈에서 우리는 두 당사자간 송금을 용이하게 하기위해 bank 모듈을 사용할 것 입니다.
이를 위해 bank 모듈의 두 가지 기능을 가져다 쓸 것 입니다.

- `SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) (sdk.Coins, error)`
- `SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error`

코드가 어떻게 구성되어 있는지 확인해 봅시다:

[expected_keepers.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/types/expected_keepers.go)

```go
package types

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
)

// 개발할 모듈이 모듈에서 사용할 기능을 명시하는 것이 좋으며, 모듈이 허용되지 않는 것을 방지하기 위해 인터페이스를 명시합니다.
type BankKeeper interface {
    SubtractCoins(ctx sdk.Context, addr sdk.AccAddress, amt sdk.Coins) (sdk.Coins, error)
    SendCoins(ctx sdk.Context, fromAddr sdk.AccAddress, toAddr sdk.AccAddress, amt sdk.Coins) error
}
```
