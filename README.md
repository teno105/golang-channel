# golang-channel

`golang-channel`는 동시성 프로그래밍을 도와주는 채널과 컨텍스를 학습니다.

## 23.1 채널 사용하기
채널(channel)이란 고루틴끼리 메시지를 전달할 수 있는 메시지 큐입니다.

### 23.1.1 채널 인스턴스 생성

```go
var messages chan string = make(chan string)
```
messages : 채널 인스턴스 변수
chan string : 채널 타입
chan : 채널 키워드
string : 메시지 타입

### 23.1.2 채널에 데이터 넣기
```go
messages <- "This is a message"
```

### 23.1.3 채널에서 데이터 빼기
```go
var msg string = <- messages
```

### 23.1.4 채널 크기

### 23.1.5 버퍼를 가진 채널

### 23.1.6 채널에서 데이터 대기

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
