# Hash Hash Hash
A concurrent hash table implementation with two different locking strategies to ensure thread safety while maintaining performance.

## Building
Run this command in the same directory where the Makefile is located to build:
```shell
make
```

## Running
1. Run the tester with a thread count and total inserts (example: 8 threads, 50k inserts):
```shell
./hash-table-tester -t 8 -s 50000
```
2. Try additional experiments (change either threads or size or both):
```shell
./hash-table-tester -t 8 -s 40000
./hash-table-tester -t 4 -s 50000
```
3. Results (sample outputs running on 8 GPU core machine):
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

This works correctly because only one thread can mess with the hash table at a time. It's like having one bathroom key that everyone has to share - nobody can go in until the person before them comes out. This completely prevents race conditions since threads can't step on each other's toes.

### Performance
```shell
$./hash-table-tester -t 8 -s 50000
Hash table base: 236,728 usec
Hash table v1: 674,478 usec
```
Version 1 is about 2.85x slower than the base version. As expected, this is pretty bad performance-wise. The slowdown happens because even though we're using 8 threads, they're all waiting in line for that one lock. It's actually worse than just running everything on one thread because we have the overhead of creating threads, context switching between them, and managing the mutex - all for no real benefit. The threads spend most of their time waiting around instead of actually doing work.

## Second Implementation
In the `hash_table_v2_add_entry` function, instead of one giant lock for everything, I gave each bucket in the hash table its own mutex. This way, threads only block each other if they're trying to access the exact same bucket.

Each hash table entry now has its own `pthread_mutex_t entry_mutex`. I also added 64 bytes of padding after each mutex to prevent different CPUs fight over cache lines. In the add_entry function, I only lock the specific bucket we're working on, not the whole table.

This is  correct because each bucket is protected individually. If thread A is adding to bucket 5 and thread B is adding to bucket 100, they can both work at the same time without any problems. They only have to wait if they both want the same bucket.

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

Version 2 is faster. With 8 threads, we get about a 3.77x speedup over the base implementation as we are using multiple cores instead of making them all wait in line. The speedup isn't quite 8x because sometimes threads still collide when they hash to the same bucket.

The padding helps the speed as well, even though threads are working on different buckets, the CPU cores would fight over cache lines and slow everything down.

## Cleaning up
To clean up and remove the executables created, use this command:
```shell
make clean
```

