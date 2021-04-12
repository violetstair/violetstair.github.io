---
layout: post
title:  "Cosmos-SDK nameservice 03"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Start your application

## application

새 앱을 만들어 시작하려면 scaffold tool에서 제공하는 lvl-1 앱을 사용할 것 입니다.

`user`는 Github 사용자 이름으로, `repo`는 생성중인 저장소의 이름으로 채울 수 있습니다.

```bash
scaffold app lvl-1 [user] [repo] [flags]

cd [repo]
```

`app.go`에서 애플리케이션이 트랜잭션을 수신할 때 수행하는 작ㅇ버을 정의합니다.
그러나 먼저 올바른 순서로 거래를 받을 수 있어야 합니다.
이것인 [Tendermint consensus engine](https://github.com/tendermint/tendermint)의 역할 입니다.

가져온 각 모듈 및 패키지에 대한 godocs 링크:

- [`log`](https://godoc.org/github.com/tendermint/tendermint/libs/log): Tendermint의 로거
- [`auth`](https://godoc.org/github.com/cosmos/cosmos-sdk/x/auth): the Cosmos SDK의 `auth` 모듈
- [`dbm`](https://godoc.org/github.com/tendermint/tm-db): Tendermint 데이터베이스 작업을 위한 코드
- [`baseapp`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp): 아래 참조

여기에 몇 가지 패키지가 `tendermint` 패키지 입니다.
Tendermint는 [ABCI](https://docs.tendermint.com/master/spec/abci/)라는 인터페이스를 통해 네트워크에서 애플리케이션으로 트랜잭션을 전달합니다.
구축중인 블록체인 노드의 아키텍쳐를 보면 다음과 같습니다.

```text
+---------------------+
|                     |
|     Application     |
|                     |
+--------+---+--------+
         ^   |
         |   | ABCI
         |   v
+--------+---+--------+
|                     |
|                     |
|     Tendermint      |
|                     |
|                     |
+---------------------+
```

다행히 개발자는 ABCI 인터페이스를 구현할 필요는 없습니다.
Cosmos SDK는 [`baseapp`](https://godoc.org/github.com/cosmos/cosmos-sdk/baseapp) 형식으로 boilerplate를 제공합니다.

`baseapp`가 하는 역할은 다음과 같습니다:

- Tendermint consensus engine에서 받은 트랜잭션을 디코딩 합니다.
- 트랜잭션에서 메시지를 추출하고 기본적인 타당성 검토를 진행합니다.
- 메시지를 처리할 수 있도록 적절한 모듈로 라우트 합니다. `baseapp`에는 사용하려는 특정 모듈에 대한 지식이 없습니다. `app.go`에 이러한 모듈에 대해 선언하는 것은 개발자의 역할이다(튜토리얼 뒷 부분에서 확인 가능합니다.) `baseapp`은 모든 모듈에 적용할 수 있는 핵심 라우팅 로직만 구현합니다.
- ABCI 메시지가 [`DeliverTx`](https://docs.tendermint.com/master/spec/abci/abci.html#delivertx) 이면 commit 합니다. ([`CheckTx`](https://docs.tendermint.com/master/spec/abci/abci.html#checktx) 변경 사항은 영구적이지 않습니다.).
- 각 블록의 시작과 끝에서 실행되는 로직을 정의할 수 있는 두 개의 메시지인 [`Beginblock`](https://docs.tendermint.com/master/spec/abci/abci.html#beginblock)과 [`Endblock`](https://docs.tendermint.com/master/spec/abci/abci.html#endblock)의 설정을 지원합니다. 실제로 각 모듈은 `BeginBlock`와 `EndBlock` 하위 로직을 구현하고 앱의 역할은 모든 것을 함께 집계하는 것 입니다. (_참고: 애플리케이션에서 이러한 메시지를 사용하지 않습니다._)
- 상태 초기화를 지원합니다.
- 쿼리 설정을 지원합니다.

이제 `appName` & `NewApp` types의 이름을 앱 이름인 `nameservice` & `NameServiceApp`로 변경합니다.
이 type은 `baseapp`을 포함합니다 (다른 언어의 상속과 유사한 Go의 embedding). 즉 `baseapp`의 모든 메소드에 액에스할 수 있습니다.

이제 애플리케이션을 시작할 수 있습니다.
현재 작동중인 블록체인을 튜토리얼 전반에 걸쳐 우리가 원하는대로 기능을 추가할 것입니다.

`baseapp`은 애플리케이션에서 사용하려는 경로 또는 사용자 인터페이스에 대한 지식이 없습니다.
애플리케이션의 주요 역할은 이러한 경로를 정의하는 것 입니다. 또 다른 역할은 초기 상태를 정의하는 것 입니다. 이 두 가지 모두 애플리케이션에 모듈을 추가해야 합니다.

[application design](./app-design.md) 색션에서 확인할 수 있듯이 nameservice를 위한 몇 가지 모듈이 필요합니다.: `auth`, `bank`, `staking`, `distribution`, `slashing`, `nameservice`.
처음 5개는 이미 존재하지만 `nameservice` 모듈은 개발자가 개발해야할 state-machine 입니다.
다음 단계에서 본격적으로 개발을 진행해 볼 것입니다.

### In order to complete your application, you need to include modules. Go ahead and start building your nameservice module. You will come back to `app.go` later
