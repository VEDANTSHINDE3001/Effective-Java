> - # Enforce Non-Instantiability with a Private Constructor
>   ## Core Idea:         
>   If a class is meant to be a utility class (only static methods/fields), you should prevent anyone from creating its instance.
>   You do this by adding a private constructor.
>
>   ## Why is this needed?          
>   If you don’t define a constructor, Java automatically provides a public default constructor which enables instantiability even though that object:
>   - has no state
>   - has no instance methods
>   - serves no purpose
>
>   ## Real-World Examples
>   Java standard library uses this pattern heavily:
>   - java.util.Collections
>   - java.util.Arrays
>   - java.lang.Math
>
>   ## 🧠 When Should You Use This?
>   Use private constructor when your class:
>   1. Is a utility class
>      ```java
>      public class StringUtils {
>
>        private StringUtils() {
>          throw new AssertionError();
>        }
>
>        public static boolean isEmpty(String s) {
>          return s == null || s.isBlank();
>        }
>      }
>      ```
>      Commonly used for:
>      - Date utilities
>      - Validation utilities
>      - Encryption helpers
>      - Mapping helpers
>   2. Contains Only Static Factory Methods
>      ```java
>      public class JsonMapper {
>        private JsonMapper() {
>          throw new AssertionError();
>        }
>
>        public static ObjectMapper newDefault() {
>          return new ObjectMapper();
>        }
>      }
>      ```
>   3. Groups constants
>      ```java
>      public class AppConstants {
>        private AppConstants() {
>          throw new AssertionError();
>        }
>
>        public static final int MAX_RETRY = 3;
>        public static final String HEADER_AUTH = "Authorization";
>      }
>      ```
>
>   One might think why not make the class abstract? But that doesn't disable instantiability because it's subclass can very well instatiate it.
>
>   It is recommended to throw Exception from private constructor to avoid instantiation via reflection thereby making intention clear that the class ought not to instantiate.
>
>   ## Why it's important?
>   - Communicates intent clearly
>   - Avoid architectural smell
>   - Enforces statelessness (future accidental state introduction will be prohibited)
>  
>   ## Things to keep in mind:
>   - Always make utility classes final so as to avoid instantiability via extensibility.
>   - Rely on DI wherever possible instead of piling utlility classes.
