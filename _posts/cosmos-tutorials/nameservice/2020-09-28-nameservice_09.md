---
layout: post
title:  "Cosmos-SDK nameservice 09"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Msgs and Handlers

이제 `Keeper` 설정이 완료되었으므로, 실제로 사용자가 name을 구입하고 값을 설정할 수 있는 `Msgs`와 `Handlers`를 구현할 차례입니다.

## `Msgs`

`Msgs` 트리거 상태 전환. `Msgs`는 클라이언트가 네트워크에 제출하는 [`Txs`](https://github.com/cosmos/cosmos-sdk/blob/master/types/tx_msg.go#L34-L41)로 래핑됩니다. Cosmos SDK의 `Txs`에서 `Msgs`를 래핑 및 매핑 해제를 담당합니다. 즉 개발자는 `Msgs`만 정의하면 됩니다. `Msgs`는 다음 인터페이스를 충족해야 합니다. (다음 섹션에서 모두 구현할 것 입니다.)

```go
// Msg : Transaction message는 Msg를 충족해야 합니다
type Msg interface {
    // message type을 반환합니다.
    // 알파벳, 숫자 이더나 비어있어야 합니다.
    Type() string

    // 태그 내에서 활용하기 위해 대해 사람이 읽을 수 있는 메시지 문자열을 반환합니다.
    Route() string

    // ValidateBasic는 다른 정보에 액세스할 필요가 없기 때문에에 간단한 유효성 검사를 수행합니다
    ValidateBasic() Error

    // Msg는 표준 byte 표현을 가져옵니다
    GetSignBytes() []byte

    // 서명자는 서명해야하는 서명자의 주소를 반환합니다
    // CONTRACT: 모든 서명은 유효해야 합니다
    // CONTRACT: 결정론적인 순서로 addr을 반환합니다
    GetSigners() []AccAddress
}
```

## `Handlers`

`Handlers`는 주어진 `Msg`가 수신될때 취해야하는 조치(store 업데이트가 필요한 방법, 조건)를 정의합니다.

이 모듈에는 사용자가 애플리케이션 상태와 상호작용하기위해 보낼 수 있는 세 가지 유형의 `Msgs`인 `SetName`](./09-set-name.md), [`BuyName`](./10-buy-name.md), [`DeleteName`](./11-delete-name.md)가 있으며 각각 연관된 `Handler`가 있습니다.

이제 `Msgs`와 `Handlers`를 더 잘 이해했으므로 첫 번째 메시지 `SetName` 작업을 시작할 수 있습니다.
