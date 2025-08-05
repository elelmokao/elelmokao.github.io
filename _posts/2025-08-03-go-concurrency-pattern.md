---
title: Go Concurrency Patterns
date: 2025-08-03 00:00:00 +0800
math: true
categories: [Go]
tags: [go, concurrency, learning]
# TAG names should always be lowercase
---
## Go Concurrency Patterns

Go Concurrency 源自於Hoare’s CSP in 1978的想法。

- 他是相互獨立執行計算的組合
- 他是一種structure software的方法，特別是使用clean code 去與真實世界做互動
- 他不是parallelism.

## A boring function

```go
func boring(msg string) {
    for i := 0; ; i++ {
        fmt.Println(msg, i)
        time.Sleep(time.Second)
    }
}
```

像上面的程式碼，每行執行時都需要等待一秒，但是`fmt.Println`都是相互獨立的。

利用Goroutine 可以將他們同時「並發（Concurrency）」，可以想像成是槍擊發出去，不管程式碼內是什麼或是否完成，都會直接進到下一行。

```go
func main() {
    go boring("boring!")
}
```

因此，上面的程式碼會在`Println`前就會結束。

```go
func main() {
    go boring("boring!")
    fmt.Println("I'm listening.")
    time.Sleep(2 * time.Second)
    fmt.Println("I'm leaving")
}
```

透過`time.Sleep`，就可以等到`boring()` 裡`println()`的觸發。

## Goroutines

- 獨立執行的function
- 擁有自己的Stack，且會根據需求成長或縮小。
- 輕量且容易大量執行
- 不是Thread，是Multiplexed dynamically onto threads as needed以確保所有的Goroutines 有在執行。

### Channels: Communications between Goroutines

```go
// declaration of channel
c := make(chan int)
// sending on a channel
c <- 1
// receiving from a channel
value = <- 1
```

## Synchronization

當執行`<- c` 時，會「等待」有東西出現在channel `c`中並將它取出。相反的，執行 `c <-` 時，會「等待」有東西return 並放進channel `c`中。兩者必須同時準備好，不然就會一直等待。意思即為channels both communicate and synchronize. （但可以透過Buffer 移除synchronize的效果）

## Go Approach

Don’t communicate by sharing memory, share memory by communicating.

## Patterns

### Generator: function that returns a channel

Channels are first-class values, just like string or integers.

```go
c := boring("boring!")
for i := 0; i < 5; i++ {
    fmt.Printf("You say: %q\n", <- c) // when receive msg, print out.
}

func boring(msg string) <- chan string {
    c := make(chan string)
    go func() {
        for i := 0; ; i++ {
            c <- fmt.Sprintf("%s %d", msg, i)
            time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
        }
    }
    return c
}
```

### Channels as a handle on a service

```go
func main() {
    joe := boring("Joe")
    ann := boring("Ann")
    for i := 0; i < 5; i++ {
        fmt.Println(<-joe)
        fmt.Println(<-ann)
    }
}
```

Channel 可以作為service handler，上述程式碼中，會先等joe 收到訊息並印出後，再等ann 收到訊息印出，然後再回到joe。

### Multiplexing

```go
func fanIn(input1, input2 <- chan string) <- chan string {
    c := make(chan string)
    go func() { for { c <- <- input1 } }()
    go func() { for { c <- <- input2 } }()
    return c
}
func main() {
    c := fanIn(boring("Joe"), boring("Ann"))
    for i := 0; i < 10; i++ {
        fmt.Println(<- c)
    }
}
```

透過`fanIn()` 我們可以不用等ann 收到訊息，而是誰先有訊息進來就`Println` 誰

### Restoring sequencing

```go
type Message struct {
    str  string
    wait chan bool // make goroutine to wait until its turn
}

func boring(msg string, c chan Message) {
    for i := 0; ; i++ {
        waitForIt := make(chan bool) // shared between all messages
        c <- Message{
            str:  fmt.Sprintf("%s: %d", msg, i),
            wait: waitForIt,
        }
        time.Sleep(time.Duration(rand.Intn(2e3)) * time.Millisecond)
        <-waitForIt // wait for go-ahead before continuing
    }
}

func main() {
    c := make(chan Message)

    go boring("Joe", c)
    go boring("Ann", c)

    for i := 0; i < 5; i++ {
        msg1 := <-c; fmt.Println(msg1.str)
        msg2 := <-c; fmt.Println(msg2.str)

        // Give permission to both messages to proceed
        msg1.wait <- true
        msg2.wait <- true
    }

    fmt.Println("You're both boring; I'm leaving.")
}
```

`boring()` 會在`c` channel 內產生訊息，然後等到`msg.wait <- true` 後，才會結束。這保證了joe 先發送訊息到c 後，會一直等待到ann 也發送了訊息，兩者一起`wait <- true` 後才會接到下一輪循環。

### Select

```go
select {
    case v1 := <- c1:
        fmt.Printf("received %v from c1\n", v1)
    case v2 := <- c2:
        fmt.Printf("received %v from c2\n", v1)
    case c3 <- 23:
        fmt.Printf("send %v to c1\n", 23)
    default:
        fmt.Printf("no one was ready to communicate\n")
}
```

就像是switch，會檢測每個`channel` 情況，若複數個情況符合，則會選擇pseudo-randomly。`default`會在當沒有符合的情況下「立刻」 執行。

### Fan-in using select

```go
func fanIn(input1, input2 <-chan string) <-chan string {
    c := make(chan string)
    go func() {
        for (
            select {
                case s:= <-input1: c <- s
                case s:= <-input2: c <- s
            }
        )
    }()
    return c
}
```

使用select，讓`fanIn()` 可以每當有`input1` or `input2` 有新訊息時，迅速傳進`c` channel 裡。

### Timeout using select

```go
func main() {
    c := boring("Joe")
    for {
        select {
        case s := <- c: fmt.Println(s)
        case <- time.After(1 * time.Second):
            fmt.Println("Timeout")
            return
        }
    }
}
```

在檢測時，若`c` channel在一秒內沒有收到新的回覆，則timeout。

### Timeout for whole conversation using select

我們也可以設定開始檢測到直接timeout 的時間: 

```go
func main() {
    c := boring("Joe")
    timeout := time.After(1 * time.Second)
    for {
        select {
        case s := <- c: fmt.Println(s)
        case <- timeout:
            fmt.Println("Timeout")
            return
        }
    }
}
```

### Quit channel

```go
quit := make(chan bool)
c := boring("Joe", quit)
for i := rand.Intn(10); i >= 0; i-- {fmt.Println(<-c)}
quit <- "Bye!"
fmt.Printf("Joe says: %q\n", <- quit)

select {
case c <- fmt.Sprintf("%s: %d", msg, i)
case <- quit: 
    cleanup()
    quit <- "See you"
    return
}
```

quit 會在接受到`"Bye!"` 後，清除，並在塞入`"See you"` 的message。

### Daisy-chain

```go
func f(left, right chan int) { left <- 1 + <- right }

func main() {
    const n = 100000
    leftmost := make(chan int)
    right := leftmost
    left  := leftmost
    for i = 0; ; < n; i++ {
        right = make(chan int)
        go f(left, right)
        left = right
    }
    go func(c chan int) { c <- 1 }(right)
    fmt.Println(<-leftmost)
}
```

透過這個daisy chain，他可以將右值+1傳遞到左值，重複10000遍。所以初始值為1下，最後會是100001。

## Example: optimisation of Google Search

最一開始的構想：Send query to web search, image search, youtube, maps, … then mix all results.

```go
// version 1
var (
    Web = fakeSearch("web")
    Image = fakeSearch("image")
    Video = fakeSearch("video")
)

type Search func(query string) Result

func fakeSearch(kind string) Search {
    return func(query string) Result {
        time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
        return Result(fmt.Sprintf("%s result for %q", kind, query))
    }
}

func Google(query string) (results []Result) {
    results = append(results, Web(query))
    results = append(results, Image(query))
    results = append(results, Video(query))
    return
}

func main() {
    results := Google("Golang")
}
```

Version 1 會逐步搜尋Web → Image → Video

```go
// version 2: Use goroutine for research
func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() {c <- Web(query)} ()
    go func() {c <- Image(query)} ()
    go func() {c <- Video(query)} ()
    
    for i := 0; i < 3; i++ {
        result := <- c
        results = append(results, result)
    }
    return
}
```

同步搜尋Web, Image, Video，然後當`c` channel收到訊息時放入results。

```go
// version 2.1
func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() {c <- Web(query)} ()
    go func() {c <- Image(query)} ()
    go func() {c <- Video(query)} ()
    
    timeout := time.After(80 * time.Millisecond)
    for i := 0; i < 3; i++ {
        select {
        case result := <- c:
            results = append(results, result)
        case  <- timeout:
            fmt.Println("timeout")
            return
        }
    }
    return
}
```

2.1 版中，加上了當超過80秒，後面的訊息都會被截斷無法接收。

解法：Replicate servers, send requests to multiple replicas, and then use the first response

→ 透過複數個search server 減少latency

```go
// version 3.0
func first(query string, replicas ...Search) Result {
    c := make(chan Result)
    searchReplica := func(i int) { c <- replicas[i](query) }
    for i := range replicas {
        go searchReplica(i)
    }
}
func Google(query string) (results []Result) {
    c := make(chan Result)
    go func() {c <- First(query, Web1, Web2))} ()
    go func() {c <- First(query, Image1, Image2))} ()
    go func() {c <- First(query, Video1, Video2))} ()
    
    timeout := time.After(80 * time.Millisecond)
    for i := 0; i < 3; i++ {
        select {
        case result := <- c:
            results = append(results, result)
        case  <- timeout:
            fmt.Println("timeout")
            return
        }
    }
    return
}
```

---

Ref: 

[https://www.youtube.com/watch?v=f6kdp27TYZs](https://www.youtube.com/watch?v=f6kdp27TYZs)

[https://web.archive.org/web/20150904025302/http://concur.rspace.googlecode.com/hg/talk/concur.html#](https://web.archive.org/web/20150904025302/http://concur.rspace.googlecode.com/hg/talk/concur.html#slide-5)
