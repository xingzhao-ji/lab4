# Hash Hash Hash
TODO introduction

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
2. Try additional experiments (change either threads or size):
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
In the `hash_table_v1_add_entry` function, I added TODO

### Performance
```shell
TODO how to run and results
```
Version 1 is a little slower/faster than the base version. As TODO

## Second Implementation
In the `hash_table_v2_add_entry` function, I TODO

### Performance
```shell
TODO how to run and results
```

TODO more results, speedup measurement, and analysis on v2

## Cleaning up
To clean up and remove the executables created, use this command:
```shell
make clean
```
