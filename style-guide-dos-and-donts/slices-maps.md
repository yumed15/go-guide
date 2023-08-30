# Slices, Maps

Slices and maps contain pointers to the underlying data so be wary of scenarios when they need to be copied.

### Declaring Empty Slices

When declaring an empty slice, prefer

<pre class="language-go"><code class="lang-go"><strong>var t []string
</strong></code></pre>

over

```go
t := []string{}
```

The former declares a nil slice value, while the latter is non-nil but zero-length. They are functionally equivalent—their `len` and `cap` are both zero—but the nil slice is the preferred style.

### **Copying** **Slices and Maps**

Keep in mind that users can modify a map or slice you received as an argument if you store a reference to it.

```go
// BAD
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
trips[0] = ...
```

```go
// GOOD
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// We can now modify trips[0] without affecting d1.trips.
trips[0] = ...
```

### **Returning Slices and Maps**

Similarly, be wary of user modifications to maps or slices exposing internal state.

```go
// BAD
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot returns the current stats.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot is no longer protected by the mutex, so any
// access to the snapshot is subject to data races.
snapshot := stats.Snapshot()
```

```go
// GOOD
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

### [Don't create a pointer to a loop iteration variable](https://tam7t.com/golang-range-and-pointers/)

Slice

```go
var result []interface{}

// i and value are loop iteration values.
for i, item := range mySlice {

  // Bad
  result = append(result, &item) // NEVER create a pointer to any iteration variable.
  
  // Good
  result = append(result, &mySlice[i]) // Alternative, create a pointer using the index

  // Good (by copying)
  temp := item                   // Alternative, first copy the iteration variable.
  result = append(result, &temp) // Then create a pointer to the copy. 
} 
```

Map

```go
var result []interface{}

// key and value are loop iteration values.
for key, value := range myMap {

  // Bad
  result = append(result, &value) // NEVER create a pointer to any iteration variable.
  
  // Good
  temp := value                   // Instead, first copy the iteration variable.
  result = append(result, &temp)  // Then create a pointer to the copy. 
} 
```

### Don't create new variables with append

```go
// Bad
notnew := append(old, add) // notnew might not be a new independent slice, but point to the same array as old, so modification to old can affect notnew.  

// Good
copied := append([]byte(nil), old...) // First do a proper copy
copied = append(copied, add)          // Then append to itself

// Ok, but weird and easy to get wrong
copied := append(old[0:0:0], old...)  // Another way to copy: `old[0:0:0]` is empty slice with same type as old. Note not including the third zero is BAD.
copied = append(copied, add)          // Then append to itself

// Ok, but easy to get wrong
copied := make([]byte, len(old))      // Yet another way to copy; make new SAME LENGTH slice 
copy(copied, old)                     // Copy using the copy command (note it copies the minimum length of copied and old!)
copied = append(copied, add)          // Then append to itself
```

