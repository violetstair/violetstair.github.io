---
layout: post
title:  "Golang Chapter 8"
author: violetstair
categories: [ Golang ]
tags: [golang, programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 8 : 표준 라이브러리"
featured: false
hidden: true
---

`Golang`은 표준 라이브러리에 대한 의존도가 높은 편이며, 하위 호환성이 일정 수준 이상 보장되므로 버전에 따른 이슈가 상대적으로 적다.

[표준 라이브러리 정보](https://golang.org/pkg/)

표준 라이브러리는 `$GOROOT/src/pkg` 폴더에 위치하며, `Golang`의 배포 패키지의 일부로 이미 컴파일되어 있다. 이 컴파일된 파일들을 archive files라고 하며 `.a` 확장자를 가진다.

## 로깅

기본적으로 `log` 패키지에서 로그의 출력은 `fmt` 패키지의 `Print`를 사용하는 방식과 유사하다

```go
func init() {
    log.SetPrefix("추적 : ")
    log.SetFlags(log.Ldate | log.Lmicroseconds | log.Llongfile)
}

func main() {
    log.Println("로그 메시지")

    // Fatalln은 Println을 실행 후 os.Exit(1)을 추가로 호출한다
    log.Fatalln("치명적 오류")

    // Panicln은 Println을 실행 후 panic()을 추가로 후출한다
    log.Panicln("패닉 메시지")
}
```

`init` 함수를 통해 현재 프로세스에서 사용 할 로그의 형태를 지정할 수 있다.
또한 로그의 플래그를 정의할 때 파이프 연산자(`|`)를 통해 여러 플래그를 조합하여 정의할 수 있다.

`panic`은 현재 실행 중인 함수를 즉시 종료 한다. 이때 `panic`이 발생하면 즉시 모든 `defer`를 실행 후 즉시 `return`을 실행한다. `panic`은 상위 함수에도 동일하게 적용되며 프로그램 에러를 반환하고 종료된다. `panic`에 의한 상태를 정상 상태로 되돌리는데 `recover`를 사용한다.

### 사용자 로거 정의

```go
import (
    "io"
    "io/ioutil"
    "log"
    "os"
)

var (
    Info    *log.Logger // 중요 정보
    Warning *log.Logger // 경고
    Error   *log.Logger // 치명적 오류
    Trace   *log.Logger // 기타
)

func init() {
    file, err := os.OpenFile("error.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalln("에러 로그 파일을 열 수 없습니다.", err)
    }

    Info = log.New(os.Stdout, "info: ", log.Ldate|log.Ltime|log.Lshortfile)
    Warning = log.New(os.Stdout, "warning : ", log.Ldate|log.Ltime|log.Lshortfile)
    Error = log.New(io.MultiWriter(file, os.Stderr), "error : ", log.Ldate|log.Ltime|log.Lshortfile)
    Trace = log.New(ioutil.Discard, "trace: ", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
    Info.Println("특별한 정보를 위한 로그 메시지")
    Warning.Println("경고성 로그 메시지")
    Error.Println("에러 로그 메시지")
    Trace.Println("일반적인 로그 메시지")
}
```

## 인코딩 디코딩

### JSON 데이터 다루기

```go
package main

import (
    "encoding/json"
    "fmt"
)

// json 데이터를 매핑할 구조체
type Student struct {
    Name  string `json:"name"`
    Age   int    `json:"age"`
    Class string `json:"class"`
}

func main() {
    jane := Student{
        Name:  "jane",
        Age:   22,
        Class: "CS",
    }
    var jane2 map[string]interface{}

    // JSON 인코딩
    jsonBytes, err := json.Marshal(jane)
    if err != nil {
        panic(err)
    }

    // JSON 바이트를 문자열로 변경
    jsonString := string(jsonBytes)

    fmt.Println(jsonString)

    // map[string]interface에 디코딩
    err = json.Unmarshal(jsonBytes, &jane2)
    if err != nil {
        panic(err)
    }

    fmt.Println(jane2["name"], jane2["age"], jane2["class"])
}
```

구조체 선언에서 각 필드위에 붙은 문자열들은 `tag`라고 하면 `JSON` 문서와 구조체 타입간 필드 매핑을 위한 메타데이터이다.
`JSON` 데이터에 유현한 접근을 위해 `map[]interface{}`로 선언된 변수에 디코딩(`Unmarshal`)할 수 있는데 이때는 구조체 데이터와 다르게 핸들링 한다
