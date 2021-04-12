---
layout: post
title:  "Cosmos-SDK nameservice 15"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Codec File

인코딩/디코딩 할 수 있도록 [Amino에 type 등록](https://github.com/tendermint/go-amino#registering-types)하려면 `./x/nameservice/types/codec.go`에 배치해야하는 약간의 코드가 필요합니다.
생성한 모든 인터페이스와 인터페이스를 구현하는 구조체는 `RegisterCodec` 함수에서 선언해야 합니다.
이 모듈에서는 세 가지 `Msg` 규현(`SetName`, `BuyName`, `DeleteName`)을 등록해야 하지만 `Whois` 쿼리의 반환 유형은 등록하지 않습니다.
또한 나중에 사용할 모듈별 코덱을 정의합니다.

[codec.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/types/codec.go)

## 다음으로 모듈과의 CLI 상호작용을 정의해야 합니다
