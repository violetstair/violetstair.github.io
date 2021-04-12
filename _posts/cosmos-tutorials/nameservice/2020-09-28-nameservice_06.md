---
layout: post
title:  "Cosmos-SDK nameservice 06"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Errors

types 폴더내에 있는 `errors.go` 파일을 수정합니다.
`errors.go` 파일 내에서 코드와 함께 모듈에 맞는 에러를 정의합니다.

[errors.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/types/errors.go)

```go
package types

import (
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

var (
    ErrNameDoesNotExist = sdkerrors.Register(ModuleName, 1, "name does not exist")
)

```

오류 처리시 호출될 해당 메서드를 추가해야 합니다. 예를들어 store에 없는 name을 삭제하려고 한다면 name이 존재하지 않으므로 오류가 발생합니다.
이 메서드가 호출되는 모습은 튜토리얼 뒷 부분에서 확인할 수 있습니다.

이제 모듈을 위한 Keeper를 작성할 순서입니다.
