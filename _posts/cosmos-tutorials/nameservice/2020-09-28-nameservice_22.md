---
layout: post
title:  "Cosmos-SDK nameservice 22"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# go.mod and Makefile

## `Makefile`

프로젝트의 루트 디렉토리에 `./Makefile`을 작성하여 사용자가 애플리케이션을 빌드할 수 있도록 합니다. scaffold tool이 기본적은 makefile을 생성합니다.

> _*NOTE*_: 아래 Makefile에는 Cosmos SDK 및 Tendermint Makefile과 동일한 명령어가 일부 포함되어 있습니다.

[Makefile](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/Makefile)

### Ledger Nano S 지원 기능을 포함하는 방법?

이를 위해서는 몇 가지 작은 수정이 필요합니다.

- 다음과 같이 `Makefile.ledger`파을을 추가합니다.

[Makefile](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/Makefile.ledger)

- Makefile에 `include Makefile.ledger`를 추가합니다

[Makefile](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/Makefile)

## `go.mod`

Golang에는 몇 가지 dependency management tools가 있습니다. 이 튜토리얼에서는 [`Go Modules`](https://github.com/golang/go/wiki/Modules)를 사용합니다.
`Go Modules`는 프로젝트 루트 디렉토리에 있는 `go.mod` 파일을 사용하여 애플리케이션에 필요한 종속성을 정의합니다.
Cosmos SDK 앱은 현재 일부 라이브러리의 특정 버전의 의존합니다. 아래 manifest에는 필요한 모든 버전이 포함되어 있습니다. 시작할려면 `./go.mod` 파일의 내용을 아래 `constraints`와 `overrides` 로 변경합니다.

> _*NOTE*_: 자신의 저장소를 사용하는경운 이를 반영하도록 모듈의 improt 경로를 (`github.com/{ .Username }/{ .Project.Repo }`)와 같이 변경해야 합니다.

- 애플리케이션이 사용하는 모든 모듈을 가져오려면 `go get ./...` 을 실행해야 합니다. 이 명령은 `go.mod` 파일에 명시된 종속성 버전을 가져옵니다.
- 특정 버전의 종속성을 사용하려면 `go get github.com/<github_org>/<repo_name>@<version>`을 실행하면 됩니다.

[go.mod](../go.mod)

## Building the app

```bash
# 앱을 $GOBIN에 설치
make install

# 이제 다음 명령을 실행할 수 있습니다.
nsd help
nscli help
```

### nameservice 개발을 완료 했습니다. 실행하고 동작을 확인해 볼 수 있습니다
