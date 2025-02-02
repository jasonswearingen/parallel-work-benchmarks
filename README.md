# Archived
These benchmarks were created in the net6/8 timeframe, and offer great insight as to perf characteristics, but no further development is expected/needed.

# lowlevel-benchmarks
benchmarking ways of doing lowlevel work in dotnet.



# TLDR;
1. **Don't do cheap work in parallel**.  For example, in some of these benchmarks I get a 2x speedup for 16x the cpu cost.
1. For lookups, `Span` is fastest.   Prefer `Array`/`List` for lookups in hotpaths over `Dictionary`/`ConcurrentDictionary`.
1. Avoid add/remove from `ConcurrentDictionary` in hotpaths due to allocations.
1. `Interlocked` is expensive to use in hotpaths.   do per-thread sums or write out to a seperate results span for processing back on the main thread. Using a `ForRange()` parallel work function is best, for example: `/Helpers/Extras/ForRange.cs`
    - As per `Kozi` on the C# Discord: 
      >   The more important lesson here is "don't write to the same memory region from multiple threads if possible".  Writes within the same cache line will slow access on other threads.
      > And THAT'S why doing it per-thread is better.  And only summing at the end.  You minimise the writes to a shared cache line.  https://www.youtube.com/watch?v=WDIkqP4JbkE&t=247s
1. Linq and PLinq are not that bad.  Not super great, but not that bad.
1. `MemoryOwner<T>` is your friend.



# The Benchmarks

these are the benchmarks, contained in subfolders of `/Benchmarks/`.  Look at each sub folder for a `ReadMe.md` with individual findings:

- `Collections_Threaded` checks speed/correctness of doing collection read/writes from threads
- `Parallel_Work` checks doing work on `Span<T>` from threads
- `Parallel_Lookup` checks a real-world critical path scenario, random access lookup of 100,000 entities.  Benchmark tests using different backing storage collections and Sequential vs Parallel.

# How to use
1. open solution in visual studio 2022
2. run solution
3. pick a benchmark
4. wait a long time for benchmarks to run


# Folder Structure
- `Program.cs` - entrypoint
- `Benchmarks/*/*.cs` - benchmark tests
- `Helpers` - helpers for the benchmarking, such as:
   - `DumbWork.cs` - helper containing input data and output verification logic
   - `Data.cs` - helper containing structure of test data worked on in benchmarks
   - `zz_Extensions.cs` - extension method for `Span<T>` and `Array` to make parallel easier.

