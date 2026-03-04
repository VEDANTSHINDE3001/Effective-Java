> # Item 14: Consider implementing `Comparable`
>
> If your class has a natural ordering, implement `Comparable<T>`.
> This allows objects to be:
> - Sorted
> - Stored in sorted collections
> - Compared consistently across the system
>
> ## What Is Comparable?
> It defines:
> ```java
> int compareTo(T other);
> ```        
> Return:
> - `< 0` → this < other
> - `0` → equal
> - `> 0` → this > other
>
> ## Real-World Example
> Domain: `User` sorted by email
> ```java
> class User implements Comparable<User> {
>
>   private final String email;
>
>   public User(String email) {
>     this.email = email;
>   }
>
>   @Override
>   public int compareTo(User other) {
>     return this.email.compareTo(other.email);
>   }
> }
> ```
> Now you get sorting automatically:
> ```java
> List<User> users = ...
> Collections.sort(users);
> ```
>
> Or store in:
> ```java
> Set<User> users = new TreeSet<>();
> ```
>
> ## Why This Matters in Real Systems    
> Sorted structures like:
> - `TreeSet`
> - `TreeMap`
> - Priority queues        
> depend on `compareTo()`       
> Without `Comparable`, you must always pass a `Comparator`
>
> ## Critical Rule: Must Be Consistent with `equals()`
> If:
> ```java
> a.equals(b) == true
> ```
> Then ideally:
> ```java
> a.compareTo(b) == 0
> ```
> If not → subtle bugs.
>
> ### Bug example
> ```java
> class User implements Comparable<User> {
>    private String email;
>    private int age;
>
>    @Override
>    public int compareTo(User o) {
>       return Integer.compare(this.age, o.age);
>    }
>
>    @Override
>    public boolean equals(Object o) {
>       return email.equals(((User)o).email);
>    }
> }
> ```
> Now:
> ```java
> User u1 = new User("a@gmail.com", 25);
> User u2 = new User("a@gmail.com", 30);
> ```
>
> - `equals()` → true (same email)
> - `compareTo()` → not zero (age differs)           
>       
> If two keys are deemed equal by `compareTo` (returns 0), the TreeMap treats them as the same key, even if their `equals()` method would return false.
> The comparator defines equality.
> 
> #### If put into `TreeSet` you may get:
> - Duplicate logical objects
> - Missing elements
> Production nightmare.
>
> #### If `compareTo()` and `equals()` disagree → inconsistent data behavior.
>
> ## Best Practice Implementation
> Use `Comparator` utilities:
> ```java
> @Override
> public int compareTo(User other) {
>    return Comparator
>            .comparing(User::getEmail)
>            .thenComparing(User::getAge)
>            .compare(this, other);
> }
> ```
>
> ## Senior-Level Insight
> Implement `Comparable` only when:
> - The class has ONE clear natural ordering.
> - It won’t change frequently.    
> If multiple orderings exist → don’t implement `Comparable`.      
> Instead provide `Comparator`s.
