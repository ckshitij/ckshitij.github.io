## 🌟 [In-Memory Datastore in Go](https://github.com/ckshitij/memstore)

Golang 1.18+ brings powerful generics support, allowing developers to write cleaner, reusable, and type-safe code. In this post, we'll explore a practical use case—**a generic in-memory datastore with built-in TTL (Time-To-Live)**, sweeping for stale data, and concurrency safety using mutexes.

We'll also cover potential pitfalls (like silent goroutine panics), address common usage patterns, and walk through real-world examples.

---

## 🚀 What We'll Build

We’re going to implement a **thread-safe, generic in-memory datastore** with:

- ✅ Type-safe key-value storage using generics  
- 🕓 TTL support for each value  
- 🧹 Auto-sweeping of expired values  
- 🔐 Concurrency-safe access using sync.RWMutex  
- 🔄 Graceful goroutine error handling  

---

## 📦 Basic Building Block: `CacheItem`

Each stored value is wrapped with metadata:

```go
type CacheItem[T any] struct {
    Value     T
    CreatedAt time.Time
    TTL       time.Duration
}
```

We also define a constructor to allow default TTL fallback:

```go
func NewCacheItem[T any](value T, ttl time.Duration) CacheItem[T] {
    if ttl <= 0 {
        ttl = 24 * time.Hour // default TTL
    }
    return CacheItem[T]{
        Value:     value,
        CreatedAt: time.Now().UTC(),
        TTL:       ttl,
    }
}
```

---

## 🗃️ The `Datastore` Structure

Our generic datastore is defined as:

```go
type Datastore[K MapKey, T any] struct {
    elements map[K]CacheItem[T]
    ttl      time.Duration
    mutex    sync.RWMutex
    opts     Options
}
```

Where `K` is a `comparable` type (as required by Go maps), and `T` is the value type.

---

## 🛠️ Creating a New Datastore

```go
ds, err := NewDatastore[string, int](ctx, 10*time.Second)
```

Optional sweep interval and configurations are passed as functional options.

### Example Use Cases:
```go
ctx := context.Background()

// A datastore storing integer counters with 30-second TTL
intStore, _ := NewDatastore[string, int](ctx, 30*time.Second)

// Store value
intStore.Put("user:123", 42)

// Get value
val, ok := intStore.Get("user:123")
if ok {
    fmt.Println("Found:", val.Value)
}
```

---

## ⚙️ Expiry and Sweeping

The `isExpired()` method checks if an item is stale:

```go
func (ds *Datastore[K, T]) isExpired(item CacheItem[T]) bool {
    return time.Since(item.CreatedAt) > item.TTL
}
```

### 🧹 Sweep Routine

We run a goroutine that periodically removes expired values:

```go
func (ds *Datastore[K, T]) sweep(ctx context.Context, interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            ds.mutex.Lock()
            for key, val := range ds.elements {
                if ds.isExpired(val) {
                    delete(ds.elements, key)
                }
            }
            ds.mutex.Unlock()
        case <-ctx.Done():
            fmt.Println("sweep closed")
            return
        }
    }
}
```

---

## 🧪 Testing with Custom Structs

The datastore works seamlessly with structs too:

```go
type Session struct {
    UserID    string
    AuthToken string
}

sessionStore, _ := NewDatastore[string, Session](ctx, 15*time.Minute)
sessionStore.Put("sess:001", Session{"u123", "token_xyz"})
```

---

## 🛡️ Final Thoughts and Improvements

✅ This generic datastore is already highly usable. However, you can enhance it by:
- Logging expired deletions
- Adding max-size or eviction policies
- Persisting values to disk
- Supporting batch operations

---

## 📚 Summary

With the power of Go generics and the concurrency-safe patterns, we’ve built a fast, type-safe, flexible in-memory cache that’s useful in many production services like:

- Session stores  
- API rate limiting  
- Local in-process caching  
- Short-lived auth tokens  

Happy Go-ing! 🎯

