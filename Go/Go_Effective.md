# Effective Go
## [Embedding](http://golang.org/doc/effective_go.html#embedding)
```Go
type ReadWriter interface {
	Reader
    Writer
}
```
In the above, `ReadWriter`, `Reader`, and `Writer` are interfaces from the `io` package. The above says what it looks like: A `ReadWriter` can do what a `Reader` can do and what a `Writer` can do. It is literally the **union** of the embedded interfaces (which must be disjoint sets of methods). Only interfaces can be embedded within interfaces.

Structs be embedded into structs with more far-reaching implications. In the below example, the `bufio.Reader` and `bufio.Writer` structs implement the analogous interfaces from `io`. We could create a `ReadWriter`:
```Go
type ReadWriter struct {
	reader *Reader
    writer *Writer
}
```
The `ReadWriter` stores a pointer to the interfaces it embeds as its fields. The above can be also written as
```Go
type ReadWriter struct {
	reader *Reader
    writer *Writer
}
```
but to use `ReadWriter.Read` or `ReadWriter.Write`, you'll need to provide "forwarding methods":
```Go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
	return rw.reader.Read(p)
}
func (rw *ReadWriter) Write(p []byte) (n int, err error) {
	return rw.writer.Write(p)
}
```
Of course, in both situations, you'll need initialize `Reader` and `Writer` first for them to be used in `ReadWriter`.

Embedding can also serve as a simple convenience. The next example shows an embedded field alongside a regular, named field:
```Go
type Job struct {
	Command string
    *log.Logger
}
``` 
The `Job` type now has the `Log`, `Logf` and other methods of `log.Logger`. You don't need to give `*log.Logger` a name.

	Job.Log("Starting now...")

The `Logger` is a regular field of the `Job` struct, so we can initialize it in the usual way inside the constructor for `Job`:
```Go
func NewJob(command string, logger *log.Logger) *Job {
	return &Job{command, logger}
}
// or
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
  ```  
If you need to refer to the embedded field directly, the type name of the field (ignoring the package qualifier) serves as the field name.
```Go
func (job *Job) Logf(format string, args ...interface{}) {
	job.Logger.Logf("%q: %s, job.Command, fmt.Sprintf(format, args...))
}
```
## `new` vs `make`
`new` initializes everything to 0, and should be used for structs.

`make` should only be used for slices, maps, and channels.

## [text/template](http://golang.org/pkg/text/template/)
### Overview
Templates are executed by applying them to a data structure. Execution of the template walks the structure and sets the cursor, represented by a period '.' (called the dot) to the value at the current location in the structure as execution proceeds.

Actions - data evaluations or control structures - are delimited by "{{" and "}}"; all text outside the actions is copied unchanged.

Here is a trivial example that prints "17 items are made of wool"
```Go
package main
import "text/template"

const tmplString = "{{.Count}} items are made of {{.Material}}"

type Inventory struct {
	Material string
    Count uint
}

func main() {
	sweaters := Inventory{"wool", 17 }
    
	tmpl, err := template.New("test").Parse(tmplString)
    if err != nil { panic(err) }
    err = tmpl.Execute(os.Stdout, sweaters)
    if err != nil { panic(err) }
}
```
### Actions

* `{{ /* a comment */ }}`
* `{{ pipeline }}`
* `{{if pipeline}} T1 {{end}}`
* `{{range pipeline}} T1 {{end}}`
* `{{range pipeline}} T1 {{else}} T0 {{end}}`
* `{{template "name"}}`
* `{{template "name" pipeline}}`
* `{{with pipeline}} T1 {{end}}`
* `{{with pipeline}} T1 {{else}} T0 {{end}}`
    
### Arguments

* Go type literal
* `nil`
* '.'
* `$variableName`
* `.Field` or `$x.Field`
* `.Method` or `$x.Method`
* `fun` (invokes `fun()`)
* `print (.F1 arg1) (.F2 arg2)`
* `(.StructValueMethod "arg").Field`

### Pipeline

* Argument
* `.Method [Argument...]`
* `functionName [Argument...]`

A pipeline may be "chained" by separating a sequence of commands with the pipe character: `'|'`. In a chained pipeline, the result of each command is passed as the last argument of the following command.

All of the following produces the quoted word "output":

    {{"\"output\""}}
        A string constant.
    {{`"output"`}}
        A raw string constant.
    {{printf "%q" "output"}}
        A function call.
    {{"output" | printf "%q"}}
        A function call whose final argument comes from the previous
        command.
    {{printf "%q" (print "out" "put")}}
        A parenthesized argument.
    {{"put" | printf "%s%s" "out" | printf "%q"}}
        A more elaborate call.
    {{"output" | printf "%s" | printf "%q"}}
        A longer chain.
    {{with "output"}}{{printf "%q" .}}{{end}}
        A with action using dot.
    {{with $x := "output" | printf "%q"}}{{$x}}{{end}}
        A with action that creates and uses a variable.
    {{with $x := "output"}}{{printf "%q" $x}}{{end}}
        A with action that uses the variable in another action.
    {{with $x := "output"}}{{$x | printf "%q"}}{{end}}
        The same, but pipelined.

### Functions

* `and`
* `call`
  * `"call .X.Y 1 2"` is `dot.X.Y(1, 2)`, where `dot` is the current context object.
* `html`
* `index`
  * `"index x 1 2 3"` is `x[1][2][3]`
* `js`
* `not`
* `or`
* `print`
  * alias for `fmt.Sprint`
* `printf`
  * alias for `fmt.Sprintf`
* `println`
  * alias for `fmt.Sprintln`
* `urlquery`
* `eq, ne, lt, l;e, gt, ge`

### Named Templates

	{{define "T1"}}ONE{{end}}
	{{define "T2"}}TWO{{end}}
	{{define "T3"}}{{template "T1"}} {{template "T2"}}{{end}}
	{{template "T3"}}

### Example of template with http server
```Go
package main

import (
	"flag"  // cmd-line parsing
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address")  // Q=17, R=18
var templ = template.Must(template.New("qr").Parse(templateStr))  // Must panics if error

func main() {
	flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
    	log.Fatal("ListenAndServe:", err)
    }
    
}

func QR(w http.ResponseWriter, req *http.Request) {
	templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{ if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
```    `

