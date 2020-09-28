---
layout: post
title:  "Golang Chapter 3"
author: violetstair
categories: [ Programming ]
tags: [Golang, Programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 3 : Golang 기본 문법"
featured: true
hidden: true
---

## 변수 선언

### 기본적인 변수 선언

```go
var name string
var age int
```

### 변수 초기화

```go
var a, b, c int = 1, 2, 3

x, y, z := "hello", 1, true

name, _ := GetName()
name, _ = GetName2()
name, err := GetName3()
```

### 상수

```go
const s string = "상수"
const s = "성수"

const (
    s1 = "상수"
    s2 = true
    s3 = 1
    s4 = 'A'
)
```

### 자료형

* `bool`
* `string`
* `int`, `int8`, `int16`, `int32`, `int64`
* `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
* `byte` : `int8`의 alias
* `rune` : `int32`의 alias, unocode의 `byte` 표현
* `float32`, `float64`
* `complex64`, `complex128` : 복소수 표현, `float32`(`complex128`은 `float62`)의 실수부와 허수부로 구성되어 있다

## 분기/반복문

### IF/ELSE

```go
if true {
    fmt.Println("True")
} else if false {
    fmt.Println("False")
} else {
    fmt.Println("No way")
}
```

### SWITCH

```go
switch grade {
    case 1:
        fmt.Println("1")
    case 2:
        fmt.Println("2")
    case 3:
        fmt.Println("3")
    case 4:
        fmt.Println("4")
    default:
        fmt.Println("5")
}
```

### FOR

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
fmt.Println(sum)
```

### WHILE

```go
// golang은 while이 없음
for {
}
```

## defer

지연 실행문(종료 예고문), 함수가 종료될 때 실행되는 내용을 함수 초반에 정의한다
`close` 함수 등을 실행할 때 유용하다

```go
func main() {
    defer fmt.Println("World")
    fmt.Println("Hello")
}

// 실행 결과
Hello
World
```

여러개의 `defer`는 `stack`에 넣고 실행 하며, 마지막에 선언된 `defer`가 함수 종료 시점에서 가장 먼저 실행 되게 된다.

```go
func main() {
    fmt.Println("counting")

    for i := 0; i < 10; i++ {
        defer fmt.Println(i)
    }

    fmt.Println("done")
}

// 실행 결과
counting
done
9
8
7
6
5
4
3
2
1
0
```

## 포인터

메모리의 주소를 가리키는 포인터도 사용할 수 있다

```go
i, j := 42, 2701

p := &i         // i의 포인트
fmt.Println(*p) // 포인터를 통해 i의 값을 출력
*p = 21         // 포인터를 통해 i에 값을 수정
fmt.Println(i)  // 포인터를 통해 변경된 i의 값을 출력

// 함수의 참조 값 전달
var name := "hello"

// name을 참조 값 형태로 foo 함수에 전달
foo(&name)

// foo 함수는 name 변수를 참조값으로 전달 받음
func foo(name *string) {
    ...Todo
}
```

## 함수

```go
func foo(name string) string {
    return name
}
```

### 익명 함수

함수명을 갖지 않는 함수

```go
func main() {
    // 익명함수 정의
    sum := func(n ...int) int {
        s := 0
        for _, i := range n {
            s += i
        }
        return s
    }
    result := sum(1, 2, 3, 4, 5) // 익명함수 호출
}
```

### 일급 함수

`golang`은 함수는 기본 타입과 동일하게 취급되므로 다른함수의 파라미터로 전달하거나 리턴값으로 사용될 수 있다

```go
func main() {
    //변수 add 에 익명함수 할당
    add := func(i int, j int) int {
        return i + j
    }

    // add 함수 전달
    r1 := calc(add, 10, 20)
    println(r1)

    // 직접 첫번째 파라미터에 익명함수를 정의함
    r2 := calc(func(x int, y int) int { return x - y }, 10, 20)
    println(r2)
}

func calc(f func(int, int) int, a int, b int) int {
    result := f(a, b)
    return result
}
```

### 함수 원형 정의

`type`문은 구조체, 인터페이스 등 사용자 정의 type을 정의하기 위해 사용된다
`type`문은 함수 원형도 정의할 수 있는데 `JAVA`의 추상 메소드와 비슷하다

```go
// 원형 정의
type calculator func(int, int) int

// calculator 원형 사용
func calc(f calculator, a int, b int) int {
    result := f(a, b)
    return result
}
```
