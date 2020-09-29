---
layout: post
title:  "Cosmos-SDK nameservice 17"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# NameService Module Rest Interface

모듈은 HTTP 핸들러를 정의하여 REST 인터페이스를 노출하여 모듈의 기능에 프로그래밍 방식으로 액세스 할 수 있으며,
`./x/nameservice/client/rest/rest.go`파일에 `imports`와 `const`를 추가하여 시작할 수 있습니다.

> _*NOTE*_: 애플리케이션은 앞서 작성한 코드를 가져와야 합니다. 여기서 import 경로는 이 저장소(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`)로 설정되며, 자신의 저장소를 사용하는 경우 (`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`)와 같이 import 경로를 변경해야 합니다.

```go
package rest

import (
    "fmt"

    "github.com/cosmos/cosmos-sdk/client/context"

    "github.com/gorilla/mux"
)

const (
    restName = "name"
)
```

## RegisterRoutes

먼저 `RegisterRoutes` 함수에서 모듈에 대한 REST 클라이언트 인터페이스를 정의합니다.
다른 모듈의 경로와 namespace 충돌 방지를 위해 모든 경로는 모듈의 이름으로 시작하는게 좋습니다.

```go
// RegisterRoutes - 메인 애플리케이션에서 등록할 경로를 정의하는 통합기능
func RegisterRoutes(cliCtx context.CLIContext, r *mux.Router, storeName string) {
    r.HandleFunc(fmt.Sprintf("/%s/names", storeName), namesHandler(cliCtx, storeName)).Methods("GET")
    r.HandleFunc(fmt.Sprintf("/%s/names", storeName), buyNameHandler(cliCtx)).Methods("POST")
    r.HandleFunc(fmt.Sprintf("/%s/names", storeName), setNameHandler(cliCtx)).Methods("PUT")
    r.HandleFunc(fmt.Sprintf("/%s/names/{%s}", storeName, restName), resolveNameHandler(cliCtx, storeName)).Methods("GET")
    r.HandleFunc(fmt.Sprintf("/%s/names/{%s}/whois", storeName, restName), whoIsHandler(cliCtx, storeName)).Methods("GET")
    r.HandleFunc(fmt.Sprintf("/%s/names", storeName), deleteNameHandler(cliCtx)).Methods("DELETE")
}
```

## Query Handlers

먼저 모든 쿼리를 저장할 `query.go` 파일을 만듭니다

다음으로 위에서 언급한 핸들러를 정의합니다. 이는 이전에 정의한 CLI 메소드와 매우 유사합니다. `whois`와 `resolve` 쿼리 부터 만들어 봅시다.

[query.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/client/rest/query.go)

위 코드에 대한 설명:

- 데이터를 가져오기 위해 동일한 `cliCtx.QueryWithData` 함수를 사용합니다.
- 이러한 기능은 해당 CLI 기능과 거의 동일합니다.

## Tx Handlers

먼저 모든 tx rest 엔드포인트를 작성할 `tx.go` 파일을 정의합니다.

이제 `buyName`, `setName`, `deleteName` 트랜잭션 경로를 정의합니다.이들은 실제로 이름을 구힙, 설정 및 삭제하기 위해 트랜잭션을 보내는 것이 아닙니다. 보안 문제가될 요청과 함께 암호를 보내야 합니다. 대신 이러한 엔드포인트는 각각의 특정 트랜잭션을 빌드하고 반환하여 안전한 방식으로 서명한 후 `/txs`와 같은 표준 엔드포인트를 사용하여 네트워크에 브로드캐스트 할 수 있습니다.

[tx.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/client/rest/tx.go)

위 코드에 대한 설명:

- [`BaseReq`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/utils#BaseReq)에는 트랜잭션을 만드는데 필요한 기본 필드(사용할 key, decode 방법, 사용중인 chain, 등...)가 포함되어 있으며 그림과 같이 포함되도록 설계 되었습니다.
- `baseReq.ValidateBasic`는 응답코드 설정을 처리하므로 이러한 삼수를 사용할 때 오류 또는 성공 처리에 대해 걱정할 필요가 없습니다.

## 다음에는 AppModule 인터페이스를 구현하여 `nameservice`를 보강할 차례입니다
