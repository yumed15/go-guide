# Function Types

### **Function Signatures and Type Aliases**

{% code lineNumbers="true" %}
```go
type GetTransactionsOptsFn func(opts url.Values)

func WithTimeRange(fromDate, toDate time.Time) GetTransactionsOptsFn {
	return func(filterVals url.Values) {
		filterVals.Set("fromCreatedAt", fromDate.In(locale.TimezoneAfricaJohannesburg).Format(time.RFC3339))
		filterVals.Set("toCreatedAt", toDate.In(locale.TimezoneAfricaJohannesburg).Format(time.RFC3339))
	}
}

GetTransactions(ctx context.Context, opts ...GetTransactionsOptsFn) ([]Transaction, error)
```
{% endcode %}

{% code lineNumbers="true" %}
```go
fromDate := time.Now().AddDate(0, 0, -4)
toDate := time.Now().AddDate(0, 0, 1)
txs, err := GetTransactions(ctx, WithTimeRange(fromDate, toDate))
```
{% endcode %}

### **Variadics**

* allow a function to accept any number of parameters.

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

