아래는 실습 순서에 맞춰 다시 작성한 `README.md`입니다.

---

# golang-channel

`golang-channel`는 Golang으로 작성된 간단한 애플리케이션으로, 채널의 사용을 익히기 위한 실습입니다.

# 23.1.1 채널 인스턴스 생성

```go
var messages chan string = make(chan string)
```

# 23.1.2 채널에 데이터 넣기
```go
messages <- "This is a message"
```

