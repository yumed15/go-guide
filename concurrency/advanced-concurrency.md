# Advanced Concurrency

### Package singleflight <a href="#package-singleflight" id="package-singleflight"></a>

* provides **a duplicate function call suppression mechanism.**
* executes and returns the results of the given function, making sure that only one execution is in-flight for a given key at a time. If a duplicate comes in, the duplicate caller waits for the original to complete and receives the same results.

<pre class="language-go" data-line-numbers><code class="lang-go">package weather

type Info struct {
    TempC, TempF int // temperature in Celsius and Farenheit
    Conditions string // "sunny", "snowing", etc
}

<strong>var group singleflight.Group
</strong>
func City(city string) (*Info, error) {
<strong>    results, err, _ := group.Do(city, func() (interface{}, error) {
</strong>        info, err := fetchWeatherFromDB(city) // slow operation
        return info, err
    })
    if err != nil {
        return nil, fmt.Errorf("weather.City %s: %w", city, err)
    }
    return results.(*Info), nil
}
</code></pre>



