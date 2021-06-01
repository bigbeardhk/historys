
```java

public static boolean isEmpty(@Nullable Collection<?> collection) {
      return collection == null || collection.isEmpty();
}

public static boolean isEmpty(@Nullable Map<?, ?> map) {
    return map == null || map.isEmpty();
}
  

public static boolean isEmpty(@Nullable Object[] array) {
      return array == null || array.length == 0;
}

```
