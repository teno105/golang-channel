# golang-channel

`golang-channel`은 동시성 프로그래밍을 도와주는 채널과 컨텍스를 학습니다.

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

### > 예제
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
fatal error: all goroutines are asleep - deadlock!  // 5.

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

1. 채널에 데이터를 10번 넣습니다.
2. for range 구문을 사용하면 채널에서 데이터를 계속 기다릴 수 있습니다.
3. wg.Wait() 메서드로 작업이 완료되기를 기다립니다. 하지만 for range 구문은 채널에 데이터가 들어오기를 계속 기다리기 때문에 
4. 가 실행되지 않고 모든 고루틴이 멈추게 되어
5. deadlock 이 표시됩니다.

### > 위 예제의 문제를 수정
```go
package main
import (
    "fmt"
    "sync"
    "time"
)

func square(wg *sync.WaitGroup, ch chan int) {
    for n := range ch {     // 2. 채널이 닫히면 종료
        fmt.Printf("number: %d, Square: %d\n", n, n*n)
        time.Sleep(time.Second)
    }
    wg.Done()
}

func main() {
    var wg sync.WaitGroup
    ch := make(chan int)

    wg.Add(1)
    go square(&wg, ch)
    
    for i := 0; i < 10; i++ {
        ch <- i * 2
    }
    close(ch)               // 1. 채널 닫음
    fmt.Println("close(ch)")
    wg.Wait()
}
```
```go
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
close(ch)
```
1. 데이터를 모두 넣고 채널이 더는 필요없기 때문에 close(ch)를 호출해 닫아줍니다.
2. for range에서 데이터를 모두 처리하고 난 다음에 채널이 닫힌 상태이면 for문을 종료합니다.

### 23.1.7 select문
채널에서 데이터가 들어오기를 대기하는 상황에서 만약 데이터가 들어오지 않으면 다른작업을 하거나, 아니면 여러 채널을 동시에 대기하고 싶을 때 어떻게 할까요?
바로 select문을 사용해서 대기하면 됩니다.
```go
select {
case n := <- ch1:
    ...                 // ch1 채널에서 데이터를 빼낼 수 있을 때 실행
case n2 := <- ch2:
    ...                 // ch2 채널에서 데이터를 빼낼 수 있을 때 실행
case ...
}
```
select 문은 위와 같이 여러 채널을 동시에 기다릴 수 있습니다.
하나의 채널에서 데이터를 읽어오면 해당 구문을 실행하고 select문이 종료됩니다.
반복해서 데이터를 처리하고 싶다면 for문과 함께 사용해야 합니다.

### > 예제
```go
package main
import (
    "fmt"
    "sync"
    "time"
)

func square(wg *sync.WaitGroup, ch chan int, quit chan bool) {
    for {
        select {                // 2. ch와 quit 양쪽을 모두 기다림
        case n:= <- ch:
            fmt.Printf("number: %d, Square: %d\n", n, n*n)
            time.Sleep(time.Second)
        case <-quit:
            wg.Done()
            return
        }
    }
}

func main() {
    var wg sync.WaitGroup
    ch := make(chan int)
    quit := make(chan bool)     // 1. 종료 채널

    wg.Add(1)
    go square(&wg, ch, quit)
    
    for i := 0; i < 10; i++ {
        ch <- i * 2
    }

    quit <- true
    wg.Wait()
}
```
```go
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
```
1. quit 종료 채널을 만들어서 square() 루틴을 만들 때 알려줍니다.
2. select문에서 ch와 quit 채널 모두를 기다립니다. ch 태널을 먼저 시도하기 때문에 ch 채널에서 데이터를 읽을 수 있으면 계속 읽습니다.
그래서 10개의 제곱이 모두 출력되고 quit 채널에서 데이터를 읽어온다음 square() 함수가 종료됩니다.

### 23.1.8 일정 간격으로 실행
1초 간격으로 다른 일을 수행해야 한다고 가정해봅시다.
time 패키지의 Tick() 함수로 원하는 시간 간격으로 신호를 보내주는 채널을 만들 수 있습니다.

### > 예제
```go
package main
import (
    "fmt"
    "sync"
    "time"
)

func square(wg *sync.WaitGroup, ch chan int) {
    tick := time.Tick(time.Second)          // 1. 1초 간격 시그널
    terminate := time.After(10*time.Second) // 2. 10초 이후 시그널

    for {                                   // 3. tick, terminate, ch 순서대로 처리
        select {
        case <-tick:
            fmt.Println("Tick")
        case <-terminate:
            fmt.Println("Terminated!")
            wg.Done()
            return
        case n:= <- ch:
            fmt.Printf("number: %d, Square: %d\n", n, n*n)
            time.Sleep(time.Second)
        }
    }
}

func main() {
    var wg sync.WaitGroup
    ch := make(chan int)

    wg.Add(1)
    go square(&wg, ch)
    
    for i := 0; i < 10; i++ {
        ch <- i * 2
    }

    wg.Wait()
}
```
```go
number: 0, Square: 0
number: 2, Square: 4
Tick
number: 4, Square: 16
Tick
number: 6, Square: 36
number: 8, Square: 64
Tick
number: 10, Square: 100
Tick
number: 12, Square: 144
Tick
number: 14, Square: 196
Tick
number: 16, Square: 256
number: 18, Square: 324
Tick
Terminated!
```

1. time.Tick()은 일정 시간 간격 주기로 신호를 보내주는 채널을 생성해서 반환하는 함수입니다. 이 함수가 반환한 채널에서 데이터를 읽어오면, 일정 시간 간격으로 현재 시각을 나타내는 Time 객체를 반환합니다.
2. time.After()는 현재 시간 이후로 일정 시간 경과 후에 신호를 보내주는 채널을 생성해서 반환하는 함수입니다. 이 함수가 반환한 채널에서 데이터를 읽으면, 일정 시간 경과 후에 현재 시각을 나타내는 Time 객체를 반환합니다.
3. select문을 이용해서 tick, terminate, ch 순서로 채널에서 데이터 읽기를 시도합니다.


### 23.1.9 채널로 생산자 소비자 패턴 구현하기
고루틴에서 뮤텍스를 사용하지 않는 방법 중 두 번째 방법인 채널을 이용해서 역할을 나누는 방법을 알아보겠습니다.

### > 예제
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Car struct {
	Body  string
	Tire  string
	Color string
}

var wg sync.WaitGroup
var startTime = time.Now()

func main() {
	tireCh := make(chan *Car)
	paintCh := make(chan *Car)

	fmt.Printf("Start Factory\n")

	wg.Add(3)
	go MakeBody(tireCh)
	go InstallTire(tireCh, paintCh)
	go PaintCar(paintCh)

	wg.Wait()
	fmt.Println("Close the factory")
}

func MakeBody(tireCh chan *Car) {
	tick := time.Tick(time.Second)
	after := time.After(10 * time.Second)

	for {
		select {
		case <-tick:
			// Make a body
			car := &Car{}
			car.Body = "Sports car"
			tireCh <- car
		case <-after:
			close(tireCh)
			wg.Done()
			return
		}
	}
}

func InstallTire(tireCh, paintCh chan *Car) {
	for car := range tireCh {
		// Make a body
		time.Sleep(time.Second)
		car.Tire = "Winter tire"
		paintCh <- car
	}
	wg.Done()
	close(paintCh)
}

func PaintCar(paintCh chan *Car) {
	for car := range paintCh {
		// Make a body
		time.Sleep(time.Second)
		car.Color = "Red"
		duration := time.Now().Sub(startTime)
		fmt.Printf("%.2f Complete Car: %s %s %s\n", duration.Seconds(), car.Body, car.Tire, car.Color)
	}
	wg.Done()
}
```
```go
Start Factory
3.00 Complete Car: Sports car Winter tire Red
4.00 Complete Car: Sports car Winter tire Red
5.00 Complete Car: Sports car Winter tire Red
6.00 Complete Car: Sports car Winter tire Red
7.00 Complete Car: Sports car Winter tire Red
8.00 Complete Car: Sports car Winter tire Red
9.00 Complete Car: Sports car Winter tire Red
10.00 Complete Car: Sports car Winter tire Red
11.01 Complete Car: Sports car Winter tire Red
12.01 Complete Car: Sports car Winter tire Red
Close the factory
```
이와 같이 한쪽에서 데이터를 생성해서 넣어주면 다른 쪽에서 생성된 데이터를 빼서 사용하는 방식을 생산자 소비자 패턴(Producer Consumer Pattern)이라고 합니다.
| 생산자 | 소비자 |
| --- | --- |
| MakeBody() | InstallTire() |
| InstallTire() | PaintCar() |

MakeBody() 루틴 생산자, InstallTire() 루틴은 소비자입니다.
또 InstallTire()는 PaintCar() 루틴에 대해 생산자가 되는 구조입니다.

## 23.2 컨텍스트 사용하기
컨텍스트(context)는 context 패키지에서 제공하는 기능으로 작업을 지시할 때 작업 가능 시간, 작업 취소 등의 조건을 지시할 수 있는 작업 명세서 역할을 합니다.

### 23.2.1 작업 취소가 가능한 컨텍스트
```go
    ctx, cancel := context.WithCancel(context.Background())
```
취소 가능한 컨텍스트를 생성합니다. 상위 컨텍스트를 인수로 넣으면 그 컨텍스트를 감싼 새로운 컨텍스트를 만들어 줍니다.
상위 컨텍스트가 없다면 가장 기본적인 컨텍스트인 context.Background() 를 사용합니다.
리턴값으로 값을 2개를 반환하는데 첫 번째가 컨텍스트 객체이고 두번째가 취소 함수입니다.

### > 예제
```go
package main
import (
    "fmt"
    "sync"
    "time"
    "context"
)

var wg sync.WaitGroup

func main() {
    wg.Add(1)
    ctx, cancel := context.WithCancel(context.Background()) // 1. 컨텍스트 생성
    go PrintEverySecond(ctx)
    time.Sleep(5*time.Second)
    cancel()                    // 2. 취소

    wg.Wait()
}

func PrintEverySecond(ctx context.Context) {
    tick := time.Tick(time.Second)
    for {
        select {
        case <-ctx.Done():      // 3. 취소 확인
            wg.Done()
            return
        case <-tick:
            fmt.Println("Tick")
        }
    }
}
```
```go
Tick
Tick
Tick
Tick
Tick
```
2. main() 함수에서 5초 이후에 취소 함수를 호출해 취소를 알립니다. 그러면 컨텍스트의 Done() 채널에 시스널을 보냅니다.
3. PrintEverySecond() 에서 컨테스트가 완료될 때 Done() 채널에 시그널을 넣기 때문에, 여기서 메시지를 수신하면 고루틴을 종료합니다.

### 23.2.2 작업 시간을 설정한 컨텍스트
```go
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
```

앞의 예제에서 "1. 컨텍스트 생성" 부분에 위의 코드를 넣으면 3초 뒤에 종료합니다.

### 23.2.3 특정 값을 설정한 컨텍스트
```go
    ctx, cancel := context.WithValue(context.Background(), "number", 9)
```
작업을 지시할 때 별도 지시사항을 추가할 수 있습니다.
context.WithValue() 함수를 이용하여 컨텍스트에 값을 설정할 수 있습니다.

### > 예제
```go
package main
import (
    "fmt"
    "sync"
    "context"
)

var wg sync.WaitGroup

func main() {
    wg.Add(1)

    ctx := context.WithValue(context.Background(), "number", 9) // 1. 컨텍스트에 값을 추가
    go square(ctx)

    wg.Wait()
}

func square(ctx context.Context) {
    if v := ctx.Value("number"); v != nil {    // 2. 컨텍스트에서 값을 읽기
        n := v.(int)
        fmt.Printf("Square: %d\n", n*n)
    }
    wg.Done()
}
```
```go
Square: 81
```
1. "number"를 키로 값을 9로 설정한 컨텍스트를 만듭니다. 이렇게 만든 컨텍스트를 square() 함수 인수로 넘겨서 값을 사용할 수 있도록 합니다.
2. 컨텍스트 ctx의 Value() 메서드로 값을 읽어옵니다. Value() 메서드의 반환 타입은 빈 인터페이스입니다. int 타입으로 변환하여 사용합니다.

### Context 심화
취소도 되면서 값도 설정하는 컨텍스트는 어떻게 만들까요?
컨텍스트를 만들 때 항상 상위 컨텍스트 객체를 인수로 넣어줘야 했습니다.
일반적으로 context.Background()를 넣어줬는데, 이미 만들어진 컨텍스트 객체를 넣어줘도 됩니다.
```go
ctx, cancel := context.WithCancel(context.Background())
ctx := context.WithValue(ctx, "number", 9)
ctx := context.WithValue(ctx, "keyword", "Lilly")
```
위와 같은 방법으로 컨텍스트를 여러 번 감싸서 여러 값을 설정할 수 있습니다.
