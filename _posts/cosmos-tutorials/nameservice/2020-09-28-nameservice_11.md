---
layout: post
title:  "Cosmos-SDK nameservice 11"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# BuyName

## MsgBuyName

Now it is time to define the `Msg` for buying names and add it to the `./x/nameservice/types/msgs.go` file.
This code is very similar to `SetName`:

```go
// MsgBuyName defines the BuyName message
type MsgBuyName struct {
    Name  string         `json:"name"`
    Bid   sdk.Coins      `json:"bid"`
    Buyer sdk.AccAddress `json:"buyer"`
}

// NewMsgBuyName is the constructor function for MsgBuyName
func NewMsgBuyName(name string, bid sdk.Coins, buyer sdk.AccAddress) MsgBuyName {
    return MsgBuyName{
        Name:  name,
        Bid:   bid,
        Buyer: buyer,
    }
}

// Route should return the name of the module
func (msg MsgBuyName) Route() string { return RouterKey }

// Type should return the action
func (msg MsgBuyName) Type() string { return "buy_name" }

// ValidateBasic runs stateless checks on the message
func (msg MsgBuyName) ValidateBasic() error {
    if msg.Buyer.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Buyer.String())
    }
    if len(msg.Name) == 0 {
        return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name cannot be empty")
    }
    if !msg.Bid.IsAllPositive() {
        return sdkerrors.ErrInsufficientFunds
    }
    return nil
}

// GetSignBytes encodes the message for signing
func (msg MsgBuyName) GetSignBytes() []byte {
    return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}

// GetSigners defines whose signature is required
func (msg MsgBuyName) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{msg.Buyer}
}
```

Next, in the `./x/nameservice/handler.go` file, add the `MsgBuyName` handler to the module router:

```go
// NewHandler returns a handler for "nameservice" type messages.
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        switch msg := msg.(type) {
        case MsgSetName:
            return handleMsgSetName(ctx, keeper, msg)
        case MsgBuyName:
            return handleMsgBuyName(ctx, keeper, msg)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
        }
    }
}
```

Finally, define the `BuyName` `handler` function which performs the state transitions triggered by the message.
Keep in mind that at this point the message has had its `ValidateBasic` function run so there has been some input verification.
However, `ValidateBasic` cannot query application state.
Validation logic that is dependent on network state (e.g. account balances) should be performed in the `handler` function.

```go
// Handle a message to buy name
func handleMsgBuyName(ctx sdk.Context, keeper Keeper, msg MsgBuyName) (*sdk.Result, error) {
    // Checks if the the bid price is greater than the price paid by the current owner
    if keeper.GetPrice(ctx, msg.Name).IsAllGT(msg.Bid) {
        return nil, sdkerrors.Wrap(sdkerrors.ErrInsufficientFunds, "Bid not high enough") // If not, throw an error
    }
    if keeper.HasOwner(ctx, msg.Name) {
        err := keeper.CoinKeeper.SendCoins(ctx, msg.Buyer, keeper.GetOwner(ctx, msg.Name), msg.Bid)
        if err != nil {
            return nil, err
        }
    } else {
        _, err := keeper.CoinKeeper.SubtractCoins(ctx, msg.Buyer, msg.Bid) // If so, deduct the Bid amount from the sender
        if err != nil {
            return nil, err
        }
    }
    keeper.SetOwner(ctx, msg.Name, msg.Buyer)
    keeper.SetPrice(ctx, msg.Name, msg.Bid)
    return &sdk.Result{}, nil
}
```

First check to make sure that the bid is higher than the current price.
Then, check to see whether the name already has an owner.
If it does, the former owner will receive the money from the `Buyer`.

If there is no owner, your `nameservice` module "burns" (i.e. sends to an unrecoverable address) the coins from the `Buyer`.

If either `SubtractCoins` or `SendCoins` returns a non-nil error, the handler throws an error, reverting the state transition.
Otherwise, using the getters and setters defined on the `Keeper` earlier, the handler sets the buyer to the new owner and sets the new price to be the current bid.

> _*NOTE*_: This handler uses functions from the `coinKeeper` to perform currency operations. If your application is performing currency operations you may want to take a look at the [godocs for this module](https://godoc.org/github.com/cosmos/cosmos-sdk/x/bank#BaseKeeper) to see what functions it exposes.

### Great, now owners can `BuyName`s! But what if they don't want the name any longer? Your module needs a way for users to delete names! Let us define define the `DeleteName` message
# BuyName

## MsgBuyName

이제 name 구매를 위한 `Msg`를 `./x/nameservice/types/msgs.go` 파일에 정의할 차례입니다.
이 코드는 `SetName`과 유사합니다.

```go
// MsgBuyName : BuyName 메시지 정의
type MsgBuyName struct {
    Name  string         `json:"name"`
    Bid   sdk.Coins      `json:"bid"`
    Buyer sdk.AccAddress `json:"buyer"`
}

// NewMsgBuyName : MsgBuyName의 생성자
func NewMsgBuyName(name string, bid sdk.Coins, buyer sdk.AccAddress) MsgBuyName {
    return MsgBuyName{
        Name:  name,
        Bid:   bid,
        Buyer: buyer,
    }
}

// Route : 모듈의 이름을 반환
func (msg MsgBuyName) Route() string { return RouterKey }

// Type : 작업 반환
func (msg MsgBuyName) Type() string { return "buy_name" }

// ValidateBasic : 메시지에대한 stateless checks
func (msg MsgBuyName) ValidateBasic() error {
    if msg.Buyer.Empty() {
        return sdkerrors.Wrap(sdkerrors.ErrInvalidAddress, msg.Buyer.String())
    }
    if len(msg.Name) == 0 {
        return sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "Name cannot be empty")
    }
    if !msg.Bid.IsAllPositive() {
        return sdkerrors.ErrInsufficientFunds
    }
    return nil
}

// GetSignBytes : 서명을 위한 메시지 인코딩
func (msg MsgBuyName) GetSignBytes() []byte {
    return sdk.MustSortJSON(ModuleCdc.MustMarshalJSON(msg))
}

// GetSigners : 어떤 사용자의 사인이 필요한지 정의
func (msg MsgBuyName) GetSigners() []sdk.AccAddress {
    return []sdk.AccAddress{msg.Buyer}
}
```

다음으로 `./x/nameservice/handler.go` 파일에서 `MsgBuyName` 핸들러를 모듈 라우터에 추가합니다.

```go
// NewHandler : "nameservice" type message에 대한 핸들러 반환
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) (*sdk.Result, error) {
        switch msg := msg.(type) {
        case MsgSetName:
            return handleMsgSetName(ctx, keeper, msg)
        case MsgBuyName:
            return handleMsgBuyName(ctx, keeper, msg)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, fmt.Sprintf("Unrecognized nameservice Msg type: %v", msg.Type()))
        }
    }
}
```

마지막으로 메시지에의해 트리거된 상태 전환을 수행하는 `BuyName`의 `handler` 함수를 정의합니다.
메시지에는 `ValidateBasic` 함수가 구현되었으며 입력값에 대한 확인을 진행합니다. 그러나 `ValidateBasic`는 애플리케이션 state를 쿼리할 수는 없습니다.
네트워크 상태(예 : account balance)에 의존하는 ValidateBasic 로직은 `handler` 기능에서 수행되어야 합니다.

```go
// handleMsgBuyName : name 구매 메시지 처리
func handleMsgBuyName(ctx sdk.Context, keeper Keeper, msg MsgBuyName) (*sdk.Result, error) {
    // 입찰 가격이 현재 소유자가 지불한 가격보다 큰지 확인
    if keeper.GetPrice(ctx, msg.Name).IsAllGT(msg.Bid) {
        return nil, sdkerrors.Wrap(sdkerrors.ErrInsufficientFunds, "Bid not high enough") // 에러메시지 반환
    }
    if keeper.HasOwner(ctx, msg.Name) {
        err := keeper.CoinKeeper.SendCoins(ctx, msg.Buyer, keeper.GetOwner(ctx, msg.Name), msg.Bid)
        if err != nil {
            return nil, err
        }
    } else {
        _, err := keeper.CoinKeeper.SubtractCoins(ctx, msg.Buyer, msg.Bid) // sender로부터 입찰금액 공제
        if err != nil {
            return nil, err
        }
    }
    keeper.SetOwner(ctx, msg.Name, msg.Buyer)
    keeper.SetPrice(ctx, msg.Name, msg.Bid)
    return &sdk.Result{}, nil
}
```

먼저 입찰가가 현재 가격보다 높은지 확인합니다, 그런 다음 name에 이미 소유자가 있는지 확인 합니다. 만약 이전 소유자가 있을경우 `Buyer`로부터 돈을 받게 됩니다.

소유자가 없는 name의 경우 `nameservice` 모듈이  `Buyer`의 코인을 `소각`(즉, 복구할 수 없는 주소로 전송) 합니다.

`SubtractCoins` 또는 `SendCoins`가 nil이 아닌 오류를 반환하면 핸들러는 오류를 발생시켜 상태 전환을 되돌립니다.
그렇지 않으면 이전에 `Keeper`에 정의된 getter/setter를 사용하여 핸들러가 구매자를 새 소유자로 설정하고 새 가격을 현재 입찰로 설정합니다.

> _*NOTE*_: 이 핸들러는 `coinKeeper`의 함수를 사용하여 통화작업을 수행합니다. 애플리케이션이 통화 연산을 수행하는 경우 [bank 모듈의 godocs](https://godoc.org/github.com/cosmos/cosmos-sdk/x/bank#BaseKeeper)를 살펴보고 어떤 함수가 노출되는지 확인할 수 있습니다.

### 이제 `BuyName`을 사용할 수 있습니다. 그러나 소유자가 더 이상 이름을 원하지 않는다면? 모듈에는 사용자가 이름을 삭제할 수 있는 방법이 필요합니다. `DeleteName` 메시지를 정의하겠습니다
