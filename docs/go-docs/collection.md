## [Collection](https://github.com/ckshitij/collection)

## Building a Generic & Concurrent-Safe Collection Library in Go using Generics

Go is known for its simplicity and performance, but when it comes to data structures like priority queues, stacks, and lists, developers often rely on custom implementations. With the introduction of **generics in Go 1.18**, it has become easier and more type-safe to build reusable data structures.

In this blog, I'll walk through the **collection package** I built, which includes:

- A **concurrent-safe doubly linked list**
- A **generic priority queue** built on heap logic
- A **queue** and **stack** abstraction
- Support for **primitive types** with optimized helper functions
- And yes — **thread-safety and performance** considerations

---

### 1. 📋 Concurrent-Safe Doubly Linked List

We designed the `list` package with concurrency in mind. The list internally uses a `sync.RWMutex` to make operations like `PushFront`, `PushBack`, `PopFront`, `PopBack`, and traversal (`IterateForward`, `IterateBackward`) thread-safe.

```go
list := list.NewList[int]()
list.PushBack(10)
list.PushFront(5)
fmt.Println(list.Len())  // Output: 2
```

---

## 2. 🚦 Priority Queue: Heap-Based & Generic

The `priority_queue` package defines a **generic `PriorityQueue[T]`** that is heap-backed. A comparator function defines whether the queue behaves as a min-heap or max-heap.

### Example:

```go
pq := pq.NewMinIntPQ(5, 1, 3, 4)
for !pq.Empty() {
    fmt.Println(pq.Pop()) // Output: 1 3 4 5
}
```

To simplify usage with primitives, I created helper functions like:

- `NewMinIntPQ`, `NewMaxIntPQ`
- `NewMinFloat32PQ`, `NewMaxFloat32PQ`
- `NewMinStringPQ`, `NewMaxStringPQ`

This means you can get **typed priority queues without redefining comparator logic.**

---

## 3. 🎡 Queue: FIFO with Thread Safety

The `queue` package provides a **generic and concurrent-safe queue**:

```go
q := queue.NewQueue[int]()
q.Enqueue(1)
q.Enqueue(2)
value, _ := q.Dequeue()
fmt.Println(value)  // Output: 1
```

It also includes **primitive helpers** like `NewIntQueue`, `NewStringQueue`, etc.

---

## 4. 🥞 Stack: LIFO with Generics

Similarly, the `stack` package offers:

```go
st := stack.NewStack[string]()
st.Push("hello")
fmt.Println(st.Top()) // Output: hello
```

You can also reset the stack using `Clear`.

---

## 5. ✅ Thread Safety & Performance

- Each data structure is guarded with appropriate **read-write mutexes** to support concurrent access.
- Underlying data structures are **efficient and optimized** for their respective use-cases.
- Extensive tests with **>90% coverage** ensure reliability.

```bash
go test ./... -race -cover
```

---

## 6. 🛠️ Contribution Guide

This project is open-source and available on [GitHub](https://github.com/ckshitij/collection). I'm welcoming contributions for:

- More data structures
- Performance improvements
- Additional features like iterators

If you'd like to contribute:

1. Fork the repo
2. Create a feature branch
3. Submit a pull request with relevant tests

---

## 7. 📚 Closing Thoughts

Go's generics unlocked a new level of reusability for data structures. Through this project, I've built reusable, type-safe, and concurrent-ready collections that can be a solid utility in your Go projects.

You can explore the full source here:  
👉 **[https://github.com/ckshitij/collection](https://github.com/ckshitij/collection)**

---

Feel free to try it out, give feedback, or contribute!

> _Happy coding!_ 🚀




