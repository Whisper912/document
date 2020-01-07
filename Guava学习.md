### Basic Utilities

#### Using/avoiding null

Optional
```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```

```java
public void sayHello(String name){
    name = Optional.fromNullable(name).or("游客");
    System.out.println("Hello, "+name);
}
```
#### Preconditions
```java
public void sayHello(String name){
    name = Preconditions.checkNotNull(name); //如果name为null，还是会抛出异常，但是可以让人看出这里name不能为null
    //...
}
```

#### Objects method
1. Objects.equals
因为运行java的null.equals(), 会报空指针异常，
```java
Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true
```
2. Objects.toString
```java
   // Returns "ClassName{x=1}"
   MoreObjects.toStringHelper(this)
       .add("x", 1)
       .toString();

   // Returns "MyObject{x=1}"
   MoreObjects.toStringHelper("MyObject")
       .add("x", 1)
       .toString();
```

3. compareTo
使用Guava提供的ComparisonChain
```java
public int compareTo(Foo that) {
     return ComparisonChain.start()
         .compare(this.aString, that.aString)
         .compare(this.anInt, that.anInt)
         .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
         .result();
}
```
