---
layout: post
title:  "Golang Chapter 4"
author: violetstair
categories: [ Golang ]
tags: [Golang, Programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 4 : 배열 / 슬라이스 / 맵"
featured: true
hidden: true
---

## 배열

배열은 동일한 타입의 데이터들이 고정된 길이의 공간에 연속적으로 저장된 형태의 데이터 타입

```go
// 배열의 선언
var array [5]int
// [ ][ ][ ][ ][ ]

// 배열의 초기화
array := [5]int{1, 2, 3, 4, 5}
// [1][2][3][4][5]

// 자동으로 길이가 결정되는 배열
// 배열이 초기화될 때 길이가 정해짐
array := [...]int{1, 2, 3, 4, 5}
// [1][2][3][4][5]

// 배열의 일부만 초기화
array : [5]int{1: 10, 2: 20}
// [ ][1][2][ ][ ]

// 배열의 사용
array := [5]int{1, 2, 3, 4, 5}
array[2] = 35

// 원소의 포인터
array := [5]*int{0: new(int), 1: new(int)}
// [ ][address_1][address_2][ ][ ]
// address_1 = 0
// address_2 = 0

*array[0] = 10
*array[1] = 20
// [ ][address_1][address_2][ ][ ]
// address_1 = 10
// address_2 = 20

// 배열의 값 대입
array1 := [5]int{1, 2, 3, 4, 5}
array2 := [5]int // 배열의 값을 대입하려면 같은 길이의 배열이어야 한다
// [1][2][3][4][5]
// [ ][ ][ ][ ][ ]
array2 = array1
// [1][2][3][4][5]
// [1][2][3][4][5]
```

### 다차원 배열

```go
// 배열의 선언
var array [4][2]int

// 배열의 초기화
var array [4][2]int := {
    {1, 2},
    {3, 4},
    {5, 6},
    {7, 8},
}
// [ [1][2] ][ [3][4] ][ [5][6] ][ [7][8] ]

// 이차원 배열의 사용
array[1][0] = 10
// [ [1][2] ][ [10][4] ][ [5][6] ][ [7][8] ]
```

### 배열의 사용

```go
var array [5]int

// array 배열을 foo 함수로 전달
foo(array)

// 정수형의 배열을 전달 받는 함수 foo
func foo(array [5]int) {
    ...Todo
}
```

## 슬라이스

슬라이스는 배열과 비슷한 개념을 가지며 동적 배열과 같이 사용할 수 있다
그러한 속성 때문에 슬라이스의 길이를 늘리거나 줄일 수 있고, 메모리를 연속적으로 사용하기 때문에
인덱싱, iteration 사용 및 GC 최적화 등의 장점을 가지고 있다

### 슬라이스의 구조

```go
// [ 메모리 주소 포인터 ][ 현재 길이 ][ 최대 길이 ]

// [ addres_1 ][ 3 ][ 5 ]
// address_1 : [1][2][3][ ][ ]
```

### 슬라이스의 생성과 초기화

```go
// 길이와 용량이 5인 문자열 슬라이스
slice := make([]string, 5)

// 길이가 3, 용량이 5인 정수 슬라이스
slice := make(int, 3, 5)

// 슬라이스 리터럴을 이용한 슬라이스 초기화
// 길이와 용량이 4인 문자열 슬라이스
slice := []string{"apple", "banana", "chery", "durian"}

// 길이가 100인 빈 문자열 슬라이스
slice := []string{99: ""}

// nil slice
slice := []int
// [ nil ][ 0 ][ 0 ]

// 빈 슬라이스
slice := make([]int, 0)
slice := []int{}
// [ address ][ 0 ][ 0 ]
// 빈 슬라이스 != nil 슬라이스는
```

#### 슬라이스와 배열의 차이

`[]` 연산자에 값을 지정하면 배열이 값을 지정하지 않으면 슬라이스가 생성된다

```go
array := [3]int{1, 2, 3}
slice := []int{1, 2, 3}
```

### 슬라이스의 사용

기본적으로 슬라이스의 사용법은 배열과 동일하다

```go
slice := []int{1, 2, 3}
// [ address ][ 3 ][ 3 ]
// address : [1][2][3]
slice[1] = 4
// address : [1][4][3]
```

### 슬라이스의 활용

#### 슬라이스 잘라내기

```go
// [ address_1 ][ 5 ][ 5 ]
slice := []int{1, 2, 3, 4, 5}
// address_1 : [1][2][3][4][5]

// slice의 일부만 잘라내 새로운 슬라이스를 생성
newSlice := slice[1:3]
// [ address_2 ][ 3 ][ 4 ]
// 슬라이스의 내부 배열을 갖는 슬라이스 즉 다른 슬라이스의 일부를 잘라 갖는 슬라이스는
// 원 슬라이스의 0번 인덱스부터 계산된 길이를 가진다

slice := make([]int, l, m)
// slice: [ address_1 ][ l ][ m ]
// array : [address_1][address_2][address_3][address_4][address_5]
newSlice := slice[i:k]
// newSlice : [ address_2 ][ k ][ y ]
// k = j - i
// y = m - i
```

#### 슬라이스 확장

```go
// 슬라이스는 필요에 따라 용량을 확장할 수 있다
slice := []int{1, 2, 3, 4, 5}
slice = append(slice, 6)
```

슬라이스 복사와 확장에 따른 주의 사항

```go
slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:3]
// [ address_1 ][ 5 ][ 5 ]
// [1][2][3][4][5]
// [ address_2 ][ 2 ][ 4 ]

newSlice = append(newSlice, 6)
// [ address_1 ][ 5 ][ 5 ]
// [1][2][3][6][5]
// [ address_2 ][ 3 ][ 4 ]
```

```go
slice := []int{1, 2, 3, 4, 5}
// [ address_1 ][ 5 ][ 5 ]
// [1][2][3][4][5]

newSlice = append(slice, 6)
// [ address_1 ][ 5 ][ 5 ]
// address_1 : [1][2][3][6][5]

// [ address_2 ][ 6 ][ 6 ]
// address_2 : [1][2][3][6][5][6]
```

#### 슬라이스 표현식

```go
slice := []int{1, 2, 3, 4, 5}
// 슬라이스 표현식
slice[2:3:4] // slice[시작 인덱스:길이 인덱스:용량 인덱스]
// [3][ ]

// slice[2:3:3]
// [3]
```

#### 두 슬라이스 합치기

```go
slice1 := []int{1, 2, 3}
slice2 := []int{4, 5, 6}

newSlice := append(slice1, slice2...)
```

## 맵

맵은 `key/value` 쌍의 정렬 없는 컬렉션을 제공하는 데이터 구조
해시테이블 기반으로 구현되어 정렬이 되지 않기 때문에 동일한 맵을 반환할 때 서로 순서가 다를 수 있다

### 맵 생성

```go
// make를 이용한 생성
// string을 key로 하고 int를 value로 하는 맵
dict := make(map[string]int)

// 맵 리터럴을 이용한 생성
dict := map[string]string{"apple":"green", "banana":"yellow", "cherry": "red"}

// int를 key로 하고 string 슬라이스를 value로 하는 맵
dict := map[int][]string{}
```

### 맵 사용

```go
dict := map[string]string{"apple":"green", "banana":"yellow", "cherry": "red"}

// 맵에 아이템 추가
dict["grape"] = "purple"

// 맵 아이템의 값 변경
dict["apple"] = "red"

// 맵에서 아이템 제거
delete(dict, "apple")

// 리터럴 속성을 이용한 반복문
for key, value := range dict {
    fmt.Printf("키 : %s / 값 : %s\n", key, value)
}
```

### 맵 사용의 주의 사항

```go
// nil 맵
dict := map[string]string
dict["apple"] = "red" // error

// 빈 맵
dict := map[string]string{}
dict["apple"] = "red" // ok
```
