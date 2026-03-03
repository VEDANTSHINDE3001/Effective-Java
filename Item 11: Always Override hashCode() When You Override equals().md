> # Item 11: Always Override `hashCode()` When You Override `equals()`
>
> ## The Rule
> If two objects are equal according to `equals()`,        
> they must return the same `hashCode()`.          
> If you override `equals()` and not `hashCode()`, your class is broken.
>
> ## Why This Matters?        
> Hash-based collections use `hashCode` first, then `equals`.      
> Examples:
> - `HashMap`
> - `HashSet`
> - `ConcurrentHashMap`
> - `Many caching frameworks (like Spring cache)`
>
> Flow inside `HashMap`:
> ```
> 1. Compute hashCode()
> 2. Find bucket
> 3. Compare using equals()
> ```
> If hashCodes differ → `equals()` is NEVER even called.
>
> ## Real-World Bug Example
> ```java
>   class User {
>     private String email;
>
>     public User(String email) {
>       this.email = email;
>     }
>
>     @Override
>     public boolean equals(Object o) {
>       if (!(o instanceof User)) return false;
>       User u = (User) o;
>       return email.equals(u.email);
>     }
>  }
> ```
> Now:
> ```java
> Set<User> users = new HashSet<>();
> users.add(new User("a@gmail.com"));
> System.out.println(users.contains(new User("a@gmail.com"))); 
> ```
> ❌ Output: false
> 
> Why?      
> Because `hashCode()` is still from `Object`, which is memory-based.           
> So the two equal objects:
> - have different hashCodes
> - go into different buckets
> - are treated as different
>        
> ## In Production, This Causes
> - Duplicate entries in `HashSet`
> - Cache misses
> - Broken `HashMap` lookups
> - Hard-to-debug bugs
> - Memory leaks in maps
> - Subtle data corruption
>
> ## Interview Insight
> If interviewer asks:
>> Why override hashCode with equals?           
>
> Strong answer:
>>  Because hash-based collections use hashCode for bucket selection. If equal objects return different hashCodes, they will end up in different buckets and never be compared using equals, violating collection contracts.
>
> ## One-Line Mental Model
>> `equals()` defines logical equality.
>> `hashCode()` ensures equal objects land in the same bucket.
>     
> ## (Optional) How is hashcode calculated?
> For multiple fields:
> ```java
> @Override
> public int hashCode() {
>   int result = Integer.hashCode(id);
>   result = 31 * result + name.hashCode();
>   result = 31 * result + Double.hashCode(price);
>   return result;
> }
> ```
> Why 31?
> - Prime number
> - Good distribution
> - Optimized by JVM `(31 * x == (x << 5) - x)`             
