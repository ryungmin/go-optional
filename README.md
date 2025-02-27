# go-optional [![.github/workflows/check.yml](https://github.com/moznion/go-optional/actions/workflows/check.yml/badge.svg)](https://github.com/moznion/go-optional/actions/workflows/check.yml) [![codecov](https://codecov.io/gh/moznion/go-optional/branch/main/graph/badge.svg?token=0HCVy6COy4)](https://codecov.io/gh/moznion/go-optional) [![GoDoc](https://godoc.org/github.com/moznion/go-optional?status.svg)](https://godoc.org/github.com/moznion/go-optional)

A library that provides [Go Generics](https://go.dev/blog/generics-proposal) friendly "optional" features.

## Synopsis

```go
some := optional.Some[int](123)
fmt.Printf("%v\n", some.IsSome()) // => true
fmt.Printf("%v\n", some.IsNone()) // => false

v, err := some.Take()
fmt.Printf("err is nil: %v\n", err == nil) // => err is nil: true
fmt.Printf("%d\n", v) // => 123

mapped := optional.Map(some, func (v int) int {
    return v * 2
})
fmt.Printf("%v\n", mapped.IsSome()) // => true

mappedValue, _ := some.Take()
fmt.Printf("%d\n", mappedValue) // => 246
```

```go
none := optional.None[int]()
fmt.Printf("%v\n", none.IsSome()) // => false
fmt.Printf("%v\n", none.IsNone()) // => true

_, err := none.Take()
fmt.Printf("err is nil: %v\n", err == nil) // => err is nil: false
// the error must be `ErrNoneValueTaken`

mapped := optional.Map(none, func (v int) int {
    return v * 2
})
fmt.Printf("%v\n", mapped.IsNone()) // => true
```

and more detailed examples are here: [./examples_test.go](./examples_test.go).

## Docs

[![GoDoc](https://godoc.org/github.com/moznion/go-optional?status.svg)](https://godoc.org/github.com/moznion/go-optional)

### Supported Operations

- [Option[T]#IsNone() bool](https://pkg.go.dev/github.com/moznion/go-optional#Option.IsNone)
- [Option[T]#IsSome() bool](https://pkg.go.dev/github.com/moznion/go-optional#Option.IsSome)
- [Option[T]#Unwrap() T](https://pkg.go.dev/github.com/moznion/go-optional#Option.Unwrap)
- [Option[T]#Take() (T, error)](https://pkg.go.dev/github.com/moznion/go-optional#Option.Take)
- [Option[T]#TakeOr(fallbackValue T) T](https://pkg.go.dev/github.com/moznion/go-optional#Option.TakeOr)
- [Option[T]#TakeOrElse(fallbackFunc func() T) T](https://pkg.go.dev/github.com/moznion/go-optional#Option.TakeOrElse)
- [Option[T]#Filter(predicate func(v T) bool) Option[T]](https://pkg.go.dev/github.com/moznion/go-optional#Option.Filter)
- [Option[T]#IfSome(f func(v T))](https://pkg.go.dev/github.com/moznion/go-optional#Option.IfSome)
- [Option[T]#IfSomeWithError(f func(v T) error) error](https://pkg.go.dev/github.com/moznion/go-optional#Option.IfSomeWithError)
- [Option[T]#IfNone(f func())](https://pkg.go.dev/github.com/moznion/go-optional#Option.IfNone)
- [Option[T]#IfNoneWithError(f func() error) error](https://pkg.go.dev/github.com/moznion/go-optional#Option.IfNoneWithError)
- [Option.Map[T, U any](option Option[T], mapper func(v T) U) Option[U]](https://pkg.go.dev/github.com/moznion/go-optional#Map)
- [Option.MapOr[T, U any](option Option[T], fallbackValue U, mapper func(v T) U) U](https://pkg.go.dev/github.com/moznion/go-optional#MapOr)
- [Option.MapWithError[T, U any](option Option[T], mapper func(v T) (U, error)) (Option[U], error)](https://pkg.go.dev/github.com/moznion/go-optional#MapWithError)
- [Option.MapOrWithError[T, U any](option Option[T], fallbackValue U, mapper func(v T) (U, error)) (U, error)](https://pkg.go.dev/github.com/moznion/go-optional#MapOrWithError)
- [Option.FlatMap[T, U any](option Option[T], mapper func(v T) Option[U]) Option[U]](https://pkg.go.dev/github.com/moznion/go-optional#FlatMap)
- [Option.FlatMapOr[T, U any](option Option[T], fallbackValue U, mapper func(v T) Option[U]) U](https://pkg.go.dev/github.com/moznion/go-optional#FlatMapOr)
- [Option.FlatMapWithError[T, U any](option Option[T], mapper func(v T) (Option[U], error)) (Option[U], error)](https://pkg.go.dev/github.com/moznion/go-optional#FlatMapWithError)
- [Option.FlatMapOrWithError[T, U any](option Option[T], fallbackValue U, mapper func(v T) (Option[U], error)) (U, error)](https://pkg.go.dev/github.com/moznion/go-optional#FlatMapOrWithError)
- [Option.Zip[T, U any](opt1 Option[T], opt2 Option[U]) Option[Pair[T, U]]](https://pkg.go.dev/github.com/moznion/go-optional#Zip)
- [Option.ZipWith[T, U, V any](opt1 Option[T], opt2 Option[U], zipper func(opt1 T, opt2 U) V) Option[V]](https://pkg.go.dev/github.com/moznion/go-optional#ZipWith)
- [Option.Unzip[T, U any](zipped Option[Pair[T, U]]) (Option[T], Option[U])](https://pkg.go.dev/github.com/moznion/go-optional#Unzip)
- [Option.UnzipWith[T, U, V any](zipped Option[V], unzipper func(zipped V) (T, U)) (Option[T], Option[U])](https://pkg.go.dev/github.com/moznion/go-optional#UnzipWith)

### JSON marshal/unmarshal support

This `Option[T]` type supports JSON marshal and unmarshal.

If the value wanted to marshal is `Some[T]` then it marshals that value into the JSON bytes simply, and in unmarshaling, if the given JSON string/bytes has the actual value on corresponded property, it unmarshals that value into `Some[T]` value.

example:

```go
type JSONStruct struct {
	Val Option[int] `json:"val"`
}

some := Some[int](123)
jsonStruct := &JSONStruct{Val: some}

marshal, err := json.Marshal(jsonStruct)
if err != nil {
	return err
}
fmt.Printf("%s\n", marshal) // => {"val":123}

var unmarshalJSONStruct JSONStruct
err = json.Unmarshal(marshal, &unmarshalJSONStruct)
if err != nil {
	return err
}
// unmarshalJSONStruct.Val == Some[int](123)
```

Elsewise, when the value is `None[T]`, the marshaller serializes that value as `null`. And if the unmarshaller gets the JSON `null` value on a property corresponding to the `Optional[T]` value, or the value of a property is missing, that deserializes that value as `None[T]`.

example:

```go
type JSONStruct struct {
	Val Option[int] `json:"val"`
}

none := None[int]()
jsonStruct := &JSONStruct{Val: none}

marshal, err := json.Marshal(jsonStruct)
if err != nil {
	return err
}
fmt.Printf("%s\n", marshal) // => {"val":null}

var unmarshalJSONStruct JSONStruct
err = json.Unmarshal(marshal, &unmarshalJSONStruct)
if err != nil {
	return err
}
// unmarshalJSONStruct.Val == None[int]()
```

## Tips

- it would be better to deal with an Option value as a non-pointer because if the Option value can accept nil it becomes worthless

## Known Issues

The runtime raises a compile error like "methods cannot have type parameters", so `Map()`, `MapOr()`, `MapWithError()`, `MapOrWithError()`, `Zip()`, `ZipWith()`, `Unzip()` and `UnzipWith()` have been providing as functions. Basically, it would be better to provide them as the methods, but currently, it compromises with the limitation.

## Author

moznion (<moznion@mail.moznion.net>)

