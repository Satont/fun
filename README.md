# Iterator and functional utilities
[![Go Reference](https://pkg.go.dev/badge/github.com/rprtr258/fun.svg)](https://pkg.go.dev/github.com/rprtr258/fun)
[![Go Report Card](https://goreportcard.com/badge/github.com/rprtr258/fun)](https://goreportcard.com/report/github.com/rprtr258/fun)
![Go](https://github.com/rprtr258/fun/workflows/Test/badge.svg?branch=master)
![CodeQL](https://github.com/rprtr258/fun/workflows/CodeQL/badge.svg?branch=master)
![Coverage](https://img.shields.io/badge/Coverage-33.1%25-yellow)

The design is inspired by [samber/lo](https://github.com/samber/lo) and [iterator proposal](https://github.com/golang/go/issues/61897).

## Root package
Root package `github.com/rprtr258/fun` provides common slice and functional utilities:

### slice
```go
func Map[R, T any, F interface {
	func(T) R | func(T, int) R
}](slice []T, f F) []R

func Filter[T any, F interface {
	func(T) bool | func(T, int) bool
}](slice []T, f F) []T

func FilterMap[R, T any, F interface {
	func(T) (R, bool) | func(T, int) (R, bool) |
		func(T) Option[R] | func(T, int) Option[R]
}](slice []T, f F) []R

func MapDict[T comparable, R any](collection []T, dict map[T]R) []R

func MapErr[R, T any, E interface {
	error
	comparable
}, FE interface {
	func(T) (R, E) | func(T, int) (R, E)
}](slice []T, f FE) ([]R, E)

func Deref[T any](ptr *T) T

func MapToSlice[K comparable, V, R any](dict map[K]V, f func(K, V) R) []R

func MapFilterToSlice[K comparable, V, R any](dict map[K]V, f func(K, V) (R, bool)) []R

func Keys[K comparable, V any](dict map[K]V) []K

func Values[K comparable, V any](dict map[K]V) []V

// FindKeyBy returns the key of the first element predicate returns truthy for.
func FindKeyBy[K comparable, V any](dict map[K]V, predicate func(K, V) bool) (K, bool)

// Uniq returns unique values of slice.
func Uniq[T comparable](collection []T) []T

// Index returns first found element by predicate along with it's index
func Index[T comparable](slice []T, find func(T) bool) (T, int, bool)

// Contains returns true if an element is present in a collection.
func Contains[T comparable](slice []T, needle T) bool

// SliceToMap returns a map containing key-value pairs provided by transform function applied to elements of the given slice.
// If any of two pairs would have the same key the last one gets added to the map.
// The order of keys in returned map is not specified and is not guaranteed to be the same from the original array.
func SliceToMap[K comparable, V, T any, F interface {
	func(T) (K, V) | func(T, int) (K, V)
}](slice []T, f F) map[K]V

// FromMap makes slice of key/value pairs from map.
func FromMap[A comparable, B any](kv map[A]B) []Pair[A, B]

// Copy slice
func Copy[T any](slice []T) []T

// ReverseInplace reverses slice in place.
func ReverseInplace[A any](xs []A)

// Subslice returns slice from start to end without panicking on out of bounds
func Subslice[T any](slice []T, start, end int) []T

// Chunk divides slice into chunks of size chunkSize
func Chunk[T any](slice []T, chunkSize int) [][]T

// ConcatMap is like Map but concatenates results
func ConcatMap[T, R any](slice []T, f func(T) []R) []R

// All returns true if all elements satisfy the condition
func All[T any](slice []T, condition func(T) bool) bool

// Any returns true if any element satisfies the condition
func Any[T any](slice []T, condition func(T) bool) bool

// SortBy sorts slice in place by given function
func SortBy[T any, R cmp.Ordered](slice []T, by func(T) R)

// GroupBy groups elements by key
func GroupBy[T any, K comparable](slice []T, by func(T) K) map[K][]T
```

### cmp
```go
// Min returns the minimum of the given values
func Min[T cmp.Ordered](xs ...T) T

// Max returns the maximum of the given values
func Max[T cmp.Ordered](xs ...T) T

// Clamp returns x clamped between low and high
func Clamp[T cmp.Ordered](x, low, high T) T

// MinBy returns the minimum of the given values using the given order function
func MinBy[T any, R cmp.Ordered](order func(T) R, xs ...T) T

// MaxBy returns the maximum of the given values using the given order function
func MaxBy[T any, R cmp.Ordered](order func(T) R, xs ...T) T
```

### optional values
```go
// Option is either value or nothing.
type Option[T any] struct {
	Value T
	Valid bool
}

func Invalid[T any]() Option[T] {
	return Option[T]{}
}

func Valid[T any](t T) Option[T] {
	return Option[T]{
		Value: t,
		Valid: true,
	}
}

func Optional[T any](value T, valid bool) Option[T]

func (o Option[T]) Unpack() (T, bool)

func (o Option[T]) Or(other Option[T]) Option[T]

func (o Option[T]) OrDefault(value T) T

func FromPtr[T any](ptr *T) Option[T]

func (opt Option[T]) Ptr() *T

func OptMap[I, O any](o Option[I], f func(I) O) Option[O]

func OptFlatMap[I, O any](o Option[I], f func(I) Option[O]) Option[O]
```

### fp
```go
// Pair is a data structure that has two values.
type Pair[K, V any] struct {K K; V V}

func Zero[T any]() T

// ToString converts the value to string.
func ToString[A any](a A) string {

// DebugP returns function that prints prefix with element and returns it.
// Useful for debug printing.
func DebugP[V any](prefix string) func(V) V

// Debug returns function that prints element and returns it.
// Useful for debug printing.
func Debug[V any](v V) V

func Has[K comparable, V any](dict map[K]V, key K) bool

func Cond[R any](defaultValue R, cases ...func() (R, bool)) R

func Ptr[T any](t T) *T

func Pipe[T any](t T, endos ...func(T) T) T
```

#### functional if
```go
func IF[T any](predicate bool, ifTrue, ifFalse T) T

func If[T any](predicate bool, value T) ifElse[T]

func IfF[T any](predicate bool, value func() T) ifElse[T]

func (i ifElse[T]) ElseIf(predicate bool, value T) ifElse[T]

func (i ifElse[T]) ElseIfF(predicate bool, value func() T) ifElse[T]

func (i ifElse[T]) Else(value T) T

func (i ifElse[T]) ElseF(value func() T) T

func (i ifElse[T]) ElseDeref(value *T) T
```

#### functional switch
```go
// Switch is a pure functional switch/case/default statement.
func Switch[R any, T comparable](predicate T, defVal R) *switchCase[T, R]

// SwitchZero is a pure functional switch/case/default statement with default
// zero value.
func SwitchZero[R any, T comparable](predicate T) *switchCase[T, R]

func (s *switchCase[T, R]) Case(val T, result R) *switchCase[T, R]

func (s *switchCase[T, R]) End() R
```

## Iterators
`github.com/rprtr258/fun/iter` introduces iterator primitives from which `Seq[T]` is basic:

```go
type Seq[V any] func(yield func(V) bool) bool
```

Which is a function which accepts function to `yield` values from iteration. `yield` must return `false` when iteration must stop (analogous to `break`). Iterator function returns `true` if it yielded all values and no `break` happened, or `false` otherwise.

Example iterator yielding numbers from 1 to `n`, including `n`:
```go
func Range(n int) iter.Seq[int] {
	return func(yield func(int) bool) bool {
		for i := 1; i <= n; i++ {
			if !yield(i) {
				return false
			}
		}
		return true
	}
}
```

## Set
`github.com/rprtr258/fun/set` introduces `Set[T]` primitive for collections of unique `comparable` values.
