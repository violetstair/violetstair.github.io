---
layout: post
title:  "Cosmos-SDK nameservice 13"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# Queriers

## Query Types

querier type을 정의하기 위해 `./x/nameservice/types/querier.go` 파일을 수정해 보겠습니다.

[querier.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/keeper/querier.go)

## Querier

`./x/nameservice/keeper/querier.go`파일에 사용자가 애플리케이션 상태에 대해 작성할 수 있는 쿼리를 작성하게 됩니다.
`nameservice` 모듈은 세 가지 쿼리를 노출합니다.

- `resolve`: `name`을 가져와 `nameservice`가 저장한 `value`를 반환 합니다. 이것은 DNS query와 유사합니다.
- `whois`: `name`을 받아 name의 `price`, `value`, `owner`를 반환합니다. name을 사고싶을 때 name이 얼마인지 알아내는데 사용됩니다.
- `name` : 매개변수를 취하지 않고 `nameservice` store에 저장된 모든 name을 반환합니다.

이미 정의된 `NewQuerier`를 확인할 수 있을것 입니다. 이 함수는 모듈에 대한 쿼리의 하위 라우터 역할을 합니다(`NewHandler` 함수와 유사합니다.). 쿼리에 대해 `Msg`와 유사한 인터페이스가 없기 때문에 switch문을 수동으로 정의해야 합니다 (`.Route()` 쿼리 함수에서 가져올 수 없음)

```go
package keeper

import (
    "github.com/cosmos/cosmos-sdk/codec"
    "github.com/cosmos/sdk-tutorials/nameservice/x/nameservice/types"

    sdk "github.com/cosmos/cosmos-sdk/types"
    abci "github.com/tendermint/tendermint/abci/types"
)

// nameservice 쿼리에서 지원하는 쿼리 엔드포인트
const (
    QueryResolve = "resolve"
    QueryWhois   = "whois"
    QueryNames   = "names"
)

// NewQuerier : 상태 쿼리를 위한 모듈 레벨 라우터
func NewQuerier(keeper Keeper) sdk.Querier {
    return func(ctx sdk.Context, path []string, req abci.RequestQuery) (res []byte, err error) {
        switch path[0] {
        case QueryResolve:
            return queryResolve(ctx, path[1:], req, keeper)
        case QueryWhois:
            return queryWhois(ctx, path[1:], req, keeper)
        case QueryNames:
            return queryNames(ctx, req, keeper)
        default:
            return nil, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "unknown nameservice query endpoint")
        }
    }
}
```

라우터가 정의되었으므로 각 쿼리에 대한 입력 및 응답을 정의합니다.

```go
func queryResolve(ctx sdk.Context, path []string, req abci.RequestQuery, keeper Keeper) ([]byte, error) {
    value := keeper.ResolveName(ctx, path[0])

    if value == "" {
        return []byte{}, sdkerrors.Wrap(sdkerrors.ErrUnknownRequest, "could not resolve name")
    }

    res, err := codec.MarshalJSONIndent(keeper.cdc, types.QueryResResolve{Value: value})
    if err != nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
    }

    return res, nil
}

func queryWhois(ctx sdk.Context, path []string, req abci.RequestQuery, keeper Keeper) ([]byte, error) {
    whois := keeper.GetWhois(ctx, path[0])

    res, err := codec.MarshalJSONIndent(keeper.cdc, whois)
    if err != nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
    }

    return res, nil
}

func queryNames(ctx sdk.Context, req abci.RequestQuery, keeper Keeper) ([]byte, error) {
    var namesList types.QueryResNames

    iterator := keeper.GetNamesIterator(ctx)

    for ; iterator.Valid(); iterator.Next() {
        namesList = append(namesList, string(iterator.Key()))
    }

    res, err := codec.MarshalJSONIndent(keeper.cdc, namesList)
    if err != nil {
        return nil, sdkerrors.Wrap(sdkerrors.ErrJSONMarshal, err.Error())
    }

    return res, nil
}
```

위 코드에 대한 참고사항:

- 여기서 `Keeper`의 getter와 setter가 많이 사용됩니다. 이 모듈을 사용하는 다른 애플리케이션을 빌드할 때 필요한 상태 조각에 액세스 하려면 돌아가서 더 많은 getter/setter를 정의해야 할 수 있습니다.
- 관례에 따라 각 출력 유형은 JSON marshal 가능하고 문자열로(Golang의 `fmt.Stringer` 인터페이스 구현) 반환 가능해야 합니다. 반환된 바이트는 출력결과의 JSON 인코딩 이어야 합니다.
  - 따라서 `resolve` 출력 type의 경우 JSON marshal 가능하고 `.String()` 매서드가 있는 `QueryResResolve`라는 구조체에 문자열을 래핑합니다.
  - Whois의 출력을 위해 일반 Whois 구조체는 이미 JSON marshal 가능하지만 `.String()` 메서드를 추가해야 합니다.
  - name 쿼리의 출력과 동일하게 `[]string`은 이미 기본적으로 marshal 가능하지만 `.String()` 메서드를 추가해야 합니다.
- Whois 타입은 `./x/nameservice/types/querier.go` 파일에 정의되어 있지 않습니다. 앞서 `./x/nameservice/types/types.go` 파일에 정의해놨기 때문입니다.

### 모듈 상태를 변경하고 볼 수 있는 방법이 생겼으므로 이제부터 마무지 작업을 할 차계입니다. 모듈의 최상위 레벨로 가져오려는 변수와 type을 정의합니다.
