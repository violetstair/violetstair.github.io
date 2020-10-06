---
layout: post
title:  "Golang Chapter 2"
author: violetstair
categories: [ Golang ]
tags: [golang, programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 2 : Golang 간단히 살펴보기"
featured: false
hidden: true
---

## Golang 설치하기

[Golang 설치 문서](https://golang.org/doc/install)

## Golang 환경 설정

```bash
> go env

GOARCH="amd64"
GOOS="darwin"
GOPATH="/Users/violetstair/workspace/go"
GOROOT="/usr/local/opt/go/libexec"
```

* GOARCH : `Golang`이 빌드될 때 사용하는 아키텍쳐
* GOOS : `Golang`이 빌드될 때 사용하는 시스템
* GOPATH : 프로젝트 진행 위치 (Workspace, library, build file, package, 등...)
* GOROOT : `Golang`이 설치된 위치, `JAVA_HOME`과 유사한 개념

## 패키지

`Golang`은 패키지라는 단위로 프로그램 그룹을 관리한다

프로그램의 시작은 `main` 패키지의 `main` 함수를 시작으로 동작하게 되며, `main` 함수는 `main.go` 파일에 존재하게 된다.
프로그램의 진입점이 되는 시작 파일이 `main.go`일 필요는 없으나 명시성을 위해 일반적으로 `main.go`로 작성한다.
`golang`에서는 패키지의 사이즈를 작게 유지할 것을 권장한다.

## 코드의 구성

```go
// 패키지 이름을 정의한다.
// 주석과 공백을 제외하고 코드의 첫 라인은 패키지 이름을 명시하는 코드가 와야 한다.
package main

// 사용할 라이브러리를 import 한다.
// import 하는 라이브러리가 여러개 일 경우 괄호로 묶어준다
import (
    // 기본 라이브러리
    "log"
    "os"

    // 원격 라이브러리
    "github.com/violetstair/study-golang/pkg/hello"
    // 패키지 alias
    sdk "github.com/violetstair/study-golang/pkg/hello/sdk"
    // 코드에서 사용하진 않지만 컴파일에 포함할 라이브러리를 익명 할당
    _ "github.com/violetstair/study-golang/pkg/hello/driver"
)

// init 함수는 main 함수보다 먼저 호출된다.
func init() {
    // 표준 출력으로 로그를 출력하도록 변경한다.
    log.SetOutput(os.Stdout)
}

// main 함수는 프로그램의 진입점이다.
func main() {
    // sdk 패키지의 Message 함수를 실행한다.
    msg := sdk.Message()
    hello.World(msg)
}
```

## 내장 도구

### `go get`

패키지 다운로드, `git clone`과 동작 방식은 유사하며 `$GOPATH`에 패키지를 다운 받는다
패키지를 다운로드 받은 이후 `go install`을 통해 패키지를 설치한다

### `go env`

`golang`의 환경 설정 정보를 볼 수 있다

```bash
go env
```

### `go build`

패키지를 컴파일 한다.

```bash
go build [package path]
go build [source_code].go
```

### `go clean`

빌드된 파일을 정리 한다

```bash
go clean
```

### `go run`

소스 파일을 빌드 후 코드를 실행한다
하나의 파일의 실행을 확인할 때 혹은 인터프리터언어의 코드처럼 실행하기 원할 때 사용한다

```bash
go run [source_code].go
```

### `go install`

패키지를 빌드 후 `go env`의 `bin` 디렉터리에 저장한다

```bash
go install [package path]
```

## 개발 관련 도구

### lint 툴

#### `go vet`

code lint, code fomatting 등 간단한 오류 검사 확인 도구

```bash
go vet [package path]
```

#### `golint`

코드를 좀더 `golang` 스럽게 확이해 주는 도구

```bash
# 설치
go get -u golang.org/x/lint/golint

golint [package path]
```

#### `go fmt`

코드 들여쓰기, 괄호나 탭/스페이스 정리 등 소스코드를 일관성 있게 정리해주는 역할을 한다.

```bash
go fmt [package path]
```

### 의존성 관리 도구

#### `dep`

`golang`의 의존성 관리도구

설치 방법

```bash
go get -u github.com/golang/dep/cmd/dep
```

homebrew를 이용한 설치

```bash
brew upgrade dep
```

프로젝트 초기화

```bash
dep init

ls
./Gopkg.lock
./Gopkg.toml
./vendor/
```

* `Gopkg.toml` : dep을 이용한 의존성 관리에 대한 설정 파일
* `Gopkg.lock` : dep을 이용한 의존성 관리에 대한 스냅샷 `dep ensure`를 통해 변경할 수 있다
* `vendor/` : dep을 이용한 의존성 패키지의 소스코드가 다운로드 되는 위치

의존성 패키지 추가

```bash
dep ensure -add github.com/violetstair/packages
```

의존성 관리

```bash
dep ensure
```

#### `module`

`Golang` 1.11 버전 이후에 추가된 공식 의존성 관리도구

go module 초기화

```bash
go mod init

ls
./go.mod
```

* `go.mod` : 의존성 관리를 위한 설정 파일

`dep`와는 다르게 `golang` 커맨드(get, test, build, run 등)를 실행할 때 소스코드에 import된 패키지를 확인하고 관리한다

```bash
go test

ls
./go.mod
./go.sum
```

* `go test`를 실행 후 `go.sum` 파일이 생긴것을 확인할 수 있는데 `go.sum` 파일은 의존성 관리를 위한 스냅샷 이다.
* 의존성 패키지 목록은 `go list -m all`로도 확인할 수 있다.
