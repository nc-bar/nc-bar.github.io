# Notes on go

Some notes on the go programming language.

## General

The language is similar to c, but:

- provides garbage collection
- removes pointer arithmetic
- namespaces (modules)
- extensive standard library (http servers, etc.)
- first-class and anonymous functions
- has more useful built-in types: maps, slices, interfaces
- compact iteration over maps, slices, etc
- generic types and interfaces
- native utf-8
- defer
- errors are just values (no try-except-finally exceptions!)
- built in concurrency constructs
- tools (`go`, `gofmt`, `gopls`, etc.)

Usual numeric types:

	int, int8, int16, int32, int64
	uint, uint8, uint16, uint32, uint64
	float32, float64
	complex

The types go to the right:

	var x int
	var f float
	var x, y someType // you can factor types
	func f(x int, f float, x,y someType)

Only one (flexible) keyword for iteration:

	for i = 0; i < n; i++ {}
	for i < n {} // equivalent to while (i < n)
	for {} // equivalent to while (1)
	for index, value := range slice {}
	for k, v := range someMap {}
	etc.

with the `continue` and `break` statements working like they do in `c`.

The function `panic` takes a string, prints it on `stderr` and aborts the program execution.

Some quirks:

- unused variables are an error
- unused imported packages are an error
- standard source code style given by `gofmt`

## Structs

- Similar to c structs and objects in other languages
- Contain related data
- Fields can be accessed (fields with names starting with uppercase are exported)
- Can associate methods to them

Definning a struct type:

	type bookInfo struct {
		name string
		isbn string
		id   int
	}

It's possible to define annonymous structs (without type name):

	anon := struct {
		name         string
		page, offset int
	}{
		name:   "SomeName",
		page:   10,
		offset: 100,
	}

They are useful for structuring data once, without having to define a type; for example when unmarshaling json data from an api.

To instantiate a struct:

	b := bookInfo{
		name: "book name",
		isbn: "xxxxxxxxxx",
		id:   100, // notice the comma
	}

You can ignore the field names:

	b := bookInfo{
		"book name",
		"xxxxxxxxxx",
		100, // notice the comma
	}

If a field is not initiallized, it takes it's zero value.

Structs can have methods, similar to objects in other languages

	func (b bookInfo) Name() string {
		return b.name
	}

As in regular functions, the `b` parameter is passed by value. To use it as a reference, use a pointer:

	func (b *bookInfo) Name() string {
		return b.name
	}

If the method receives a value, the following code works:

	b := bookInfo{
		name: "book name",
		isbn: "xxxxxxxxxx",
		id:   100, // notice the comma
	}
	fmt.Println((&b).Name())

it's equivalent to

	b := bookInfo{
		name: "book name",
		isbn: "xxxxxxxxxx",
		id:   100, // notice the comma
	}
	fmt.Println(b.Name())

If the method expects a pointer and receives a value, the compiler takes the reference of the value (if it's possible) and passes it automatically.


## Slices

They have a more precise definition, but in general are just used as lists. To define a slice:

	var s someType[]

The zero value of a slice is `nil`. To declare and initialize at the same time:

	s := someType[]{} //empty
	t := someType[]{v1, v2, v3}

Sometimes it's useful to create a slice of anon structs:

	s := []struct {
		page   int
		offset int
	}{
		{1, 100},
		{2, 40},
		{10, 0},
	}

To access the i-th element in a slice:

	v = s[i]

You can copy a slice with a similar syntax as in `python` list ranges:

	ss := s[i:n]

with `i` the index of the first element and `n` last element to be copied (is non inclusive). If `i` or `n` are not explicit, they default to `0` and the remaining string respectively.

	ss := s[:]   // entire slice
	ss := s[2:4] // s[2] s[3]
	ss := s[:3]  // s[0] s[1] s[2]
	ss := s[1:]  // s[1] s[2] s[3] ... s[n]

The built-in function `len` returns the length of an slice

	s := []int{1, 2, 3}
	fmt.Prinln(len(s)) // prints 3

With the built in function `append`, you can create a new slice equal to an existing slice plus an element:

	s := []int{1, 2, 3}
	ns := append(s, 4) // ns is 1, 2, 3, 4

Iterate over a slice with the `for` and `range` keywords:

	for i, v := range slice {
		fmt.Println("Value: ", v, "on index: ", i)
	}
	
	for i := range slice {
		fmt.Println("Value: ", slice[i], "on index: ", i)
	}


## Maps

Maps associates a set of keys with a set of values. They are similar to `python` dictionaries. To define map between two types, for example `keyType` and `valueType`,

	var m map[keyType]valueType

The zero value for a map is `nil` and no keys can be added to it. To initialize an empty map:

	m := make(map[keyType]valueType)

To initialize it with some values:

	m := map[keyType]valueType{
		k1: v1,
		...
		kn: vn,
	}

To get the value for a key in a map do:

	value = m[key]

If the key is not defined in the map it will return the zero value of the value type.

It's possible to know if the key was found, by receiving two values:

	value, ok := m[key]
	if ok {
		fmt.Println("Key was found with value: ", value)
	} else {
		fmt.Println("Key not found")
	}

A common pattern is:
	
	if value, ok := m[key]; ok {
		use(value)
	}
	fmt.Println("Key not found")

Or:

	if value, ok := m[key]; !ok {
		fmt.Println("Key not found")
	}
	use(value)

To remove a mapping:

	delete(m, key)

You can iterate over a mapping, receiving in each iteration a key value-pair:

	for key, value := range m {
		fmt.Println("key: ", key, "value: ", value)
	}


## Functions

Functions are defined with the syntax:

	func funcName(parameters) returnTypes {
		// body
	}

where `parameters` are a list of variable definitions (with the usual syntax) and `returnTypes` is either a type or a list of types in the format:

	(type1, type2, ... , typeN)

For example:

	func f() error {
		return error("error")
	}

takes no arguments and returns an error (`nil` denotes the absence of error). But this function:

	func divideThem(x, y float64) (float64, error) {
		if y == 0 {
			return 0, error("con't devide by 0")
		}

		return x / y, nil
	} 

If there is some code which should execute at the end of a function, use `defer`. It's useful for keeping the source code of some statements close, because their functions are closely related, but their execution isn't. For example, in this function from [here](https://go.dev/blog/defer-panic-and-recover):

	func CopyFile(dstName, srcName string) (written int64, err error) {
		src, err := os.Open(srcName)
		if err != nil {
			return
		}
		defer src.Close()

		dst, err := os.Create(dstName)
		if err != nil {
			return
		}
		defer dst.Close()

		return io.Copy(dst, src)
	}

the `defer src.Close()` is kept close to the `os.Open(srcName)` but their execution isn't. It's good to keep those two statements close because it's easier to read and write that code (no need to remember to close the file, or to remember to check if the code you are reading closes all the files). Without `defer`:

	func CopyFile(dstName, srcName string) (written int64, err error) {
		src, err := os.Open(srcName)
		if err != nil {
			return
		}

		dst, err := os.Create(dstName)
		if err != nil {
			return
		} 
		
	 	w, err := io.Copy(dst, src)

		src.Close() // easy to forget!
		dst.Close()
		return w, err
	}


## Interfaces

Similar concepts as `haskell` typeclasses.

They define a set of methods. If a type implements those methods, that type implements that interface.

Just like a `struct`, an interface is a type also. It can be passed as arguments to functions or variables can be declared to have that type (although with some restrictions). Something typed as an interface can represent any value from any type which implements that interface.

A stupid example:

	package main
	
	import (
		"fmt"
	)
	
	type animal interface {
		Eat()
		isAlive() bool
	}
	
	type dog struct {
		name string
		alive bool
	}
	
	func (d dog) Eat() {
		return
	}
	
	func (d dog) isAlive() bool {
		return d.alive
	}
	
	type cat struct {
		name           string
		remainingLives int
	}
	
	func (c cat) Eat() {
		return
	}
	
	func (c cat) isAlive() bool {
		return c.remainingLives > 0
	}
	
	func countLiving(as []animal) int {
		alive := 0
		for _, a := range as {
			if a.isAlive() {
				alive++
			}
		}
		return alive
	}
	
	func main() {
		animals := []animal{
			dog{"teo", false,},
			dog{"chicho", false,},
			dog{"alfie", true,},
			cat{"gata", 0,},
		}
	
		fmt.Println(countLiving(animals))
	}

By definition, the empty interface is implemented by every type.

	type everyType interface{}

A shorthand for `interface{}` is `any`. This are equivalent:

	type everyType interface{}
	type everytype any

To run code depending on the type of an interface value you can use type assertions and type switches.

Type assertions allow to assert the type and get the underlying value of an interface value:

	var i any := "some string"
	v, ok := i.(string)
	// now v is "some string" and ok is true
	
	v, ok = i.(int)
	// v is 0 (int's zero value) and ok is false

If the type assersion is used returning only a value, the program panics if the type is incorrect

	var i any := "some string"
	v = i.(int) // panics

A type switch switches the execution according to the type of an interface value:

	var i any := "some string"
	switch v := i.(type) {
	case string:
		fmt.Println(v)
	case int:
		fmt.Println("Is an int: ", v)
	default:
		fmt.Println("Not an int nor a string")
	}

## Generics

A way to define types and functions receiving types as parameters. Example taken from [here](https://go.dev/doc/tutorial/generics):

	func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
		var s V
		for _, v := range m {
			s += v
		}
		return s
	}

Here, `K` has to be in the `comparable` class (types with `==` and `!=` defined), and `V` is either `int64` or `float64`.

To use a generic function:

	SumIntsOrFloats[string, int64](ints)
	SumIntsOrFloats[string, float64](floats)

A `struct` can also take generic types:

	type vector[T int | float32 | float64] struct {
		x, y T
	}

An interface can be used as a type constraint

	type number interface {
		int | float32 | float64
	}

	type vector[T number] struct {
		x, y T
	}


## Concurrency

Go implements concurrency using goroutines. A goroutine creates a thread which executes a function concurrently.

	go f(x, y, z) 

Runs the function `f` in a new goroutine. First it evaluates `f`, `x`, `y` and `z` in the current thread and then it creates the goroutine.

Goroutines communicate through typed channels. You send values from one end of a `channel` and read them from the other; they are typed because it's only possible to send and read values of that type. The channels are, by default, blocking. Like threads, goroutines execute in the same address space.

Create a channel:

	ch := make(chan int)

Then, you can send and read ints through `ch`.

	ch <- i // i is of type int

	n = <-ch // read int from ch into variable n

Passing a buffer length to `make` you can create a buffered channel:

	ch := make(chan int, 10)

The buffered channel will only block when the buffer is full.

Channels can be closed by the sender using the built-in function `close`:

	ch := make(chan int)
	...
	close(ch)

when an end closes a `channel` it can't send more data through it. To test if a `channel` is closed, receive two parameter:

	n, ok := <-ch

When closed, `ok` will be `false`.

It's possible to iterate over a `channel`. The iteration gives the value read from the `channel` in each iteration. The `for` ends when `ch` is closed.

	for i := range ch {
		...
	}

To wait on a set of channels there's `select`:

	for {
		select {
		case x <- ch:
			fmt.Println("From channel ch: ", x)
		case y <- ch2:
			fmt.Println("From channel ch2: ", x)
		default:
			fmt.Println("No value received")
			time.Sleep(time.Second)
		}
	}

The execution blocks until one of the channels receives an input. To execute code if no value was sent, you can put it on the `default` case.

## Links and biblio

- [Effective go](https://go.dev/doc/effective_go)
- [How to write go code](https://go.dev/doc/code)
- [A tour of go](https://go.dev/tour/)
