# Performance Optimisation

### Prefer strconv over fmt

When converting primitives to/from strings, `strconv` is faster than `fmt`.

```go
// BAD
// BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

```go
// GOOD
// BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

### Avoid string-to-byte conversion

Do not create byte slices from a fixed string repeatedly. Instead, perform the conversion once and capture the result.

```go
// BAD
// BenchmarkBad-4   50000000   22.2 ns/op
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

```go
// GOOD
// BenchmarkGood-4  500000000   3.25 ns/op
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

### Prefer Specifying Container Capacity

Specify container capacity where possible in order to allocate memory for the container up front. This minimises subsequent allocations (by copying and resizing of the container) as elements are added.

*   **Specifying Map Capacity Hints**\


    Where possible, provide capacity hints when initialising maps with `make()`.\


    ```go
    make(map[T1]T2, hint)
    ```

    \
    Providing a capacity hint to `make()` tries to right-size the map at initialization time, which reduces the need for growing the map and allocations as elements are added to the map.\
    \
    Note that, unlike slices, map capacity hints do not guarantee complete, preemptive allocation, but are used to approximate the number of hashmap buckets required. Consequently, allocations may still occur when adding elements to the map, even up to the specified capacity.

```go
// BAD
// m is created without a size hint; there may be more allocations at assignment time.
m := make(map[string]os.FileInfo)

files, _ := os.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

```go
// GOOD
// m is created with a size hint; there may be fewer allocations at assignment time.
files, _ := os.ReadDir("./files")

m := make(map[string]os.DirEntry, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

*   **Specifying Slice Capacity**

    \
    Where possible, provide capacity hints when initializing slices with `make()`, particularly when appending.\


    ```go
    make([]T, length, capacity)
    ```

    \
    Unlike maps, slice capacity is not a hint: the compiler will allocate enough memory for the capacity of the slice as provided to `make()`, which means that subsequent `append()` operations will incur zero allocations (until the length of the slice matches the capacity, after which any appends will require a resize to hold additional elements).

```go
// BAD
// BenchmarkBad-4    100000000    2.48s
for n := 0; n < b.N; n++ {
  data := make([]int, 0)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

```go
// GOOD
// BenchmarkGood-4   100000000    0.21s
for n := 0; n < b.N; n++ {
  data := make([]int, 0, size)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```
