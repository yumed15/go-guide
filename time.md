# Time

### **Use `time.Time` for instants of time**

Use [`time.Time`](https://golang.org/pkg/time/#Time) when dealing with instants of time, and the methods on `time.Time` when comparing, adding, or subtracting time.

```go
// BAD
func isActive(now, start, stop int) bool {
  return start <= now && now < stop
}
```

```go
// GOOD
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```

### **Use `time.Duration` for periods of time**

Use [`time.Duration`](https://golang.org/pkg/time/#Duration) when dealing with periods of time.

```go
// BAD
func poll(delay int) {
  for {
    // ...
    time.Sleep(time.Duration(delay) * time.Millisecond)
  }
}

poll(10) // was it seconds or milliseconds?
```

```go
// GOOD
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```

### Getting Next Day

Going back to the example of adding 24 hours to a time instant, the method we use to add time depends on intent. If we want the same time of the day, but on the next calendar day, we should use [`Time.AddDate`](https://golang.org/pkg/time/#Time.AddDate). However, if we want an instant of time guaranteed to be 24 hours after the previous time, we should use [`Time.Add`](https://golang.org/pkg/time/#Time.Add).

```go
newDay := t.AddDate(0 /* years */, 0 /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```

\
\
