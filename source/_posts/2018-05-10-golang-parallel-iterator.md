---
title: golang-parallel-iterator
date: 2018-05-10 17:02:10
tags:
---

## Parallel Iterator

As well as we know, if we has a component can be visit each item of a slice or array, we called the component is `iterator`. so, `parallel iterator` mean the iterator can be parllel vistit item.



## Parllel Iterator for Golang

```go
// @Example 1:
// example of iterator
 mySlice := []int{1,2,3,4,5}
 for k,v := range mySlice {
     log.Printf("index:%d item:%d\n",k,v)
 }
```

In the exmaple 1, its very fast,let we change the logic to:

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

The blow example is a simple web fetcher for get website response and save it to database. the method `Fetch` is very slow(beacuse the request is http). we conside each request need N seconds, we need fetch M urls, so the total fetch time is N * M(O(N)).

we can easy to reduce overall processing time with goroutine. 

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

We parllel to fetch each url current, the total fetch time is equal the max of thoose.  Gorountine looks like very powerful to solove  the issue,but it's still has another problem.

We need N gorontinues to process each url, we can't control gorontinue's number(), its very danger, we need find a way to control max gorontinue's number limit.

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

In the last example. we use gorontinue for parllel and control concurrence with  `WaitGroup`.


### More Examples

* if we need load remote resource of each item when i load a list from database.


### Best Practice

1. load list from storage engine(redis/RDMS/MongoDB).
2. apply Parllel Iterator when we need still call remote resource(other network behavior) for each item。

### References

* [Wait Group of Golang](https://golang.org/pkg/sync/#WaitGroup)


































