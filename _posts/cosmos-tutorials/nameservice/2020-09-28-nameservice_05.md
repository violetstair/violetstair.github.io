---
layout: post
title:  "Cosmos-SDK nameservice 05"
author: violetstair
categories: [ Cosmos-SDK-Tutorial ]
tags: [golang, programming, cosmos-sdk]
image: assets/images/cosmos-sdk.png
description: "COSMOS SDK로 만들어보는 nameservice"
featured: false
hidden: false
---

# Key

types 폴더에 있는 `key.go` 파일 번저 수정해 보겠습니다.

`key.go` 파일 내에서 모듈을 만드는 동안 사용될 key가 이미 생성된 것을 볼 수 있습니다.
애플리케이션 전체에서 사용할 key를 정의하면 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 코드를 작성하는데 도움이 됩니다.

[key.go](https://github.com/cosmos/sdk-tutorials/blob/master/nameservice/x/nameservice/types/key.go)

```go

package types

const (
    // ModuleName : 모듈의 이름
    ModuleName = "nameservice"

    // StoreKey : KVStore 생성시 사용할 StoreKey
    StoreKey = ModuleName

    // RouterKey : name 모듈의 라우터 키
    RouterKey = ModuleName

    // QuerierRoute : 메시지 쿼리에 사용할 쿼리 라우트
    QuerierRoute = ModuleName
)
```
