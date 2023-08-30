# Functions

**Variadics**

Variadics allow a function to accept any number of parameters.

```go
// `nums` is treated like a slice of int
func sum(nums ...int) int {
    sum := 0
    // iterate through each argument to the function
    for _, n := range nums {
        sum += n
    }
    return sum
}

a := []int{1, 2, 3}
b := []int{4, 5, 6}

all := append(a, b...)     // slices can be expanded with ...
answer := sum(all...)      // each element will be an argument to the function

// same as above
answer = sum(1, 2, 3, 4, 5, 6)    // many arguments
```

