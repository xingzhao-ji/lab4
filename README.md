# Hash Hash Hash
This lab compares different hash table implementations and analyzes their performance in terms of execution time and accuracy across single-threaded, single-mutex multi-threaded, and multiple-mutex multi-threaded approaches.

## Building
Run this command in the same directory where the Makefile is located to build:
```shell
make
```
## Running
1. Run the tester with a thread count and total inserts (example: 8 threads, 50000 inserts):
```shell
./hash-table-tester -t 8 -s 50000
```
2. Try additional experiments (change either threads or size or both):
```shell
./hash-table-tester -t 8 -s 40000
./hash-table-tester -t 4 -s 50000
```
3. Results (example outputs running on my 8 GPU core machine):
```shell
$./hash-table-tester -t 8 -s 50000
Generation: 52,087 usec
Hash table base: 236,728 usec
  - 0 missing
Hash table v1: 674,478 usec
  - 0 missing
Hash table v2: 62,701 usec
  - 0 missing

$./hash-table-tester -t 8 -s 40000
Generation: 48,314 usec
Hash table base: 149,372 usec
  - 0 missing
Hash table v1: 433,012 usec
  - 0 missing
Hash table v2: 37,306 usec
  - 0 missing

$./hash-table-tester -t 4 -s 50000
Generation: 33,724 usec
Hash table base: 67,602 usec
  - 0 missing
Hash table v1: 206,681 usec
  - 0 missing
Hash table v2: 16,443 usec
  - 0 missing
```

## First Implementation
In the `hash_table_v1_add_entry` function, I added a single mutex to protect the entire hash table. Basically, I just put one big lock on the whole thing - whenever any thread wants to add something, it has to grab the lock first, do its work, then release it.

I also added `pthread_mutex_t mutex` field to the hash table struct, initialized it in the create function, and then wrapped the entire add_entry function with lock/unlock calls.

This works correctly because only one thread can touch the hash table at a time. When a thread grabs the mutex, everyone else has to wait their turn. This prevents any race conditions since threads can't accidentally overwrite each other's work.

### Performance
```shell
$./hash-table-tester -t 8 -s 50000
Hash table base: 236,728 usec
Hash table v1: 674,478 usec
```
Version 1 is about 2.85x slower than the base version. This makes sense since it's actually slower than single-threaded. The main problems are thread creation overhead where we waste time creating and managing threads, everyone waiting for one lock since all threads line up for the same mutex so no actual parallelism happens, context switching as the OS keeps swapping between threads that are mostly just waiting, and lock overhead from getting and releasing the mutex. We essentially get all the downsides of threading (overhead from creating threads, context switches, lock management) with none of the benefits since everything still runs one at a time.

## Second Implementation
In the `hash_table_v2_add_entry` function, instead of one giant lock for everything, I gave each bucket in the hash table its own mutex. This way, threads only block each other if they're trying to access the exact same bucket.

Each hash table entry now has its own `pthread_mutex_t entry_mutex`. I also added 64 bytes of padding after each mutex to prevent different CPUs fight over cache lines. In the add_entry function, I only lock the specific bucket we're working on, not the whole table.

This works correctly because each bucket gets its own lock. If thread A needs bucket 5 and thread B needs bucket 100, they can both work at the same time since they're using different mutexes. They only have to wait if they both want the same bucket, but with 4096 buckets and a decent hash function, this doesn't happen too often.

### Performance
```shell
$./hash-table-tester -t 8 -s 50000
Hash table base: 236,728 usec
Hash table v2: 62,701 usec
  - Speedup: 3.77x faster than base
  - 10.76x faster than v1

$./hash-table-tester -t 4 -s 50000
Hash table base: 67,602 usec
Hash table v2: 16,443 usec
  - Speedup: 4.11x faster than base
  - 12.57x faster than v1
```

Version 2 is much faster. With 8 threads, we get about a 3.77x speedup over the base implementation. Now we're actually using multiple cores properly since different threads can insert into different buckets at the same time.

The padding I added prevents false sharing, which is when different CPU cores mess up each other's caches even though they're working on different data. Without the padding, threads would constantly invalidate each other's cache lines and performance would tank.

## Cleaning up
To clean up and remove the executables created, use this command:
```shell
make clean
```



