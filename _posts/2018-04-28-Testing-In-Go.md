Go is a beautiful language. It makes writing high performing concurrent code as easy as building your website. But a crucial part of writing code is testing it. Go provides a set of tools that allow one to write tests with ease. Let us go into more detail.

Go has a built in testing framework. It is provided by the go test command.

Here's an example.

```go
package strings_test

import (
    "strings"
    "testing"
)

func TestIndex(t *testing.T) {
    const s, sep, want = "chicken", "ken", 4
    got := strings.Index(s, sep)
    if got != want {
        t.Errorf("Index(%q,%q) = %v; want %v", s, sep, got, want)
    }
}

```

## Table Driven Tests

Table driven tests are tests where you specify test case with each test case has input and expected output. This code will make things more clear.

```go
func TestIndex(t *testing.T) {
    var tests = []struct {
        s   string
        sep string
        out int
    }{
        {"", "", 0},
        {"", "a", -1},
        {"fo", "foo", -1},
        {"foo", "foo", 0},
        {"oofofoofooo", "f", 2},
        // etc
    }
    for _, test := range tests {
        actual := strings.Index(test.s, test.sep)
        if actual != test.out {
            t.Errorf("Index(%q,%q) = %v; want %v", test.s, test.sep, actual, test.out)
        }
    }
}
```

### T

The *testing.T argument  is used for error reporting.

```go
t.Errorf("got bar = %v, want %v", got, want)
t.Fatalf("Frobnicate(%v) returned error: %v", arg, err)
t.Logf("iteration %v", i)
```

We can also enable paralle testing by 

```
t.Parallel()
```

We can also control whether a test runs at all.

```
if runtime.GOARCH == "arm" {
    t.Skip("this doesn't work on ARM")
}
```

### Running Tests

We can run tests for a specific package by running ```go test``` in the directory of the specific package.
```shell
$ go test
PASS

$ go test -v
=== RUN TestIndex
--- PASS: TestIndex (0.00 seconds)
PASS
```

### Test Coverage 

We can also get test coverage via the ```go test -cover``` command.

```shell
$ go test -cover
PASS
coverage: 96.4% of statements
ok      strings    0.692s
```

We can also generate coverage profiles that can be interpreted via the cover tool.

```shell
$ go test -coverprofile=cover.out
$ go tool cover -func=cover.out
strings/reader.go:    Len             66.7%
strings/strings.go:   TrimSuffix     100.0%
... many lines omitted ...
strings/strings.go:   Replace        100.0%
strings/strings.go:   EqualFold      100.0%
total:                (statements)    96.4%
```

HTML reports can also be generated via

```
go tool cover -html=cover.out
```

## Conclusion

1. Write tests for unexported functions inside the package
2. Write tests outside the package in a file named _pkg_test.go_
3. Use concurrency primitives to test concurrent code.
