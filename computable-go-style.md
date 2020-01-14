# Use Go Like So

*COGS* - COmputable Golang Style

## Introduction
The goal of this document is to create consistently styled Go code as followed by Computable. 

These are general guidelines, for further in-depth resources please see the following:

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [Common Go Mistakes](https://github.com/golang/go/wiki/CodeReviewComments)

## Tooling (optional)
Vscode users may get [annoying linting issues](https://github.com/golang/lint/issues/389)] via the default linter.
An improvement over the default linter is using [GolangCI-Lint](https://github.com/golangci/golangci-lint)

Can be installed with the following command: 
`go get -u github.com/go-delve/delve/cmd/dlv`

Please follow the repo instructions to setup correctly.

## Project Structure
For further detailed resources see the following:

1. [How to structure your Go apps by Kat Zien (video)](https://youtu.be/oL6JBUk6tj0)
2. [Best practices for Industrial Programming by Peter Bourgon (video)](https://youtu.be/PTE4VJIdHPg)
3. [Golang project layout](https://github.com/golang-standards/project-layout)

`/cmd`
* Main application for this project 
* Don't put a lot of code in the application directory. If the code can be imported and used in other projects, it should live in `/pkg`.
* It is common to have small `main()` functions. e.g.

```go
func main() {
  if err := run(); err != nil {
    fmt.Printf(os.Stderr, "%v", err)
    os.Exit(1)
  }
}
```

`/pkg`
* Code that's okay to be used by external applications.
* Specifically we will be grouping packages by context as related by Domain Driven Design.

`/internal`
* Private applications and library code. 
* Code you don't want others importing.
* This layout pattern is enfored by the Go compiler.
* You can have more than one internal directory at any level in the tree.

`/vendor`
* Application dependencies

`/configs`
* Configuration files

## Domain Driven Design
The goal of DDD is to create a ubiquitous language allowing easy communication between cross-functional groups.
It creates clarity by standardizing the use of words with pre-agreed meanings.

It also informs us how to structure our projects. This is known as "grouping by context."
Our project structure will follow Domain Driven Design as best described by Kat Zien [here](https://youtu.be/oL6JBUk6tj0?t=982).

Every repository with be accompanied with a `D3.md` featuring Ubiquitous Language that should be enforced by the package structure and actual code itself.

## Style
### Group Similar Declarations

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go

const a = 1
const b = 2



var a = 1
var b = 2



type Area float64
type Volume float64
```

</td><td>

```go
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```
</td></tr>
</tbody></table>

Do not group unrelated declarations.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
  ENV_VAR = "MY_ENV"
)
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

const ENV_VAR = "MY_ENV"
```

</td></tr>
</tbody></table>

### Import Group Ordering

There should be two import groups:
* Standard library
* Everything Else

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
  "fmt"
  "os"

  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>

### Reduce Nesting
Code should reduce nesting where possible. This can be done by handling special cases/errors early.
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

And avoiding unnecessary Else's.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if ok {
  err := something()
  if err != nil {
    return errors.Wrap(err, "something")
  }
  // do things
} else {
  return errors.New("not ok")
}
```

</td><td>

```go
if !ok {
  return errors.New("not ok")
}

err := something()
if err != nil {
  return errors.Wrap(err, "something")
}
// do things
```

</td></tr>
</tbody></table>

### Don't Panic
If errors occur, the function must return an error and allow the caller
to decide how to handle.
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

</td><td>

```go
func foo(bar string) error {
  if len(bar) == 0 {
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

</td></tr>
</tbody></table>

### Errors strings
Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation.
Errors are usually usually printed following other context. 

E.g. use `fmt.Errorf("something bad")` not `fmt.Errorf("Something bad")`, 
so that `log.Printf("Reading %s: %v", filename, err)` formats without a spurious capital letter mid-message.
### Use field names to initialize Structs

Explicit should be favored over implicit.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

### Reduce scope of variables.

Reduce variable scope where possible. Do not reduce scope if it conflicts with reducing nesting.
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

</td><td>

```go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

</td></tr>
</tbody></table>

### Use Raw String Literals to Avoid Escaping

Go supports [raw string literals](https://golang.org/ref/spec#raw_string_lit),
which can span multiple lines and include quotes. Use these to avoid
hand-escaped strings which are much harder to read.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

### Avoid using `new`

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Declaring empty slices

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := []string{}
```

</td><td>

```go
var t []string
```

</td></tr>
</tbody></table>

