# A Tour of Go: Basics

## Hello world
```Go
package main

import "fmt"

func main() {
	fmt.Println("Hello world!")
}
```
Programs start in package `main`. They can import packages through import paths. E.g. `"fmt"` and `"math/rand"`:
```Go
package main

import (
	"fmt"
    "math/rand"
)

func main() {
	rand.Seed(2)
    fmt.Println("My fav num is ", rand.Intn(10))
}
```
## Functions, Variables, & Types
### Functions
```Go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
    y = sum - x
    return
}

func main() {
   	fmt.Println(split(17))
}
```
The func `split` takes a parameter `sum` of type `int` and returns a "tuple" of `x` and `y`, both are of type `int`.

### Variables
Variables can be declared in two ways:
```Go
var i, j int  # = 1, 2

k := 3  # "short" assignment
```
### Basic Types

	bool
    
    string
    
    int		int8	int16	int32	int64
    uint	uint8	uint16	uint32	uint64	uintptr
    
    byte // alias for uint8
    rune // alias for int32 (unicode code point)
	
    float32	float64
    complex64 complex128

## Flow control
### For
```Go
package main
import "fmt"
func main() {
	sum := 0
    for i :=0; i < 10; i++ {
    	sum += i
    }
    fmt.Println(sum)
}
```
Like in C/Java, you can leave out the pre/post-allocation of the for-loop:
```Go
sum := 1
for ; sum < 1000; {
	sum += sum
}
```
You can also remove the semicolons, making it look more like a **while** loop
```Go
sum := 1
for sum < 1000 {
	sum += sum
}
```
Like the **while** loop, omitting the condition --> inifinite while loop:

	for {
    }

You can combine the `range` clause with the `for` clause to iterate elements in a slice/map.

### If
```Go
if x < 0 {
	fmt.Println(sqrt(-x) + "i")
} else {
	fmt.Println(sqrt(x) + "i")
}
```
Similar to C, you can perform assignment in an if-statement (this only works with short assignment):
```Go
if v := math.Pow(x, n); v < lim {
	return v
}
```
### Switch
```Go
package main
import (
	"fmt"
    "runtime"
)
func main() {
	fmt.Print("Go runs on ")
    switch os := runtime.GOOS; os {
    	case "darwin":
        	fmt.Println("OS X.")
        case "linux":
        	fmt.Println("Linux.")
        default:
        	fmt.Printf("%s.", os)
    }
}
```
In a switch statement, Go evaluates from top to bottom, stopping when a case is matched (unlike in C).

A switch statement with no condition is the same as `switch true`: the `true` is matched with the expression in the following cases:
```Go
package main
import (
	"fmt"
    "time"
)
func main() {
	t := time.Now()
    switch {
    	case t.Hour() < 12:
        	fmt.Println("Good morning!")
        case t.Hour() < 17:
        	fmt.Println("Good afternoon.")
        default:
        	fmt.Println("Good evening...")
    }
}
```
### Defer
```Go
package main
import "fmt"
func main() {
	defer fmt.Println("world")
    fmt.Println("hello")
}
```
In the above example, `fmt.Println("world")` will be executed right after `main()` returns (i.e. when its functional scope exits).

Defers can be stacked!
```Go
package main
import "fmt"
func main() {
	defer fmt.Println("blastoff!")
    for i := 0; i < 10; i++ {
    	defer fmt.Println(i)
    }
    fmt.Println("start countdown")
}
```
Defer is great for decorations (e.g. locks/mutexes or header/footer annotations):
```Go
mu.Lock()
defer mu.Unlock()

printHeader()
defer printFooter()
```
### Panic & Recover
When the function `F` calls `panic`, execution of `F` stops, and any deferred functions in `F` are executed. `F` returns to the caller but behaves like a call to `panic`. This recursive process continues up the stack until all functions in the current goroutine have returned, at which point the program crashes. Panics can also be initiated by runtime errors, such as out-of-bounds array access or stack overflow.

`recover` is a built-in function that regains control of a panicking goroutine. Recover is only useful inside deferred functions; `recover` inside normal execution will return `nil` and have no side-effects.
```Go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```
This will output

    Calling g.
    Printing in g 0
    Printing in g 1
    Printing in g 2
    Printing in g 3
    Panicking!
    Defer in g 3
    Defer in g 2
    Defer in g 1
    Defer in g 0
    Recovered in f 4
    Returned normally from f.

## More types: Structs, Slices, and Maps
### Pointers
```Go
package main
import "fmt"
func main() {
	i, j := 42, 2701
    p := &i;
    fmt.Println(*p)
    *p = 21
    fmt.Println(i)
    p = &j
    *p = *p / 37
    fmt.Println(j)
}
```
The above will output (like in C):

	42
    21
    73

Go, however, has no pointer arithmetic.

### Structs
```Go
package main
import "fmt"

type Vertex struct {
	X int
    Y int
}

func main() {
	v := Vertex{1, 2}
	fmt.Printf("%T(%v) or %#v", v, v, v)
}
``` 
We have constructed a struct literal. You can access struct fields as you do in C or Python:
	
    v.X = 4

**So far we have learned about pointers and structs. We will later see why we ever need to point to a struct in interfaces**

### Arrays
Arrays are statically allocated. If the length of the array will dynamically change, consider slices.
```Go
package main
import "fmt"
func main() {
	var a[2]string
    a[0] = "Hello"
    a[1] = "World"
    fmt.Println(a[0], a[1])
    fmt.Println(a)
}
```
### Slices
Arrays are `[N]T` with `N` as a constant number literal (e.g. `[14]bool`), while slices are to `[]T` (e.g. `[]int`).
```Go
p := []int{2, 3, 5, 7, 9, 11, 13}
ps1 := p[1:4]
``` 
Slice index ranges are left-inclusive, right-exclusive, and 0-indexed.

Slices can also be created with the `make` function. The slice is initialized with '0' bits.
```Go
a := make([]int, 5)  // len(a)=5
b := make([]int, 0, 5)  // len(b)=0, cap(b)=5
b = b[:cap(b)]  // len(b)=5, cap(b)=5
b = b[1:]  // len(b)=4, cap(b)=4
```
Indexing slices in Go is a bit weird. You can think of `len` as a "soft length", and `cap` as a "hard length". Indexing is always with respect to the "hard length" (the `cap`). That is why you can do the following
```Go
b := make([]int, 0, 5)
c := b[:2]
d := c[2:5]
```
even though `c` is length 2... `2:5` refers to index 2 and 5 of the underlying array/vector. Note, however, that `cap(d) < cap(b)`, since the lower index is where capacity is counted from. You can think of slices as just pointers to indices of an underlying array/vector.

Here is a summary to help remember indexing:

* `lo` index - `cap` of _this_ slice is counted from `lo` to `cap` of `make([]T, len, cap)`
* `hi` index - `len` of _this_ slice is counted from `lo` to `hi`

The "zero" value of a slice is `nil`. A `nil` slice has length and capacity 0. Slices are declared as `nil` if not initialized/`make`'d.

#### Appending & Copying
Slices can append elements:
```Go
package main

import "fmt"

func main() {
    var a []int
    //a := make([]int, 0, 0)
    printSlice("a", a)

    // append works on nil slices.
    a = append(a, 0)
    printSlice("a", a)

    // the slice grows as needed.
    a = append(a, 1)
    printSlice("a", a)

    // we can add more than one element at a time.
    a = append(a, 2, 3, 4)
    printSlice("a", a)
}

func printSlice(s string, x []int) {
    fmt.Printf("%s len=%d cap=%d %v\n",
        s, len(x), cap(x), x)
}
```
This will output:

	a len=0 cap=0 []
	a len=1 cap=2 [0]
	a len=2 cap=2 [0 1]
	a len=5 cap=8 [0 1 2 3 4]

The capacity doubles whenever the length is bigger than capacity.

You can also copy one slice into another slice (overwrite a dst slice with a src slice):
```Go
// double the capacity of s
func copy(dst, src []T) int { ... }
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```
### Range
```Go
package main
import "fmt"
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
func main() {
	for i, v := range pow {
    	fmt.Printf("2**d = %d\n", i, v)
    }
}
```
Range allows you to iterate over a slice/map.

### Maps
Maps, like Slices, must be created with `make`. `nil` refers to a 0-valued map.
```Go
package main
import "fmt"

type Vertex struct {
	Lat, Long float64
}

func main() {
	m = make(map[string]Vertex)
    m["Bell Labs"] = Vertex{
    	40.68, -74.40
    }
    fmt.Println(m["Bell Labs"])
}
```
Maps can also be constructed as literals. It looks like a mix between Python and C++:
```Go
m := map[string]Vertex{
	"Bell Labs": {40.68, -74.40},
    "Google": {37.42, -122.08},
}
```
Map assignment is easy:
```Go
m[key] = elem
elem = m[key]
elem, ok = m[key]  # alt: ok is a bool representing presence of key
```
### Function
Functions are values too.
```Go
package main
import (
	"fmt"
    "math"
)
func main() {
	hypot := func(x, y float64) float64 {
    	return math.Sqrt(x*x + y*y)
    }
    fmt.Printf("%T, %v", hypot, hypot)
}
```
Returns

	func(float64, float64) float64, 0x20200
    
`0x20200` is the memory location of the function.

Closures:
```Go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
    return func(x int) int {
    	sum += x
        return sum
    }
}

func main() {
	pos, neg := adder(), adder()
    for i := 0; i < 10; i++ {
    	fmt.Println(
        	pos(i),
            neg(-2*i),
        )
    }
}
```
You can see that `adder`, when called, returns a function of type `func(int) int`. Closures have memory/state of inside variables. The variable is in essence bound to the lifetime of the closure object (e.g. `adder`). Thus, closures have 2 purposes:

1. Store an internal variable
2. Return a function that when called can alter the stored internal variable

