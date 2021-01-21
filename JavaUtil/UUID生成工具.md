```java
public class RandomUUID{

    public static String getUUID()
    {
      String uuid =  UUID.randomUUID().toString().replaceAll("-", "");
      return uuid;
    }

}
```
