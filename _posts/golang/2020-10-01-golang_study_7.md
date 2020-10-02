---
layout: post
title:  "Golang Chapter 7"
author: violetstair
categories: [ Golang ]
tags: [golang, programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 7 : 동시성 패턴"
featured: false
hidden: true
---

## Runner 패키지

runner 패키지를 이용해 cron과 유사한 채널을 이용해 프로그램의 실행 시간을 관찰하고 프로그램이 너무 오래 실행되면 종료하는 프로그램 예제

* 지정된 시간 동안 작업을 모두 처리한 경우 종료
* 프로그램이 제 시간에 작업을 완료하지 못해 스스로 종료
* 운영체제로부터 인터럽트 신호가 발생한 경우 종료

```go
package runner

import (
    "errors"
    "os"
    "os/signal"
    "time"
)

// Runner 타입은 주어진 타임아웃 시간동안 일련의 작업을 수행하고, 운영체제 인터럽트에 의해 실행이 종료
type Runner struct {
    // 운영체제로부터 전달되는 인터럽트 신호를 수신하기 위한 채털
    interrupt chan os.Signal

    //초리가 종료되었음을 알리기 위한 채널
    complete chan error

    // 지정된 시간이 초과했음을 알리기 위한 채널
    timeout <-chan time.Time

    // 처리될 작업을 목록을 저장하기 위한 슬라이스
    tasks []func(int)
}

// timeout 채널에서 값을 수신하면 ErrTimeout을 리턴
var ErrTimeout = errors.New("시간을 초과했습니다.")

// 운영체제 이벤트를 수신하면 ErrInterrupt를 리턴
var ErrInterrupt = errors.New("운영체제 인터럽트 신호를 수신했습니다.")

// 실행할 Runner 타입 값을 리턴하는 함수
func New(d time.Duration) *Runner {
    return &Runner{
        interrupt: make(chan os.Signal, 1),
        complete:  make(chan error),
        timeout:   time.After(d),
    }
}

// Runner 타임에 작업을 추가하는 메서드, 작업은 int형 ID를 매개변수로 전달받는 함수
func (r *Runner) Add(tasks ...func(int)) {
    r.tasks = append(r.tasks, tasks...)
}

// 저장된 모든 작업을 실행하고 채널 이벤트를 관찰
func (r *Runner) Start() error {
    // 모든 종류의 인터럽트 신호를 수신
    signal.Notify(r.interrupt, os.Interrupt)

    // 각각의 작업을 각기 다른 고루틴을 통해 실행
    go func() {
        r.complete <- r.run()
    }()

    select {
    // 작업 완료 신호를 수신한 경우
    case err := <-r.complete:
        return err
    // 작업 시간 초과 신호를 수신하는 경우
    case <-r.timeout:
        return ErrTimeout
    }
}

// 개별 작업을 실행하는 메서드
func (r *Runner) run() error {
    for id, task := range r.tasks {
        // 운영체제로부터 인터럽트 신호를 수신했는지 확인
        if r.gotIntereupt() {
            return ErrInterrupt
        }

        // 작업 실행
        task(id)
    }

    return nil
}

// 인터럽트 신호가 수신되었는지 확인하는 메서드
func (r *Runner) gotIntereupt() bool {
    select {
    // 인터럽트 이벤트가 발생한 경우
    case <-r.interrupt:
        // 이후 발생하는 인터럽트 신호를 더이상 수신하지 않도록 한다
        signal.Stop(r.interrupt)
        return true

    //작업을 계속해서 실행하게 한다
    default:
        return false
    }
}
```

```go
package main

import (
    "log"
    "os"
    "time"

    "GOPATH/runner"
)

// 프로그램의 실행 시간
const timeout = 3 * time.Second

func createTask() func(int) {
    return func(id int) {
        log.Printf("프로세서 - 작업 #%d", id)
        time.Sleep(time.Duration(id) * time.Second)
    }
}

func main() {
    log.Println("작업을 시작합니다.")

    // 실행 시간을 이용해 새로운 task runner 생성
    r := runner.New(timeout)

    // 수행할 작업 등록
    r.Add(createTask(), createTask(), createTask())

    // 작업을 실행하고 결과를 처리
    if err := r.Start(); err != nil {
        switch err {
        case runner.ErrTimeout:
            log.Println("지정된 작업 시간을 초과했습니다")
            os.Exit(1)
        case runner.ErrInterrupt:
            log.Println("운영체제 인터럽트가 발생했습니다")
            os.Exit(2)
        }
    }

    log.Println("프로그램을 종료합니다.")
}
```

## 풀링

버퍼가 있는 채널을 이용하여 공유가 가능한 리소스 풀 구현해본다. 데이터베이스 연결이나 메모리 버퍼 같은 정적인 리소스를 관리할 때 고루틴은 리소스를 할당받아 사용하고 사용 후 풀에 반환하는 구조로 동작한다

```go
package pool

import (
    "errors"
    "io"
    "log"
    "sync"
)

// Pool 구조체는 여러 개의 고루틴에 안전하게 공유하기 위한 리소스의 집합을 관리한다
// 이 풀에서 관리하기 위한 리소스는 io.Closer 인터페이스를 반드시 구현해야 한다
type Pool struct {
    m         sync.Mutex
    resources chan io.Closer
    factory   func() (io.Closer, error)
    closed    bool
}

// ErrPoolClosed 에러는 리소스를 획득하려 할 때 풀이 닫혀있는 경우 발생
var ErrPoolClosed = errors.New("풀이 닫혔습니다.")

// New 함수는 리소스 관리 풀을 생성한다.
// 풀은 새로운 리소스를 할당받기 위한 함수와 풀의 크기를 매배변수로 정의한다
func New(fn func() (io.Closer, error), size uint) (*Pool, error) {
    if size <= 0 {
        return nil, errors.New("풀 사이즈가 너무 작습니다.")
    }

    return &Pool{
        factory:   fn,
        resources: make(chan io.Closer, size),
    }, nil
}

// 풀에서 리소스를 획득하는 메서드
func (p *Pool) Acquire() (io.Closer, error) {
    select {
    // 사용 가능한 리소스가 있는지 검사한다.
    case r, ok := <-p.resources:
        log.Println("리소스 획득 :", "공유된 리소스")
        if !ok {
            return nil, ErrPoolClosed
        }
        return r, nil
    // 사용 가능한 리소스가 없는 경우 새로운 리소스 생성
    default:
        log.Println("리소스 획득 :", "새로운 리소스")
        return p.factory()
    }
}

// 풀에 있는 리소스를 반환하는 메서드
func (p *Pool) release(r io.Closer) {
    // 안전한 작업을 위해 점금 설정
    p.m.Lock()
    defer p.m.Unlock()

    // 풀이 닫혔으면 리소스를 해제
    if p.closed {
        r.Close()
        return
    }

    select {
    // 새로운 리소스를 큐에 추가
    case p.resources <- r:
        log.Println("리소스 반환 : ", "리소스 큐에 반환")
    // 리소스 큐가 가득찬 경우 리소스 해제
    default:
        log.Println("리소스 반환: ", "리소스 해제")
        r.Close()
    }
}

// 풀을 종료하고 생성된 모든 리소스를 해제하는 메서드
func (p *Pool) Close() {
    // 안전한 작업을 위해 잠금 설정
    p.m.Lock()
    defer p.m.Unlock()

    // 풀이 이미 닫혀 있으면 아무런 작업도 수행하지 않음
    if p.closed {
        return
    }

    // 풀을 닫힌 상태로 전환
    p.closed = true

    // 리소스를 해제하기에 앞서 채널을 먼저 닫음, 그렇지 않으면 데드락에 덜릴 수 있음
    close(p.resources)

    // 레소스 해제
    for r := range p.resources {
        r.Close()
    }
}
```

pool 패키지를 이용해 연결 풀을 관리하는 예제 코드

```go
package main

import (
    "io"
    "log"
    "math/rand"
    "sync"
    "sync/atomic"
    "time"

    "GOPATH/pool"
)

const (
    maxGoroutines   = 25 // 실행할 수 있는 고루틴의 최대 개수
    pooledResources = 2  // 풀이 관리할 리소스의 개수
)

// 공유 자원을 표현한 구조체
type dbConnection struct {
    ID int32
}

// dbConnection 타입이 풀에의해 관리될 수 있도록 io.Close 인터페이스를 구현한다
// Close 메소드는 자원의 해제를 담당한다
func (dbConn *dbConnection) Close() error {
    log.Println("닫힘 : 데이터베이스 연결", dbConn.ID)
    return nil
}

// 각 데이터베이스에 유일한 id를 할당하기 위한 변수
var idCounter int32

// 풀이 새로운 리소스가 필요할 때 호출함
// 팩토리 메서드
func createConnection() (io.Closer, error) {
    id := atomic.AddInt32(&idCounter, 1)
    log.Println("생성 : 새 데이터베이스 연결", id)

    return &dbConnection{id}, nil
}

// 데이터베임스 연결 리소스 풀을 테스트
func performQuery(query int, p *pool.Pool) {
    // 풀에서 데이터베이스 연결 리소스 획득
    conn, err := p.Acquire()
    if err != nil {
        log.Println(err)
        return
    }

    // 데이터베이스 연결 리소스를 다시 풀로 되돌림
    defer p.Release(conn)

    // 쿼리문이 실행되는 것 처럼 일시 대기
    time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
    log.Printf("쿼리 : QID[%d] CID[%d]\n", query, conn.(*dbConnection).ID)
}

func main() {
    var wg sync.WaitGroup
    wg.Add(maxGoroutines)

    // 데이터베이스 연결을 관리할 풀을 생성
    p, err := pool.New(createConnection, pooledResources)
    if err != nil {
        log.Println(err)
    }

    // 풀에서 데이터베이스 연결을 가져와 쿼리를 실행
    for query := 0; query < maxGoroutines; query++ {
        // 각 고루틴에는 쿼리의 복사본을 전달해야 한다, 그렇지 않으면 고루틴들이 동일한 쿼리를 가지게 된다
        go func(q int) {
            performQuery(q, p)
            wg.Done()
        }(query)
    }

    // 고루틴이 종료될때까지 대기
    wg.Wait()

    // 풀을 닫는다
    log.Println("프로그램을 종료합니다.")
    p.Close()
}
```

## work 패키지

버퍼가 없는 채널을 이용하여 원하는 개수 만큼 작업을 독시적으로 실행할 수 있는 고루틴 풀 생성

```go
package work

import "sync"

// 작업 풀을 사용하려는 타입은 Worker 인터페이스를 구현해야 한다
type Worker interface {
    Task()
}

// Worker 인터페이스를 실행하는 고루틴의 풀을 제공하기 위한 Pool 구조체
type Pool struct {
    work chan Worker
    wg   sync.WaitGroup
}

// 새로운 작업 풀을 생성하는 함수
func New(maxGoroutines int) *Pool {
    p := Pool{
        work: make(chan Worker),
    }

    p.wg.Add(maxGoroutines)
    for i := 0; i < maxGoroutines; i++ {
        go func() {
            for w := range p.work {
                w.Task()
            }
            p.wg.Done()
        }()
    }

    return &p
}

// 풀에 새로운 작업을 추가하는 메서드
func (p *Pool) Run(w Worker) {
    p.work <- w
}

// 모든 고루틴을 종료할 때까지 대기하는 메서드
func (p *Pool) Shutdown() {
    close(p.work)
    p.wg.Wait()
}
```

고루틴을 생성하고 작업 풀에 등록/실행하는 예제

```go
package main

import (
    "log"
    "sync"
    "time"

    "GOPATH/work"
)

// 화면에 출력할 이름을 슬라이스로 선언
var names = []string{
    "steve",
    "bob",
    "alen",
    "jason",
    "will",
}

// 이름을 출력하기 위한 구조체
type namePrinter struct {
    name string
}

// Woeker 인터페이스를 구현하기 위해 Task 메서드 선언
func (m *namePrinter) Task() {
    log.Println(m.name)
    time.Sleep(time.Second)
}

func main() {
    // 두 개의 고루틴 작업풀 생성
    p := work.New(2)

    var wg sync.WaitGroup
    wg.Add(100 * len(names))

    for i := 0; i < 100; i++ {
        // 이름 슬라이스 반복
        for _, name := range names {
            // namePrinter 변수 선언하고 name 지정
            np := namePrinter{
                name: name,
            }

            go func() {
                // 실행할 메서드 등록, Run 메서드가 리턴되면 해당 작업이 처리된 것으로 간주
                p.Run(&np)
                wg.Done()
            }()
        }
    }

    wg.Wait()

    // 작업풀이 종료하고 이미 등록된 작업들이 종료될 때 까지 대기
    p.Shutdown()
}
```
