# Effective-Java





> ## Creating and Destroying Objects
>
> - ### Consider static factory methods instead of constructors
>   
>   It's a static method that returns instance of the class instead of using a constructor.
>
>   Doesn't mean never use constructors - but static factory methods often provide more flexibility and clarity than constructors.
>   
>   ```java
>   public class User {
>     private final String name;
>     private final int age;
>
>     private User(String name, int age) {
>         this.name = name;
>         this.age = age;
>     }
>
>      public static User of(String name, int age) {
>          return new User(name, age);
>      }
>    }
>   ```
>      - ### ✅ Advantages of static factory methods
>      1. ### They have names (better readability)
>         Constructors  must use the class name, so intent is often unclear.
>         ```java
>         new BigInteger(1, 2, 3); // What do these mean?
>         ```
>         ```java
>         BigInteger.probablePrime(64, random);
>         ```
>         Real world examples:
>         - LocalDate.now()
>         - Path.of(...)
>         - Optional.ofNullable(...)
>         - List.copyOf(...)
>         - List.of(...)
>        
>         Code becomes more readable and self-documenting.
>      2. ### They are not required to create a new object
>         They can reuse existing instances, cache them or return singletons.
>         ```java
>          public static Boolean valueOf(boolean b) {
>            return b ? Boolean.TRUE : Boolean.FALSE;
>          }
>         ```
>         Instead of creating new Boolean objects, Java reuses two constants.
>         Real world examples:
>         - Integer.valueOf(int)
>         - Enum constants
>         - Flyweight objects (connection pools and thread pools)
>         They offer benefits such as less memory usage, better peformance and enables immutability.
>      3. ### They can return any subtype of the return type
>                              
>         Constructors must return exactly the class type.
>         Static factory methods can return any implementation.
>         ```java
>         public static List<String> emptyList() {
>            return new ArrayList<>();
>          }
>         ```
>         Caller does not depend on the concrete implementation and they don't even need to know (better security)
>         Real world usage:
>         ```java
>         List.of(1, 2, 3);      // returns an immutable list
>         Collections.emptyList(); // returns optimized implementation
>         ```
>         This enables:
>         - API evolution (it enables to change the which subtype is getting returned without affecting the client's code since client it not tied to any concrete implementation)
>         - Interface based design
>         - Implementation hiding
>        
>      4. ### Returned class can vary on input
>                                
>         Static factories can decide which class to return dynamically.
>         ```java
>         public static Shape create(String type) {
>           return switch (type) {
>              case "CIRCLE" -> new Circle();
>              case "RECT"   -> new Rectangle();
>              default -> throw new IllegalArgumentException();
>           };
>         }
>         ```
>         Real world examples:
>         - EnumSet.of(...)
>         - Collectors.toList()
>         - JDBC DriverManager.getConnection(...)
>        
>         This utilizes a famous pattern -> Factory Method
>      5. ### Returned class need not exist at compile time
>  
>         Static factory methods can return classes loaded later (via SPI, reflection, plugins).
>         eg: Service Provider Interface (SPI)
>         ```java
>         ServiceLoader<PaymentProvider> loader = ServiceLoader.load(PaymentProvider.class);
>         ```
>         The actual implementation is discovered at runtime.
>         Real world examples:
>         - JDBC Drivers
>         - Logging framweorks
>         - Java SPI (ServiceLoader)     
>         
>         Enables modular, pluggable architectures.
>            
>    - ### ❌ Disadvantages of static factory methods
>      1. ### Classes without public constructors cannot be subclassed
>               
>         If constructors are private, subclassing becomes impossible.
>         ```java
>         public final class UtilityClass {
>            private UtilityClass() {}
>         }
>         ```
>         Trade-off:
>         - Less extensible
>         - Safer, immutable, controlled design
>             
>         This is often intentional, not a flaw.
>                        
>      2. ### Static factory methods are not easily discoverable
>          
>         Constructors appear automatically in IDE suggestions.
>
>         Static methods require knowing their names.
>
>         Follow naming conventions to mitigate this issue:
>         - of -> Aggregate creation
>         - valueOf -> type conversion
>         - getInstance -> singleton
>         - newInstance -> new object
>         - from -> conversion
>         - copyOf -> defensive copy
>      
>      3. ### Can lead to too many factory methods
>
>         Poorly designed APIs may expose many similar static methods.
>         Tips:
>         - Keep factories cohesive
>         - Avoid overloading with unclear differences
>         - Follow naming conventions
>
>   - ### 🏗 Real-world design example
>
>     Without static factory (bad)
>     ```java
>     new User("Vedant", 25);
>     new User("Vedant", 25, true);
>     new User("Vedant", 25, false, Role.ADMIN);
>     ```
>
>     With static factory (good)
>      ```java
>      User.of("Vedant", 25);
>      User.admin("Vedant", 25);
>      User.guest("Vedant");
>      ```
>
>      Clear intent, fewer bugs, easier refactoring.
>
>   - ### When should you prefer static factory methods?
>     ✅ Prefer when:
>      - You want readable APIs
>      - You need caching or reuse
>      - You want to hide implementation
>      - You want immutability
>      - You design library/framework code
>
>     ❌ Prefer constructors when:
>      - Simple DTOs
>      - Subclassing is required
>      - API simplicity matters more than flexibility
