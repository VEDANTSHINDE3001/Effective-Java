> # Prefer Dependency Injection to Hardwiring Resources
>
> ## What does “hardwiring resources” mean?
> Hardwiring = creating dependencies directly inside your class.
> ```java
>   public class SpellChecker {
>      private final Dictionary dictionary = new EnglishDictionary();
>
>      public boolean isValid(String word) {
>        return dictionary.contains(word);
>      }
>    }
> ```
> ### Problems:
> - Always uses `EnglishDictionary`
> - Hard to test
> - Cannot switch to `SpanishDictionary`
> - Violates Open/Closed Principle
> - Tight coupling
> This is rigid design.
>
> ## What is Dependency Injection (DI)?
> Instead of creating the dependency, you inject it from outside.
> ```java
>   public class SpellChecker {
>      private final Dictionary dictionary;
>
>      public SpellChecker(Dictionary dictionary) {
>        this.dictionary = Objects.requireNonNull(dictionary);
>      }
>
>      public boolean isValid(String word) {
>        return dictionary.contains(word);
>      }
>    }
> ```
>
> ```java
>  SpellChecker english = new SpellChecker(new EnglishDictionary());
>  SpellChecker spanish = new SpellChecker(new SpanishDictionary());
> ```
>
> This is:
> - ✔ Flexible
> - ✔ Testable
> - ✔ Reusable
> - ✔ Loosely coupled
>
> ## Why Constructor Injection is recommended
> Constructor injection is recommended over setter injection
> Why?
> 1. Ensures immutability
> 2. Dependency cannot be null
> 3. Object is fully initialized when created
> ```java
>   public class Service {
>      private final Repository repository;
>
>      public Service(Repository repository) {
>        this.repository = Objects.requireNonNull(repository);
>      }
>   }
> ```
> This is exactly what spring promotes.
>
> ## Real-World Example (Backend Service)
> ### Hardwired Service
> ```java
>   public class PaymentService {
>     private final RazorpayClient client = new RazorpayClient();
>
>     public void pay() {
>       client.process();
>     }
>   }
> ```
> Problems:
> - Cannot mock `RazorpayClient`
> - Cannot switch to Stripe
> - Tight coupling to vendor
>
> ### With Dependency Injection
> ```java
>   public interface PaymentGateway {
>      void process();
>   }
>
>   public class RazorpayGateway implements PaymentGateway {
>     public void process() { }
>   }
>
>   public class StripeGateway implements PaymentGateway {
>     public void process() { }
>   }
>
>   public class PaymentService {
>     private final PaymentGateway gateway;
>
>     public PaymentService(PaymentGateway gateway) {
>       this.gateway = gateway;
>     }
>
>     public void pay() {
>       gateway.process();
>     }
>   }
> ```
>
> Now:
> ```java
>   PaymentService service = new PaymentService(new StripeGateway());
> ```
>
> You can switch vendors without touching `PaymentService`.
>
> ## Why DI is Extremely Important in Real Systems
> 1. Testing becomes easy
>    ```java
>    class FakePaymentGateway implements PaymentGateway {
>      boolean called = false;
>      public void process() {
>        called = true;
>      }
>    }
>    ```
>    Now unit testing is simple:
>    ```java
>    FakePaymentGateway fake = new FakePaymentGateway();
>    PaymentService service = new PaymentService(fake);
>
>    service.pay();
>    assertTrue(fake.called);
>    ```
>    Without DI → you'd need PowerMock or ugly hacks.
> 2. Enables Clean Architecture
>
> ## Note:        
> - Use Dependency Injection Even Without Frameworks.        
>   You do not need Spring. It's just a framework that automates wiring.                    
>   DI is design principle - not a framework feature.
>
> ## DI and immutability (Important Design Insight)
> Constructor injection + final fields:
> ```java
> private final Repository repository;
> ```
> This gives:
> - Thread safety
> - Safe publication
> - No reconfiguration bugs
> - Clear object lifecycle
> This is aligned with other Java clean code items such as:
> - Favour immutability
> - Minimizr mutability
> - Use final fields.
>
> ## Common Mistakes TO Avoid:
> - Using field injection (Spring)
>   Problems:
>   - Hard to test
>   - Hidden dependency
>   - Not immutable
>   - Can be null before injection
>  - Never use DI on value objects, simple data holder and stateless utility methods. Use it for Services, Clients, Data sources.
>
> ## DI Encourages Good Design:
> It forces you to:
> - Define interfaces
> - Think in abstractions
> - Separate policy from implementation
> - Reduce coupling
> - Improve modularity
>
> ## The Core Idea in One Line
> >  Classes should not create the resources they depend on — they should receive them.
>
> ## Supplier<> (learn about this later)
