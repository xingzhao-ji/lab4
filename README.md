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
3. Results (example outputs running on a 8-core CPU machine):
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
In the `hash_table_v1_add_entry` function, I added a single pthread_mutex_t guarding the entire hash table; each add_entry call acquires the lock, performs the insertion in the target bucket, and releases the lock before returning.

I also added `pthread_mutex_t mutex` member to the hash table structure, initialized it in the create routine, and bracketed add_entry with pthread_mutex_lock/pthread_mutex_unlock to enforce mutual exclusion.

We can guarantee the correctness because the table-wide mutex is acquired at the start of `hash_table_v1_add_entry` and released on every exit path. While the lock is held, one thread computes the bucket, walks the list, and updates or inserts the entry, so no two threads modify the table at the same time. The mutex is initialized in `hash_table_v1_create` and destroyed in `hash_table_v1_destroy`.

### Performance
```shell
$./hash-table-tester -t 8 -s 50000
Hash table base: 236,728 usec
Hash table v1: 674,478 usec
```
Version 1 is ~2.85x slower than the base (674,478 usec / 236,728 usec). Version 1 is slower because of thread creation overhead and threads wait for each other instead of working in paralle

## Second Implementation
In the `hash_table_v2_add_entry` function, I added a `pthread_mutex_t` to each bucket, replacing the table-wide lock. With per-bucket locking, operations on different buckets proceed in parallel, and threads contend only when they access the same bucket.

I added a `pthread_mutex_t entry_mutex` to each bucket and 64 bytes of padding after the mutex so two buckets do not sit in the same 64-byte memory block. This keeps updates to different buckets from slowing each other down. In `hash_table_v2_add_entry`, only the target bucket’s mutex is locked and no table wide lock is used.

This works because each bucket has its own lock. Operations on different buckets run at the same time; only operations targeting the same bucket wait for one another. With 4,096 buckets (for example, 50,000 inserts ≈ 12 per bucket on average), simultaneous requests for the same bucket are relatively infrequent, so waiting is much lower than with a single table-wide lock.

### Performance
```shell
$./hash-table-tester -t 8 -s 50000
Hash table base: 236,728 usec
Hash table v2: 62,701 usec

$./hash-table-tester -t 4 -s 50000
Hash table base: 67,602 usec
Hash table v2: 16,443 usec
```

Version 2 is about ~3.78x faster than the base (236,728 usec / 62,701 usec). The increase of speed is due to allowing threads to operate on different buckets concurrently rather than waiting behind a single lock.

I also added 64-byte padding after each bucket’s mutex to avoid false sharing. 

## Cleaning up
To clean up and remove the executables created, use this command:
```shell
make clean
```







