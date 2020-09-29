---
layout: post
title:  "Cosmos-SDK nameservice 16"
author: violetstair
categories: [ CosmosSDK ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: true
---

# Nameservice Module CLI

Cosmos SDK는 CLI를 위해 [`cobra`](https://github.com/spf13/cobra) 라이브러리를 사용합니다.
이 라이브러리를 사용하면 각 모듈의 자체 명령을 쉽게 노출할 수 있습니다.
사용자와 모듈간의 상호작용을 CLI에 대해 정의하려면, 다음 파일을 생성해 시작할 수 있습니다.

- `./x/nameservice/client/cli/query.go`
- `./x/nameservice/client/cli/tx.go`

## Queries

Start in `query.go`. Here, define `cobra.Command`s for each of your modules `Queriers` (`resolve`, and `whois`):

[query.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/client/cli/query.go)

위 코드에 대한 참고사항:

- CLI는 새로운 `context`를 도입합니다: [`CLIContext`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/context#CLIContext). CLI에 필요한 사용자 입력 및 애플리케이션 구성에 대한 데이터를 전달합니다.
- `cliCtx.QueryWithData()` 함수에 필요한 `path`는 쿼리 라우터의 이름에 직접 매핑합니다.
  - 경로의 첫 번째 부분은 SDK 애플리케이션에 가능한 쿼리 유형을 구분하는데 사용됩니다. `custom`은 `Queriers`를 위한것 입니다.
  - 두 번째 부분(`nameservice`)은 쿼리를 라우팅할 모듈의 이름입니다.
  - 마지막으로 후촐될 모듈의 특정한 querier아 위치합니다.
  - 이 예에서 네 번째 부분은 쿼리 입니다. 이것은 쿼리 매개변수가 간단한 문자열이기 때문에 동작합니다. 더 복잡한 쿼리 입력을 사용하려면 [`.QueryWithData()`](https://godoc.org/github.com/cosmos/cosmos-sdk/client/context#CLIContext.QueryWithData) 함수의 두 번째 인수를 사용하여 `data`를 전달해야 합니다. 이에 대한 예제는 [queriers in the Staking module](https://github.com/cosmos/cosmos-sdk/blob/5af6bd77aa6c0e8facc936947a3365416892e44d/x/staking/keeper/querier.go)를 참조하면 됩니다.

## Transactions

이제 쿼리 인터페이스가 정의되었으므로 `tx.go`에서 트랜잭션으로 넘어갈 차례입니다.

> _*NOTE*_: 애플리케이션은 방금 작성한 코드를 가져와야 합니다. 여기서 가져오기 경로는 이 저장소(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`)로 설정 됩니다. 자신의 저장소에서 작업하는 경우 이를 반영하도록 import 경로를 다음과 같이 변경해야 합니다. (`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

[tx.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/client/cli/tx.go)

위 코드에 대한 참고사항:

- 여기서는 `authcmd` 패키지가 사용됩니다. [authcmd에 대한 godocs](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth/client/cli#GetAccountDecoder). CLI로 제어되는 계정에 대한 맥세스를 제공하고 서명을 용이하게 합니다.

### 이제 REST 클라이언트가 모듈과 통신하는데 사용할 경로를 정의할 준비가 되었습니다
