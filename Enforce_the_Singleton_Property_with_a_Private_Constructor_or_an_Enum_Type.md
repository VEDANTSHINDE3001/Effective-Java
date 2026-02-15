> - # Enforce the Singleton Property with a Private Constructor or an Enum Type
>   
>   ## What is a Singleton?
>   A Singleton ensures:
>   - Only one instance of a class exists in the JVM
>   - Provides a global access point to that instance          
>
>   ## Real world scenarios where Singleton can be used and why:
>   - Logger -> One logging configuration shared app-wide
>   - Configuration manager -> Same config used everywhere
>   - Cache manager -> Shared in-memory cache
>   - Thread pool manager ->cone shared executor
>   - Feature flag service -> Centralized feature control
>   Spring beans' default scope in java is singleton.       
>
>   ## Approach 1 - Private Constructor + Static Field (Classic Singleton -> Eager initialization)
>   ```java
>     public class ConfigManager {
>        private static final ConfigManager INSTANCE = new ConfigManager();
>
>        private ConfigManager() {
>          // Prevent external instantiation
>        }
>
>        public static ConfigManager getInstance() {
>          return INSTANCE;
>        }
>     }
>   ```
>     Why this works:
>     - Constructor is `private` → cannot call `new`
>     - Instance created once
>     - Thread-safe because of class loading guarantees
>     ## Problems with Classic Singleton
>     Even if constructor is private, singleton can be broken by:
>     1. Reflection
>        ```java
>        Constructor<ConfigManager> constructor = ConfigManager.class.getDeclaredConstructor();
>        constructor.setAccessible(true);
>        ConfigManager anotherInstance = constructor.newInstance();
>        ```
>
>     2. Serialization
>        Deserialization creates new instance unless you implement:
>        ```java
>        private Object readResolve() {
>          return INSTANCE;
>        }
>        ```
>   ## Approach 2 — Enum Singleton (Recommended)
>   ```java
>   public enum AppConfig {
>     INSTANCE;
>
>     public String getProperty(String key) {
>       return "value";
>     }
>   }
>   ```
>   Usage:
>   ```java
>   AppConfig config = AppConfig.INSTANCE;
>   ```
>   ### Why Enum is Better:
>   It automatically handles:
>   - Serialization
>   - Reflection attacks
>   - Thread safety
>   - Single instance guarantee
>   JVM guarantees one instance per enum constant
>   Much simpler code
>        
>   ### What's happening under the hood and how the class is instantiated? (as you can see there's no constructor involved)
>   ```java
>   public final class AppConfig extends Enum<AppConfig> {
>     public static final AppConfig INSTANCE = new AppConfig();
>
>     private AppConfig() {
>       super("INSTANCE", 0);
>     }
>   }
>   ```
>
>   Important things happening:
>   -  Enum becomes a final class
>   -  It extends `java.lang.Enum`
>   -  Constructor is implicitly private
>   -  JVM creates INSTANCE exactly once
>   -  You cannot call new AppConfig() anywhere
>      And that's how singleton is attained.
>         
>   ## Real-World Example (Production Style)
>   Example: Feature Flag Service
>   ```java
>   public enum FeatureFlagService {
>     INSTANCE;
>
>     private final Map<String, Boolean> flags = new HashMap<>();
>
>     public void enable(String feature) {
>       flags.put(feature, true);
>     }
>
>     public boolean isEnabled(String feature) {
>       return flags.getOrDefault(feature, false);
>     }
>   }
>   ```
>   Usage:
>   ```java
>   FeatureFlagService.INSTANCE.enable("new-checkout");
>   if (FeatureFlagService.INSTANCE.isEnabled("new-checkout")) {
>      // execute new logic
>   }
>   ```
>        
>   ## Important points to remember:
>   - Dependency Injection vs Singleton
>     Spring beans are singleton by default.
>     Spring container ensures one instance per application context.
>     Prefer DI over manual singleton.
>     Manual Singleton is mostly useful for:
>     1. Libraries
>     2. Framework Code
>        
>   - When NOT to Use Singleton:
>     1. If object has different configurations
>     2. If object is stateful per request (i.e each request has unique data taht is tored in the object...if single object on jvm data will be overwritten and cause anomalies - >it will create a problem for concurrency. If singleton is used in such cases it will introduce race conditions, data leakage between users, thread safety issues)
>     3. If scaling across multiple JVMs (singleton is per JVM, not per cluster)
