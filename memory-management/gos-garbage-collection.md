# Go's Garbage Collection

The Go runtime uses a garbage collector to periodically scan the program's heap (the area of memory where objects are allocated), looking for objects that are no longer reachable. Once the garbage collector determines that an object is no longer reachable, it will release the memory associated with that object and make it available for reuse.

* uses a <mark style="background-color:yellow;">**concurrent**</mark>, <mark style="background-color:yellow;">**stop-the-world algorithm mark**</mark> (=while the garbage collector is running, the program is paused and cannot continue execution.) <mark style="background-color:yellow;">**and sweep garbage collector**</mark>
  *   **Mark phase**: _Identify and mark the objects that are no longer needed by the program._

      **Sweep phase**: _For every object marked “unreachable” by the mark phase, free up the memory to be used elsewhere._
* uses a technique called "<mark style="background-color:yellow;">**tracing**</mark>" to determine which objects are still reachable.
* starts with a set of "root" objects, such as global variables, and then "traces" the pointers from these objects to other objects, marking all of the objects that are reachable. Any objects that are not marked as reachable are considered garbage and are eligible for collection.
* uses a technique called "<mark style="background-color:yellow;">**generational**</mark>" garbage collection, which means that the garbage collector divides the heap into different generations, and it focuses on collecting objects from the older generations first.\


## Mark Phase

When a collection starts, the collector runs through three phases of work:

* Mark Setup - STW
* Marking - Concurrent
* Mark Termination - STW

**Mark Setup - STW**

* When a collection starts, the first activity that must be performed is turning on the Write Barrier. The purpose of the Write Barrier is to allow the collector to maintain data integrity on the heap during a collection since both the collector and application goroutines will be running concurrently.
* In order to turn the Write Barrier on, every application goroutine running must be stopped. This activity is usually very quick, within 10 to 30 microseconds on average. That is, as long as the application goroutines are behaving properly.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**Marking - Concurrent**

* Once the Write Barrier is turned on, the collector commences with the Marking phase.
* The collector takes 25% of the available CPU capacity for itself. The collector uses Goroutines to do the collection work.
* The Marking phase consists of marking values in heap memory that are still in-use. This work starts by inspecting the stacks for all existing goroutines to find root pointers to heap memory. Then the collector must traverse the heap memory graph from those root pointers.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

**Mark Termination - STW**

* Once the Marking work is done, the next phase is Mark Termination. This is when the Write Barrier is turned off, various clean up tasks are performed, and the next collection goal is calculated.\


## Pacing

* The collector has a pacing algorithm which is used to determine when a collection is to start. The algorithm depends on a feedback loop that the collector uses to gather information about the running application and the stress the application is putting on the heap. Stress can be defined as how fast the application is allocating heap memory within a given amount of time. It’s that stress that determines the pace at which the collector needs to run.



Resources:

* [https://www.golinuxcloud.com/golang-garbage-collector/](https://www.golinuxcloud.com/golang-garbage-collector/)
* [https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html](https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html)
