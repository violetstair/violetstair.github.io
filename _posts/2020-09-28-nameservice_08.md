---
layout: post
title:  "Cosmos-SDK nameservice 08"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# The Keeper

Cosmos SDK 모듈의 핵심은 `Keeper`라는 모듈입니다.
store와 상호작용을 처리하고, 모듈간 상호작용을 위해 다른 Keeper에 대한 참조를 가지며, 모듈의 핵심 기능 대부분을 포함합니다.

## Keeper Struct

SDK 모듈을 시작하려면, `./x/nameservice/keeper/keeper.go` 파일에 `nameservice.Keeper`를 정의해야 합니다.
scafold tool에 의해 생성된 파일에 정의된 내용에 대해서는 이번에는 다루지 않을 것 이기때문에 `keeper.go` 파일에 있는 내용을 모두 지우고 새로 작성합니다.

```go
package keeper

import (
    "github.com/cosmos/cosmos-sdk/codec"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/violetstair/nameservice/x/nameservice/types"
)

// Keeper는 데이터 저장소에 대한 링크를 유지하고 state-machine의 다양한 부분에 대한 getter/setter 메서드를 노출합니다.
type Keeper struct {
    CoinKeeper types.BankKeeper

    storeKey  sdk.StoreKey // sdk.Context에서 저장소에 액세스하기 위한 노출되지 않는 키

    cdc *codec.Codec // 바이너리 인코딩/디코딩을 위한 wire codec
}
```

위 코드에 대한 몇 가지 참고 사항:

- 애플리케이션에 대한 두 개의 `cosmos-sdk` 패키지와 `types`을 가져옵니다.:
  - [`codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec) - `codec`은 Cosmos 인코딩 형식인 [Amino](https://github.com/tendermint/go-amino)로 작업할 수 있는 도구를 제공합니다.
  - [`types` (sdk의)](https://godoc.org/github.com/cosmos/cosmos-sdk/types) - 여기에는 SDK 전체에서 일반적으로 사용되는 type이 포함됩니다..
  - `types` - 이전 섹션에서 정의한 `BankKeeper`가 포함되어 있습니다.
- `Keeper` 구조체. 이 keeper에는 몇 가지 핵심 요소가 있습니다.:
  - `types.BankKeeper` - 이전 섹션에서 `bank` 모듈을 사용하기 위해 정의한 인터페이스 입니다. 이를 포함하면 모듈의 코드가 `bank` 모듈의 함수를 호출할 수 있습니다. SDK는 애플리케이션 상태 섹션에 액세스하기 위해 [object capabilities](https://en.wikipedia.org/wiki/Object-capability_model) 접근 방식을 사용합니다. 이는 개발자가 최소 권한 접근 방식을 이용하여 모듈에 결합이 있거나 악의적인 모듈의 접근에 의해 state에 영향을 주지 않도록 제한하기 위한것 입니다.
  - [`*codec.Codec`](https://godoc.org/github.com/cosmos/cosmos-sdk/codec#Codec) - binary structs를 인코딩 및 디코딩하기 위해 Amino에서 사용하는 코덱에 대한 포인터입니다.
  - [`sdk.StoreKey`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#StoreKey) - 애플리케이션의 상태를 유지하는 `sdk.KVStore`에 대한 액세스를 게이트하는 store key.

## Getters and Setters

이제 `Keeper`를 통해 store와 상호작용을 하는 메소드를 추가할 차례입니다.
먼저 주어진 name을 Whois에 설정하는 함수를 추가합니다.

```go
// SetWhois : name에 대한 Whois 메타데이터 구조체를 설정
func (k Keeper) SetWhois(ctx sdk.Context, name string, whois types.Whois) {
    if whois.Owner.Empty() {
        return
    }
    store := ctx.KVStore(k.storeKey)
    store.Set([]byte(name), k.cdc.MustMarshalBinaryBare(whois))
}
```

이 메서드에서는 먼저 `Keeper`의 `storeKey`를 사용하여 `map[name]Whois`에 대한 store object를 가져옵니다.

> _*NOTE*_: 이 함수는 [`sdk.Context`](https://godoc.org/github.com/cosmos/cosmos-sdk/types#Context)를 사용합니다. 이 객체는 `blockHeight`와 `chainID`와 같은 상태의 어러 중요한 부분에 액세스하는 함수를 보유합니다.

다음으로 `.Set([]byte, []byte)` 메서드를 사용하여 `<name, whois>` 쌍을 store에 저장합니다.
store는 `[]byte`만 사용하므로 Amino라는 Cosmos SDK 인코딩 라이브러리를 사용하여 `Whois` 구조체를 `[]byte`로 marshal하여 store에 저장합니다.

`Whois`의 소유자 필드가 비어있으면 존재하는 모든 name에 소유자가 있으야 하므로 store에 아무것도 쓰지 않습니다.

다음으로 name을 확인할 메소드를 추가합니다 (예 : `name`에 대한 `Whois` 조회)

```go
// GetWhois : name에 대한 전체 Whois 메타 데이터 구조체를 조회
func (k Keeper) GetWhois(ctx sdk.Context, name string) types.Whois {
    store := ctx.KVStore(k.storeKey)
    if !k.IsNamePresent(ctx, name) {
        return types.NewWhois()
    }
    bz := store.Get([]byte(name))
    var whois types.Whois
    k.cdc.MustUnmarshalBinaryBare(bz, &whois)
    return whois
}
```

여기서는 `SetName` 메소드와 마찬가지로 먼저 `storeKey`를 사용하여 store에 액세스합니다.
다음으로 store key에 `Set` 메소드를 사용하는 대신 `.Get([]byte) []byte`  메소드를 사용합니다.
함수의 매개변수로 전달된 `name`을 `[]byte` type으로 캐스팅해 store에 key로 전달하고 `[]byte` type의 결과를 반환 받는다.
우리는 Amino를 사용하지만 여기서는 byte slice를 `Whois` 구조체로 unmarshal해 반환한다.

현재 store에 name이 없는 경우 minimumPrice로 초기화된 새로운 `Whois`가 반환된다.

다음으로 name을 삭제하는 기능을 추가합니다.

```go
// DeleteWhois : name에 대한 Whois 메타 데이터 구조체를 삭제합니다.
func (k Keeper) DeleteWhois(ctx sdk.Context, name string) {
    store := ctx.KVStore(k.storeKey)
    store.Delete([]byte(name))
}
```

여기서는 `SetName` 메소드와 마찬가지로 먼저 `storeKey`를 사용하여 store에 액세스 합니다.
다음으로 `.Delete([]byte)` 메소드를 사용하여 store에서 name을 삭제합니다.
store는 `[]byte`만 사용하므로 문자열 형식으로 전달된 함수의 매개변수 `name`는 `[]byte`로 캐스팅 됩니다.

이제 name을 기반으로 store에서 특정 배개 변수를 가져오는 함수를 추가합니다.
store의 getters, setters를 사용하는 대신, `GetWhois`, `SetWhois` 함수를 재사용 합니다.
예를 들어 필드를 설정하려면 먼저 `Whois` 객체 데이터를 가져온 후 특정 필드를 업데이트한 다음 업데이트된 버전의 객체를 store에 저장합니다.

```go
// ResolveName : name으로 조회되는 Whois 객체의 값을 반환합니다
func (k Keeper) ResolveName(ctx sdk.Context, name string) string {
    return k.GetWhois(ctx, name).Value
}

// SetName : name으로 조회되는 Whois객체의 값을 설정합니다
func (k Keeper) SetName(ctx sdk.Context, name string, value string) {
    whois := k.GetWhois(ctx, name)
    whois.Value = value
    k.SetWhois(ctx, name, whois)
}

// HasOwner : name의 소유자가 있는지 여부를 반환합니다
func (k Keeper) HasOwner(ctx sdk.Context, name string) bool {
    return !k.GetWhois(ctx, name).Owner.Empty()
}

// GetOwner : name의 소유자 주소를 반환힙니다
func (k Keeper) GetOwner(ctx sdk.Context, name string) sdk.AccAddress {
    return k.GetWhois(ctx, name).Owner
}

// SetOwner : name의 소유자를 설정합니다
func (k Keeper) SetOwner(ctx sdk.Context, name string, owner sdk.AccAddress) {
    whois := k.GetWhois(ctx, name)
    whois.Owner = owner
    k.SetWhois(ctx, name, whois)
}

// GetPrice : name의 가격을 가져옵니다
func (k Keeper) GetPrice(ctx sdk.Context, name string) sdk.Coins {
    return k.GetWhois(ctx, name).Price
}

// SetPrice : name의 가격을 설정합니다
func (k Keeper) SetPrice(ctx sdk.Context, name string, price sdk.Coins) {
    whois := k.GetWhois(ctx, name)
    whois.Price = price
    k.SetWhois(ctx, name, whois)
}

// IsNamePresent : name이 store에 있는지 확인 합니다
func (k Keeper) IsNamePresent(ctx sdk.Context, name string) bool {
    store := ctx.KVStore(k.storeKey)
    return store.Has([]byte(name))
}
```

SDK에는 store의 특정 지점에 있는 모든 `<Key, Value>` 쌍에 대한 iterator를 반환하는 `sdk.Iterator`라는 기능도 포함되어 있습니다.
store에 있는 모든 name에 대한 iterator를 가져오는 함수를 추가합니다.

```go
// GetNamesIterator : store에 있는 모든 name에 대한 whois 값을 iterator하게 가져옵니다.
func (k Keeper) GetNamesIterator(ctx sdk.Context) sdk.Iterator {
    store := ctx.KVStore(k.storeKey)
    return sdk.KVStorePrefixIterator(store, []byte{})
}
```

`./x/nameservice/keeper/keeper.go` 파일에 필요한 마지막 코드는 `Keeper`의 생성자 함수입니다.

```go
// NewKeeper : nameservice Keeper의 새 인스턴스를 만듭니다
func NewKeeper(cdc *codec.Codec, storeKey sdk.StoreKey, coinKeeper types.BankKeeper) Keeper {
    return Keeper{
        cdc:        cdc,
        storeKey:   storeKey,
        CoinKeeper: coinKeeper,
    }
}
```

다음으로 `Msgs`와 `Handlers`를 사용하여 사용자가 새 store와 상호작용하는 방식을 설명할 차례입니다.
