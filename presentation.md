# Sorting Algorithms

**Computer Science Fundamentals Series**

Comparison sorts · Non-comparison sorts · Complexity analysis · Divide and conquer · Stability · Real-world implementations

*Mid-level software engineer track -- 21 slides*

---

## Table of Contents

1. [Why Study Sorting?](#slide-02--why-study-sorting)
2. [Sorting Taxonomy](#slide-03--sorting-taxonomy)
3. [Bubble Sort](#slide-04--bubble-sort)
4. [Selection Sort](#slide-05--selection-sort)
5. [Insertion Sort](#slide-06--insertion-sort)
6. [Merge Sort](#slide-07--merge-sort)
7. [Quick Sort](#slide-08--quick-sort)
8. [Heap Sort](#slide-09--heap-sort)
9. [The Divide-and-Conquer Paradigm](#slide-10--the-divide-and-conquer-paradigm)
10. [Lower Bound for Comparison Sorts](#slide-11--lower-bound-for-comparison-sorts)
11. [Counting Sort](#slide-12--counting-sort)
12. [Radix Sort](#slide-13--radix-sort)
13. [Bucket Sort](#slide-14--bucket-sort)
14. [Stability, Adaptivity & In-Place](#slide-15--stability-adaptivity--in-place)
15. [Complexity Comparison Table](#slide-16--complexity-comparison-table)
16. [When to Use Which Sort](#slide-17--when-to-use-which-sort)
17. [Real-World Implementations](#slide-18--real-world-implementations)
18. [Parallel & External Sorting](#slide-19--parallel--external-sorting)
19. [Practical Considerations](#slide-20--practical-considerations)
20. [Summary & Further Reading](#slide-21--summary--further-reading)

---

## Slide 02 -- Why Study Sorting?

### Sorting is everywhere

Sorting is the most fundamental algorithmic problem in computer science. Efficient sorting enables efficient searching, merging, and data presentation.

- Database `ORDER BY` clauses execute a sort
- Binary search requires sorted input -- `O(log n)` vs `O(n)`
- Merge operations (joins, set intersection) assume sorted streams
- Rendering leaderboards, timelines, search results
- Scheduling, priority queues, and resource allocation

### What we will cover

1. **Comparison-based sorts** -- Bubble, Selection, Insertion, Merge, Quick, Heap
2. **Non-comparison sorts** -- Counting, Radix, Bucket
3. **Complexity analysis** -- time, space, best/worst/average cases
4. **Properties** -- stability, adaptivity, in-place
5. **Real-world implementations** -- Timsort, Introsort, pdqsort

---

## Slide 03 -- Sorting Taxonomy

### Simple / Quadratic

Easy to implement. `O(n²)` average and worst case. Practical for small inputs or nearly-sorted data.

`Bubble` | `Selection` | `Insertion`

*Insertion sort is the best of the three in practice.*

### Efficient / Linearithmic

Divide-and-conquer or heap-based. `O(n log n)` average case. The workhorse algorithms.

`Merge Sort` | `Quick Sort` | `Heap Sort`

*Quick sort is fastest in practice for random data.*

### Non-Comparison

Exploit structure of keys to beat `Ω(n log n)`. Linear time under constraints on key range or distribution.

`Counting` | `Radix` | `Bucket`

*Require assumptions about input distribution.*

> **Key insight:** no single algorithm is best for all inputs. The right choice depends on data size, distribution, memory constraints, and whether stability is required.

---

## Slide 04 -- Bubble Sort

Repeatedly swap adjacent elements if they are in the wrong order. The largest unsorted element "bubbles up" to its correct position each pass.

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:
            break  # already sorted
```

| Property | Value |
|----------|-------|
| Best case | `O(n)` -- already sorted |
| Average case | `O(n²)` |
| Worst case | `O(n²)` -- reverse sorted |
| Space | `O(1)` |
| Stable | Yes |
| In-place | Yes |
| Adaptive | Yes (with early exit) |

> The early-exit optimisation (`swapped` flag) makes bubble sort **adaptive** -- it runs in `O(n)` on already-sorted input.

> **When to use:** Almost never in production. Useful only for teaching or when code simplicity is paramount and `n < 20`.

---

## Slide 05 -- Selection Sort

Find the minimum element in the unsorted region and swap it into the next position of the sorted region. Exactly `n - 1` swaps -- minimal writes.

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
```

| Property | Value |
|----------|-------|
| Best case | `O(n²)` |
| Average case | `O(n²)` |
| Worst case | `O(n²)` |
| Space | `O(1)` |
| Stable | No (default implementation) |
| In-place | Yes |
| Adaptive | No |

> **Minimal swaps:** selection sort performs exactly `O(n)` swaps regardless of input. Good when writes are expensive (e.g. flash memory).

> **Not stable:** swapping distant elements can change the relative order of equal keys. A linked-list variant can be made stable.

---

## Slide 06 -- Insertion Sort

Build a sorted sub-array one element at a time by inserting each new element into its correct position. The way most people sort a hand of playing cards.

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
```

| Property | Value |
|----------|-------|
| Best case | `O(n)` -- already sorted |
| Average case | `O(n²)` |
| Worst case | `O(n²)` -- reverse sorted |
| Space | `O(1)` |
| Stable | Yes |
| In-place | Yes |
| Adaptive | Yes |

> **The best quadratic sort.** Excellent cache locality, low overhead, and adaptive. Used as the base case in Timsort and Introsort for small sub-arrays (`n ≤ 32`).

> **Online algorithm:** insertion sort can sort a stream as elements arrive -- it does not need all input up front.

---

## Slide 07 -- Merge Sort

Divide the array in half, recursively sort each half, then merge the two sorted halves. The canonical divide-and-conquer sorting algorithm.

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(L, R):
    result = []
    i = j = 0
    while i < len(L) and j < len(R):
        if L[i] <= R[j]:
            result.append(L[i]); i += 1
        else:
            result.append(R[j]); j += 1
    result.extend(L[i:])
    result.extend(R[j:])
    return result
```

| Property | Value |
|----------|-------|
| Best case | `O(n log n)` |
| Average case | `O(n log n)` |
| Worst case | `O(n log n)` |
| Space | `O(n)` |
| Stable | Yes |
| In-place | No |
| Adaptive | No (standard version) |

> **Guaranteed O(n log n).** The only common comparison sort with no quadratic worst case. Preferred when stability is required and extra memory is acceptable.

> **External sorting:** merge sort naturally extends to disk-based sorting -- merge sorted runs that each fit in memory.

---

## Slide 08 -- Quick Sort

Choose a pivot, partition the array so all elements less than the pivot come before it and all greater come after, then recursively sort the partitions.

```python
def quicksort(arr, lo, hi):
    if lo < hi:
        p = partition(arr, lo, hi)
        quicksort(arr, lo, p - 1)
        quicksort(arr, p + 1, hi)

def partition(arr, lo, hi):
    pivot = arr[hi]
    i = lo - 1
    for j in range(lo, hi):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[hi] = arr[hi], arr[i + 1]
    return i + 1
```

| Property | Value |
|----------|-------|
| Best case | `O(n log n)` |
| Average case | `O(n log n)` |
| Worst case | `O(n²)` -- bad pivot |
| Space | `O(log n)` stack |
| Stable | No |
| In-place | Yes |
| Adaptive | No (standard version) |

> **Fastest in practice** for random data due to excellent cache behaviour -- sequential memory access within partitions. Small constant factor.

> **Pivot selection matters.** Naive last-element pivot degrades to `O(n²)` on sorted input. Median-of-three or random pivot avoids this.

> **Adversarial inputs:** an attacker who knows your pivot strategy can craft `O(n²)` inputs. Randomised pivot or Introsort mitigates this.

---

## Slide 09 -- Heap Sort

Build a max-heap from the input, then repeatedly extract the maximum element and place it at the end of the array.

```python
def heap_sort(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)

def heapify(arr, n, i):
    largest = i
    left, right = 2 * i + 1, 2 * i + 2
    if left < n and arr[left] > arr[largest]:
        largest = left
    if right < n and arr[right] > arr[largest]:
        largest = right
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
```

| Property | Value |
|----------|-------|
| Best case | `O(n log n)` |
| Average case | `O(n log n)` |
| Worst case | `O(n log n)` |
| Space | `O(1)` |
| Stable | No |
| In-place | Yes |
| Adaptive | No |

> **O(n log n) guaranteed + O(1) space.** The only comparison sort that is both worst-case optimal and in-place.

> **Cache-unfriendly.** Heap sort makes many non-sequential memory accesses (parent/child jumps), making it slower than quicksort in practice despite identical asymptotic complexity.

---

## Slide 10 -- The Divide-and-Conquer Paradigm

### Three steps

1. **Divide** -- split the problem into smaller sub-problems of the same type
2. **Conquer** -- recursively solve each sub-problem (base case: trivially sorted)
3. **Combine** -- merge the solutions of sub-problems into the overall solution

> **Master Theorem:** for recurrences of the form `T(n) = aT(n/b) + O(n^d)`, the solution depends on the relationship between `log_b(a)` and `d`.

### Merge Sort vs Quick Sort

| Aspect | Merge Sort | Quick Sort |
|--------|-----------|------------|
| Divide cost | `O(1)` -- split at midpoint | `O(n)` -- partition |
| Combine cost | `O(n)` -- merge | `O(1)` -- nothing |
| Recurrence | `T(n) = 2T(n/2) + O(n)` | `T(n) = T(k) + T(n-k-1) + O(n)` |
| Work location | On the way **up** (merge) | On the way **down** (partition) |

> **Key difference:** merge sort does the hard work when combining (merging). Quick sort does the hard work when dividing (partitioning). Both achieve `O(n log n)` average case.

---

## Slide 11 -- Lower Bound for Comparison Sorts

### The decision-tree argument

Any comparison-based sorting algorithm can be modelled as a binary decision tree, where each internal node is a comparison and each leaf is a permutation of the input.

- For `n` elements there are `n!` possible permutations
- The tree must have at least `n!` leaves
- A binary tree with `L` leaves has height at least `log₂ L`
- Therefore: height ≥ `log₂(n!)`
- By Stirling's approximation: `log₂(n!) = Ω(n log n)`

> **Conclusion:** no comparison-based sort can do better than `Ω(n log n)` comparisons in the worst case. Merge sort and heap sort are **asymptotically optimal**.

> **This is why non-comparison sorts exist.** By exploiting key structure (integer ranges, digit decomposition), we can bypass the `Ω(n log n)` barrier.

---

## Slide 12 -- Counting Sort

Count occurrences of each key value, compute prefix sums, then place elements directly into their final positions. No comparisons needed.

```python
def counting_sort(arr, max_val):
    count = [0] * (max_val + 1)
    output = [0] * len(arr)

    for x in arr:
        count[x] += 1
    for i in range(1, max_val + 1):
        count[i] += count[i - 1]

    for x in reversed(arr):
        output[count[x] - 1] = x
        count[x] -= 1
    return output
```

| Property | Value |
|----------|-------|
| Time | `O(n + k)` where `k` = key range |
| Space | `O(n + k)` |
| Stable | Yes |
| In-place | No |
| Comparison-based | No |

> **Stable by design.** The reverse traversal preserves the original order of equal elements. This property is critical when counting sort is used as a sub-routine in radix sort.

> **Constraint:** keys must be non-negative integers (or mappable to them). When `k >> n`, the space and time become impractical.

> **Best for:** sorting integers in a known, small range -- ages, scores, characters, pixel values.

---

## Slide 13 -- Radix Sort

Sort integers digit by digit, from least significant to most significant (LSD) or vice versa (MSD). Uses a stable sub-sort (typically counting sort) at each digit position.

```python
def radix_sort(arr):
    if not arr:
        return arr
    max_val = max(arr)
    exp = 1
    while max_val // exp > 0:
        counting_sort_by_digit(arr, exp)
        exp *= 10

def counting_sort_by_digit(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10
    for x in arr:
        idx = (x // exp) % 10
        count[idx] += 1
    for i in range(1, 10):
        count[i] += count[i - 1]
    for x in reversed(arr):
        idx = (x // exp) % 10
        output[count[idx] - 1] = x
        count[idx] -= 1
    arr[:] = output
```

| Property | Value |
|----------|-------|
| Time | `O(d(n + k))` -- `d` digits, base `k` |
| Space | `O(n + k)` |
| Stable | Yes (LSD variant) |
| In-place | No |
| Comparison-based | No |

> **Effectively linear** when the number of digits `d` is constant (e.g. 32-bit integers have at most 10 decimal digits). Time becomes `O(n)`.

> **LSD vs MSD:** LSD processes least-significant digit first and is simpler. MSD processes most-significant first and can short-circuit on prefixes -- useful for strings.

---

## Slide 14 -- Bucket Sort

Distribute elements into buckets based on their value, sort each bucket individually (often with insertion sort), then concatenate the buckets.

```python
def bucket_sort(arr):
    if not arr:
        return arr
    n = len(arr)
    buckets = [[] for _ in range(n)]
    min_val, max_val = min(arr), max(arr)
    spread = max_val - min_val or 1

    for x in arr:
        idx = int((x - min_val) / spread * (n - 1))
        buckets[idx].append(x)

    for bucket in buckets:
        insertion_sort(bucket)

    return [x for bucket in buckets for x in bucket]
```

| Property | Value |
|----------|-------|
| Best / average | `O(n + k)` -- uniform distribution |
| Worst case | `O(n²)` -- all in one bucket |
| Space | `O(n + k)` |
| Stable | Yes (if sub-sort is stable) |
| In-place | No |
| Comparison-based | Hybrid |

> **Assumes uniform distribution.** If all elements land in one bucket, performance degrades to the sub-sort's complexity -- `O(n²)` with insertion sort.

> **Best for:** floating-point numbers uniformly distributed over a known range. Also used for sorting strings by first character.

---

## Slide 15 -- Stability, Adaptivity & In-Place

### Stability

A sort is **stable** if elements with equal keys retain their original relative order.

- Sort students by name, then stable-sort by grade → students with the same grade are still alphabetical
- Required for multi-key sorting and radix sort correctness

**Stable:** Insertion, Merge, Counting, Radix, Bucket, Bubble, Timsort

**Unstable:** Quick, Heap, Selection, Introsort

### Adaptivity

An **adaptive** sort exploits existing order in the input to run faster.

- Insertion sort: `O(n)` on sorted data vs `O(n²)` on random
- Timsort detects and merges pre-sorted "runs"
- Non-adaptive sorts (heap, merge) always do the same work

**Adaptive:** Insertion, Bubble, Timsort, pdqsort

**Non-adaptive:** Merge, Heap, Selection, Counting

### In-Place

An **in-place** sort uses only `O(1)` or `O(log n)` extra memory beyond the input array.

- Swap-based sorts (bubble, selection, insertion, quick, heap) are in-place
- Merge sort requires `O(n)` auxiliary space
- Counting / radix / bucket need `O(n + k)`

**In-place:** Bubble, Selection, Insertion, Quick, Heap

**Not in-place:** Merge, Counting, Radix, Bucket

> **Trade-off triangle:** no comparison sort achieves all three of `O(n log n)` worst case + stable + in-place simultaneously. Merge sort sacrifices in-place. Heap sort sacrifices stability. Quick sort sacrifices worst-case guarantee.

---

## Slide 16 -- Complexity Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable | In-Place |
|-----------|------|---------|-------|-------|--------|----------|
| **Bubble Sort** | `O(n)` | `O(n²)` | `O(n²)` | `O(1)` | Yes | Yes |
| **Selection Sort** | `O(n²)` | `O(n²)` | `O(n²)` | `O(1)` | No | Yes |
| **Insertion Sort** | `O(n)` | `O(n²)` | `O(n²)` | `O(1)` | Yes | Yes |
| **Merge Sort** | `O(n log n)` | `O(n log n)` | `O(n log n)` | `O(n)` | Yes | No |
| **Quick Sort** | `O(n log n)` | `O(n log n)` | `O(n²)` | `O(log n)` | No | Yes |
| **Heap Sort** | `O(n log n)` | `O(n log n)` | `O(n log n)` | `O(1)` | No | Yes |
| **Counting Sort** | `O(n + k)` | `O(n + k)` | `O(n + k)` | `O(n + k)` | Yes | No |
| **Radix Sort** | `O(d(n + k))` | `O(d(n + k))` | `O(d(n + k))` | `O(n + k)` | Yes | No |
| **Bucket Sort** | `O(n + k)` | `O(n + k)` | `O(n²)` | `O(n + k)` | Yes* | No |

`n` = number of elements · `k` = range of key values · `d` = number of digits · *if sub-sort is stable

---

## Slide 17 -- When to Use Which Sort

| Scenario | Best choice | Why |
|----------|------------|-----|
| Small array (`n ≤ 32`) | **Insertion sort** | Low overhead, excellent cache locality, adaptive |
| General-purpose, stability required | **Merge sort / Timsort** | Guaranteed `O(n log n)`, stable |
| General-purpose, speed priority | **Quick sort / Introsort** | Fastest average case, in-place |
| Worst-case guarantee + minimal memory | **Heap sort** | `O(n log n)` guaranteed, `O(1)` space |
| Integers in small range | **Counting sort** | Linear time when `k = O(n)` |
| Fixed-width integers or strings | **Radix sort** | Linear time, stable |
| Uniformly distributed floats | **Bucket sort** | Expected linear time |
| Data too large for memory | **External merge sort** | Merge sorted runs from disk |
| Nearly sorted data | **Insertion sort / Timsort** | Adaptive -- exploits existing order |

> **Rule of thumb:** use your language's built-in sort (Timsort in Python/Java, Introsort in C++). Only reach for a specialised algorithm when profiling reveals a bottleneck.

---

## Slide 18 -- Real-World Implementations

### Timsort

Hybrid merge + insertion sort. Designed by Tim Peters for Python (2002). Now used in Python, Java, Android, Swift, Rust (stable sort).

- Detects natural "runs" (pre-sorted sequences)
- Extends short runs with insertion sort (minrun ~32--64)
- Merges runs using a stack-based strategy with galloping mode
- `O(n)` on already-sorted data
- `O(n log n)` worst case, stable
- `O(n)` auxiliary space

### Introsort

Hybrid quick + heap + insertion sort. Default in C++ `std::sort`, .NET, Go.

- Starts with quicksort (fast average case)
- Monitors recursion depth -- if it exceeds `2 log n`, switches to heapsort
- Falls back to insertion sort for small partitions
- `O(n log n)` guaranteed worst case
- In-place, **not** stable
- Best of all three algorithms' strengths

### pdqsort

Pattern-defeating quicksort. Used in Rust (unstable sort), Boost, Go 1.19+.

- Builds on Introsort with better pivot selection
- Detects and handles adversarial patterns
- Uses block partitioning for better branch prediction
- Adaptive: `O(n)` on sorted, reverse-sorted, and many repeated elements
- `O(n log n)` worst case
- Fastest known general-purpose unstable sort

**Language defaults:** Python → Timsort | C++ → Introsort | Rust → Timsort (stable) / pdqsort (unstable) | Java → Timsort (objects) / Dual-Pivot QS (primitives)

---

## Slide 19 -- Parallel & External Sorting

### Parallel sorting

Modern hardware has many cores. Sorting algorithms can exploit parallelism to achieve significant speedups.

| Approach | Description |
|----------|------------|
| **Parallel merge** | Divide array into chunks, sort each chunk on a separate thread, then merge. `O(n log n / p)` with `p` processors. |
| **Bitonic sort** | Oblivious sorting network. Fixed comparison pattern regardless of data -- ideal for GPUs and SIMD. |
| **Sample sort** | Generalisation of quicksort to `p` partitions. Sample pivots to ensure balanced partitions across processors. |

**In practice:** `std::sort` with parallel execution policies (C++17), Java's `Arrays.parallelSort`, and Python's `concurrent.futures` for chunk-based parallel sorts.

### External sorting

When data exceeds available RAM, we sort in chunks that fit in memory, write sorted "runs" to disk, then merge the runs.

| Phase | Description |
|-------|------------|
| **Phase 1: Sort** | Read `M` bytes into RAM, sort with an efficient in-memory algorithm, write sorted run to disk. |
| **Phase 2: Merge** | K-way merge of sorted runs using a min-heap. Each run contributes one buffer page. |
| **I/O complexity** | `O(N/B · log_{M/B}(N/B))` disk I/Os, where `B` = block size, `M` = memory. |

> **Used everywhere:** database `ORDER BY` on large tables, `sort` Unix command on huge files, MapReduce shuffle phase, and any data pipeline sorting terabytes.

> **Replacement selection:** a tournament-tree technique that produces runs of average length `2M` -- reducing the number of merge passes.

---

## Slide 20 -- Practical Considerations

### Cache and memory effects

- Asymptotic complexity ignores constant factors and cache behaviour
- Quicksort's sequential access pattern is **cache-friendly** -- data in the same partition stays in L1/L2 cache
- Heapsort's parent/child jumps cause frequent **cache misses**
- Merge sort's auxiliary array doubles memory footprint -- relevant when `n` is large
- For linked lists, merge sort is ideal (no random access needed, `O(1)` extra space)

> **Branch prediction:** pdqsort uses branchless comparison and block partitioning to minimise branch mispredictions -- a major source of overhead in quicksort.

### Language and API considerations

- `Python list.sort()` is Timsort -- stable, `O(n log n)`. Use `key=` parameter for custom comparisons, not `cmp`
- `C++ std::sort` is Introsort -- unstable. Use `std::stable_sort` when stability matters
- `Java Arrays.sort` uses Timsort for objects (stable) and dual-pivot quicksort for primitives (faster, no stability needed)
- Always prefer built-in sorts over hand-rolled implementations -- they are tuned, tested, and optimised for the runtime

> **Comparison function contract:** your comparator must define a strict weak ordering (irreflexive, asymmetric, transitive). Violating this causes undefined behaviour in C++ and subtle bugs everywhere.

---

## Slide 21 -- Summary & Further Reading

### Key takeaways

- Comparison sorts have a fundamental lower bound of `Ω(n log n)` -- merge sort and heap sort are asymptotically optimal
- Non-comparison sorts (counting, radix, bucket) bypass this limit by exploiting key structure
- No single sort wins on every metric -- stability, memory, adaptivity, and cache behaviour all matter
- Insertion sort is the best quadratic sort and is used as a sub-routine in all major hybrid algorithms
- Real-world sorts are hybrids: Timsort (merge + insertion), Introsort (quick + heap + insertion), pdqsort (quicksort + heap + insertion + pattern detection)
- Use your language's built-in sort unless profiling reveals a specific bottleneck
- External merge sort handles data larger than memory
- Understanding sorting deeply improves your ability to reason about algorithmic trade-offs in general

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- the canonical reference for sorting algorithms and analysis |
| **Sedgewick** | *Algorithms*, 4th ed. -- excellent treatment of quicksort, mergesort, and practical implementations |
| **Knuth** | *The Art of Computer Programming, Vol. 3: Sorting and Searching* -- exhaustive mathematical analysis |
| **Peters (2002)** | [listsort.txt](https://github.com/python/cpython/blob/main/Objects/listsort.txt) -- Tim Peters' original Timsort description |
| **Musser (1997)** | *Introspective Sorting and Selection Algorithms* -- the Introsort paper |
