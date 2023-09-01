# Benchmarking, Profiling, Tracing

* **Benchmarking** - focus on a particular piece of code, allowing measurement of time and/or memory information.
* **Profiling** - aggregated data collected through sampling during program (or test) execution. Profiling has no timeline.
* **Tracing** - data collected through events occurring during program (or test) execution. They give a chronological view of program's execution with detailed information about heap, GC, goroutines, core usage, ...

### Running benchmarks

Run tests:

* with benchmarks (time) : <mark style="color:yellow;">`go test ./fibonacci -bench .`</mark>
* with benchmarks (time and memory) : <mark style="color:yellow;">`go test ./fibonacci -bench . -benchmem`</mark>

### Profiling benchmarks

Get profiling data from the benchmarks:

* CPU profiling using <mark style="color:yellow;">`-cpuprofile=cpu.out`</mark>
* Memory profiling using <mark style="color:yellow;">`-benchmem -memprofile=mem.out`</mark>

{% code lineNumbers="true" %}
```bash
go test ./fibonacci \
  -bench BenchmarkSuite \
  -benchmem \
  -cpuprofile=cpu.out \
  -memprofile=mem.out
```
{% endcode %}

### Viewing profiling data

* through command line : <mark style="color:yellow;">`go tool pprof cpu.out`</mark>
* with a browser : <mark style="color:yellow;">`go tool pprof -http=localhost:8080 cpu.out`</mark>

### Tracing

{% code lineNumbers="true" %}
```bash
go test ./fibonacci \
  -bench BenchmarkSuite \
  -trace=trace.out

go tool trace trace.out
```
{% endcode %}

### Getting Memory Statistics

* <mark style="color:yellow;">`runtime.ReadMemStats`</mark> allows you to retrieve memory statistics for the current Go process.

{% code lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	var memStats runtime.MemStats

	runtime.ReadMemStats(&memStats)

	fmt.Printf("Total allocated memory (in bytes): %d\n", memStats.Alloc)
	fmt.Printf("Heap memory (in bytes): %d\n", memStats.HeapAlloc)
	fmt.Printf("Number of garbage collections: %d\n", memStats.NumGC)
}
```
{% endcode %}



Packages:

* Go testing package : [https://golang.org/pkg/testing/](https://golang.org/pkg/testing/)
* Go runtime package : [https://golang.org/pkg/runtime/](https://golang.org/pkg/runtime/)
* Go trace package : [https://golang.org/pkg/runtime/trace/](https://golang.org/pkg/runtime/trace/)
* Go pprof package : [https://golang.org/pkg/runtime/pprof/](https://golang.org/pkg/runtime/pprof/)

Resources:

* [Uber Talk on Profiling and Optimizing Go](https://www.youtube.com/watch?v=N3PWzBeLX2M)
