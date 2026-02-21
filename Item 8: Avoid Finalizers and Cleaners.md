> # Item 8: Avoid Finalizers and Cleaners
>
> ## What Are Finalizers and Cleaners?
> ### Finalizer
> ```java
>   @Override
>   protected void finalize() throws Throwable {
>     // cleanup code
>   }
> ```
>
> ### Cleaner
> Replacement for finalizers
> ```java
>   Cleaner cleaner = Cleaner.create();
>   cleaner.register(this, () -> {
>     // cleanup action
>   });
> ```
>
>> It's recommended not to use both of them
>
> ### Problems:
> - Non-deterministic (we don't know when it will be called)
> - Performance overhead
>
> ## Why Finalizers/Cleaners Are Dangerous?
> ### 1. No Guarantee They Run
>    The GC may:
>    - Run late
>    - Run never (if JVM exits)
>    - Be delayed under load          
>    **If you have a class that has `finalize()` method and it writes to a file; if GC doesn't run"**
>      - You hit "Too many files open"
>      - Production crash
>       This might occur when system is under load and GC doesn't kick in.
> ### 2. Unpredictable Timing
>    GC timing depends on:
>    - Memory pressure
>    - Heap size
>    - System load
>    So resource release is unpredictable.
> ### 3. Performance Penalty
>    Objects with finalizers:
>    - Are treated specially by GC
>    - Survive longer
>    - Go through at least two GC cycles        
>       
>    Meaning:
>    - Slower GC
>    - More memory pressure
>    - Throughput degradation        
>
> ## So What Should We Do Instead?
> ### Use Explicit Termination (Autocloseable)
> Correct pattern
> ```java
>   class FileWriterWrapper implements AutoCloseable {
>     private final FileOutputStream out;
>
>     public FileWriterWrapper(String path) throws Exception {
>       this.out = new FileOutputStream(path);
>     }
>
>     @Override
>     public void close() throws Exception {
>       out.close();
>     }
>   }
> ```
>
> Usage: (try-with-resources)
> ```java
>   try (FileWriterWrapper writer = new FileWriterWrapper("file.txt")) {
>      // use writer
>   }
> ```
> This is:
> - Deterministic
> - Safe
> - Fast
> - Production ready        
>      
> ## Real World Examples
> - JDBC connections    
>
>   Wrong:
>   ```java
>   connection = dataSource.getConnection();
>   // hope GC closes it
>   ```
>
>   Correct:
>   ```java
>   try (Connection con = dataSource.getConnection()) {
>    // try-with-resources
>   }
>   ```
> - Thread Pools
>   ```java
>   executor.shutdown();
>   executor.awaitTermination(...);
>   ```
> - Native Resources (File handles, sockets, streams)
>   Anything outside JVM memory must be explicitly closed.              
>               
>   If you rely on GC:
>   - You will exhaust DB connection pool
>   - Your production system will go down
>
> ## Word of advice - You should never write `finalize()`.
>
> ## Design Rules
> 1. Any class holding a resource must:
>    - Implement `AutoCloseable`
>    - Be used with try-with-resources
> 2. Never rely on GC for:
>    - DB connections
>    - File IO
>    - Sockets
>    - Thread pools
>    - Locks
> 3. Prefer Composition Over Finalization
>    Use:
>    - `@PreDestroy`
>    - `DisposableBean`
>    - Bean lifecycle hooks
>    **Never finalizers**
>
> ## Core Principle Behind This Item
>>  Garbage Collection is for memory management, not for resource management.   
>   Memory is inside JVM.    
>   Files, sockets, DB connections are outside JVM.    
>   GC only understands memory.
>
> ## Interview-Level Summary
> ### Why avoid finalizers?
> Answer:
> - Non-deterministic execution
> - Performance penalty
> - Can cause object resurrection
> - Deprecated
> - Explicit resource management is safer
>
>           
