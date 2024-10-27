# golang-channel

`golang-channel`는 동시성 프로그래밍을 도와주는 채널과 컨텍스를 학습니다.

## 23.1 채널 사용하기
채널(channel)이란 고루틴끼리 메시지를 전달할 수 있는 메시지 큐입니다.

### 23.1.1 채널 인스턴스 생성

```go
var messages chan string = make(chan string)
```
| 구문 | 설명 |
| --- | --- |
| `messages` | 채널 인스턴스 변수 |
| `chan string` | 채널 타입 |
| `chan` | 채널 키워드 |
| `string` | 메시지 타입 |

### 23.1.2 채널에 데이터 넣기
```go
messages <- "This is a message"
```
| 구문 | 설명 |
| --- | --- |
| `messages` | 채널 인스턴스 |
| `<-` | 연산자 |
| `This is a message` | 넣을 데이터 |

### 23.1.3 채널에서 데이터 빼기
```go
var msg string = <- messages
```
| 구문 | 설명 |
| --- | --- |
| `var msg string` | 빼낸 데이터를 담을 변수 |
| `<-` | 연산자 |
| `messages` | 채널 인스턴스 |

### 예제
```go
package main
import (
    "fmt"
    "sync"
    "time"
)

func main() {
    var wg sync.WaitGroup
    ch := make(chan int)        // 채널 생성

    wg.Add(1)
    go square(&wg, ch)          // 고루틴 생성
    ch <- 9                     // 채널에 데이터 넣음
    wg.Wait()                   // 작업이 완료되길 기다림
}

func square(wg *sync.WaitGroup, ch chan int) {
    n := <-ch                   // 데이터 빼옴

    time.Sleep(time.Second)     // 1초 대기
    fmt.Printf("Square: %d\n", n*n)
    wg.Done()
}
```
```go
Square: 81
```

### 23.1.4 채널 크기
일반적으로 채널을 생성하면 크기가 0인 채널이 만들어집니다.
크기가 0이라는 뜻은 채널에 들어온 데이터를 담아줄 곳이 없다는 얘기가 됩니다.
채널에서 데이터를 가져가지 않아서 프로그램이 멈추는 경우를 보겠습니다.

```go
package main
import "fmt"

func main() {
    ch := make(chan int)        // 채널 생성

    ch <- 9                     // 채널에 데이터 넣음
    fmt.Println("Never print")
}
```
```go
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
        /Users/teno/study/golang-channel/cmd/golang-channel/main.go:8 +0x34
make: *** [run] Error 2
```


### 23.1.5 버퍼를 가진 채널
내부에 데이터를 보관할 수 있는 메모리 영역을 버퍼(buffer)라고 부릅니다. 그래서 보관함을 가지고 있는 채널을 버퍼를 가진 채널이라고 말합니다.
make() 함수에서 뒤에 버퍼 크기를 적어주면 됩니다.
```go
var chan string messages = make(chan string, 2)
```
위과 같이 채널을 생성하면 버퍼가 2개인 채널이 만들어집니다.

### 23.1.6 채널에서 데이터 대기
```go
package main
import (
    "fmt"
    "sync"
    "time"
)

func square(wg *sync.WaitGroup, ch chan int) {
    for n := range ch {       // 2. 데이터를 계속 기다림
        fmt.Printf("number: %d, Square: %d\n", n, n*n)
        time.Sleep(time.Second)
    }
    wg.Done()                   // 4. 실행되지 않음
}

func main() {
    var wg sync.WaitGroup
    ch := make(chan int)

    wg.Add(1)
    go square(&wg, ch)
    
    for i := 0; i < 10; i++ {
        ch <- i * 2             // 1. 데이터를 넣음
    }
    wg.Wait()                   // 3. 작업 완료를 기다림
}
```
```go
==> Running golang-channel...
number: 0, Square: 0
number: 2, Square: 4
number: 4, Square: 16
number: 6, Square: 36
number: 8, Square: 64
number: 10, Square: 100
number: 12, Square: 144
number: 14, Square: 196
number: 16, Square: 256
number: 18, Square: 324
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [semacquire]:
sync.runtime_Semacquire(0x14000002101?)
        /usr/local/go/src/runtime/sema.go:71 +0x2c
sync.(*WaitGroup).Wait(0x1400000e0f0)
        /usr/local/go/src/sync/waitgroup.go:118 +0x74
main.main()
        /Users/teno/study/golang-channel/cmd/golang-channel/main.go:27 +0xd0

goroutine 4 [chan receive]:
main.square(0x1400000e0f0, 0x140000200e0)
        /Users/teno/study/golang-channel/cmd/golang-channel/main.go:10 +0xb4
created by main.main in goroutine 1
        /Users/teno/study/golang-channel/cmd/golang-channel/main.go:22 +0x98
make: *** [run] Error 2
```



### 23.1.7 select문

### 23.1.8 일정 간격으로 실행

### 23.1.9 채널로 생산자 소비자 패턴 구현하기

## 23.2 컨텍스트 사용하기
컨텍스트(context)는 context 패키지에서 제공하는 기능으로 작업을 지시할 때 작업 가능 시간, 작업 취소 등의 조건을 지시할 수 있는 작업 명세서 역할을 합니다.

### 23.2.1 작업 취소가 가능한 컨텍스트
```go
    ctx, cancel := context.WithCancel(context.Background())
```

### 23.2.2 작업 시간을 설정한 컨텍스트
```go
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
```

### 23.2.3 특정 값을 설정한 컨텍스트
```go
    ctx, cancel := context.WithValue(context.Background(), "number", 9)
```
