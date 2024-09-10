## [Cache](https://github.com/ckshitij/cache)

Cache library in go-lang which can be used to store the key-value pair in memory and also provide the auto cleanup functionality using time-to-live duration.

### Types

Currently there are two type of cache are supported:
 - Datastore
   - Common in-memory datastore pretty much like to store key-values, with get and set functionality and provide the clean-up based on time-to-live parameters.
 - AutoReload
   - This cache is mostly used for read-heavy functionality in one go, there is not a set method since it mainly load all the data on startup then auto reload the data after given interval.
   - e.g, mostly this could be used if user need to load all the data at once from db to memory.

### Installation

- Use the below command to install the library
    ```sh
    go get github.com/ckshitij/cache
    ```

### Examples

#### Datastore Test cases

```go
func TestAutoSweep(t *testing.T) {
  sweepInterval := 500 * time.Millisecond
  ds, err := NewDatastore[string](context.Background(), 1*time.Second, cache.WithSweeping(sweepInterval))
  if err != nil {
    t.Fatalf("init failed")
  }
  key, value := "tempKey", "tempValue"
  ds.Put(key, value)

  // Wait for TTL to expire and cleanup to run
  time.Sleep(2 * time.Second)

  // Verify that the key has been cleaned up
  _, ok := ds.Get(key)
  if ok {
    t.Errorf("expected key to be cleaned up, but it still exists")
  }
}

func TestConcurrentAccess(t *testing.T) {
  ds, err := NewDatastore[string](context.Background(), 5*time.Second)
  if err != nil {
    t.Fail()
  }

  // Run concurrent writes using integer range
  for i := range make([]struct{}, 100) {
    go ds.Put(fmt.Sprintf("key%d", i), fmt.Sprintf("value%d", i))
  }

  // Run concurrent reads using integer range
  for i := range make([]struct{}, 100) {
    go func(i int) {
      ds.Get(fmt.Sprintf("key%d", i))
    }(i)
  }

  // Ensure all values are accessible
  time.Sleep(1 * time.Second)
  for i := range make([]struct{}, 100) {
    val, ok := ds.Get(fmt.Sprintf("key%d", i))
    if !ok || val.Value != fmt.Sprintf("value%d", i) {
      t.Errorf("expected value %v, got %v", fmt.Sprintf("value%d", i), val)
    }
  }
}
```

#### AutoReload Test cases

```go
// Mock DataFunc that returns some mock data for testing
func mockLoadFuncSuccess() (map[string]cache.CacheElement[string], error) {
	data := map[string]cache.CacheElement[string]{
		"key1": {Value: "value1"},
		"key2": {Value: "value2"},
	}
	return data, nil
}

func TestNewAutoReload_Success(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	// Test cache creation with a successful load function
	autoCache, err := NewAutoReload[string](ctx, "testCache", mockLoadFuncSuccess)
	require.NoError(t, err, "Error should be nil on creating cache")
	assert.NotNil(t, autoCache, "Cache should not be nil after initialization")

	// Test that data is loaded into cache immediately
	data, ok := autoCache.Get("key1")
	assert.True(t, ok, "Cache key 'key1' should exist")
	assert.Equal(t, "value1", data.Value, "Cache value for 'key1' should be 'value1'")

	// Test Get for non-existent key
	_, ok = autoCache.Get("invalidKey")
	assert.False(t, ok, "Cache should return false for non-existent keys")

	// Test refresh duration
	assert.Equal(t, time.Second*10, autoCache.GetRefreshDuration(), "Refresh duration should be 1 minute")

	// Test that the initial data is available in cache
	data, ok = autoCache.Get("key2")
	assert.True(t, ok, "Cache key 'key2' should exist")
	assert.Equal(t, "value2", data.Value, "Cache value for 'key2' should be 'value2'")
}

```