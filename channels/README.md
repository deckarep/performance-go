## Shared channel select is slow with many P

[Further Reading](https://github.com/golang/go/issues/20351)

Note that the idle channels below are never closed and are never selectable, but the other two are, and stay busy. But when the channel is private to the goroutine (uncontended), things are fast. When it's a shared channel, things are slow.

```go
package selbench
        
import "testing"
        
func BenchmarkSelectShared(b *testing.B) {
        idleShared := make(chan struct{})
        b.RunParallel(func(pb *testing.PB) {
                ch := make(chan int, 1)
                for pb.Next() {
                        select {
                        case ch <- 1:
                        case <-ch:
                        case <-idleShared:
                        }
                }
        })      
}       

func BenchmarkSelectPrivate(b *testing.B) {
        b.RunParallel(func(pb *testing.PB) {
                ch := make(chan int, 1)
                idlePrivate := make(chan struct{})
                for pb.Next() {
                        select {
                        case ch <- 1:
                        case <-ch:
                        case <-idlePrivate:
                        }
                }
        })
} 
```

```sh
go test -v -bench=Select -cpu=1,2,4,8,16,32,64

BenchmarkSelectShared           10000000                194 ns/op
BenchmarkSelectShared-2         10000000                147 ns/op
BenchmarkSelectShared-4          5000000                395 ns/op
BenchmarkSelectShared-8          3000000                449 ns/op
BenchmarkSelectShared-16         5000000                354 ns/op
BenchmarkSelectShared-32         5000000                320 ns/op
BenchmarkSelectShared-64         5000000                296 ns/op
BenchmarkSelectPrivate          10000000                192 ns/op
BenchmarkSelectPrivate-2        20000000               98.0 ns/op
BenchmarkSelectPrivate-4        30000000               49.3 ns/op
BenchmarkSelectPrivate-8        50000000               25.5 ns/op
BenchmarkSelectPrivate-16       100000000              13.8 ns/op
BenchmarkSelectPrivate-32       200000000              7.07 ns/op
BenchmarkSelectPrivate-64       200000000              6.31 ns/op
```