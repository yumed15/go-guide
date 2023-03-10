# Field tags in marshaled structs

Any struct field that is marshaled into JSON, YAML, or other formats that support tag-based field naming should be annotated with the relevant tag.

The serialized form of the structure is a contract between different systems. Changes to the structure of the serialized form--including field names--break this contract. Specifying field names inside tags makes the contract explicit, and it guards against accidentally breaking the contract by refactoring or renaming fields.

```go
// BAD
type Stock struct {
  Price int
  Name  string
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "UBER",
})
```

```go
// GOOD
type Stock struct {
  Price int    `json:"price"`
  Name  string `json:"name"`
  // Safe to rename Name to Symbol.
}

bytes, err := json.Marshal(Stock{
  Price: 137,
  Name:  "UBER",
})
```

