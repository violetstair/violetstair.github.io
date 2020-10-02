---
layout: post
title:  "Golang Chapter 6"
author: violetstair
categories: [ Golang ]
tags: [golang, programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 6 : 동시성"
featured: false
hidden: true
---

`golang`에서 동시성이란, 함수들을 다른 함수들과 독립적으로 실행할 수 있는 기능을 의미한다
함수를 고루틴으로 생성하면 곧바로 실행되는 것이 아니라 실행해야 할 함수로 예약된 후
논리 프로세스가 여력이 있을 때 실행되는 독립적인 작업 단위로 취급된다
또한 고루틴 사이에 데이터 교환은 동시 접속으로 데이터를 잠그는 기법이 아니라 채널을 통해 메시지를 전달하는 모델이다.
여러개의 물리 프로세스에서 실행되는 병렬성과는 다른 개념이다

## 고루틴

아래 코드를 실행해 보면 코드의 역순인 대문자 부터 실행되는 것을 확인할 수 있다
(단지 코드의 실행 시간이 짧아서임)

```go
func main()  {
    // 스케쥴러가 사용할 하나의 논리 프로세서를 할당한다
    runtime.GOMAXPROCS(1)

    // wg는 프로그램의 종료를 대기하기 위해 사용한다
    var wg sync.WaitGroup
    wg.Add(2) // 두 개의 wait group

    fmt.Println("고루틴 실행")

    // 고루틴 익명함수
    go func() {
        // 익명 함수가 종료될 때 실행
        defer wg.Done() // wait group 완료

        for count := 0; count < 3; count++ {
            for char := 'a'; char < 'a' + 26; char++ {
                fmt.Printf("%c ", char)
            }
        }
    }()

    // 고루틴 익명함수
    go func() {
        // 익명 함수가 종료될 때 실행
        defer wg.Done() // wait group 완료

        for count := 0; count < 3; count++ {
            for char := 'A'; char < 'A' + 26; char++ {
                fmt.Printf("%c ", char)
            }
        }
    }()

    fmt.Println("대기 중 ...")
    wg.Wait() // wait group이 0보다 크면 대기하게 된다

    fmt.Println("\n프로그램을 종료합니다...")
}
```

아래의 소수를 출력하는 코드를 실행해 보면 A와 B의 순서가 실행때마다 다른것을 확인할 수 있가
이는 각각의 고루틴을 실행 중 유휴시간을 다른 고루틴에게 할당하기 때문이다

```go
// wg는 프로그램의 종료를 대기하기 위해 사용한다
var wg sync.WaitGroup

func main()  {
    // 스케쥴러가 사용할 하나의 논리 프로세서를 할당한다
    runtime.GOMAXPROCS(1)

    wg.Add(2) // 두 개의 wait group

    fmt.Println("고루틴 실행")

    // 고루틴 실행
    go printPrime("A")
    go printPrime("B")

    fmt.Println("대기 중 ...")
    wg.Wait() // wait group이 0보다 크면 대기하게 된다

    fmt.Println("\n프로그램을 종료합니다...")
}

func printPrime(prefix string) {
    defer wg.Done()

next: // continue에 대한 라벨 변수
    for outer := 2; outer < 5000; outer++ {
        for inner := 2; inner < outer; inner++ {
            if outer % inner == 0 {
                // 라벨에 의해 inner 루프문 건너뛰고 다음 outer 루프 바로 실행
                continue next
            }
        }
        fmt.Printf("%s:%d\n", prefix, outer)
    }
    fmt.Printf("완료 : %s\n", prefix)
}
```

### 고루팀의 병렬 실행

앞서 실행했던 알파벳 출력 코드의 논리 프로세스 수를 수정하여 실행해보면 실행 결과가 다르게 나타나는 것을 알 수 있다

```go
func main()  {
    // 물리 프로세스 수 만큼 논리 프로세스 할당
    runtime.GOMAXPROCS(runtime.NumCPU())

    // wg는 프로그램의 종료를 대기하기 위해 사용한다
    var wg sync.WaitGroup
    wg.Add(2) // 두 개의 wait group

    fmt.Println("고루틴 실행")

    // 고루틴 익명함수
    go func() {
        // 익명 함수가 종료될 때 실행
        defer wg.Done() // wait group 완료

        for count := 0; count < 3; count++ {
            for char := 'a'; char < 'a' + 26; char++ {
                fmt.Printf("%c ", char)
            }
        }
    }()

    // 고루틴 익명함수
    go func() {
        // 익명 함수가 종료될 때 실행
        defer wg.Done() // wait group 완료

        for count := 0; count < 3; count++ {
            for char := 'A'; char < 'A' + 26; char++ {
                for a := 0; a < 30; a++ {
                }
                fmt.Printf("%c ", char)
            }
        }
    }()

    fmt.Println("대기 중 ...")
    wg.Wait() // wait group이 0보다 크면 대기하게 된다

    fmt.Println("\n프로그램을 종료합니다...")
}
```

고루틴이 논리 프로세스에 의해 동시에 실행되며 출력 결과가 뒤섞이는걸 볼 수 있다

> 단 논리 프로세스의 수를 늘리는게 무조건 성능 향상을 이루는건 아니다

## 레이스 컨디션 탐지

Race Condition : 두 개 혹은 그 이상의 고루틴이 동기화 없이 동시에 공유된 자원에 접근하여 읽거나 쓰기를 시도하게 되는 상태, 이를 방지하기 위해 공유 자원에 대한 원자성, 즉 어느 한 시점에 단 하나의 고루팅만 그 자원에 접근해야 한다.

```go
import (
    "fmt"
    "runtime"
    "sync"
)

var (
    // 모든 고루틴이 값을 증가하려고 시도하는 변수
    counter int

    // 프로그램이 종료될 때까지 대기할 WaitGroup
    wg sync.WaitGroup
)

// 패키지 수주에 정의된 counter 변수의 값을 증가시키는 함수
func incCounter(id int) {
    // 함수 실행이 종료되면 main 함수에 알리기 위한 Done 함수 호출 예약
    defer wg.Done()

    for count := 0; count < 2; count++ {
        // counter 변수의 값을 읽음
        value := counter

        // 다른 고루틴이 CPU를 사용할 수 있도록 양보
        runtime.Gosched()

        // 카운터 증가
        value++

        // 원래 변수에 증가된 값을 다시 저장
        // * value는 증가 했지만 다른 고루틴에서 변경된 값은 반영되지 않았다 *
        // * 즉 다른 고루틴의 동작과 상관 없이 counter는 2로 덮어써지게 된다 *
        counter = value
    }
}

func main() {
    // 고루틴당 하나씩 총 두 개의 카운터 추가
    wg.Add(2)

    // 고루틴 생성
    go incCounter(1)
    go incCounter(2)

    // 고루틴 실행 종료까지 대기
    wg.Wait()
    fmt.Println("최종 결과  : ", counter)
}
```

빌드 시 레이스 컨디션 플래그를 통해 실행 시 레이스 컨디션 상태를 확인해 볼 수 있다

```bash
go build -race

./test
==================
WARNING: DATA RACE
Read at 0x000001275320 by goroutine 8:
  main.incCounter()
      /GOPATH/go/src/main.go:25 +0x7c

Previous write at 0x000001275320 by goroutine 7:
  main.incCounter()
      /GOPATH/go/src/main.go:34 +0x9d

Goroutine 8 (running) created at:
  main.main()
      /GOPATH/go/src/main.go:44 +0x89

Goroutine 7 (finished) created at:
  main.main()
      /GOPATH/go/src/main.go:43 +0x68
==================
최종 결과  :  4
Found 1 data race(s)
```

## 공유 자원 잠금

공유 자원에 대한 접근을 잠금으로써 고루틴을 동기활 할 수 있는 기능들을 제공한다.

### atomic

오직 한 개의 고루틴만이 변수에 접근하여 값을 변경할 수 있도록 동기화 한다.

```go
import (
    "fmt"
    "runtime"
    "sync"
    "sync/atomic"
)

var (
    // 공유 자원으로 활용될 변수
    counter int64

    // 프로그램이 종료될 때까지 대기할 WaitGroup
    wg sync.WaitGroup
)

// 패키지 수주에 정의된 counter 변수의 값을 증가시키는 함수
func incCounter(id int) {
    // 함수 실행이 종료되면 main 함수에 알리기 위한 Done 함수 호출 예약
    defer wg.Done()

    for count := 0; count < 2; count++ {
        // counter 변수에 안전하게 1을 증가
        atomic.AddInt64(&counter, 1)

        // 다른 고루틴이 CPU를 사용할 수 있도록 양보
        runtime.Gosched()
    }
}

func main() {
    // 고루틴당 하나씩 총 두 개의 카운터 추가
    wg.Add(2)

    // 고루틴 생성
    go incCounter(1)
    go incCounter(2)

    // 고루틴 실행 종료까지 대기
    wg.Wait()
    fmt.Println("최종 결과  : ", counter)
}
```

두 번째 예제

```go
import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"
)

var (
    // 실행중인 고루틴들의 종료 신호로 사용될 플래그
    shutdown int64

    // 프로그램이 종료될 때까지 대기할 WaitGroup
    wg sync.WaitGroup
)

// 실행 중 종료 플래그를 확인하여 종료하는 함수
func doWork(work string) {
    // 함수 실행의 종료를 main 함수에 알리기 위한 Done 호출 예약
    defer wg.Done()

    for {
        fmt.Printf("do work : %s\n", work)
        time.Sleep(250 * time.Millisecond)

        // 종료 플래그를 확인하고 실행 종료
        if atomic.LoadInt64(&shutdown) == 1 {
            fmt.Printf("stop work : %s\n", work)
            break
        }
    }
}

func main() {
    wg.Add(2)

    go doWork("A")
    go doWork("B")

    // 고루팅 실행 시간 할당
    time.Sleep(1 * time.Second)

    // 종료 신호 플랙스 설정
    fmt.Println("프로그램 종료")
    atomic.StoreInt64(&shutdown, 1)

    // 고루틴 실행이 종료될 때 까지 대기
    wg.Wait()
}
```

### mutex

상호 배타의 개념에서 유래, 임계지역을 생성하여 한번에 하나의 고루틴만이 시행할 수 있도록 보장한다.

```go
import (
    "fmt"
    "runtime"
    "sync"
)

var (
    //공유 자원을 활용될 변수
    counter int

    // 프로그램이 종료될 때 까지 대기할 WaitGroup
    wg sync.WaitGroup

    // 코드의 임계영역을 지정할 때 사용할 mutex
    mutex sync.Mutex
)

// 패키지 수준에서 정의된 counter 변수의 값을 mutex를 이용해 안전하게 증가시키는 방법
func incCounter(id int) {
    // 함수 실행이 종료를 알리기 위한 Done 함수 호출 예약
    defer wg.Done()

    for count := 0; count < 2; count++ {
        // 이 임계 지역에는 한 번에 하나의 고루틴만 접근 가능
        mutex.Lock()
        {
            // counter 값 조회
            value := counter

            // 다른 고루틴이 CPU를 사용할 수 있도록 양보
            runtime.Gosched()

            // 현재 카운터 값 증가
            value++

            // 원래 변수에 증가된 값을 다시 저장
            counter = value
        }
        // 대기중인 다른 고루틴이 접근할 수 있도록 잠금 해제
        mutex.Unlock()
    }
}

func main() {
    // 고루틴 당 하나 씩 총 두 개의 카운터 추가
    wg.Add(2)

    // 두 개의 고루틴 생성
    go incCounter(1)
    go incCounter(2)

    // 고루틴 실행이 종료될 때 까지 대기
    wg.Wait()
    fmt.Printf("최종 결과 : %d\n", counter)
}
```

## 채널

채널은 필요한 공유 자원을 다른 고루틴에게 보내거나 받아 고루틴 사이에 동기화를 지원하는 개념으로, 고루틴 사이에 파이프처럼 동작하며, 고루틴간 데이터 교환에 동기화를 보장한다.
`make`로 채널을 생성을 선언하며, `<-` 연산자로 데이터를 전달한다.

### 버퍼가 없는 채널

값을 전달받기 전 어떤 값을 얼마나 보유할 수 있을지 크기가 결정 되지 않은 채널

버퍼가 없는 채널을 이용한 테니스 경가 예제

```go
import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// 프로그램이 종료될 때 까지 대기할 WaitGroup
var wg sync.WaitGroup

func init() {
    rand.Seed(time.Now().UnixNano())
}

// 선수의 행동을 모방하는 player
func player(name string, court chan int) {
    // 함수 실행이 종료될 때 Done 함수를 호출하도록 예약
    defer wg.Done()

    for {
        // 공이 되돌아올 때까지 대기
        ball, ok := <-court
        if !ok {
            // 채널이 닫혔으면 승리로 간주
            fmt.Printf("%s의 승리\n", name)
            return
        }

        // 랜덤 값을 이용해 공을 받아치지 못했는지 확인
        n := rand.Intn(100)
        if n%13 == 0 {
            fmt.Printf("%s의 반격 실패\n", name)

            // 채널을 닫아 현재 선수가 패배 했다고 알림
            close(court)
            return
        }

        // 선수가 반격한 횟수를 출력하고 그 값을 증가 시킨다
        fmt.Printf("%s의 %d번째 반격\n", name, ball)
        ball++

        // 공을 상대에게 전달
        court <- ball
    }
}

func main() {
    // 버퍼가 없는 채널 생성
    court := make(chan int)

    // 고루틴당 하나씩 총 두개의 카운터 추가
    wg.Add(2)

    // 두 선수 생성
    go player("나달", court)
    go player("조코비치", court)

    // 경기 시작
    court <- 1

    // 경기 종료 대기
    wg.Wait()
}
```

버퍼가 없는 채널을 이용한 계주 경기 예제

```go
import (
    "fmt"
    "sync"
    "time"
)

// 프로그램이 종료될 때 까지 대기할 WaitGroup
var wg sync.WaitGroup

// 계주의 각 주자를 표현하는 Runner 함수
func Runner(baton chan int) {
    var newRunner int

    // 바통을 전달받을 떄까지 대기
    runner := <-baton

    // 트랙을 달린다
    fmt.Printf("%d 번째 주자가 바통을 받아 달리기 시작\n", runner)

    // 새로운 주자가 교체 지점에서 대기
    if runner != 4 {
        newRunner = runner + 1
        fmt.Printf("%d 번째 주자 대기 중\n", newRunner)
        go Runner(baton)
    }

    // 트랙을 달림
    time.Sleep(100 * time.Millisecond)

    // 결기가 끝났는지 확인
    if runner == 4 {
        fmt.Printf("%d 번째 주자가 도착했습니다. 경기가 끝났습니다.\n", runner)
        wg.Done()
        return
    }

    // 다음 주자에게 바통을 넘김
    fmt.Printf("%d 번째 주자가 %d 번째 주자에게 바통을 넘김\n", runner, newRunner)
    baton <- newRunner
}

func main() {
    // 버퍼가 없는 채널 생성
    baton := make(chan int)

    // 마지막 주자를 위한 하나의 카운터 생성
    wg.Add(1)

    // 첫 번째 주자 경기 준비
    go Runner(baton)

    // 경기 시작
    baton <- 1

    // 경기 종료 대기
    wg.Wait()
}
```

### 버퍼가 있는 채널

고루틴이 값을 받아가기 전까지 채널에 보관할 수 있는 값의 개수를 지정할 수 있다.
이 채널을 이용하면 보내고 받는 동작이 반드시 동시에 이루어지지 않아도 된다.
값을 받는 작업의 경우 채널내의 값일 없을때만, 값을 넣는 작업의 경우 채널에 값이 가득 찼을때만 채널의 잠금 상태를 확인해도 된다.

```go
import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

const (
    numbnerGoroutines = 4 // 실행할 고루틴의 개수

    taskLoad = 10 // 처리할 작업의 개수
)

// 프로그램이 종료될 때까지 대기할 WaitGroup
var wg sync.WaitGroup

func init() {
    rand.Seed(time.Now().Unix())
}

func worker(tasks chan string, worker int) {
    defer wg.Done()

    for {
        // 작업이 할당될 때까지 대기
        task, ok := <-tasks
        if !ok {
            // 채널이 닫힌 경우
            fmt.Printf("작업자: %d : 종료 합니다.\n", worker)
            return
        }

        // 작업을 시작하는 메시지 출력
        fmt.Printf("작업자 : %d : 작업 시작 : %s\n", worker, task)

        // 작업을 처리(랜덤한 시간 대기)
        sleep := rand.Int63n(100)
        time.Sleep(time.Duration(sleep) * time.Millisecond)

        // 작업이 완료되었다는 메시지 출력
        fmt.Printf("작업자 : %d : 작업 완료 : %s\n", worker, task)
    }
}

func main() {
    // 작업 부하를 관리하기 위한 버퍼가 있는 채널을 생성
    tasks := make(chan string, taskLoad)

    // 작업을 처리할 고루틴 생성
    wg.Add(numbnerGoroutines)
    for gr := 1; gr <= numbnerGoroutines; gr++ {
        go worker(tasks, gr)
    }

    // 실행할 작업 추가
    for post := 1; post <= taskLoad; post++ {
        tasks <- fmt.Sprintf("작업 : %d", post)
    }

    // 작업을 모두 처리하면 채널을 닫음
    close(tasks)

    // 모든 작업이 처리될 때까지 대기
    wg.Wait()
}
```

