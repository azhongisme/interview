# defer快照
defer快照指defer语句定义时所捕获的变量的状态
## 失效
1. 匿名函数闭包，变量的值在defer定义后发生变化
```go
func main() {
    x := 0
    defer fmt.Println(x)
    x = 1
    fmt.Println(x)
}
```
2. 引用类型
