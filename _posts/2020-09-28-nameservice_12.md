---
layout: post
title:  "Cosmos-SDK nameservice 12"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# Delete Name

## MsgDeleteName

이제 이름을 삭제할 `Msg`를 정의하고 `./x/nameservice/types/msgs.go` 파일에 추가할 차례입니다. 이 코드는 `SetName`와 유사합니다.:

```go
// MsgDeleteName : DeleteName 메시지 정의
type MsgDeleteName struct {
    Name  string         `json:"name"`
    Owner sdk.AccAddress `json:"owner"`
}

// NewMsgDeleteName : MsgDeleteName 함수의 생성자
func NewMsgDeleteName(name string, owner sdk.AccAddress) MsgDeleteName {
    return MsgDeleteName{
        Name:  name,
        Owner: owner,
    }
}

// Route : 모듈의 이름을 반환
func (msg MsgDeleteName) Route() string { return RouterKey }

// Type : 작업 반환
func (msg MsgDeleteName) Type() string { return "delete_name" }

// ValidateBasic : 메시지에 대한 stateless checks 실행
func (msg MsgDeleteName) ValidateBasic() error {
    if msg.Owner.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Owner.String())
    }
    if len(msg.Name) == 0 {
        return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name cannot be empty")
    }
    return nil
}

// GetSignBytes : 서명을 위한 메시지 인코딩
func (msg MsgDeleteName) GetSignBytes() []byte {
    return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}

// GetSigners : 어떤 사용자의 사인이 필요한지 정의
func (msg MsgDeleteName) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{msg.Owner}
}
```

다음으로 `./x/nameservice/handler.go` 파일에 `MsgDeleteName` 핸들러를 모듈 라우터에 추가합니다.

```go
// NewHandler : "nameservice" type messages에 대한 핸들러 반환
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        switch msg := msg.(type) {
        case MsgSetName:
            return handleMsgSetName(ctx, keeper, msg)
        case MsgBuyName:
            return handleMsgBuyName(ctx, keeper, msg)
        case MsgDeleteName:
            return handleMsgDeleteName(ctx, keeper, msg)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
        }
    }
}
```

마지막으로 메시지에 의해 트리거된 상태 전환을 수행하는 `DeleteName`에 대한 `handler` 함수를 정의합니다
이 시점에서 메시지에는 `ValidateBasic` 함수가 정의되었으므로 입력 값에 대한 확인을 진행합니다.
그러나 `ValidateBasic`은 애플리케이션의 상태를 쿼리할 수 없습니다.
네트워크 상태(예: account balance)에 의존하는 ValidateBasic 로직은 `handler` 함수에서 진행되어야 합니다.

```go
// handleMsgDeleteName : name 삭제를 위한 메시지 처리
func handleMsgDeleteName(ctx sdk.Context, keeper Keeper, msg MsgDeleteName) (*sdk.Result, error) {
    if !keeper.IsNamePresent(ctx, msg.Name) {
        return nil, sdkerrors.Wrap(types.ErrNameDoesNotExist, msg.Name)
    }
    if !msg.Owner.Equals(keeper.GetOwner(ctx, msg.Name)) {
        return nil, sdkerrors.Wrap(sdkerrors.ErrUnauthorized, "Incorrect Owner")
    }

    keeper.DeleteWhois(ctx, msg.Name)
    return &sdk.Result{}, nil
}
```

먼저 name이 현재 store에 있는지 확인하고 없는 경우 오류를 발생시키고 사용자에게 반환합니다.
그런 다음 `Msg` sender가 실제로 name의 소유자(`keeper.GetOwner`)인지 확인합니다.
이후 `Keeper`에서 함수를 호출하여 name을 삭제할 수 있습니다.

### 이제 `Messages`와  `Handlers`를 정의했으므로 이러한 트랜잭션에서 데이터를 만드는 방법을 확인해볼 차례입니다
