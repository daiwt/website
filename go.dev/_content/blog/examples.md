---
title: Testable Examples in Go
date: 2015-05-07
by:
- Andrew Gerrand
tags:
- godoc
- testing
summary: How to add examples, which double as tests, to your packages.
---

## Introduction

Godoc [examples](https://golang.org/pkg/testing/#hdr-Examples) are snippets of
Go code that are displayed as package documentation and that are verified by
running them as tests.
They can also be run by a user visiting the godoc web page for the package
and clicking the associated "Run" button.

Having executable documentation for a package guarantees that the information
will not go out of date as the API changes.

The standard library includes many such examples
(see the [`strings` package](https://golang.org/pkg/strings/#Contains),
for instance).

This article explains how to write your own example functions.

## Examples are tests

Examples are compiled (and optionally executed) as part of a package's test
suite.

As with typical tests, examples are functions that reside in a package's
`_test.go` files.
Unlike normal test functions, though, example functions take no arguments
and begin with the word `Example` instead of `Test`.

The [`stringutil` package](https://godoc.org/github.com/golang/example/stringutil/)
is part of the [Go example repository](https://github.com/golang/example).
Here's an example that demonstrates its `Reverse` function:

	package stringutil_test

	import (
		"fmt"

		"github.com/golang/example/stringutil"
	)

	func ExampleReverse() {
		fmt.Println(stringutil.Reverse("hello"))
		// Output: olleh
	}

This code might live in `example_test.go` in the `stringutil` directory.

Godoc will present this example alongside the `Reverse` function's documentation:

{{image "examples/reverse.png"}}

Running the package's test suite, we can see the example function is executed
with no further arrangement from us:

	$ go test -v
	=== RUN TestReverse
	--- PASS: TestReverse (0.00s)
	=== RUN: ExampleReverse
	--- PASS: ExampleReverse (0.00s)
	PASS
	ok  	github.com/golang/example/stringutil	0.009s

## Output comments

What does it mean that the `ExampleReverse` function "passes"?

As it executes the example,
the testing framework captures data written to standard output
and then compares the output against the example's "Output:" comment.
The test passes if the test's output matches its output comment.

To see a failing example we can change the output comment text to something
obviously incorrect

	func ExampleReverse() {
		fmt.Println(stringutil.Reverse("hello"))
		// Output: golly
	}

and run the tests again:

	$ go test
	--- FAIL: ExampleReverse (0.00s)
	got:
	olleh
	want:
	golly
	FAIL

If we remove the output comment entirely

	func ExampleReverse() {
		fmt.Println(stringutil.Reverse("hello"))
	}

then the example function is compiled but not executed:

	$ go test -v
	=== RUN TestReverse
	--- PASS: TestReverse (0.00s)
	PASS
	ok  	github.com/golang/example/stringutil	0.009s

Examples without output comments are useful for demonstrating code that cannot
run as unit tests, such as that which accesses the network,
while guaranteeing the example at least compiles.

## Example function names

Godoc uses a naming convention to associate an example function with a
package-level identifier.

	func ExampleFoo()     // documents the Foo function or type
	func ExampleBar_Qux() // documents the Qux method of type Bar
	func Example()        // documents the package as a whole

Following this convention, godoc displays the `ExampleReverse` example
alongside the documentation for the `Reverse` function.

Multiple examples can be provided for a given identifier by using a suffix
beginning with an underscore followed by a lowercase letter.
Each of these examples documents the `Reverse` function:

	func ExampleReverse()
	func ExampleReverse_second()
	func ExampleReverse_third()

## Larger examples

Sometimes we need more than just a function to write a good example.

For instance, to demonstrate the [`sort` package](https://golang.org/pkg/sort/)
we should show an implementation of `sort.Interface`.
Since methods cannot be declared inside a function body, the example must
include some context in addition to the example function.

To achieve this we can use a "whole file example."
A whole file example is a file that ends in `_test.go` and contains exactly one
example function, no test or benchmark functions, and at least one other
package-level declaration.
When displaying such examples godoc will show the entire file.

Here is a whole file example from the `sort` package:

{{raw `
	package sort_test

	import (
		"fmt"
		"sort"
	)

	type Person struct {
		Name string
		Age  int
	}

	func (p Person) String() string {
		return fmt.Sprintf("%s: %d", p.Name, p.Age)
	}

	// ByAge implements sort.Interface for []Person based on
	// the Age field.
	type ByAge []Person

	func (a ByAge) Len() int           { return len(a) }
	func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
	func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

	func Example() {
		people := []Person{
			{"Bob", 31},
			{"John", 42},
			{"Michael", 17},
			{"Jenny", 26},
		}

		fmt.Println(people)
		sort.Sort(ByAge(people))
		fmt.Println(people)

		// Output:
		// [Bob: 31 John: 42 Michael: 17 Jenny: 26]
		// [Michael: 17 Jenny: 26 Bob: 31 John: 42]
	}
`}}

A package can contain multiple whole file examples; one example per file.
Take a look at the [`sort` package's source code](https://golang.org/src/sort/)
to see this in practice.

## Conclusion

Godoc examples are a great way to write and maintain code as documentation.
They also present editable, working, runnable examples your users can build on.
Use them!
