# A Tour of Go: Concurrency

## Goroutines
A goroutine is a lightweight thread managed by the Go runtime.
```Go
	go f(x, y, z)
```
The evaluation of `f`, `x`, `y`, and `z` happens in the current goroutine. The execution happens in a new goroutine. Like threads, goroutines run in the same address space. Thusly, you'll need the `sync` package for thread synchronization works.

## Channels
Channels are a typed conduit through which you can send and received typed values with the channel operator: `<-`:
```Go
	ch <- v		// send v to channel ch.
    v := <-ch	// recv from ch & assign to v
```
Like maps and slices, channels must be created with `make`. You can think of `make` as a sort of `malloc`:
```Go
	ch := make(chan int)
```
By default, sends and receives block until the other side is erady. This allows goroutines to synchronize w/o explicit lokcs or condition variables
```Go
package main
import "fmt"
func sum(a []int, c chan int) {
	sum := 0
    for _, v := range a {
    	sum += v
    }
    c <- sum // send sum to c
}
func main() {
	a := []int{7, 2, 8, -9, 4, 0}
    c := make(chan int)
    go sum(a[:len(a)/2], c)
    go sum(a[len(a)/2:], c)
    x, y := <-c, <-c  // block until received both vals from same chan
    fmt.Println(x, y, x+y)
}
```
Channels can be buffered. This allows sends to block ONLY when the buffer is full.
```Go
	ch := make(chan int, 100)
``` 
In the below example, we set the channel buffer size to 1. The first send fills the buffer. The second send is blocked because the buffer is full (until we receive it... which will never happen in this case, as you'll see):
```Go
package main
import "fmt"
func main() {
	c := make(chan int, 1)
    c <- 1
    c <- 2
    // never reach here
    fmt.Println(<-c)
    fmt.Println(<-c)
}
``` 
This situation is a specific case of deadlock. To prevent this deadlock, make sure your recv's are interspersed with the sends.

Did you know that channels are also iterables? That means they can be iterated with `range` in a for-loop:
```Go
	for i := range c  // recv values from channel c, assign to i, until c is closed
```
To close a channel, just perform `close(c)`.

## Select
The `select` statement lets a goroutine wait on _multiple_ communication operations. A `select` blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.
```Go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
    for {
    	select {
        	case c <- x:
            	x, y = y, x+y
            case <- quit:
            	fmt.Println("quit")
                return
        }
    }
}

func main() {
	c := make(chan int)
    quit := make(chan int)
    go func() {
    	for i := 0; i < 10; i++ {
        	fmt.Println(<-c)
        }
        quit <- 0
    }()
    fibonacci(c, quit)
}
```
Select allows channels to act like communication/event channels. In the `select` block, you setup channel "handlers" in the form of `case` expressions that listen to events from channels.

The `default` case of `select` is run when no other cases are ready (i.e. the channels are still blocked)
```Go
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(100 * time.Millisecond)
    boom := time.After(500 * time.Millisecond)
    for {
        select {
        case <-tick:
            fmt.Println("tick.")
        case <-boom:
            fmt.Println("BOOM!")
            return
        default:
            fmt.Println("    .")
            time.Sleep(50 * time.Millisecond)
        }
    }
}
```

