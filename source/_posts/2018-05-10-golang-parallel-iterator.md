---
title: Golang Parllel Iterator
date: 2018-05-10 17:02:10
tags:
---

## Parallel Iterator

As well as we know, if we have a component can visit each item of a slice or array, we called is `iterator`. 

if an `Iterator` can visit more than 1 item same time, we called it is `Parallel Iterator`.

## Parllel Iterator for Golang

```go
// @Example 1:
// example of iterator
 mySlice := []int{1,2,3,4,5}
 for k,v := range mySlice {
     log.Printf("index:%d item:%d\n",k,v)
 }
```

In example 1, it's showing an iterator for an integer slice. It's simple and fast, looks like everything is ok. 

Let me define a `Web Site Fetcher`, the Fetcher can be download a website data.

```go
// @Example 2:
// Simple web fetch component
type WebFetcher struct {
    url string
}

func (c *WebFetcher) Fetch() ([]byte,error) {
    resp,err := http.Get(c.url)
    if err != nil {
        return nil,err
    }
    defer resp.Body.Close()
    return ioutil.ReadAll(resp.Body)
}

func main() {

urls := []string{"url1","url2","url3"}

for _,url := range urls {
    fetcher := &WebFetch{url:url}
    data,err := fetcher.Fetch()
    if err != nil {
        //save data to database
        // ...
    }
}
}
```
{% diagram "Example 2 Work Flow"  %}
digraph G {
graph [
rankdir = "LR"
];
node [
fontsize = "16"
shape = "ellipse"
];
edge [
];

"node0" [
label = "<f0> 1| <f1> HTTP Request | <f2> 2 | <f3> HTTP Request | <f4> 3 |<f5> HTTP Request  | <f6> 4 | <f7> HTTP Request  | <f8> 5 | <f9> HTTP Request"
shape = "record"
];

"node0":f0 -> "node0":f1
"node0":f1 -> "node0":f2
"node0":f2  -> "node0":f3
"node0":f3  -> "node0":f4
"node0":f4  -> "node0":f5
"node0":f5  -> "node0":f6
"node0":f6  -> "node0":f7
"node0":f7  -> "node0":f8
"node0":f8  -> "node0":f9
}
{% enddiagram %}

The above example is a simple web fetcher for getting website response and saves it to a database. The method `Fetch` is very slow(because the request is public HTTP). 

we assume each request need N seconds to get data , the total fetch time is N * M(O(N)) when we need to fetch M URLs.

Don't worry,  we can easy to reduce overall processing time with goroutine.  Let we still focus on code.

```go
// @example 3
for _,url := range urls {
    go func(url string){
        fetcher := &WebFetch{url:url}
        data,err := fetcher.Fetch()
        if err != nil {
            //save data to database
            // ...
        }
    }(url)
}
}

```

{% diagram "Example 3 Work Flow"  %}

 digraph G {
graph [
rankdir = "LR"
];
node [
fontsize = "16"
shape = "ellipse"
];
edge [
];

"node0" [
label = "<f0> 1| <f1> 2|<f2> 3|<f3> 4|<f4> 5"
shape = "record"
];

"node1" [
label = "<f0> Goroutine 1| <f1> Goroutine 2|<f2> Goroutine 3|<f3> Goroutine 4|<f4> Goroutine 5"
shape = "record"
];

"node2" [
label = "<f0> CPU Core 1| <f1> CPU Core 2"
shape = "record"
];

"node3" [
label = "<f0> HTTP Request 1| <f1> HTTP Request 2|<f2> HTTP Request 3|<f3> HTTP Request 4|<f4> HTTP Request 5"
shape = "record"
];

"node0":f0 -> "node1":f0
"node0":f1 -> "node1":f1
"node0":f2 -> "node1":f2
"node0":f3 -> "node1":f3
"node0":f4 -> "node1":f4

"node1":f0 -> "node2":f1
"node1":f1 -> "node2":f0
"node1":f2 -> "node2":f0
"node1":f3 -> "node2":f1
"node1":f4 -> "node2":f1

"node2":f0 -> "node3":f0
"node2":f0 -> "node3":f1
"node2":f1 -> "node3":f2
"node2":f1 -> "node3":f3
"node2":f1 -> "node3":f4

"node3":f0 -> Done
"node3":f1 -> Done
"node3":f2 -> Done
"node3":f3 -> Done
"node3":f4 ->Done

}

{% enddiagram %}


The above example looks very fast. because the method `Fetch` is run in parallel current, the total fetch time is equal to the max of those.  Goroutine looks like very powerful to solve the issue, but it still has another problem.

We need N goroutines to process each URL, we can't control goroutine's number, we need to create a lot goroutine and database resource when we have a lot of URLs, its very danger in production environment, we need to find a way to control max gorontinue's number.

In Golang world, it's also easy.  you need some time to analyze the example.

```go
// @example 4

func Parallel(lstRaw interface{},parallelCount int,callback func(idx int,item interface{})) {
	rt := reflect.TypeOf(lstRaw)
	rv := reflect.ValueOf(lstRaw)
	if rt.Kind() != reflect.Slice && rt.Kind() != reflect.Array {
		panic("can't use non-slice type for parallel")
	}
	l := rv.Len()
	totalBatch := int(math.Ceil(float64(l) / float64(parallelCount)))
	for i := 0 ;i<totalBatch;i++ {
		startOffset := i * parallelCount
		endOffset := (i+1) * parallelCount
		if endOffset > l {
			endOffset = l
		}
		group := &sync.WaitGroup{}
		parallelSize := endOffset - startOffset
		group.Add(parallelSize)
		v := rv.Slice(startOffset,endOffset)
		vl := v.Len()
		for i := 0;i <vl;i++ {
			go func(idx int,item interface{}){
				defer group.Done()
				callback(idx,item)
			}(startOffset + i,v.Index(i).Interface())
		}
		group.Wait()
	}
}

n := 2 //max gorontinue's number

// Amzing! Very easily to parllel iter.
Parallel(urls,n,func(idx int,item interface{}){
     fetcher := &WebFetch{url:item.(string)}
        data,err := fetcher.Fetch()
        if err != nil {
            //save data to database
            // ...
        }
})

```

In the last example. we used goroutine for parllel and control parllel with  `WaitGroup`.

Same as its name, `WaitGroup` is a way to control(Wait these all be done.) a group(more than 1 goroutines). You can be found the detail in [here](https://golang.org/pkg/sync/#WaitGroup).

### More Examples

* If we need load remote resource of each item when i load a list from database.


### Best Practice

1. load list from storage engine(redis/RDMS/MongoDB).
2. apply Parllel Iterator when we need still call remote resource(other network behavior) for each item。

### References

* [Wait Group of Golang](https://golang.org/pkg/sync/#WaitGroup)


































