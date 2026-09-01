# Improving-Counting-Sort-Via-Data-Locality
A Java implementation and empirical evaluation of a cache friendly hybrid sorting algorithm that combines a modified Quicksort preprocessing step with Counting Sort. This project replicates and analyses the results of the paper "Improving Counting Sort Algorithm Via Data Locality" (Mahmud, Haque & Choudhury, ACSME 2022)

# Overview 
Counting sort runs in O(n + r) time, but its real world performance suffers on random input because accesses to the count[] and output[] arrays are non-sequential, causing frequent CPU cache misses. This project implements the paper's proposed fix, partition the input into cache sized subarrays with a modified Quicksort, the run Counting Sort independently on each subarray. Because the subarrays are ordered relative to one another, the final output is globally sorted, with far better cache locality. 

The algorithm works in two phases : 
1. Preprocessing (modified Quicksort) : Recursively partitions the array using Hoare partitioning with median of three pivot selection, stopping once a subarray satisfies the cache constraint (maxValue - minValue) + (high - low) <= C
2. Partitioned Counting Sort : Applies Counting Sort independently to each resulting cache sized subarray

# Implementation 
- countingSort() : Classical Counting Sort baseline, no cache optimisation
- quicksortModified() : Recursive preprocessing step that partitions the array unto cache constrained subarrays without fully sorting them
- partition() : Hoare partition scheme with median of three pivot selection
- countingSortByPartitions() : Applies Counting Sort segment by segment over the partitions produced by quicksortModified()
- countingSortSegment() : Helper that sorts a single segment in place using its local min/max range

# Notable Implementation Challenges 
- Incomplete sort misunderstanding : quicksortModified() deliberately does not fully sort the array, it only partitions it, leaving final sorting to per partition Counting sort
- Infinite recursion on duplicate heavy input : When maxValue == minValue, the cache constraint condition depends only on high-low, which can cause infinite recursion on large runs of identical values, fixed with an early exit check
- Per-partion VS whole array Counting sort : An initial version ran Counting Sort once over the whole array post partitioning, which was correct but missed the cache locality benefit, fixed by sorting each partition independently via countingSortByPartitions()

# Experimental Setup 
- Hardware : Apple M1 MacBook Air (192KB L1 instruction cache, 128KB L1 data cache per core, 12MB shared L2 cache)
- Language : Java, timed with System.nanoTime()
- Input : Uniformly distributed random integers in [0, r-1], fixed random seed (21) for reproducibility

