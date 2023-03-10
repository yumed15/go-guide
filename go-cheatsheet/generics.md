# Generics

**Generic functions**

With Generics, you can create functions with types as parameters. Instead of writing separate functions for each type like:

```go
func LastInt(s []int) int {
    return s[len(s)-1]
}

func LastString(s []string) string {
    return s[len(s)-1]
}
```

you can write a function with a type parameter:

```go
func Last[T any](s []T) T {
    return s[len(s)-1]
}
```

You can call a generic function like any other function:

```go
func main() {
    data := []int{1, 2, 3}
    fmt.Println(Last(data))

    data2 := []string{"a", "b", "c"}
    fmt.Println(Last(data2))
}
```
