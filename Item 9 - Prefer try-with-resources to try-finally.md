> # Item 9: Prefer try-with-resources to try-finally
>
> ## The Problem with try–finally
>
> Earlier resource clean-up looked like this:
> ```java
> BufferedReader br = new BufferedReader(new FileReader("data.txt"));
> try {
>   return br.readLine();
> } finally {
>   br.close();
> }
> ```
>
> This seems fine until there are mutiple resources.
>
> When multiple resources:
> ```java
>   BufferedReader br = new BufferedReader(new FileReader("input.txt"));
>   BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"));
>
>   try {
>     bw.write(br.readLine());
>   } finally {
>     bw.close();
>     br.close();
>   }
> ```
>
> What if:
> - `bw.close()` throws exception?
> - Then `br.close()` never executes.
> Now you’ve leaked a resource.
>
> ## Even Worse: Exception Suppression Problem
> ```java
>   try {
>     throw new RuntimeException("Main Exception");
>   } finally {
>     throw new RuntimeException("Close Exception");
>   }
> ```
>
> Output:
> ``` Close Exception ```
> The original exception is LOST.
> This makes debugging production issues a nightmare.
>
> ## try-with-resources (Java 7+) for the rescue
> ```java
>   try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
>     return br.readLine();
>   }
> ```
> - No finally block.
> - Automatic cleanup.
>
> ### Multiple Resources (Correct Handling)
> ```java
>   try (
>     BufferedReader br = new BufferedReader(new FileReader("input.txt"));
>     BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"))
>   ) {
>     bw.write(br.readLine());
>   }
> ```
>
> What happens?
> - Resources closed in reverse order
> - All close() calls attempted
> - If exceptions occur → they are properly chained     
>   We will get suppressed exceptions now by doing `e.getSuppressed();`
>
> ## Why It Works?
> It works because the resource implements:
> ```java
> AutoCloseable
> ```
> Which means it has:
> ```java
> void close() throws Exception;
> ```
>
> ## Real-World Usage
> - JDBC Connections
>   Bad:
>   ```java
>   Connection con = dataSource.getConnection();
>   try {
>     ...
>   } finally {
>      con.close();
>   }
>   ```
>
>   Good:
>   ```java
>   try (Connection con = dataSource.getConnection()) {
>     ...
>   }
>   ```      
>   If you forget close():
>   - Connection pool exhaustion
>   - Production outage
> - File Handling
> - Streams
> - Custom Resource
>   If your class manages:
>   - DB connection
>   - File
>   - Native memory
>   - Socket
>   - Thread pool      
>   It should implement:
>   ```java
>   class MyResource implements AutoCloseable {
>     @Override
>     public void close() {
>       // release resource
>     }
>   }
>   ```
>   Then it can be used safely:
>   ```java
>   try (MyResource r = new MyResource()) {
>    ...
>   }
>   ```
>
> ## When Is try-finally Still Okay?
> Rare cases:
> - When no resource is involved
> - Pure logic cleanup
> - Lock unlocking (sometimes)
>
> Example:
> ```java
> lock.lock();
> try {
>    ...
> } finally {
>   lock.unlock();
> }
> ```
> Here try-with-resources isn’t natural unless you wrap the lock in an AutoCloseable adapter.
>
> ## Mental Model
> If something needs `close()`,    
> it belongs inside `try (...)`.
>           
>           
>               
