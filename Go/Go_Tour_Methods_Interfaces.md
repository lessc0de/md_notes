# A Tour of Go: Methods and Interfaces

## Structs and Methods
C++ has classes. Javascript has prototypes. Go has interfaces.

You can define methods on struct types.
```Go
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
    fmt.Println(v.Abs())
}
```
Let's briefly explain what is going on. We see the dot notation used in the call of `Abs` by `v` of type `Vertex`. The object to the left of the dot (**`v`**`.Abs`) is matched to `Abs`'s declaration signature (`func `**`(v Vertex)`**` Abs() float64`).

There's another way to think about this. `func (v Vertex) Abs() float64 {...}` is _defined_ to a method receiver (`(v Vertex)`). This means that the `Vertex` struct has access, in addition to its internal fields, the function `Abs`. Now, the dot notation is usually a way of referencing a field of a struct (e.g. `v.X`). The dot notation can now be used to reference a method defined on the struct!

The above can be done with struct pointers:
```Go
package main

import (
    "fmt"
    "math"
)

type Vertex struct {
    X, Y float64
}

func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := &Vertex{3, 4}
    v.Scale(5)
    fmt.Println(v, v.Abs())
}
```
The result is the same, only that the method receiver (aka the struct) is passed by pointer, NOT by copy. This distinction is important for one reason: if the method mutates the structure's field, then passing-by-pointer will persist the mutation. Passing-by-value will discard the change once the function scope ends because the method receiver is really a copy of the original structure.

## Interfaces
So we know that a method requires a method receiver to be bound to. Well, an interface is a set of methods. And previously, we said that a structure can bind to a method. Well, not only structures, but also functions! When a function/structure binds to an interface, that object can now call any of the methods within that interface set.
```Go
package main

import (
	"fmt"
    "math"
)

type MyFloat float64

type Abser interface {
	Abs() float64
}

func (f MyFloat) Abs() float64 {
	if f < 0 {
    	return float64(-f)
    }
    return float64(f)
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	var a Abser
    f := MyFloat(-math.Sqrt2)
    v := Vertex{3, 4}
    
    a = f
    a = &v
    
    fmt.Println(a.Abs())
}
```
You can read the above as the following:

1. Create a custom type `MyFloat` that is really just an alias to `float64` (at this point)
2. Create an interface that declares the `Abs()` method which returns `float64`
3. Implement `Abs()` (from the `Abser` interface) for a specific method receiver: `MyFloat`
4. Implement `Abs()` (from the `Abser` interface) for a specific method receiver: `*Vertex`

You can see that a sort of polymorphism come about from this interface. `v` and `f` both implement `Abs` from the `Abser` interface, but they both have different implementations of `Abs`. Polymorphism is really useful when you want to call a common method on many different objects, so long as all those objects belong to the same interface.

Note that interfaces are satisfied implicitly. This implicitness decouples implementation packages from interface packages (kind of like C headers [`.h`] vs sources [`.c`]).

### Useful Interfaces
A well known package is `io`, which defines the `Reader` and `Writer` interfaces. `fmt` defines the `Stringer` interface, which exposes the `string` method used for printing/formatting values. `fmt.Println` looks for this interface to print values.
```Go
package main
import "fmt"
type Person struct {
	Name string
    Age int
}
func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}
func main() {
	a := Person{"Arthur Dent", 42}
    z := Person{"Mashi Shmo", 11}
    fmt.Println(a, z)
}
```
When defining your own objects, it is useful to `import "fmt"` (to grab the `Stringer` interface that exposes `String()`), and define `func (T) String() string` for that object.

The `error` interface is also useful if you want to expose error functionality.
```Go
package main
import (
	"fmt"
    "time"
)
type MyError struc {
	When time.Time
    What string
}
func (e *MyError) Error() string {  // MyError type implements Error() method from error interface
	return fmt.Sprintf("at %v, %s",
    	e.When, e.What)
}
func run() error {
	return &MyError{
    	time.Now(),
        "it didn't work",
    }
}
func main() {
	if err := run(); err != nil {
    	fmt.Println(err)
    }
}
```
Another useful interface is the already mentioned `io.Reader`. This is useful for reading io (files, sockets, compressors, ciphers, etc.)
```Go
package main
import (
	"fmt"
    "io"
    "strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")
    
    b := make([]byte, 8)
    for {
    	n, err := r.Read(b)
        fmt.Printf("n = %v b = %v\n", n, err, b)
        fmt.Printf("B[:n] = %q\n", b[:n])
        if err == io.EOF {
        	break
        }
    }
}
```
A final useful interface is `http.Handler`, an interface that serves HTTP requests.
```Go
package http

type Handler interface {
	ServeHTTP(w ResponseWriter, r *Request)
}

...

package main

import (
	"fmt"
    "log"
    "net/http"
)

type Hello struct{}

func (h Hello) ServeHTTP(w ResponseWriter, r *Request) {
	fmt.Fprint(w, "Hello!")
}

func main() {
	var h Hello
    err := http.ListenAndServe("localhost:4000", h)
    if err != nil {
    	log.Fatal(err)
    }
}
```

