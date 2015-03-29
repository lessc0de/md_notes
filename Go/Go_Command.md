# Go command
Go code should be able to be built using the information found in the source itself. It shouldn't need a Makefile or a configuration file that describes the program's dependencies. To achieve this, Go relies on conventions. Most importantly,

* the import path is derived in a known way from the URL of the source code
* the place to store the sources in the local files system is derived in a known way from the import path
* each directory in a source tree corresponds to a single package
* the package is built using only information in the source code

Most packages follow this convention. This style is known as convention over configuration.

## Go's conventions more in depth
### 1. The import path is derived in a known way from the URL of the source code.
For GitHub, the root directory of the repo is identified by the repo's main URL, without the `http://` prefix. Subdirectories are named by adding to that path. For example, the Go example programs are obtained by running

	git clone https://github.com/golang/example
    
and thus the import path for the root directory of that repo is `github.com/golang/example`. The [`stringutil`](https://godoc.org/github.com/golang/example/stringutil) package is stored in a directory, so its import path is `github.com/golang/example/stringutil`.

### 2. The place to store sources in the local file system is derived in a known way from the import path.
Specifically, the first choice is `$GOPATH/src/<import-path>.` If `$GOPATH` is unset, Go will fall back to storing source code alongside the standard Go packages, in `$GOROOT/src/<import-path>`.

By convention, each tree contains:

* a top-level directory named `bin` for holding compiled executables
* a top-level directory named `pkg` for holding compiled packages that can be imported
* a top-level directory named `src` for holding package source files

### 3. Each directory in a source tree corresponds to a single package
This convention ties the fundamental Go unit - the package - to the file system structure. This means that the file system tools become Go package tools. Copying, moving, or deleting a package corresponds to copying, moving, or deleting a directory.

### 4. Each package is built using only the information in the source files
This makes it more likely that the tool will be able to adapt to changing build environments and conditions. For example, if we allowed extra configuration such as compiler flags or command line recipes, then that configuration would need to be updated each time the build tools changed; it would also be inherently tied to the use of a specific tool chain.

## How to write Go code
`GOPATH` is the path or set of paths to "Go workspaces." You should assign a workspace to each Go project you are working on.

	mkdir $HOME/gocode
    export GOPATH=$HOME/gocode
    mkdir -p $GOPATH/src/github.com/ulysseses
    mkdir -p $GOPATH/src/github.com/ulysseses/hello
    mkdir -p $GOPATH/src/github.com/ulysseses/stringutil
    
    edit $GOPATH/src/github.com/ulysseses/hello/hello.go
    edit $GOPATH/src/github.com/ulysseses/stringutil/stringutil.go
    edit $GOPATH/src/github.com/ulysseses/stringutil/stringutil_test.go
    
    go build $GOPATH/src/github.com/ulysseses/stringutil	# verify the package compiles
    go test $GOPATH/src/github.com/ulysseses/stringutil		# verify the package passes all tests
    go build $GOPATH/src/github.com/ulysseses/hello			# verify the program compiles
    go install $GOPATH/src/github.com/ulysseses/stringutil	# install the stringutil package
    go install $GOPATH/src/github.com/ulysseses/hello		# install the hello program (compile [if not already] and link stringutil pkg)
    
    $GOPATH/bin/hello
    ls $GOPATH/pkg/<arch>/github.com/ulysseses
    
## Remote packages
Go can import a remote package. Here is an example with Github:

	$ go get github.com/golang/example/hello
    $ $GOPATH/bin/hello
    Hello, Go examples!

After issuing the above `go get` command, the workspace directory should look like this

    bin/
        hello                           # command executable
    pkg/
        linux_amd64/
            github.com/golang/example/
                stringutil.a            # package object
    src/
        github.com/golang/example/
        .git/                       # Git repository metadata
            hello/
                hello.go                # command source
            stringutil/
                reverse.go              # package source
                reverse_test.go         # test source

As you can see, we got `github.com/golang/example/hello`, but Go automatically fetched `github.com/golang/example/stringutil`, `hello`'s single dependency.

