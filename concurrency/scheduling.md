# Scheduling

Go's mechanism for hosting goroutines is an implementation of what's called an <mark style="color:yellow;">**M:N scheduler**</mark>: which states that <mark style="color:yellow;">**M**</mark> number of goroutines can be distributed over <mark style="color:yellow;">**N**</mark> number of OS threads.

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

When a Go program starts => it is given a logical processor **P** for every virtual core => Every P is assigned an OS thread **M** => Every Go program is also given an initial G which is the path of execution for a Go program. OS threads are context-switched on and off a core, goroutines are context-switched on and off a M.

There are two run queues in the Go scheduler.

* **Global Run Queue (GRQ)**
* **Local Run Queue (LRQ)**

Each P is given given a LRQ that manages the goroutines assigned to be executed within the context of P. These goroutines take turn being context-switched on and off the M assigned to that P. GRQ is for goroutines that have not been assigned to a P yet.

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

When a goroutine is performing an asynchronous system call, P can swap the G off M and put in a different G for execution. However, when a goroutine is performing a synchronous system call, the OS thread is effectively blocked. Go scheduler will create a new thread to continue servicing the existing goroutines in the LRQ.

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Go follows a model of concurrency called the <mark style="color:yellow;">**fork-join model**</mark>:

* fork - at any point in the program, a _**child**_ branch of execution can be split off and run concurrently with its _**parent**_
* join - at some point in the future, the concurrent branches of execution will join back together

<img src="../.gitbook/assets/image (5) (1).png" alt="" data-size="original">
