---
title: trap-of-golang
date: 2018-05-10 11:55:05
tags:
---

### Intro

`Golang` is an nice backend system programming language, but it's also has some trap. we are talking some traps from my experice.


## TOP 1. Easy Concurrent and Hard Safety.

The keyword `go` is very easy than thread pool and async callback solution for concurrently programming. 

Let we showing an example:

```go
results := map[string]string

urls := []string{"url1","url2","url3"}
for _,url := range urls {
    go func(url string){
        results[url] = getData(url)
    }(url)
}
```

We can easy use Goroutines to paralle fetch remote data. but the above example has a bug related to concurrently. You can see a panic error likes `concurrently write map` and **shutdown your application**. 

Why?

Because the standard map type not allow directly share to multi goroutnes. It's not goroutinue level safely type, and nothing happen in at compile time(the behavior can't accept in Rust).

More seriously is the bug very difficult to find in the development phase.

We need to control it very carefully With Lock.

```go
results := make(map[string]string)
resultLock := &sync.Mutex{}

urls := []string{"url1","url2","url3"}
for _,url := range urls {
    go func(url string){
        resultLock.Lock()
        defer resultLock.Unlock()
        results[url] = getData(url)
    }(url)
}

```

Another way is use `sync.Map` to replace `map`, but i don't like it.


## Top 2: Nil Error Not Is Nil Error

In Golang world, the last return value is an error interface if this function might return an error(eg: can't access resource).

But the standard error just is a string value,can't set error code likes C. I am try to define a custom Error struct for error code:

```go
package main

import (
	"log"
	"fmt"
)

type CustomError struct {
    code int
    msg string
}

func NewCustomError(code int,msg string) *CustomError {
    return &CustomError{code:code,msg:msg}
}

func (e *CustomError) Error() string {
    return fmt.Sprintf("code=%d msg=%s",e.code,e.msg)
}

func (e *CustomError) String() string {
    return e.Error()
}


func (e *CustomError) Code() int {
    return e.code
}

func FuncOne(value int) (int,*CustomError) {
    if value < 0 {
        return 0,NewCustomError(101,"value need gte 0")
    }
    return value + 1,nil
}

func FuncTwo(value1 int,value2 int) (int,error) {
    v,err := FuncOne(value1)
    if err != nil {
        return v,err
    }
    return v+value2,nil
}


func main() {

{
 v ,err := FuncTwo(1,1)
 if err != nil {
     log.Println("FuncTwo error of case 1")
     return
 }
 v,err = FuncOne(v)
 if err != nil {
        log.Println("FuncOne error of case 1")
     return
 }
}
}
```

**Can you wait a few mintues for think what's thing will output ? **

[Click here,run it on https://play.glang.org](https://play.golang.org/p/l2VdD5mCMh1)

Is that what you didn't expect? 

In fact, we can found a lot issues in github.

* https://github.com/golang/go/issues/15695

[Click here, if you want to know why](https://golang.org/doc/faq#nil_error)


## Top 3: Don't Forget Close Resource

Golang have a standard HTTP package named `net/http`, it's included a HTTP Client. Let we focus on a example:

```go

func ReadRemoteData() ([]byte,error) {
remoteResourceURL := "http://url"
resp,err := http.Get(remoteResourceURL)
if err != nil {
  return nil,err   
}
return ioutil.ReadAll(resp.Body)
}
```

The above example function `ReadRemoteData` will read another HTTP Server data and return it. Looks like is fine, but it hides a huge problem.

Because HTTP protocol is a TCP Connection,the connection always opened if nobody to close it. So,the above exmaple has Resource Leak bug(TCP Connection).

In some script language(PHP),we don't need care it. Because all resource will automic release in the end. 

PHP is a linear system and hold the whole process for each request. The life cycle of any resource is clear,so we automic release resource. But,Golang world can't match the model, we need careful manage any opened resource.


```go

func ReadRemoteData() ([]byte,error) {
remoteResourceURL := "http://url"
resp,err := http.Get(remoteResourceURL)
if err != nil {
  return nil,err   
}
//we will close the connection when we don't need it.
defer resp.Body.Close()
return ioutil.ReadAll(resp.Body)
}
```
































