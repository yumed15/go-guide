# Concurrency vs Parallelism

“Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.” - Rob Pike

Concurrency is a property of the code; parallelism is a property of the running program.

> Concurrency is a _**semantic property of a program or system**_.  Concurrency is when multiple tasks are in progress for overlapping periods of time. Concurrency is a conceptual property of a program or a system, it’s more about how the program or system has been designed. Long story short, concurrency happens when you have context switching between sequential tasks.

Using the same example as Kirill Bobrov uses in [Grokking Concurrency](https://www.manning.com/books/grokking-concurrency), imagine that one cook is chopping salad while occasionally stirring the soup on the stove. He has to stop chopping, check the stove top, and then start chopping again, and repeat this process until everything is done.

As you can see, we only have one processing resource here, the chef, and his concurrency is mostly related to logistics; without concurrency, the chef has to wait until the soup on the stove is ready to chop the salad.

> Parallelism is an _**implementation property**_. It resides on the hardware layer.
>
> Parallelism is about multiple tasks or subtasks of the same task that literally run at the same time on a hardware with multiple computing resources like multi-core processor.

Back in the kitchen, now we have two chefs, one who can do stirring and one who can chop the salad. We’ve divided the work by having another processing resource, another chef.

{% hint style="info" %}
Concurrency can be parallelised but concurrency does not imply parallelism.\
e.g. In a single-core CPU, you can have concurrency but not parallelism.
{% endhint %}

\=> We don't write parallel code, only concurrent code that we hope might be ran in parallel.

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

## Concurrency in Go vs Java

<table><thead><tr><th width="170.33333333333331">Concept</th><th>Go</th><th>Java</th></tr></thead><tbody><tr><td>Multithreading</td><td>through <strong>goroutines</strong></td><td>through threads via <strong>Thread</strong> class or <strong>Runnable</strong> interface</td></tr><tr><td>Memory Space</td><td>goroutines use only <mark style="color:yellow;"><strong>2 KB</strong></mark><strong> of memory space</strong>.</td><td>threads take <mark style="color:yellow;"><strong>2 MB</strong></mark><strong> of memory space</strong></td></tr><tr><td>Communication Coordination</td><td>through built in <mark style="color:yellow;"><strong>primivate channels</strong></mark> which are built to handle race conditions => safe and prevents explicit locking; <br><br>the data structure that is shared between goroutines doesn't have to be locked</td><td><ul><li>Threaded programming uses <mark style="color:yellow;"><strong>locks</strong></mark> in order to access a shared variable. These can to lead to deadlocks and race conditions which are difficult to detect.</li><li>Can only speak to one another through <mark style="color:yellow;"><strong>return values</strong></mark> or <mark style="color:yellow;"><strong>shared (volatile) variables</strong></mark> and are highly costly to build and manage.</li></ul></td></tr><tr><td>Scheduling </td><td>scheduling of goroutines is done by <mark style="color:yellow;"><strong>go runtime</strong></mark> and hence it is quite faster => context switching is faster</td><td>the scheduling of threads is done by <mark style="color:yellow;"><strong>OS runtime</strong></mark> => context switching is slower</td></tr><tr><td>Garbage Collection</td><td>not automatically garbage collected</td><td>Once the thread dies its native memory and stack are freed immediately without needing to be GC. However, the <code>Thread</code> object is like any other object and it lives until it the GC has decided it can be freed e.g. there is no strong reference to it.</td></tr><tr><td></td><td>thousands of goroutines are multiplexed on one or two OS threads.</td><td>if you launch 1000 threads in JAVA then it would consume lot of resources and these 1000 threads needs to be managed by OS. Moreover each of these threads will be more than 1 MB in size</td></tr></tbody></table>
