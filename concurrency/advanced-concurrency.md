# Advanced Concurrency

### Package singleflight <a href="#package-singleflight" id="package-singleflight"></a>

* provides <mark style="color:yellow;">**a duplicate function call suppression mechanism**</mark>**.**
* executes and returns the results of the given function, <mark style="color:yellow;">**making sure that only one execution is in-flight for a given key at a time**</mark>. If a duplicate comes in, the duplicate caller waits for the original to complete and receives the same results.

<pre class="language-go" data-overflow="wrap" data-line-numbers><code class="lang-go">// you have a database with weather information per city and you want to expose this as an API. In some cases you might have multiple users ask for the weather for the same city at the same time.

// => just query the database, and then share the result to all the waiting requests
package weather

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



### Package errgroup

* best described as a `sync.WaitGroup` but where the tasks return errors that are propagated back to the waiter.
* useful when you have <mark style="color:yellow;">**multiple operations that you want to wait for, but you also want to determine if they all completed successfully**</mark>.

<pre class="language-go" data-overflow="wrap" data-line-numbers><code class="lang-go">// you want to lookup the weather for multiple cities at once, and fail if any of the lookups fails.

func Cities(cities ...string) ([]*Info, error) {
<strong>    var g errgroup.Group
</strong>    var mu sync.Mutex
    res := make([]*Info, len(cities)) // res[i] corresponds to cities[i]

    for i, city := range cities {
        i, city := i, city // create locals for closure below
<strong>        g.Go(func() error {
</strong>            info, err := City(city)
            mu.Lock()
            res[i] = info
            mu.Unlock()
            return err
        })
    }
    if err := g.Wait(); err != nil {
        return nil, err
    }
    return res, nil
}

</code></pre>



### Bounded concurrency

* through the use of **semaphores** by keeping track of how many tasks are running, and to block until there is room for another task to start.
* to allow up to 10 tasks to run at once, we create a channel with space for 10 items:&#x20;

`semaphore := make(chan struct{}, 10)`

* to start a new task, blocking if too many tasks are already running, we simply attempt to send a value on the channel: `semaphore <- struct{}{}`
* When a task completes, mark it as such by taking a value out of the channel: `<-semaphore`

<pre class="language-go" data-overflow="wrap" data-line-numbers><code class="lang-go">func Cities(cities ...string) ([]*Info, error) {
    var g errgroup.Group
    var mu sync.Mutex
    res := make([]*Info, len(cities)) // res[i] corresponds to cities[i]
<strong>    sem := make(chan struct{}, 10)
</strong>    for i, city := range cities {
        i, city := i, city // create locals for closure below
<strong>        sem &#x3C;- struct{}{}
</strong>        g.Go(func() error {
            info, err := City(city)
            mu.Lock()
            res[i] = info
            mu.Unlock()
<strong>            &#x3C;-sem
</strong>            return err
        })
    }
    if err := g.Wait(); err != nil {
        return nil, err
    }
    return res, nil
}
</code></pre>
