## [Multithreaded file search utility](https://github.com/ckshitij/multithread-file-search-utility/blob/main/main.go)

Golang utility to search the file in system by passing the directory path and filename,
it will show all the files with same name present under the given directory.

It's a very fast utility utilizing the multithread feature of go-lang.

- *Currently its only supported for linux/macos based machine*

## Usage

- For easy access just install make utility and execute the below command 
  ```
  make run
  ```
- update the directory and filename args in the makefile

### Manual usage if make is not install in the system
- Download the utility
  - change the permission to make it executable for linux-based machine
  ```
    chmod +x mfs
  ```
- Run the utility below is the example
  ```
  mfs --directory /User/ --filename README.md
  ```
  - here **/User/** is the example for the directory name and **README.md** as filename.

### Example

#### Code comparison example:

```go
package main

import (
	"fmt"
	"time"

	"github.com/ckshitij/multithread-file-search-utility/config"
	file_search "github.com/ckshitij/multithread-file-search-utility/search_file"
)

func GetTimeElapse(callbackFunc func()) {
	startTime := time.Now()
	callbackFunc()
	elapsed := time.Since(startTime)
	fmt.Printf("Total time taken to run the function %d ms\n", elapsed.Milliseconds())
}

func executeMultithreadedSearch() {
	fs := file_search.NewFileSearchUtility()
	fs.Add(1)
	go fs.SearchFile(config.GetDirectory(), config.GetFileName())
	fs.Wait()
	fmt.Printf("executed the multithreaded, found total %d matched files", len(fs.MatchedFiles()))
}

func executeSyncSearch() {
	fs := file_search.NewFileSearchUtility()
	fs.SyncSearchFile(config.GetDirectory(), config.GetFileName())
	fmt.Printf("executed the sync search, found total %d matched files", len(fs.MatchedFiles()))
}

func main() {
	// Compare the time difference between sync and multithread search calls.
	GetTimeElapse(executeMultithreadedSearch)
	GetTimeElapse(executeSyncSearch)
}
```