---
layout: post
title:  "Cosmos-SDK nameservice 21"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Entry points

Golang에서는 바이너리로 컴파일 되는 파일을 프로젝트의 `./cmd` 폴더에 배치하게 됩니다.
이 애플리케이션의 경우 2개의 바이너리를 생성해야 합니다.

- `nsd`: 이 바이너리는 P2P 연결을 유지하고, 트랜잭션 전파, 로컬 스토리지 처리, 네트워크와 상호작용하는 RPC 인터페이스 제공 등의 기능을 가지며, `bitcoind` 또는 기타 암호화 화폐의 데몬과 유사한 기능을 가집니다. 이 경우 Tendermint는 네트워킹 및 트랜잭션 실행에 사용됩니다.
- `nscli`: 이 바이너리는 사용자가 애플리케이션과 사호작용을 할 수 있는 명령을 제공합니다

바이너리를 인스턴스화 할 프로젝트 디렉토리에 두개의 파일을 만들어 시작합니다

- `./cmd/nsd/main.go`
- `./cmd/nscli/main.go`

## `nsd`

먼저 `cmd/nsd/main.go`에 다음과 같은 코드를 추가합니다.

> _*NOTE*_: 애플리케이션은 방금 작성한 코드를 가져와야 합니다. 여기서 가져오기 경로는 이 저장소(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`)로 설정 됩니다. 자신의 저장소에서 작업하는 경우 이를 반영하도록 import 경로를 다음과 같이 변경해야 합니다. (`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

[main.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/cmd/nsd/main.go)

위 코드에 대한 설명:

- 위 코드의 대부분은 Tendermint, Cosmos SDK 및 Nameservice 모듈의 CLI 명령을 결합합니다.

## `nscli`

`nscli` 명령을 빌드하여 완료 합니다.

> _*NOTE*_: 애플리케이션은 방금 작성한 코드를 가져와야 합니다. 여기서 가져오기 경로는 이 저장소(`github.com/cosmos/sdk-tutorials/nameservice/x/nameservice`)로 설정 됩니다. 자신의 저장소에서 작업하는 경우 이를 반영하도록 import 경로를 다음과 같이 변경해야 합니다. (`github.com/{ .Username }/{ .Project.Repo }/x/nameservice`).

[main.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/cmd/nscli/main.go)

Note:

- 이 코드는 Tendermint, Cosmos SDK 및 Nameservice 모듈의 CLI 명령을 결합합니다.
- [`cobra` CLI 문서](http://github.com/spf13/cobra)는 위 코드를 이해하는데 도움이 될 것입니다.
- 앞서 정의한 `ModuleClient`를 여기에서 확인할 수 있습니다.
- 경로가 `registerRoutes` 함수에 어떻게 포함되었는지 확인할 수 있습니다.

### 이제 바이너리에 종속성 관리를 정의하고 앱을 빌드할 시간이 되었습니다
