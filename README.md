# multithread-lab

## First Demo ##


## Cancelling Tasks ## 
* The `CancellationTokenSource.CreateLinkedTokenSource` can linked multiple `CancellationTokenSource` to stop task whatever linked token be fired.

## Waiting For Time To Pass ##


## Waiting For Task ##
* `Task.WaitAll(t1, t2)` waiting for the longest task. In the sample will be waiting the `t1` task completed
* `Task.WaitAny(t1, t2)` waiting for the first completed task. In the sample will be waiting the `t2` task completed
* `Task.WaitAny(new[] { t1, t2 }, 4000)` only waiting for 4 seconds. In the sample will be waiting the `t2` task completed

## Exception Handling ##


## Critical Sections ##
* `Interlocked.Add` to add new value to origin one.
* `Interlocked.Increment` to increment one to origin value.
* `Interlocked.Decrement` to decrement one to origin value.
* `Interlocked.MemoryBarrier` shorthand for `Thread.MemoryBarrier()`.
    * Synchronizes memory access as follows.
* `Interlocked.Exchange` sets a variable to a specified value as an atomic operation.
* `SpinLock` object for atomic operation that until you are able to execute.
    * `SpinLock(true)` enable thread owner tracking.
* Mutex object
    * Can shared between server different processes.
* Reader-Writer locks
    * Support lock recursion in ctor paramater but not recommended.


## Concurrent Collections ##
* `ConcurrentDictionary` asking to count that's an expensive operation.
* `ConcurrentQueue`
* `ConcurrentStack` provides the `TryPopRange` to pop items at once.
* `ConcurrentBag` provides no ordering guarantees.
* `BlockingCollection` is a wrapper around one of the `IProducerConsumerCollection` classes.
    * Provides blocking and bounding capabilites.

## Task Coordination ##
* Contiuation
    * `ContinueWith` api provides to continue next task.
        * Continuations can be conditional.
            * TaskContinuationOptions.NotOnFaulted.
        * Beware of waiting on continuations that might not occur by faulted. That will be waiting forever.
    * `Task.Factory.ContinueWhenAll` provides api to waiting for all tasks.
        * One-to-many continuations.
    * `Task.Factory.ContinueWhenAny` provides api to waiting any first completed task.
        * One-to-any continuations.
        
* Child task
    * `TaskCreationOptions.AttachedToParent` for attach to parent.
    * `TaskContinuationOptions.OnlyOnRanToCompletion` continue when task ran to completion.
    * `TaskContinuationOptions.OnlyOnFaulted` continue when task ran to failted.

* Synchronization Primitives
    * All do same thing. 
        * They have a counter.
        * Let you execute N threads at a time.
        * Other threads are unblocked until state changes.

    * `Barrier`
        * Blocks all threads until N are waiting , then lets those N through via `SignalAndWait()`.
    * `CountdownEvent`
        * Signaling and waiting separate; waits until signal level reaches 0, then unblocks.
    * `ManualResetEventSlim`
        * Like `CountdownEvent` with a count of 1.
        * `Set()` to release all block threads. After `Set()` signaled, threads that call `WaitOne()` do not block.
        * `Reset` starting block threads. 
    * `AutoResetEvent`
        * Resets after waiting.
    * `SemaphoreSlim`
        * Counter `CurrentCount` decreased by `Wait()` and increased by `Release(N)`.
        * Can have a maximum.

## Parallel Loops ## 
* Parallel.For/Foreach are blocking calls
    * Wait until threads completed or an exception occurred.
* Can check the state of the loop as it is executing in `ParallelLoopState`.
* Can check result of execution in `ParallelLoopResult`.
* `ParallelLoopOption` let us customize execution with:
    * Max degree of parallelism.
    * Cancellation token.
* `Parallel.Invoke`
    * Run several provided functions concurrently.
    * Is equivalent to:
        * Creating a task for each lambda.
        * Doing a `Task.WaitAll()` on all the tasks.
* `Parallel.For`
    * Uses an index [start; finish].
    * Cannot provide a step.
        * Create an `IEnumerable<int>` and use `Parallel.ForEach`
    * Partitions data into different tasks.
    * Execute provided delegate with counter value argument.
        * Might be inefficient.
* `Parallel.ForEach`
    * Like `Parallel.For` but takes an `IEnumerable<T>` instead.
    * If enumerator using `yeild return` that must watch out cause you're never going to find out how long the total number of elements is.
* Thread local storage
    * Writing to a shared variable from many tasks is inefficient.
        * Interlocked every time.
    * Can store partially evaluated results for each task.
    * Can specify a function to integrate partial results into final result.
* Partitioning
    * Data is split into chunks by a `Partitioner`.
    * Can create your own.
    * Goal is improve performance.
        * Void costly delegate creation calls.


## Parallel Linq ##
* Turn Linq query parallel by 
    * Calling `AsParallel()` on an IEnumerable.
    * Use a `ParallelEnumerable`
* Use `WithCancellation()` to provide a cancellation token.
* Catch
    * `AggregateException`
    * `OperationCanceledException` if expecting to cancel.
* `WithMergeOptions(ParallelMergeOptions.xxx)` determine how soon produced result can be consumed.
* Parallel version of `Aggregate` provides a syntax for custom per-task aggregation options.

## Async and await ##
* What does await do?
    * Registers continuation with the async operation.
        * In other words, "code that follows me is a continuation, like `ContinueWith()`"
    * Gives up the current thread.
        * It's over. We no longer do anything on the current thread.
        * The call happens on a thread from TPL thread pool.
    * If `SynchroizationContext.Current` != null, ensures the continuation is also posted there(e.g., on the UI thread)
        * If null, continuation scheduled using current task scheduler.
        * This behavior can be customized right on the call.
    * Coerces the result of the asynchronous operation.
* `Task.Run`
    * Equivalent to 
        * Task.Factory.StartNew(something,
            CancellationToken.None,
            TaskCreationOptions.DenyChildAttach,
            TaskScheduler.Default);
    * Provides 8 overloads to support combinations of 
        * Task vs Task<T>
        * Cancelable vs Non-Cancelable.
        * Synchronous vs asynchronous delegate.
* `await` can be used a the language equivalent of `Unwrap()`
* Task Utility Combinators
    * Kind of like `Task.WaitAll/WaitAny`, but block the current thread.
    * Crate brand new tasks (useful for async/await)
    * `Task.WhenAny`
        * Creates a task that will complete when any of the supplied tasks has completed.
        * await Task.WhenAny(downloadFromHttp, downloadFromFtp); 
    * `Task.WhenAll()`
        * Creates a task that will complete when all of the supplied task have completed.
        * await Task.WhneAll(measureTemperature, measurePressure);
* Asynchronous programming is enableed with the `async` and `await` keywords.
* Methods that `await` must be decorated with `async`.
* When you await, you abandon current thread, fire up a new one and then do a context-preserving continuation of the code that's left.
* You can call async method synchronously.
* .net 4.5 gives us awaitable `Task.Delay` and combinators.
