---
date: "2019-04-25T04:21:58-07:00"
draft: false
title: "Go-Lession"
tags: [go]
series: ["golang"]
categories: [“编码人生”]
toc: true
---

## 启动两个线程，合并输出
```go
package main

import "fmt"

func main() {
	ch1 := make(chan int)
    ch2 := make(chan int)
    
	go func() {
		for i := 0; i < 10; i++ {
			if i%2 == 0 {
				ch1 <- i
			}
		}
    }()
    
	go func() {
		for i := 0; i < 10; i++ {
			if i%2 == 1 {
				ch2 <- i
			}
		}
    }()
    
	tmp := 0
	for i := 0; i < 5; i++ {
		tmp = <-ch1
		fmt.Println(tmp)
		tmp = <-ch2
		fmt.Println(tmp)
	}
}

```
## for-range

~~~golang
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3}
	mp := make(map[int]*int)
	fmt.Println("for range 坑")
	for index, value := range slice {
		mp[index] = &value
	}
	//根本原因在于for range是用一个变量承接mp中的内容的
	for key, value := range mp {
		fmt.Println(key, " ", *value)
	}

	fmt.Println("等价写法")
	var index, value int
	mp2 := make(map[int]*int)
	for index, value = range slice {
		mp2[index] = &value
	}

	for key, value := range mp2 {
		fmt.Println(key, " ", *value)
	}

	fmt.Println("正确写法")
	mp3 := make(map[int]*int)
	for index, value := range slice {
		num := value
		mp3[index] = &num
	}
	//根本原因在于for range是用一个变量承接mp中的内容的
	//因为for range创建的是每个元素的拷贝，而不是直接返回每个元素的引用，如果使用该值变量的地址作为指向每个元素的指针，就会导致错误，在迭代时，返回的变量是一个迭代过程中根据切片依次赋值的新变量，所以值的地址总是相同的，导致结果不如预期。
	for key, value := range mp3 {
		fmt.Println(key, " ", *value)
	}

}

~~~
---
out
~~~golang
for range 坑
0   3
1   3
2   3
3   3
等价写法
0   3
1   3
2   3
3   3
正确写法
0   0
1   1
2   2
3   3
~~~

---

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T04:21:58-07:00|
