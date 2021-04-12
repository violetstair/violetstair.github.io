---
layout: post
title:  "Cosmos-SDK nameservice 04"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Types

가장 먼저 해야할 일은 다음과 같은 명령으로 사용하여 scaffold tool로 `/x/` 폴도에 모듈을 만드는 것 입니다.

이 튜토리얼의 경우 모듈 이름을 `nameservice`로 할것입니다.

```bash
cd x/

scaffold module [user] [repo] nameservice
```

## `types.go`

이제 모듈 생성을 계속할 수 있습니다. 모듈에 대한 사용저 정의 type을 지정합니다. `./x/nameservice/types` 폴더에 `types.go` 파일을 생성하여 시작합니다.

## Whois

각 객체에는 name과 관련된 세가지 데이터가 있습니다.

- Value - 이름이 확인하는 값입니다 .이것은 임의의 문자열일 뿐이지만 나중에 IP 주소, DNS zone file 또는 블록체인 주소와 같은 특정 형식에 적합하도록 수정할 수 있습니다.
- Owner - 현재 이름의 소유자
- Price - 이름을 구매하기위해 지불해야하는 가격

SDK 모듈을 시작하기위해 `./x/nameservice/types/types.go` 파일에서 `nameservice.Whois` 구조체를 정의합니다.

[types.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/types/types.go)

```go
package types

import (
    "fmt"
    "strings"

    sdk "github.com/cosmos/cosmos-sdk/types"
)

// MinNamePrice : 이전에 소유한 적이 없는 name의 초기 시작 가격
var MinNamePrice = sdk.Coins{sdk.NewInt64Coin("nametoken", 1)}

// Whois : name의 모둔 메타데이터를 포함하는 구조체
type Whois struct {
    Value string         `json:"value"`
    Owner sdk.AccAddress `json:"owner"`
    Price sdk.Coins      `json:"price"`
}
```

[디자인 문서](./01-app-design.md)에서 언급했듯이 name에 아직 소유자가 없는 경운 MinPrice를 사용하여 초기화 해야합니다.
