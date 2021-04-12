---
layout: post
title:  "Cosmos-SDK nameservice 10"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# SetName

## `MsgSetName`

SDK `Msgs`의 네이밍 규칙은 `Msg{ .Action }` 입니다.
구현할 첫 번째 작업은 `SetName`이므로 `MsgSetName` 이라고 정의하겠습니다.
이 `Msg`는 name 소유자를 설정하고 해당 name에 대한 결과 값을 반환합니다.
`./x/nameservice/types/msgs.go`라는 파일에 `MsgSetName`을 정의하여 시작합니다:

```go
package types

import (
    sdk "github.com/cosmos/cosmos-sdk/types"
)

const RouterKey = ModuleName // ModuleName은 key.go 파일에 정의되어 있습니다

// MsgSetName : SetName 메시지 정의
type MsgSetName struct {
    Name  string         `json:"name"`
    Value string         `json:"value"`
    Owner sdk.AccAddress `json:"owner"`
}

// NewMsgSetName : MsgSetName의 생성자
func NewMsgSetName(name string, value string, owner sdk.AccAddress) MsgSetName {
    return MsgSetName{
        Name:  name,
        Value: value,
        Owner: owner,
    }
}
```

`MsgSetName`에는 name의 값을 설정하는데 필요한 세 가지 속성이 있습니다.:

- `name` - 설정하려는 name
- `value` - name을 해석하는 값
- `owner` - name의 소유자

다음으로, `Msg` 인터페이스를 구현합니다.:

```go
// Route : 모듈의 이름을 반환
func (msg MsgSetName) Route() string { return RouterKey }

// Type : 작업 유형 반환
func (msg MsgSetName) Type() string { return "set_name" }
```

위의 함수들은 SDK에서 처리를 위해 적절한 모듈로 `Messages`를 라우팅하는데 사용됩니다.
또한 인덱싱에 사용되는 데이터베이스 태그에 사람이 읽을 수 있는 name을 추가합니다.

```go
// ValidateBasic : 메시지에 대한 stateless 검사를 진행합니다
func (msg MsgSetName) ValidateBasic() error {
    if msg.Owner.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Owner.String())
    }
    if len(msg.Name) == 0 || len(msg.Value) == 0 {
        return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name and/or Value cannot be empty")
    }
    return nil
}
```

`ValidateBasic`는 `Msg`의 유효성에 대한 기본적인 **stateless** 검사를 진행하는데 사용됩니다.
위 코드의 경우 비어있는 속성이 없는지 확인합니다.

```go
// GetSignBytes : 서명을 위해 메시지를 인코딩 합니다
func (msg MsgSetName) GetSignBytes() []byte {
    return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}
```

`GetSignBytes`는 `Msg`가 서명을 위해 인코딩 되는 방식을 정의합니다.
대부분의 경우 정렬된 JSON에 대한 marshal을 의미합니다. 출력을 수정해서는 안됩니다.

```go
// GetSigners : 누구의 서명이 필요한지 정의
func (msg MsgSetName) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{msg.Owner}
}
```

`GetSigners`는 `Tx`가 유효하기 위해 필요한 서명을 정의합니다.\
예를 들어, 이 경우 `MsgSetName`은 이름이 가리키는 것을 재설정하려고 할 때 `Owner`가 트랜잭션을 서명하도록 요구합니다.

## `Handler`

이제 `MsgSetName`이 지정되었으므로 다음 단계에서느 메시지가 수신될 때 수행해야하는 작업을 정의할 차례입니다. 이것이 `handler`의 역할입니다.

handler 파일(`./x/nameservice/handler.go`)에서 다음 코드로 시작합니다.:

```go
package nameservice

import (
    "fmt"

    "github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"

    sdk "github.com/cosmos/cosmos-sdk/types"
    sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

// NewHandler : "nameservice" type messages에 대한 핸들러를 반환합니다.
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        switch msg := msg.(type) {
        case MsgSetName:
            return handleMsgSetName(ctx, keeper, msg)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
        }
    }
}
```

`NewHandler`는 본질적으로 모듈로 들어오는 메시지를 적절한 핸들러로 보내는 서브 라우터입니다. 현재로서는 `Msg`/`Handler`가 하나뿐입니다.

이제 `handleMsgSetName`에서 `MsgSetName` 메시지를 처리하기위한 실제 로직을 정의해야 합니다.:

> _*NOTE*_: SDK 핸들러 이름에 대한 네이밍 규칙은 `handleMsg{ .Action }`입니다.

```go
// handleMsgSetName : name 설정을 위한 메시지 처리
func handleMsgSetName(ctx sdk.Context, keeper Keeper, msg MsgSetName) (*sdk.Result, error) {
    if !msg.Owner.Equals(keeper.GetOwner(ctx, msg.Name)) { // 메시지 발신자가 현재 소유자인지 확인
        return nil, sdkerrors.Wrap(sdkerrors.ErrUnauthorized, "Incorrect Owner") // 소유자가 아니면 오류 발생
    }
    keeper.SetName(ctx, msg.Name, msg.Value) // 소유자의 경우 name의 값을 지정된 값으로 설정
    return &sdk.Result{}, nil
}
```

이 함수에서 `Msg` 발신자가 실제로 name(`Keeper.GetOwner`)의 소유자인지 확인합니다.
그렇다면 `Keeper`에서 함수를 호출하여 name을 설정할 수 있습니다. 그렇지 않은경우 오류를 발생시키고 사용자에게 반환합니다.

### 이제 소유자는 `SetName`을 할 수 있습니다. 하지만 name에 소유자가 없으면 어떻게 될까요? 모듈에는 name을 구매할 수 있는 기능이 필요합니다. `BuyName` 메시지를 정의하겠습니다
