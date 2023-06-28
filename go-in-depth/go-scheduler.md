# Go Scheduler

When a Go program starts up, it's given a <mark style="background-color:green;">**Logical Processor (P)**</mark> for every virtual core on the host machine. If you have a processor with multiple threads per physical core (Hyper-Threading), each hardware thread will be represented as a virtual core.&#x20;

![](<../.gitbook/assets/image (2).png>)&#x20;

\=> 4 physical cores + inter core i7 is hyper-threading = 8 virtual cores

Every <mark style="background-color:green;">**P**</mark> is assigned an <mark style="background-color:blue;">**OS Thread (M)**</mark>**.**

Every Go program is given an initial <mark style="background-color:yellow;">**Goroutine (G)**</mark> which is the path of execution for a Go program. Just as OS Threads (<mark style="background-color:blue;">**M**</mark>) are context-switched on and off a core (<mark style="background-color:green;">**P**</mark>), Goroutines (<mark style="background-color:yellow;">**G**</mark>) are context-switched on and off an <mark style="background-color:blue;">**M**</mark>.

2 different run queues:

* <mark style="background-color:red;">**GRQ = Global Run Queue**</mark>
* <mark style="background-color:orange;">**LRQ = Local Run Queue**</mark>

Each <mark style="background-color:green;">**P**</mark> is given a <mark style="background-color:orange;">**LRQ**</mark> that manages the Goroutines assigned to be executed within the context of a P. These <mark style="background-color:yellow;">**Goroutines G**</mark> take turns being context-switched on and off the <mark style="background-color:blue;">**M**</mark> assigned to that <mark style="background-color:green;">**P**</mark>.

The <mark style="background-color:red;">**GRQ**</mark> is for Goroutines that have not been assigned to a <mark style="background-color:green;">**P**</mark> yet.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## Goroutine States

**Waiting**: G is stopped and waiting for something in order to continue. This could be for reasons like waiting for the operating system (system calls) or synchronization calls (atomic and mutex operations). These types of [latencies](https://en.wikipedia.org/wiki/Latency\_\(engineering\)) are a root cause for bad performance.

**Runnable**: This means G wants time on an M so it can execute its assigned instructions. If you have a lot of Gs that want time, then Gs have to wait longer to get time. Also, the individual amount of time any given g gets is shortened as more Gs compete for time. This type of scheduling latency can also be a cause of bad performance.

**Executing**: This means the Goroutine has been placed on an M and is executing its instructions.&#x20;

## Asynchronous System Calls <a href="#asynchronous-system-calls" id="asynchronous-system-calls"></a>

When the OS you are running on has the ability to handle a system call asynchronously, something called the [<mark style="background-color:yellow;">**network poller**</mark>](https://golang.org/src/runtime/netpoll.go) can be used to process the system call more efficiently. This is accomplished by using kqueue (MacOS), epoll (Linux) or iocp (Windows) within these respective OS’s.

\=> the scheduler can prevent Goroutines from blocking the M when those system calls are made. This helps to keep the M available to execute other Goroutines in the P’s LRQ without the need to create new Ms. This helps to reduce scheduling load on the OS.

* Goroutine-1 is executing on the M and there are 3 more Goroutines waiting in the LRQ to get their time on the M. The network poller is idle with nothing to do.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* Goroutine-1 wants to make a network system call, so Goroutine-1 is moved to the network poller and the asynchronous network system call is processed. Once Goroutine-1 is moved to the network poller, the M is now available to execute a different Goroutine from the LRQ. In this case, Goroutine-2 is context-switched on the M.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

* The asynchronous network system call is completed by the network poller and Goroutine-1 is moved back into the LRQ for the P. Once Goroutine-1 can be context-switched back on the M, the Go related code it’s responsible for can execute again. The big win here is that, to execute network system calls, no extra Ms are needed. The network poller has an OS Thread and it is handling an efficient event loop.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Synchronous System Calls <a href="#synchronous-system-calls" id="synchronous-system-calls"></a>

The network poller can’t be used and the Goroutine making the system call is going to block the M.\


* Goroutine-1 is going to make a synchronous system call that will block M1.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

*   The scheduler is able to identify that Goroutine-1 has caused the M to block. At this point, the scheduler detaches M1 from the P with the blocking Goroutine-1 still attached. Then the scheduler brings in a new M2 to service the P. At that point, Goroutine-2 can be selected from the LRQ and context-switched on M2. If an M already exists because of a previous swap, this transition is quicker than having to create a new M.

    \


    <figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
*   The blocking system call that was made by Goroutine-1 finishes. At this point, Goroutine-1 can move back into the LRQ and be serviced by the P again. M1 is then placed on the side for future use if this scenario needs to happen again.

    \


    <figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Work Stealing <a href="#work-stealing" id="work-stealing"></a>

helps to balance the Goroutines across all the P’s so the work is better distributed and getting done more efficiently.

* We have a multi-threaded Go program with two P’s servicing four Goroutines each and a single Goroutine in the GRQ. What happens if one of the P’s services all of its Goroutines quickly?

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

* Half the Goroutines are taken from P2 and now P1 can execute those Goroutines. What happens if P2 finishes servicing all of its Goroutines and P1 has nothing left in its LRQ?

<div align="left">

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

</div>

* P2 finished all its work and now needs to steal some. First, it will look at the LRQ of P1 but it won’t find any Goroutines. Next, it will look at the GRQ. There it will find Goroutine-9.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

* P2 steals Goroutine-9 from the GRQ and begins to execute the work.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Resources:&#x20;

* [https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)
