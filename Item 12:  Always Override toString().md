> ## Item 12 – Always Override `toString()`
>
> ## Why This Item Matters
> The default `Object.toString()` looks like this:
> ```java
> ClassName@1a2b3c
> ```
> That’s almost useless in real systems.
>
> In backend engineering (Spring Boot, logging, debugging production issues), `toString()` is what gets printed in logs, exceptions, monitoring tools, and debuggers.
>
> If you don’t override it:
> - Logs become meaningless
> - Debugging becomes painful
> - Observability suffers
>
> ## What a Good `toString()` Should Do
> It should:
> 1. Include all important fields
> 2. Be concise but informative
> 3. Be consistent with equals() -> (if `equals()` considers two of the three fields of the class then `toString()` should reflect that. If you only have one field in `toString()` but an object can be told apart on the basis of two fields then logs would be insufficient)
> 4. Avoid leaking sensitive data
>
> ### Good Example:
> ```java
> public class User {
>   private String name;
>   private String email;
>
>   @Override
>   public String toString() {
>     return "User{name='" + name + "', email='" + email + "'}";
>   }
> }
> ```
> Now logs show:
> ```
> User{name='Vedant', email='vedant@gmail.com'}
> ```
>
> ## Note:
> Frameworks like `SLF4J` calls `toString()` internally.      
> If not overridden → useless logs in production.           
