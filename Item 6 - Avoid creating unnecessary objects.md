> # Avoid Creating Unnecessary Objects     
>
> This item is about performance, memory efficiency, and clarity.
> It teaches:
> > Reuse objects when possible instead of creating new ones repeatedly.
>
> Especially important for:
> - High-throughput systems (APIs, schedulers, batch jobs)
> - Low-latency services
> - Tight loops
> - Large collections
>
> ## 1. Prefer Reuse to Re-Creation
> Avoid:
> ```java
> String s = new String("hello");
> ```
> Instead use:
> ```java
> String s = "hello";
> ```
> Why?:
> - `new String("hello")` cretaed a String literal "hello" in the string pool -> another String object on heap
> - `String s = "hello"` reuses the pooled String.0
>        
> It might seem trivial but if done in loop at can overload GC and spike up latency.
> eg:
> ```java
>   for (int i = 0; i < 1_000_000; i++) {
>      String status = new String("SUCCESS");
>   }
> ```
> This creates 1M unnecessary objects → GC pressure → latency spikes.
>
> ## 2. Avoid Creating Objects in Loops
> ```java
>   for (String name : names) {
>     Pattern p = Pattern.compile("[A-Z]+");
>     Matcher m = p.matcher(name);
>   }
> ```
> `Pattern.compile()` is expensive.
>
> Instead do this:
> ```java
>   private static final Pattern UPPERCASE = Pattern.compile("[A-Z]+");
>   for (String name : names) {
>     Matcher m = UPPERCASE.matcher(name);
>   }
> ```
>
> ### Why this matters?
> Imagine:
> - API receiving 10k req/sec
> - Regex compiled per request
> - Massive CPU + GC churn
> This will lead to:
> - Increased GC pauses
> - Higher latency
> - CPU waste
>
> ## 3. Use Static Factory Methods Instead of Constructors When Reuse is Possible
> ```java
>   Boolean.valueOf("true");
> ```
>
> Instead of:
> ```java
> new Boolean("true");
> ```
>
> `Boolean.valueOf()` returns cached instances.
>
> ### Why?
> Boolean has only two values: TRUE and FALSE.
> It reuses:
> - Boolean.TRUE
> - Boolean.FALSE
>
> **This is why constructors of wrapper classes are deprecated.**
>
> ## 4. Prefer Primitives Over Boxed Types
>
> Bad:
> ```java
>   Long sum = 0L;
>
>   for (long i = 0; i < 1_000_000; i++) {
>     sum += i;
>   }
> ```
>
> This causes:
> - Boxing
> - Unboxing
> - Creation of new Long objects
>
> Good:
> ```java
>   long sum = 0L;
>
>   for (long i = 0; i < 1_000_000; i++) {
>     sum += i;
>   }
> ```
> ### Why is it not a good idea?
> - Memory Overhead (Heap usage)
>   A primitive int takes 4 bytes. An Integer object requires 16-24 bytes (or more) because it contains the 4-byte value plus object headers (mark word, class pointer) and alignment padding.
> - Object Creation & Indirection
>   Boxing converts a value into a heap-allocated object.
>   When accessing the value, the CPU must first dereference a pointer to the object in the heap, rather than directly accessing a value on the stack (primitive), slowing down access speeds.
> - Garbage Collection Pressure
>   Because boxed objects are objects on the heap, creating millions of them (e.g., in a loop) generates significant work for the Garbage Collector, leading to longer pauses.
> - Immutability and Identity
>   Boxed classes are immutable. You cannot change the value inside an Integer object; you must create a new one.
>   Additionally, boxed objects have "identity." Two Integer objects might both represent the value 1000, but (obj1 == obj2) will be false because they are different instances in memory. This often leads to subtle bugs that primitives avoid entirely.
>
> ### Why this matters
> Autoboxing in:
> - Streams  (you use `IntStream`, `LongStream`, `DoubleStream`)
> - Map keys (you can use some libraries when performance is necessesary)
> - Aggregations
>   Can cause hidden performance problems
>
> Especially in:
> - Batch jobs
> - Financial calculations
>
> ## 5. Be Careful with `Optional`
>
> ```java
>  Optional<String> value = Optional.of("test");
> ```
> If done in hot loops → unnecessary object creation.
> Design Advice:
> - Don't use `Optional` for:
>   - Fields
>   - Method parameters
> - Use for return types only
>
> ## 6. Avoid Unnecessary Defensive Copies
> Sometimes defensive copies are required (immutability). But don't overdo them.
> If field/collection is already immutable then don't create defensive copies.
> This add overhead.
>
> ### Balance:
> - External API -> defensive copy (if an external api passes a field then make a defensive copy because if while you're processing the api changes the state then there would be undetected bugs.)
> - Internal code -> reuse
>
> ## 7. Avoid Creating Expensive Objects Repeatedly
> Examples:
> - `ObjectMapper` in Spring
> - `RestTemplate`
> - `ThreadPoolExecutor`
> - Database connections
>
> Create only once as `static final` fields or let Spring manage them as singleton beans.
>
> ### Real production issue:
> Creating:
> - Thread pools per request
> - DB connections per method
> - HTTP clients per call
> Will:
> - Exhaust resources
> - Kill performance
> - Crash system
>
> ## 8. Use Caching When Appropriate
> Instead of:
> - Re-querying DB every time
> - Rebuilding heavy objects
> Better:
> - Use Caffeine
> - Use Redis
> - Use Spring Cache
>
> ## 9. Watch Out for String Concatenation in Loops
> Bad:
> ```java
>   String result = "";
>   for (String s : list) {
>     result += s;
>   }
> ```
>
> Creates new String each iteration.
>
> Instead"
> ```java
>   StringBuilder sb = new StringBuilder();
>   for (String s : list) {
>      sb.append(s);
>   }
> ```
>
> ## Design-Level Takeaways:
> 1. Understand Object Lifecycle
>    - Who creates it?
>    - How often?
>    - How long does it live?
> 2. Think about GC Pressure
>    Frequent allocations → frequent minor GC → latency spikes.
> 3. Singleton / Bean Reuse in Spring
>    Let container manage expensive objects.
> 4. Avoid Accidental Autoboxing
>    Especially in collections and streams.
>
> ## Final Summary:
> Avoid creating unnecessary objects by:
> - Reusing immutable objects
> - Preferring primitives
> - Avoiding object creation in loops
> - Caching expensive instances
> - Avoiding unnecessary boxing
> - Being mindful in hot paths
