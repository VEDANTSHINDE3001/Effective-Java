> # Item 10 – Obey the General Contract When Overriding `equals()` (especially with collections like `HashSet` and `HashMap`)
>
> ## When Should You Override `equals()`?        
>  Ask yourself:
>>  If two objects contain same data, should the system treat them as the same logical thing?      
>>  If YES → override `equals()`        
>>  If NO → don’t 
>          
> Never override:
> - Thread
> - Socket
> - Runnable
> - Controller
> - Beans
>
>> Override when instances represent a value -> DTO
>
> ## The General Contract (MUST follow all 5)
> 1. Reflexive
>    ```java
>    x.equals(x) == true
>    ```
> 2. Symmetric
>    ```java
>    x.equals(y) == y.equals(x)
>    ```
> 3. Transitive
>    ```java
>    x.equals(y) && y.equals(z) ⇒ x.equals(z)
>    ```
> 4. Consistent
>    Multiple calls return same result (if state unchanged).
> 5. Non-null
>    ```java
>    x.equals(null) == false
>    ```
>
> ## Correct Implementation Pattern (Modern Java)
> ```java
>   @Override
>   public boolean equals(Object o) {
>     if (this == o) return true;
>     if (!(o instanceof User user)) return false;
>     return age == user.age &&
>       Objects.equals(name, user.name);
>     }
> ```
> Why this pattern?
> - `this == o` → fast path (simply checks addresses of both objects)
> - `instanceof` → type safety
> - Compare significant fields only
> - Use `Objects.equals()` for null safety
>
> ## Real-World Bug Example (Symmetry Violation):
>
> Bad design:
> ```java
>   class Money {
>     private final int amount;
>
>     @Override
>     public boolean equals(Object o) {
>       if (!(o instanceof Money)) return false;
>       return amount == ((Money) o).amount;
>     }
>   }
> ```
>  Subclass
> ```java
>    public class Voucher extends Money {
>      private final String store;
>
>      public Voucher(int amount, String store) {
>        super(amount);
>        this.store = store;
>      }
>
>      @Override
>      public boolean equals(Object o) {
>       if (o instanceof Voucher) {
>           Voucher other = (Voucher) o;
>           return super.equals(o) &&
>           this.store.equals(other.store);
>        }
>         return false;
>      }
>    } 
>
> ```
>
> What Goes Wrong?
> ```java
>   Money cash = new Money(100);
>   Voucher voucher = new Voucher(100, "Amazon");
>
>   cash.equals(voucher);    // ✅ true
>   voucher.equals(cash);    // ❌ false
> ```
>
> Result -> symmetry broken
>
> ### Real world impact?
>
> HashSet corruption
> ```java
>   Set<Money> set = new HashSet<>();
>   set.add(voucher);
>
>   set.contains(cash);  // unpredictable behavior
> ```
>    
> You’ll see:
>  - Duplicate entries
>  - Failed lookups>
>  - Data inconsistencies
> Solutions:
> 1. (Recommended) Make Class Final
> 2. Use `getClass()` instead of `instanceOf()`
> > Avoid inheritance in value objects.
>
> ## Always Override `hashCode()` with `equals()`        
> If you override equals() but not hashCode():
> ```java
> Set<User> users = new HashSet<>();
> users.add(new User("Vedant", 25));
>
> users.contains(new User("Vedant", 25)); // false ❌
> ```        
> Because HashSet uses `hashCode()` first.
> They use `hashCode()` to find the bucket        
> And `equals()` to cpmpare inside the bucket
> 
> If `hashCode()` differs:
> - They go to different buckets
> - `equals()` is NEVER even called
>
> Result: lookup fails
> 
> 
>
> Correct:
> ```java
>  @Override
>  public int hashCode() {
>     return Objects.hash(name, age);
>  }
> ```
>
> Why?
>>  If two objects are equal according to equals(),
>>  they MUST return the same hashCode().
>
> ## Design Advice for Modern Java      
> Prefer records for value types
> ```java
>   public record User(String name, int age) {}
> ```
> You get:
> - Correct `equals()`
> - Correct `hashCode()`
> - Correct `toString()`
>
> ## How to avoid Subclass symmetry violation?        
> If class is meant to be final → use `instanceof` safely.
> If class is non-final and equality must be strict → use:
> ```java
> if (o == null || getClass() != o.getClass()) return false;
> ```
>            
> ## Practical Summary
> - Override equals() only for value objects
> - Always override hashCode() together
> - Never break symmetry via inheritance
> - Use only significant fields
> - Prefer immutability
