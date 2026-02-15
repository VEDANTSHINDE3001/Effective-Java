> - # Consider a Builder When Faced with Many Constructor Parameters    
>   When a class has many parameters, especially when:
>   - Some are required
>   - Others are optional
>   - Many are of the samr type
>   - There are validation constraints.     
>          
>     In such cases constructors become hard to read, error-prone and difficult to  
>    maintain.
>               
>   #### Telescoping Constructor Pattern (Problemati Approach)
>   ```java
>   public class NutritionFacts {
>       private final int servingSize;  // required
>       private final int servings;     // required
>       private final int calories;     // optional
>       private final int fat;          // optional
>       private final int sodium;       // optional
>       private final int carbohydrate; // optional
>
>       public NutritionFacts(int servingSize, int servings) {
>           this(servingSize, servings, 0);
>       }
>
>       public NutritionFacts(int servingSize, int servings, int calories) {
>           this(servingSize, servings, calories, 0);
>       }
>
>       public NutritionFacts(int servingSize, int servings, int calories, int fat) {
>           this(servingSize, servings, calories, fat, 0);
>       }  
>
>       public NutritionFacts(int servingSize, int servings,
>                          int calories, int fat,
>                          int sodium) {
>           this(servingSize, servings, calories, fat, sodium, 0);
>       }
>
>       public NutritionFacts(int servingSize, int servings,
>                            int calories, int fat,
>                          int sodium, int carbohydrate) {
>           this.servingSize = servingSize;
>           this.servings = servings;
>           this.calories = calories;
>           this.fat = fat;
>           this.sodium = sodium;
>           this.carbohydrate = carbohydrate;
>       }
>   }
>   ```                                
>   Hard to read:                  
>   ```java
>   new NutritionFacts(240, 8, 100, 35, 27, 10);
>    ```
>   What is 35? Fat? Sodium?
>   - Parameter order mistakes are easy
>   - Doesn’t scale well
>   - Unusable when parameters are same type (e.g., multiple int)
>   #### JavaBeans Pattern (Another Flawed Approach)     
>   ```java
>   public class NutritionFacts {
>       private int servingSize;
>       private int servings;
>       private int calories;
>       private int fat;
>       private int sodium;
>       private int carbohydrate;
>
>       public NutritionFacts() {}
>
>       public void setServingSize(int val) { servingSize = val; }
>       public void setServings(int val) { servings = val; }
>       public void setCalories(int val) { calories = val; }
>       public void setFat(int val) { fat = val; }
>       public void setSodium(int val) { sodium = val; }
>       public void setCarbohydrate(int val) { carbohydrate = val; }
>   }
>    ```
>   Usage:
>   ```java
>   NutritionFacts cocaCola = new NutritionFacts();
>   cocaCola.setServingSize(240);
>   cocaCola.setServings(8);
>   cocaCola.setCalories(100);
>   ```
>   Problems with this approach:
>   - Object is in inconsistent state during construction
>   - Not immutable
>   - Requires Setters
>   - Hard to make thread-safe
>   - Cannot enforce required parameters
>   
>   #### The Builder Pattern (Recommended Approach)
>   ```java
>   public class NutritionFacts {
>
>       private final int servingSize;
>       private final int servings;
>       private final int calories;
>       private final int fat;
>       private final int sodium;
>       private final int carbohydrate;
>
>       public static class Builder {
>           // Required parameters
>           private final int servingSize;
>           private final int servings;
>
>           // Optional parameters - initialized to default values
>           private int calories = 0;
>           private int fat = 0;
>           private int sodium = 0;
>           private int carbohydrate = 0;
>
>           public Builder(int servingSize, int servings) {
>               this.servingSize = servingSize;
>               this.servings = servings;
>           }
>
>           public Builder calories(int val) {
>               calories = val;
>               return this;
>           }
>
>           public Builder fat(int val) {
>               fat = val;
>               return this;
>           }
>
>           public Builder sodium(int val) {
>               odium = val;
>               return this;
>           }
>
>           public Builder carbohydrate(int val) {
>               carbohydrate = val;
>               return this;
>           }
>
>           public NutritionFacts build() {
>               return new NutritionFacts(this);
>           }
>       }
>
>       private NutritionFacts(Builder builder) {
>           servingSize = builder.servingSize;
>           servings = builder.servings;
>           calories = builder.calories;
>           fat = builder.fat;
>           sodium = builder.sodium;
>           carbohydrate = builder.carbohydrate;
>       } 
>   }
>   ```
>   #### Usage:
>   ```java
>   NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
>        .calories(100)
>        .sodium(35)
>        .carbohydrate(27)
>        .build();
>   ```
>   #### Why Builder is Superior
>   - Readable and Self-Documenting (no confusion about which argument is associated to which field.)
>   - Enforces Required Parameters (have to initialize while creating object)
>   - Immutability   
>       1. All fields are `final`
>       2. No Setters
>       3. Thread-safe once constructed
>   - Scales very well (easy to add a new field)    
>   - Easy to enforce validation hence object is never created in invalid state. (add validation in build() method)
>   - Enables chaining because Builder methods always return `this`
>   - Workes well with same-type parameters
>       ```java
>       new User("John", "Doe", "Mumbai", "India");  // Bad
>                    
>       User user = new User.Builder("John", "Doe")  //Better
>        .city("Mumbai")
>        .country("India")
>        .build();
>       ```    
>           
>   #### Real World Usage
>   1. HTTP Clients
>       ```java
>       HttpRequest request = HttpRequest.newBuilder()
>        .uri(URI.create("https://example.com"))
>        .header("Content-Type", "application/json")
>        .GET()
>        .build();
>       ```
>   2. String Builder (Oracle Java API)
>       ```java
>       String result = new StringBuilder()
>           .append("Hello ")
>           .append("World")
>           .toString();
>       ```    
>   3. Lombok `@Builder`
>       ```java
>       @Builder
>       public class User {
>           private String name;
>           private int age;
>       }
>       ```
>   #### When NOT to Use Builder
>      It's an overkill when:   
>      - Class has 2-3 parameters
>      - No optional parameters
>      - Simple DTO    
