---
layout: post
title:  "Golang Chapter 5"
author: violetstair
categories: [ Golang ]
tags: [Golang, Programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 5 : Go의 Type 시스템"
featured: true
hidden: true
---

`golang`는 정적 타입 언어로, 컴파일러가 프로그램 내의 모든 값의 `type`을 정확히 알고 있어야 한다.
내장 타입 이외에 사용자 정의 타입을 `struct` 키워드로 정의할 수 있으며, 새로운 타입을 선언할 때는 내장 타입과 마찬가지로
컴파일러에게 정보의 크기와 표현 방법을 알려줘야 한다.

## 사용자 정의 Type

### `struct`

```go
// user type 정의
type user struct {
    name       string
    email      string
    age        int
    privileged bool
}

// user type 선언과 값 입력
var rose user
rose.name = "rose"
rose.email = "rose@black.pink"
rose.age = 23
rose.privileged = true

// struct 리터럴을 이용한 user type 초기화
lisa := user{
    name:       "lisa",
    email:      "lisa@black.pink",
    age:        23,
    privileged: true,
}

// 필드 이름 없이 user type 초기화
jisu := user{"jisu", "jisu@black.pink", 23, true}

// struct를 포함하는 struct type
type admin struct {
    person user
    level  string
}

// admin type의 초기화
jenny := admin{
    person: user{
        name:       "jenny",
        email:      "jenny@black.pink",
        age:        23,
        privileged: true,
    }
    level: "super",
}
```

### 새로운 타입

```go
// Duration type은 int64의 alias type
type Duration int64

var dur Duration
// Duration이 int64의 alias type이긴 하지만
// 컴파일러는 Duration과 int64를 별개의 type으로 구분한다
dur = int64(1000) // error
dur = Duration(1000) // ok
```

### method

`method`는 사용자가 정의한 `struct`의 행위를 정의하기 위한 방법
함수를 선언할 때 함수 이름 앞에 오는 `struct`를 수신자(`receiver`)라 하며
수신자가 정의된 함수를 메소드(`method`)라고 한다.

```go
// user struct
type user struct {
    name email string
}

// user struct를 수신자로 하는 notify 함수
func (u *user) notify() {
    fmt.Printf(" %s 사용자의 메일 주소 %s에 메일을 발송합니다\n", u.name, u.email)
}

// user struct를 수신자로 하는 changeEmail 함수
func (u *user) changeEmail(email string) {
    u.email = email
}

func main() {
    // user 타입의 수신자 선언
    thanos := user{"bill", "thanos@villain.com"}
    // user struct 수신자로 하는 notify 메소드 호출
    thanos.notify()

    // user 타입의 포인터를 이용한 수신자 선언
    thor := &user{"thor", "thor@asgard.king"}
    // user struct 수신자로 하는 메소드 호출
    thor.changeEmail("thor@earth.idiot") // 포인터 변수라고 해도 메소드 호출 방법은 동일하다
    (*thor).changeEmail("thor@earth.idiot") // 이렇게 사용하지 않아도 됨
    thor.notify()
}
```

## reference type

슬라이스, 맵, 채널, 인터페이스, 함수, 문자열 등 `reference type` 변수들은 `header value`를 가지고 있다
`header value`는 데이터의 포인터(메모리 주소)이며, 이러한 `reference type` 변수들은 변수 복사를 할 때
값이 아닌 `header value`만 복사하게 된다

```go
// 레퍼런스 타입 변수(슬라이스) 선언
slice1 := []int{1, 2, 3}
var slice2 []int

// 레퍼런스 타입 변수 slice1를 slice2로 복사
slice2 = slice1

// 복사된 레퍼런스 타입 변수 slice2의 값을 변경
slice2[0] = 0

// 레퍼런스 타입의 복사는 헤더 값만 복사되기 때문에 두개의 레퍼런스 타입 모두 값이 변경된다
fmt.Println(slice1) // [0, 2, 3]
fmt.Println(slice2) // [0, 2, 3]
```

## interface

`struct`가 필드의 집합체라면 `interface`는 메소드들의 집합체이다

`struct` type이 상태를 가지고 있다면

`interface` type은 기능을 가지고 있다

먼저 `method`를 이용해 도형의 넓이를 구하는 방법을 보면 다음과 같다

```go
// struct 구현
type Square struct {
    width, height float64
}

type Triangle struct {
    width, height float64
}

// struct에 대한 메소드 area 구현
func (s Square) Area() float64 { return s.width * s.height }
func (t Triangle) Area() float64 { return t.width * t.height / 2 }

func main() {
    s := Square{2, 5}
    t := Triangle{2, 5}
    fmt.Println(s.Area())
    fmt.Println(t.Area())
}
```

이때 인터페이스는 메소드에 대해 선언하기 위한 타입이다

```go
// 인터페이스는 메소드 이름과 반환형을 선언하는 것 만으로 구현된다
type Figure interface {
    Area() float64
}

func main() {
    // Figure 타입으로 변수 선언
    var s, t Figure

    // Square와 Triangle 타입으로 초기화
    s = Square{2, 5}
    t = Triangle{2, 5}

    fmt.Println(s.Area())
    fmt.Println(t.Area())
}

// 혹은 인터페이스를 슬라이스 형식으로 선언할 수 도 있다
func main() {
    slice := make([]Figure, 0)
    slice = append(slice, Square{2, 5})
    slice = append(slice, Triangle{2, 5})
    for _, v := range slice {
        fmt.Println(v.Area())
    }
}
```

인터페이스 예제 2

```go

// 알림 동작을 정의하는 notifier 인터페이스
type notifier interface {
    notify()
}

// user type
type User struct {
    name  string
    email string
}

// User 수신자에 대한 notify 메소드
func (u *User) notify() {
    fmt.Printf("%s 사용자의 메일 주소 %s 에 메일을 전송\n", u.name, u.email)
}

// Admin type
type Admin struct {
    user  User
    level string
}

func main() {
    // user type 초기화
    bill := User{"bill", "bill@email.com"}
    sendNotification(&bill)

    manager := Admin{"manager", "manager@email.com"}
    sendNotification(&manager)
}
```

## 타입 임베딩

기존 타입을 확장하거나 그 동작을 변형하는 것을 타입 임베딩이라고 한다
기존에 선언된 타입을 새로운 구조체 타입의 내부에 선언 하는 것이다

```go
// user type
type User struct {
    name  string
    email string
}

// User 수신자에 대한 notify 메소드
func (u *User) notify() {
    fmt.Printf("%s 사용자의 메일 주소 %s 에 메일을 전송\n", u.name, u.email)
}

// Admin type
type Admin struct {
    User // 타입 임베딩
    level string
}

func main() {
    // Admin type 초기화
    ad := Admin{
        User: User{
            name: "jean",
            email: "grey@x.man",
        },
        level: "dark pheonix",
    }

    // User의 메소드 notify 호출
    ad.User.notify()

    // 타입 임베딩을 통해 내부 메소드를 호출할 수 있다
    ad.notify()
}
```

`User`는 `Admin`의 내부 타입 이지만 타입 임베딩을 통해 `Admin` 타입을 이용해 `User` 타입의 내무 메소드를 실행할 수 있는 것이다

```go
// user type
type User struct {
    name  string
    email string
}

// User 수신자에 대한 notify 메소드
func (u *User) notify() {
    fmt.Printf("%s 사용자의 메일 주소 %s 에 메일을 전송\n", u.name, u.email)
}

// Admin type
type Admin struct {
    User // 타입 임베딩
    level string
}


func main() {
    // Admin type 초기화
    ad := Admin{
        User: User{
            name: "jean",
            email: "grey@x.man",
        },
        level: "dark pheonix",
    }

    // User의 메소드 notify 호출
    ad.User.notify()

    // 타입 임베딩을 통해 내부 메소드를 호출할 수 있다
    ad.notify()
}
```
