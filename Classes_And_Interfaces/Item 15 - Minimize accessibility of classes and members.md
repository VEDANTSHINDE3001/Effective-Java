> # Item 15: Minimize Accessibility of Classes and Members
> 
> ## Core Idea
> - Make each class or member as inaccessible as possible.
> - Encapsulation (information hiding) reduces coupling and increases maintainability.
> 
> ## Why This Matters (Real Engineering Impact)
> - ### Loose Coupling      
>   If fewer things are visible, fewer components depend on them.   
> - ### Safer Refactoring    
>   Private implementation can change without breaking clients.   
>   When a class’s internal details are `private`, other classes cannot depend on them.  
>   So you can safely change the implementation without breaking external code.
>   Example:
>   ```java
>   public class DiscountCalculator {
>
>    public double calculate(double price) {
>        return applyDiscount(price);
>    }
>
>    private double applyDiscount(double price) {
>        return price * 0.9;
>    }
>   }
>   ```    
>   Other classes can only use:
>   ```
>   calculator.calculate(100);
>   ```
>   They cannot access `applyDiscount()` because it is `private`.
>   Later you can refactor or even replace the whole logic with a different algorithm.      
>   No external code breaks because clients only depend on the public API (`calculate`), not the internal implementation.
> - ### Better API Design
>   You force yourself to expose only what truly belongs in the public contract.   
> - ### Easier Debugging
>   Smaller visible surface area → fewer misuse cases.
> 
> ## Access Levels in Java (Most Restrictive → Least)
> ```
> private
> (default) package-private
> protected
> public
> ```
> ### Golden rule:
>>  Always use the lowest possible access level.
> 
> ## Practical Guidelines
> ### 1. Make Top-Level Classes Package-Private Unless They Are API
>   ```java
>    // Bad
>    public class HelperUtil { }
>
>    // Good
>    class HelperUtil { }
>   ```    
>   If a class is used only inside a package, it should not be `public`.
>   #### Real-world example (Spring Boot project):
>       - `UserController` → public (API layer)
>       - `UserServiceImpl` → package-private
>       - `UserMapper` → package-private
>       - `InternalValidator` → private nested class   
>    
>   This keeps internal design hidden.
> 
> ### 2. Make Fields Private (Almost Always)
>   Why making fields `public` is a bad idea?
>   - Breaks encapsulation
>   - Can't make immutable  
>
>   It's better to make fields `private` and give a getter method.   
> 
>   Or even better → Make class immutable when possible
> 
>   How to make a class immutable?
>     - Declare the class as `final`
>     - Make all fields `private` and `final`
>     - Do not provide setter methods
>     - Return copies of mutable objects in getters
>    
> ### 3. Be Careful with `protected`
>   `protected` exposes members to:
>
>   - Subclasses
>   - Entire package   
>   Once you make something `protected` in a `public` class, you are committing to support it forever.  
>   Example: 
>   ```java
>   public class PaymentProcessor {
>
>      protected void validatePayment() {
>        // validation logic
>      }
>   }
>   ```   
>   Another developer may extend it:
>   ```java
>   class CustomPaymentProcessor extends PaymentProcessor {
>      @Override
>      protected void validatePayment() {
>        // custom behavior
>      }
>   }
>   ```
>   Now your `validatePayment()` method has become part of the external contract.  
>   If you later:
>   - remove it
>   - change its behavior
>   - change its signature
>   
>   You break existing subclasses   
> ### 4. Avoid Public Mutable Fields (Especially Arrays)
>    Very dangerous:
>    ```java
>    public static final String[] ROLES = {"ADMIN", "USER"};
>    ```
>    Anyone can modify:
>    ```java
>    User.ROLES[0] = "HACKED";
>    ```
>    Fix:
>    ```java
>    private static final String[] ROLES = {"ADMIN", "USER"};
>
>    public static final List<String> roles = List.of(ROLES);
>    ```
>    Or return a defensive copy.  
>    You'd think the array is declared `final` but it means the reference cannot be changed - the value of the object it points can be changed.   
>    This problem will not occur for mutable datatypes like `String`
> ### 5. Use Private Constructors for Utility Classes
>    ```java
>    public final class StringUtils {
>       private StringUtils() { }  // Prevent instantiation
>    } 
>    ```
> Prevents accidental creation.
> 
> ### 6. Nested Classes Should Be Private
> If a nested class is used only internally:
> ```java
> public class OrderService {
>
>    private static class OrderValidator {
>        //...
>    }
> }
> ```
> Never expose internal implementation types.
> 
> ## Special Case: Public Constants
> Allowed only if:
> - Primitive types
> - Immutable objects
> ```java
> public static final int MAX_USERS = 100;
> ```
> Avoid exposing:
> - Arrays
> - Collections (unless unmodifiable)
> 
> ## Important Design Principle
>> ### Designing a public class is designing a permanent contract. 
> Once something is `public`:
> - You cannot remove it → (other code might be using it)
> - You cannot change its behavior → (logic will break in older code that is expecting a certain behavior)
> - You cannot restrict it later → (you can't make it private since other code is using it)
>
> ## Rule of Thumb Checklist
> Before making something public, ask:
> - Is this part of my logical abstraction?
> - Will external clients depend on this?
> - Am I okay supporting this forever?
> - Can I make it package-private instead?
> - Can I make it private?   
> If unsure → choose more restrictive.