---
layout: post
title:  "Golang Chapter 9"
author: violetstair
categories: [ Golang ]
tags: [golang, programming]
image: assets/images/gopher.jpg
description: "Golang study chapter 9 : 테스트와 벤치마킹"
featured: false
hidden: true
---

## 단위 테스트

`golang`에서 코드 테스트는 다음과 같은 명령어로 실행 가능하다

```bash
go test [build/test flags] [packages] [build/test flags & test binary flags]
```

테스트 파일의 이름은 `{테스트할_파일}_test.go`와 같이 `_test.go`로 끝나야 테스트 도구가 테스트 파일을 발견하고 실행할 수 있다.
또한 테스트 파일내에 위치한 테스트 함수의 이름은 `Test`로 시작해야 한다.

### 기본 단위 테스트

```go
import (
    "net/http"
    "testing"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

func TestDownload(t *testing.T) {
    url := "http://www.naver.com"
    statusCode := 200

    t.Log("컨텐츠 다운로드 기능 테스트 시작.")
    {
        t.Logf("\tURL : \"%s\" / 상태 코드 : \"%d\"",
            url, statusCode)
        {
            resp, err := http.Get(url)
            if err != nil {
                t.Fatal("\t\tHTTP Get 요청에 대한 응답", ballotX, err)
            }
            t.Log("\t\tHTTP Get 요청에 대한 응답", checkMark)

            defer resp.Body.Close()

            if resp.StatusCode == statusCode {
                t.Logf("\t\t상태 코드가 \"%d\" 인지 확인 %v", statusCode, checkMark)
            } else {
                t.Errorf("\t\t상태 코드가 \"%d\" 인지 확인 %v %v", statusCode, ballotX, resp.StatusCode)
            }
        }
    }
}
```

### 테이블 테스트

```go
import (
    "net/http"
    "testing"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

func TestDownload(t *testing.T) {
    var urls = []struct {
        url        string
        statusCode int
    }{
        {
            "https://www.naver.com",
            http.StatusOK,
        },
        {
            "http://rss.cnn.com/rss/cnn_topstbadurl.rss",
            http.StatusNotFound,
        },
    }

    t.Log("Given the need to test downloading different content.")
    {
        for _, u := range urls {
            t.Logf("\tURL \"%s\" 의 상태코드가 \"%d\"인지 체크", u.url, u.statusCode)
            {
                resp, err := http.Get(u.url)
                if err != nil {
                    t.Fatal("\t\tHTTP Get 요청에 대한 응답.", ballotX, err)
                }
                t.Log("\t\tHTTP Get 요청에 대한 응답.", checkMark)

                defer resp.Body.Close()

                if resp.StatusCode == u.statusCode {
                    t.Logf("\t\t상태코드가 \"%d\" 인지 확인. %v", u.statusCode, checkMark)
                } else {
                    t.Errorf("\t\t상태코드가 \"%d\" 인지 확인. %v %v", u.statusCode, ballotX, resp.StatusCode)
                }
            }
        }
    }
}
```

### 모의 호출

앞서 실행한 코드의 문제점은 테스트코드가 외부의 상태에 의해 실행 결과가 결정된다는 점이다.
이를 위해 `httptest` 패키지를 사용할 수 있으며 `mock` 코드를 생성하여 테스트를 할 수 있다.

```go
import (
    "encoding/xml"
    "fmt"
    "net/http"
    "net/http/httptest"
    "testing"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

// feed변수에 mocking 문서를 선언
var feed = `<?xml version="1.0" encoding="UTF-8"?>
<rss>
<channel>
    <title>Going Go Programming</title>
    <description>Golang : https://github.com/goinggo</description>
    <link>http://www.goinggo.net/</link>
    <item>
        <pubDate>Sun, 15 Mar 2015 15:04:00 +0000</pubDate>
        <title>Object Oriented Programming Mechanics</title>
        <description>Go is an object oriented language.</description>
        <link>http://www.goinggo.net/2015/03/object-oriented</link>
    </item>
</channel>
</rss>`

// mockServer 함수는 GET 요청을 처리할 서버에 대한 포인터를 리턴
func mockServer() *httptest.Server {
    f := func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(200)
        w.Header().Set("Content-Type", "application/xml")
        fmt.Fprintln(w, feed)
    }

    return httptest.NewServer(http.HandlerFunc(f))
}

func TestDownload(t *testing.T) {
    statusCode := http.StatusOK

    server := mockServer()
    defer server.Close()

    t.Log("컨텐츠 다운로드 기능 시작")
    {
        t.Logf("\tURL \"%s\" 호출 시 상태 코드가 \"%d\"인지 확인", server.URL, statusCode)
        {
            resp, err := http.Get(server.URL)
            if err != nil {
                t.Fatal("\t\tHTTP Get 요청을 보냈는지 확인", ballotX, err)
            }
            t.Log("\t\tHTTP Get 요청을 보냈는지 확인", checkMark)

            defer resp.Body.Close()

            if resp.StatusCode != statusCode {
                t.Fatalf("\t\t상태 코드가 \"%d\" 인지 확인 %v %v", statusCode, ballotX, resp.StatusCode)
            }
            t.Logf("\t\t상태 코드가 \"%d\" 인지 확인 %v",
                statusCode, checkMark)

            var d Document
            if err := xml.NewDecoder(resp.Body).Decode(&d); err != nil {
                t.Fatal("\t\t디코딩할 수 있는 응답이 돌아왔는지 확인.", ballotX, err)
            }
            t.Log("\t\t디코딩할 수 있는 응답이 돌아왔는지 확인.", checkMark)

            if len(d.Channel.Items) == 1 {
                t.Log("\t\t피드에 \"1\" 번 아이템이 있는지 확인.", checkMark)
            } else {
                t.Error("\t\t피드에 \"1\" 번 아이템이 있는지 확인.", ballotX, len(d.Channel.Items))
            }
        }
    }
}

// RSS 문서의 아이템 태그 정의
type Item struct {
    XMLName     xml.Name `xml:"item"`
    Title       string   `xml:"title"`
    Description string   `xml:"description"`
    Link        string   `xml:"link"`
}

// RSS 문서의 채널 태그 정의
type Channel struct {
    XMLName     xml.Name `xml:"channel"`
    Title       string   `xml:"title"`
    Description string   `xml:"description"`
    Link        string   `xml:"link"`
    PubDate     string   `xml:"pubDate"`
    Items       []Item   `xml:"item"`
}

// RSA 문서 관련 필드 정의
type Document struct {
    XMLName xml.Name `xml:"rss"`
    Channel Channel  `xml:"channel"`
    URI     string
}
```

### Endpoint 테스트

웹 API를 개발할 때 실제 서버를 거치지 않고 endpoint 테스트 진행할 때 `httptest` 패키지를 통해 테스트할 수 있다.

```go
package main

import (
    "log"
    "net/http"

    "GOPATH/handlers"
)

func main() {
    handlers.Routes()

    log.Println("웹 서비스 실행 : 포트 :4000")
    http.ListenAndServe(":4000", nil)
}
```

handlers 패키지

```go
package handlers

import (
    "encoding/json"
    "net/http"
)

// 웹 서비스의 라우트 설정
func Routes() {
    http.HandleFunc("/sendjson", SendJSON)
}

// 간단한 JSON 문서를 반환
func SendJSON(rw http.ResponseWriter, r *http.Request) {
    u := struct {
        Name  string
        Email string
    }{
        Name:  "Bill",
        Email: "bill@ardanstudios.com",
    }

    rw.Header().Set("Content-Type", "application/json")
    rw.WriteHeader(200)
    json.NewEncoder(rw).Encode(&u)
}
```

실제 코드가 정상적으로 동작하는 것을 확인했지만 사람의 눈이 아닌 테스트 코드를 통해 코드의 동작상태를 확인할 수 있다
`handlers_test.go` 테스트 코드를 작성할 수 있다

```go
package handlers_test

import (
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "GOPATH/handlers"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

func init() {
    handlers.Routes()
}

// endpoint 테스트를 진행
func TestSendJSON(t *testing.T) {
    t.Log("SendJSON에대한 endpoint 테스트 시작")
    {
        req, err := http.NewRequest("GET", "/sendjson", nil)
        if err != nil {
            t.Fatal("\t웹 요청을 보냈는지 확인", ballotX, err)
        }
        t.Log("\t웹 요청을 보냈는지 확인", checkMark)

        rw := httptest.NewRecorder()
        http.DefaultServeMux.ServeHTTP(rw, req)

        if rw.Code != 200 {
            t.Fatal("\t응답코드가 \"200\"인지 확인", ballotX, rw.Code)
        }
        t.Log("\t응답코드가 \"200\"인지 확인", checkMark)

        u := struct {
            Name  string
            Email string
        }{}

        if err := json.NewDecoder(rw.Body).Decode(&u); err != nil {
            t.Fatal("\t응답 데이터에 대한 디코딩 확인", ballotX)
        }
        t.Log("\t응답 데이터에 대한 디코딩 확인", checkMark)

        if u.Name == "Bill" {
            t.Log("\t응답 데이터의 이름 확인", checkMark)
        } else {
            t.Error("\t응답 데이터의 이름 확인", ballotX, u.Name)
        }

        if u.Email == "bill@ardanstudios.com" {
            t.Log("\t응답 데이터의 메일 주소 확인", checkMark)
        } else {
            t.Error("\t응답 데이터의 메일 주소 확인", ballotX, u.Email)
        }
    }
}
```

## 예제 코드

`golang`은 코드로 부터 문서를 생성할 수 있는 `godoc`이라는 도구를 제공한다.

```go
package handlers_test

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "net/http/httptest"
)

// ExampleSendJSON 함수는 기본 예제를 제공한다
func ExampleSendJSON() {
    r, _ := http.NewRequest("GET", "/sendjson", nil)
    w := httptest.NewRecorder()
    http.DefaultServeMux.ServeHTTP(w, r)

    var u struct {
        Name  string
        Email string
    }

    if err := json.NewDecoder(w.Body).Decode(&u); err != nil {
        log.Println("ERROR:", err)
    }

    fmt.Println(u)
    // Output:
    // {Bill bill@ardanstudios.com}
}
```

예제 코드는 함수나 메서드 기반으로 생성되며 `Example`로 시작한다.
예제 코드는 항상 존재하는 함수나 메서드를 기반으로 만들어야 한다 즉 위 코드에서 예제 코드를 생성한 `SendJSON` 함수가 실제 코드내에 있어야 한다.

로컬 `godoc` 서버를 실행 하는 방법은 다음과 같다

```bash
godoc -http=":3000"
```

또한 위 코드는 하나의 단위 테스트이기도 하기때문에 `go test`를 통해 실행할 수 도 있다

```bash
go test -v -run="ExampleSendJSON"
```

## 벤치마킹

코드의 성능을 테스트하고, CPU나 메모리 이슈를 파악하기 위해 활용할 수 있다.
또한 동시성 패턴을 테스트하거나 최상의 결과를 위해 작업 풀이 올바르게 설정되었는지 판단할 때 사용한다.

```go
import (
    "fmt"
    "strconv"
    "testing"
)

// BenchmarkSprintf 함수는 fmt.Sprintf 함수의 성능을 테스트한다
func BenchmarkSprintf(b *testing.B) {
    number := 10

    b.ResetTimer()

    for i := 0; i < b.N; i++ {
        fmt.Sprintf("%d", number)
    }
}

// BenchmarkFormat 함수는 strconv.FormatInt 함수의 성능을 테스트한다
func BenchmarkFormat(b *testing.B) {
    number := int64(10)

    b.ResetTimer()

    for i := 0; i < b.N; i++ {
        strconv.FormatInt(number, 10)
    }
}

// BenchmarkItoa 함수는 strconv.Itoa 함수의 성능을 테스트한다
func BenchmarkItoa(b *testing.B) {
    number := 10

    b.ResetTimer()

    for i := 0; i < b.N; i++ {
        strconv.Itoa(number)
    }
}
```

벤체마크 테스트를 실행하는 방법은 다음과 같다

```bash
go test -v -run="none" -bench="BenchmarkSprintf"
```

벤치마킹 시간을 조정하려면 `-benchtime` 옵션을 사용하면 된다.

```bash
go test -v -run="none" -bench="BenchmarkFormat" -benchtime=3s
```

메모리의 할당 횟수와 한 번 할당할 때의 크기에 대한 정보를 확인하려면 `-benchmem` 옵션을 사용하면 된다.

```bash
go test -v -run="none" -bench=. -benchtime=3s -benchmem
```
