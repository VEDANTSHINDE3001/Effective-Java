# Item 7 – Eliminate Obsolete Object References

> This item is about preventing memory leaks in garbage-collected languages like Java.
>
> Java doesn’t have manual memory management bugs like C++ but can still have memory leaks caused by **lingering references**.
>
> ## Core Idea:
> An object becomes eligible for GC only when no live references point to it.                   
> If you accidentally keep a reference to an object that is no longer needed, the GC cannot reclaim it.
> 
>>  **Obsolete reference** – a reference that will never be used again but is still reachable.
>        
> ## Classic Example: Manual implementation of Stack
> #### Problematic implementation
> ```java
>   public class Stack {
>     private Object[] elements;
>     private int size = 0;
>
>     public Stack(int capacity) {
>       elements = new Object[capacity];
>     }
>
>     public void push(Object e) {
>       elements[size++] = e;
>     }
>
>     public Object pop() {
>       if (size == 0)
>        throw new EmptyStackException();
>        return elements[--size];
>     }
>   }
> ```
> #### What’s the Problem?
> When you `pop()`:
> - The object is removed logically.
> - But its reference still exists inside `elements[]`.
>
> So GC thinks:
>>  “This object is still reachable.”           
>
> Even though it’s not logically part of the stack anymore.
>
> #### Fix:
> ```java
>   public Object pop() {
>     if (size == 0)
>       throw new EmptyStackException();
>
>     Object result = elements[--size];
>     elements[size] = null;  // Eliminate obsolete reference
>     return result;
>   }
> ```
> Now:
> - The reference is cleared.
> - Object becomes eligible for GC.
>
> Yes, we no longer use manual implementation of stack but it's worth understanding how memory leaks happen.
>
> #### What's the problem:
> If such stack implementation is used memory usage keeps growing -> eventually:
> - High GC
> - OutOfMemoryError
> - Production crash
>
> ## Where Do Memory Leaks Commonly Happen in Java?
> ### 1. Self-Managed Memory
>    If your class manages its own memory (arrays, buffers, pools), you must null out references manually.
>    Examples:
>    - Custom cache
>    - Object pools
>    - Buffers
>    - Ring buffers
>    - Thread-local storage
>   
>     Most application code doesn’t do this, but frameworks sometimes do.
>
> ### 2. Caches
>    ```java
>    Map<String, User> cache = new HashMap<>();
>    ```
>    If entries are never removed:
>    - Objects remain forever
>    - Memory leak
>
>    #### Solutions:
>    - Caffeine
>    - Redis
>    - Time-based eviction
>    - Size-based eviction
>
> ### 3. Listeners & Callbacks
>    ```java
>    eventSource.registerListener(myListener);
>    ```
>
>       If:
>       - You never unregister
>       - Listener lives forever
>
>       Even if the listener object is unused elsewhere:
>       - It stays reachable
>       - Memory leak
>
>       #### Real word cases:
>       - Spring event listeners
>       - Scheduler callbacks
>       - Message broker consumers
>
>       If not deregistered → leak.
>
> ## The Danger of Static Fields
>    Static fields live for the lifetime of the application.
>    ```java
>      public class SessionHolder {
>        public static List<UserSession> sessions = new ArrayList<>();
>      }
>    ```
>    If sessions are never removed:
>    - Memory leak for entire app lifetime.
>
> ## ThreadLocal – Silent Memory Leak
> ```java
> private static final ThreadLocal<User> userHolder = new ThreadLocal<>();
> ```
>
>   If:
>   - You set value
>   - Forget to call remove()
>
>   In thread pools:
>   - Thread lives long
>   - Value stays attached
>   - Memory leak
>
>   ### Correct usage
>   ```java
>     try {
>       userHolder.set(user);
>         // logic
>       } finally {
>       userHolder.remove();
>     }
>   ```
>   In Spring Boot apps with thread pools → this is critical.
>
> ## Symptoms of memory leak:
> - Heap memory keeps growing
> - GC frequency increases
> - App slows down gradually
> - Eventually OOM
>
> ## Real Production Scenario
> Imagine:
> - You have a scheduler processing 1M records/day
> - You maintain an in-memory list of processed IDs
> - Never clear it
>
> Memory after:
> - 1 day → 200MB
> - 1 week → 2GB
> - Crash
>
>   The leak is not GC’s fault.
>   It’s an obsolete reference problem.
>
> ## Rule of thumb - Prefer Libraries Over Manual Memory Management     
>           
>           
