---
layout: post
title:  "Golang Chapter 1"
author: violetstair
categories: [ Golang ]
tags: [golang, programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 1 : Golang 소개"
featured: false
hidden: true
---

## Golang의 특징

### 1. 개발 효율성

* `Python`, `Javascript`와 같은 인터프리터 언어의 개발 생산성
* `C`, `JAVA`와 같은 컴파일러 언어의 타입 안전성
* `GC`를 개발자가 관리하지 않아도 된다

### 2. 동시성

스레드와 유사하지만 더 작은 코드, 더 작은 메모리를 이용한다

#### 고루틴

* 하나의 스레드에 여러개의 고루틴이 동작할 수 있다
* 함수 앞에 `go` 키워드를 이용해 고루틴으로 동작시킬 수 있다
* 실행에 대한 오버헤드가 적어 성능에 영향이 적다

#### 채널

* 고루틴 간에 안전하게 데이터 전송을 가능하게 하는 데이터 구조
* 공유 메모리를 사용하지 않는다
  * 같은 변수를 사용하는 문제가 발생하지 않음
  * 위와 같은 문제 때문에 사용하는 `lock`을 사용하지 않음

### 3. 타입 시스템

* 컴파일언어의 특징인 강한 타입 언어
* 객체지향 언어이지만 `JAVA`의 추상화 클래스, 인터페이스와는 다른 합성 디자인 패턴
* `JAVA`에서 인터페이스는 다른 구조를 가짐
  * `JAVA`는 객체간 상속 관계를 가지며 인터페이스는 객체(여러 동작)를 정의한다
  * `Golang`는 상속 관계를 가지지 않으며 인터페이스는 하나의 동작만 정의
* `C`에서 사용하는 `struct`와 유사한 `type`을 사용할 수 있다

#### 덕 타이핑(Duck Typing)

`Golang`은 `OOP`와는 조금 다른 `Duck typing` 프로그래밍을 지향한다

* 사람이 오리처럼 행동하면 오리로 봐도 무방하다는 개념
* `JAVA`와 달리 모든 객체 대한 정의를 사전에 정의할 필요가 없다
* `JAVA`는 `object`와 `interface`로 이루어지고 `Golang`은 `type`으로 이루어진다

### 4. 메모리 관리

* 메모리 관리를 개발자가 하지 않아도 된다
* `JAVA`처럼 메모리 압축은 하지 않음
* 런타임에 `TCMalloc` 방식으로 메모리 관리
  * TCMalloc
    * 구글에서 만든 메모리 관리 방식
    * 빠른 메모리 할당과 메모리 단편화 감소
    * malloc을 호출하는 것보다 속도가 빠름
    * 메모리를 4k 페이지 단위로 관리
      * `type` 선언에 따라 메모리 사용량도 달라짐

### 5. Golang의 단점

* 패키지 관리의 문제점
  * 패키지 `import` == `repository url`
* 구글이 만듬
  * 구글 스타일의 Coding convention : 코드 가독성은 ...
    * 변수는 `snake_case` or `camelCase`, Public object는 `PascalCase`, Private object는 `camelCase`

## Golang 스타일 가이드라인

### 패키지 디렉터리 구조

#### `/cmd`

프로젝트의 메인 어플리케이션이 위치한다

어플리케이션 디렉터리명은 실행하려는 실행 파일의 이름과 같아야 한다. (e.g. `myapp` : `/cmd/myapp/main.go`)
어플리케이션 디렉터리는 최소한의 코드를 포함해야 하며 비지니스 로직은 `/pkg` 디렉터리에 포함하여야 한다.
`/cmd`에는 `internal`과 `/pkg`를 임포트, 호출하는 작은 `main` 함수를 작한다.

#### `/internal`

개인적인 어플리케이션과 라이브러리 코드가 위치한다.
재사용 되지 않는 코드, 다른 라이브러리에서 사용하지 않는 코드를 작성하는 디렉터리

#### `/pkg`

외부 어플리케이션에서 사용되어도 괜찮은 라이브러리 코드, 대부분의 비지니스 로직이 위치한다.

#### `/vendor`

패키지 종성석 관리하는 디렉터리, `go mod`를 통해 패키지를 관리할 때 자동으로 생성해 주기도 한다.
저장소에는 포함시킬 필요가 없다.
(e.g. `node`의 `node_module`과 유사)

### 서비스 어플리케이션 디렉터리

#### `/api`

`OpenAPI` / `Swagger` 스펙과 JSON Schema, Protocol 정의 파일들이 위치한다

### 웹 어플리케이션 디렉터리

#### `/web`

웹 어플리케이션의 컴포넌트, Static Web Asset, Server side template, SAP 등이 위치한다

### 공통 어플리케이션 디렉터리

#### `/configs`

설정 파일 또는 설정 파일의 템플릿 들이 위치한다

#### `/init`

시스템 init(systemd, upstart, sysv)와 프로세스 매니지/슈퍼바이저 (runit, supervisord) 설정

#### `/scripts`

빌드, 설치, 분석 등을 위한 스크립트가 위치한다
루트 디렉터리의 `Makefile`이 실행할 스크립트가 위치하며, 이는 `Makefile`를 작고 간단하게 유지할 수 있게 한다

#### `/build`

CI 설정(travis, circle, drone 등), AMI, Docker, OS 패키지 설정과 스크립트 등 CI 관련된 설정을 포함한다

#### `/deployments`

Iaas, PaaS, 시스템, 컨테이너 오케스트레이션 배포 설정과 템플릿 등의 설정 파일을 포함한다.
(e.g. docker-compose, kubernetes, mesos, terraform 등)
쿠버네티스로 배포 되는 앱들은 `/deploy`로 설정하기도 한다.

#### `/test`

외부 테스트 앱 설정과 테스트 데이터 등이 위치한다.

### 그외 디렉터리

#### `/docs`

디자인과 사용자 문서들이 위치한다.

#### `/tools`

프로젝트에서 사용하는 도구들, `/pkg`나 `/internal`에서 임포트 할 수 있는 코드가 위치한다.

#### `/examples`

라이브러리 패키지의 사용 예제 코드가 위치한다.

#### `/third_party`

외부 도구들의 포크된 코드나 3rd party 유틸리티 등이 위치한다 (e.g. Swagger UI)

#### `/githooks`

Git Hook 설정 파일

#### `/assets`

repository에 사용될 asset 파일(이미지, 로고 등)

#### `/website`

github pages를 위한 데이터

### 안티 패턴

#### `/src`

`golang` 워크스페이스에서 패키지 소스코드가 포함되는 `src`이외의 `src` 디렉터리를 사용하는걸 권장하지 않는다.
