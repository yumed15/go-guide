# Function Types

_= type to identify a function signature_



Example 1

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
txs, err := b.TymebankClient().GetTransactions(ctx, tymebank.WithTimeRange(fromDate, toDate))
```
{% endcode %}



Example 2

```go
package main

import "fmt"

type area func(int, int) int

func main() {
    areaF := getAreaFunc()
    print(2, 3, areaF)

}

func print(x, y int, a area) {
    fmt.Printf("Area is: %d\n", a(x, y))
}

func getAreaFunc() area {
    return func(x, y int) int {
        return x * y
    }
}
```
