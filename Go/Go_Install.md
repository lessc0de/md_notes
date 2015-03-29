# Installing Go
## Install the Go tools
### Linux
Go to [this archive](https://golang.org/dl/) and download the `.tar.gz`. Extract it to `/usr/local`, creating a Go tree in `/usr/local/go`. For example:

	tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
    
The latest version as of 2015-01-28 is `1.4.1`.

If you would like to install in a custom location, notify Go of its path (e.g. if `go` was in `$HOME`:

	export GOROOT=$HOME/go
    export PATH=$PATH:$GOROOT/bin

## Testing the installation
`hello.go`
```Go
package main
import "fmt"
func main() {
	fmt.Println("Hello World!")
}
```
Then run it with `go`:

	$ go run hello.go
    Hello World!

