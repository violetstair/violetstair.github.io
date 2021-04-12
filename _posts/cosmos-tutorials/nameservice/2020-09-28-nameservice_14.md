---
layout: post
title:  "Cosmos-SDK nameservice 14"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Alias

`./x/nameservice/alias.go` 파일로 이동하여 시작합니다.
이 파일의 주요 목적은 순환 참조를 방지하기 위한 것 입니다.
순환 참조에 대한 자세한 내용은 여기서([Golang 순환 참조](https://stackoverflow.com/questions/28256923/import-cycle-not-allowed)) 확인할 수 있습니다.

먼저 생성한 "types" 폴더를 가져와 시작합니다.

## alias.go 파일에 세 종류의 type을 생성합니다

- constants(상수) : 불변하는 변수의 선언
- variables(변수) : 세미지와 같은 정보를 포함하는 정의
- type : type 폴더에서 생성한 type 정의

[alias.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/alias.go)

이제 필요한 constants, variables, types을 지정했으니 모듈 생성을 진행할 수 있습니다.

다음으로 Amino 인코딩 형식으로 type을 등록합니다.
