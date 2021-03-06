## Effective Java, 2nd Edition

by Joshua Bloch

*I, [Michael Parker](http://omgitsmgp.com/), own this book and took these notes to further my own learning. If you enjoy these notes, please [purchase the book](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683)!*

### Chapter 2: Creating and Destroying Objects

#### Item 1: Consider static factories instead of constructors
* An instance-controlled class is one that uses static factories to strictly control what instances exist at any time.

##### Pros
+ Have names unlike constructors
+ Not required to return a new object when invoked.
+ Can return any subtype as return type
  + Factory
+ Fourth advantage is that reduce verbosity of creating parameterized type instances

##### Cons
+ How to differentiate them from already existing static methods
+ Classes without public/private constructors cannot be subclassed.


* By convention, static factory methods for an interface named `Type` are put in a non-instantiable class named `Types`.
* When naming static factory methods, `getInstance` may return the same instance, while `newInstance` should not.

#### Item 2: Consider a builder when faced with many constructor parameters
* The builder pattern simulates optional named parameters in Ada and Python.
* A builder whose parameters have been set makes a fine abstract factory, assuming some generic `Builder<T>` interface.
```java
public class Meal {
  private final Food food;
  private final Fruit fruit;
  private final Drink drink;

  public Meal(Food food) {

  }

  public Meal(Food food, Fruit fruit) {

  }
  //... and so on multiple constructors..
}
```

The JavaBean Pattern - allows inconsistentcy. mandates mutability

Builder Pattern

```java
public class Meal {

 public static class Builder {
   // required parameters
    private final Food food;
    private final Drink drink;

    //optional
    private final Fruit fruit;
    public Builder(Food food, Drink drink) {
      this.food = food;
      this.drink = drink;

    }

    public Builder withFruit(Fruit fruit) {
      this.fruit = fruit;
    }


 }

}
```

#### Item 3: Enforce the singleton property with a private constructor or an enum type
* Adding `implements Serializable` to a singleton class is not enough, you must declare all fields `transient` and provide a `readResolve` method.
* A single-element `enum` type provides the serialization for free and is the best way to implement a singleton.

Attempt 1 - Wrong

```java
public class ClassicSingleton {
   private static ClassicSingleton instance = null;
   protected ClassicSingleton() {
      // Exists only to defeat instantiation.
   }
   //... fails in multi-threaded environment
   public static ClassicSingleton getInstance() {
      if(instance == null) {
         instance = new ClassicSingleton();
      }
      return instance;
   }
}

```

Syncronized Singleton

``` java
public synchronized static Singleton getInstance() {
   if(singleton == null) {
      // simulateRandomActivity();
      singleton = new Singleton();
   }
   logger.info("created singleton: " + singleton);
   return singleton;
}
```

Double Check Singleton
``` java
public static Singleton getInstance() {
   if(singleton == null) {
     synchronized(Singleton.class) {
       if (singleton==null)
        singleton = new Singleton();

     }
   }

  //  logger.info("created singleton: " + singleton);
   return singleton;
}

```

Private constructor and static instance
```java
public class Singleton {
   public final static Singleton INSTANCE = new Singleton();
   private Singleton() {
         // Exists only to defeat instantiation.
      }
}
```

Serializable
```java

public class Singleton implements java.io.Serializable {
   public static Singleton INSTANCE = new Singleton();
   protected Singleton() {
      // Exists only to thwart instantiation.
   }
   /// override this else it will cause deserialzation to get two Singletonss...
      private Object readResolve() {
            return INSTANCE;
      }
}
```
Best Way - Can enum be serialized?
``` java
public enum Singleton {
  INSTANCE;
}

```

#### Item 4: Enforce non-instantiability with a private constructor
```java
public class UtilityClass {
  private UtilityClass () {
    throw new AssertionError();
  }
}
```

* A private constructor not only supresses instantiation, but subclassing.

#### Item 5: Avoid creating unnecessary objects
* Often lazy initialization only complicates the implementation and yields no noticeable performance increase.

Don't do this. It creates an extra object "hello"
```java
String s = new String("hello"); // don't do this ..
```

Use static initializer to create Calendar/Timezone

```java
class Person {

  public boolean isBabyBoomer() {
    Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmt.set(1965, Calendar.JANUARY, 1, 0 ,0 );
    //do date time comparison .
  }
}

```
better to use static initializer
``` java
class Person {
  static {
      Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
      gmt.set(1965, Calendar.JANUARY, 1, 0 ,0 );
  }
}
```
* Prefer primitives to boxed primitives, as unintentional autoboxing can lead to creating many new instances.

e.g. This creates a Long object everytime.

``` java
Long sum = 0L;
for (long i = 0; i < Integer.MAX_VALUE; ++i) {
  sum+=i;
}

```


* Highly optimized garbage collectors can easily outperform object pools that do not contain heavyweight objects.

+ Exceptions maybe in case of DB Connections

#### Item 6: Eliminate obsolete object references
* Whenever a class manages its own memory, like a stack or object pool, the programmer should be alert for memory leaks.

+ Memory Leak in stack example
```java
public Object pop() {
  if (size==0)
    throw new EmptyStackException();
  Object result = elements[--size];
  //the elements array still has a reference to the result element.
  // add this line to indicate to GC to collect the element here ...
elements[size] = null;

  return result;
}
```

* Caches, listeners, and callbacks can all be sources of memory leaks, but weak references can help.
+ Clean up caches.
+ Unregister listeners.

#### Item 7: Avoid finalizers
* Do not think of finalizers as Java's analogue of C++ destructors -- there's no guarantee finalizers will be called at all!
* If an uncaught exception is thrown in a finalizer, it is ignored, and the finalization abruptly terminates.
* There is a severe performance penalty for using finalizers -- object creation and deletion increases from nanoseconds to microseconds.

+ Where to use them if at all - TODO
### Chapter 3: Methods Common to All Objects

#### Item 8: Obey the general contract when overriding `equals`
* Override `equals` when a class has a notion of logical equality that differs from mere object identity, and the superclass has not provided a suitable implementation.
* For classes that represent a value, such as `Integer` or `Date`, the `equals` method should always be overridden.
* There is no way to extend an instantiable class and add a value component (field) while preserving the `equals` contract.
* The class is private or package-private, and you are certain `equals` method will never be invoked. You should still override equals and return an exceptions
``` java
@Override public boolean equals(Object o) {
  throw new AssertionError();
}
```

#### General Contract for equals()
+ Reflexive x.equals(x)
+ Symmetric x.equals(y) ==> y.equals(x)
Breaks symmetry...
``` java

class CaseInsentiveString {
  private final String s;

  public CaseInsentiveString(String s) {
    this.s = s;
  }

  @Override
  public boolean equals(Object o) {
    if (o instanceof CaseInsentiveString) {
      return s.equalsIgnoreCase(((CaseInsentiveString)o).s);
    }
    if (o instanceof String) {
      return s.equalsIgnoreCase((String)o);
    }
  }

}
```

Now
``` java
CaseInsentiveString cis = new CaseInsentiveString(polish);
String s = "polish";
cis.equals(s)// returns true;
s.equals(cis)// returns false;
```
Further if you had
``` java
List<CaseInsentiveString> coll = new ArrayList<>();
coll.contains(cis); // indeterminate answer..
```
##### equals() symmetry is important

+ Transitive x.equals(y), y.equals(z) ==> z.equals(x)
  + Also favor composition to inheritence - Why ?
    + There is literally no way to extend a class and adding a value type without violating the equals contract.
``` java
  // class Point
  class Point {
    private final x;
    private final y;

    public Point(int x, int y){
      this.x = x;
      this.y = y;
    }

    @Override public boolean equals(Object o){
      if (!(o instanceof Point)) return false;
      Point p = (Point)o;
      return (this.x==p.x && this.y==p.y);
    }
  }

 // Color Point

 class ColorPoint extends Point {
   private final Color color;

   public boolean equals(Object o) {
     if (!(o instanceof ColorPoint)) return false;
      return false;

    return super.equals(o) && ((ColorPoint)o).color == color;

   }
   // the equals method above breaks symmetry cause

   // Point p1 = new Point(1, 2);
   // ColorPoint p2 = new ColorPoint(1,2, Color.red);
   // p1.equals(p2) ==> true
   // p2.equals(p1) ==> false

   // One possible solution for Point type do color blind comparison
   public boolean equals(Object o) {
     if (!(o instanceof Point)) return false;

     if (!(o instanceof ColorPoint)) {
       //Point type delegate to points equals method...
       return o.equals(this);
     }
    // o is a ColorPoint; do a full comparison.  
    return super.equals(o) && ((ColorPoint)o).color == color;

   }

   // However this violates the transistivity principle

   // Point p1 = new Point(1, 2);
   // ColorPoint p2 = new ColorPoint(1,2, Color.red);
   // ColorPoint p3 = new ColorPoint(1,2, Color.red);
   // p1.equals(p2) ==> true
   // p1.equals(p3) ==> true
   // p2.equals(p3) ==> false;

   //Its better do it to using composition
   public class ColorPoint {
     private final Point point ;
     private final Color color;


     @Override public boolean equals(Object o) {
       if (!(o instanceof ColorPoint)) {
         return false;
       }
       ColorPoint cp = (ColorPoint) o;
       return cp.point.equals(point) && cp.color.equals(color);
     }
   }

 }
```


+ consistent x.equals(y) ==> should return same values
+ x.equals(null) = > false when x is not  null;



* By using composition with views to internal components, you can add value components to instantiable classes without violating the `equals` contract.
* To compare `float` and `double` values, use the `Float.compare` and `Double.compare` methods to deal with `NaN` and `-0.0` values.
* For best performance, first compare fields that are more likely to differ, or less expensive to compare.

#### Item 9: Always override `hashCode` when you override `equals`
  + Otherwise you cannot use your class properly in the HashMap, HashSet and Hashtable.
  + equal objects should have equal hashcodes
  +
Don't do this
  ```java
   @ Override public int hashCode() {
    return 42;
  }
  ```
Hashfun
  + produce unequal hashcodes for unequal numbers.
  +
* To take the hash code of `float` and `double` values, use `Float.floatToIntBits` and `Double.doubleToLongBits`, respectively.

* Immutability offers the chance to cache hash codes if computing them is expensive.
* Try not to specify the behavior of your hash code method in Javadoc, as that limits your options for improving it later.

#### Item 10: Always override `toString`
* If you specify the format in Javadoc, provide a static factory method accepting a `String` parameter so a client can convert between the two forms.
* Provide programmatic information to all the information provided by `toString`, or clients may try to parse the string to retrieve it.

#### Item 11: Cloneable ? what does that do..


#### Item 12: Consider implementing `Comparable`
* Like the `equals` method, there is no way to extend and instantiable class with a new value component while preserving the `compareTo` contract.
* If `compareTo` is consistent with `equals`, note that sorted collections (e.g. `TreeSet`, `TreeMap`) use the equality test imposed by `compareTo` instead of `equals` and may break the interface (e.g. `Set`, `Map`) contract.
+ e.g. BigDecimal class
+ Big Difference between compareTo vs equals is that compareTo is parameterized whereas equals is not.
  + You don't need to type check it.

* Use methods `Double.compare` and `Float.compare` instead of relational operators, which don't obey the `compareTo` contract for floating point values.

### Chapter 4: Classes and Interfaces

#### Item 13: Minimize the accessibility of classes and members
+ encapsulation: - information hiding
  + Modules communicate each other via APIs - their implementation should be hidden from each other
  + Isolation + Testing

+ Access Levels
  private - member is accessible from the top-level class only
  package-private: default, members are accessible from packages only
  protected: sub-classes + packages
  public : anywhere


* Private and package-private members can "leak" into the exported API if the class implements Serializable.
* Even a protected member is part of the class's exported API and must be supported forever.
* With the exception of `public static final` fields to immutable objects, public classes should have no public fields - not thread safe. Can cause havoc.

+ Non-Zero array is mutable
``` java
//this is mutable.
public static final Things[] VALUES = {...};
```

To provide an array back make it private and return a copy in accessors.

#### Item 14: In public classes, use accessor methods, not public fields
* If a class is package-private or a private nested class, there's nothing wrong with exposing its data.
* A public class with immutable public fields is okay because it can enforce their invariants upon construction.



#### Item 15: Minimize mutability
* To make object immutable
  + make the class final - don't allow inheritence
  + allow fields final & private
  + don't allow direct mutation of the elements.
    + if there are mutation operations create a new object and return it.

* For immutable classes, the Java memory model requires that all fields be `final` to ensure correct behavior when passing an instance between threads without synchronization.
* Defend against "leaking" references by making defensive copies in constructors, accessors, and `readResolve` methods when needed.
* Instances of an immutable class can share internal objects with one another for efficiency.
* If a client requires performing expensive multi-stage operations on your class, expose them as primitive methods, or provide a mutable companion class (like `StringBuilder` for `String`).

#### Item 16: Favor composition over inheritance
* Inheritance violates encapsulation because the subclass depends on the implementation details of the superclass for its proper function.
  + Each subclass depends on superclass for it function properly.
  + If the code in superclass breaks, it breaks subclass even if the subclass itself didn't change at all.
e.g.
``` java
class CountingHashSet<T> extends HashSet<T> {
  private int counter = 0 ;

  @Override public boolean add(T e){
    counter++;
    return super.add(e);
  }


    @Override public boolean addAll(T e){
      counter++;
      return super.add(e);
    }
}
```

addAll in ```CountingHashSet``` would call addAll in ```HashSet``` which in turn will call add on the object i.e. ```CountingHashSet.add```.  
The key point being inheriting classes need to know about the superclass. 

* Inheritance means you inherit the scope and flaws of an API, whereas composition allows you to design a better suited one.
* If an appropriate interface exists, using composition and forwarding allows you to instrument any implementation of the interface, instead of a single implementation through inheritance.

#### Item 17: Design and document for inheritance, or else prohibit it
* The class that allows subclassing must document its self-use of overridable methods.
* The only way to test if a class is suitably designed for inheritance (e.g. provides all the necessary implementation hooks through protected methods) is to actually write subclasses.
* Eliminating a class's self use of methods, typically through introducing private helper methods, can make a class safe to subclass.

#### Item 18: Prefer interfaces to abstract classes
* Use interfaces to allow construction of non-hierarchical type frameworks.
* Simulated multiple inheritance is where a class implementing an interface can forward invocations to an instance of a private inner class that extends the skeletal implementation of that interface and hence does the bulk of the work.
* A variant of a skeletal implementation is the simple implementation, which is a concrete class that defines the simplest possible implementation, such as `AbstractMap.SimpleEntry`.

#### Item 19: Use interfaces only to define types
* Don't use constant interfaces, or interfaces that define no methods but only constants, which classes implement to access the constants without fully qualifying their names.
* If the constants are static members of a class, and you really don't want to qualify their names, use the `static import` facility for brevity.

#### Item 21: Use function objects to represent strategies
* Classes that implement concrete strategies, or simulate function pointers, should be stateless and made into singletons.
* When defining a strategy as an anonymous class that is inline in a method invocation, consider extracting the object as a `private static final` field so a new instance is not created upon every call.

#### Item 22: Favor static member classes over nonstatic
* Nonstatic member classes are ideal for providing adapters, or views of an outer class as an instance of some unrelated class.
* An anonymous class have enclosing instances if and only if they occur in a non-static context.

### Chapter 5: Generics

#### Item 23: Don't use raw types in new code
* `List<E>` is read as "list of E", and `List<String>` is read as "list of string", where `String` is the actual type parameter and `E` is the formal type parameter.
* While an instance of the raw type `List` could be designated to hold only types of a single class but opts out of type-checking, `List<Object>` is typesafe because it explicitly states that it can contain objects of any type.
* The unbound wildcard type, such as in `List<?>`, represents a list of some unknown type and so forbids inserting any element other than `null`, unlike the raw type `List` which is not typesafe.

#### Item 24: Eliminate unchecked warnings
* Always use the `@SuppressWarnings` annotation on the smallest scope possible, to not mask other, critical warnings.

#### Item 25: Prefer lists to arrays
* Arrays are covariant, so `Sub[]` is a subtype of `Super[]`, and reified, so they retain their type-information at runtime and `Super[]` can throw an `ArrayStoreException` when given a different subclass; by contrast, `List<E>` is invariant and erased.
* Consequently, arrays provide runtime safety but not compile-time safety, while a generic type like `List<E>` provides compile-time safety but not runtime safety.

#### Item 27: Favor generic methods
* You can exploit the type inference provided by generic methods and write generic static factory methods that make instances easier to create.
* If you have an immutable, singleton instance of a generic class that could be shared across all types, make it private and of type `Object`, and then let a generic singleton factory method cast it to the caller's desired type.
* The type bound `<T extends Comparable<T>>` may be read as "for every type T that can be compared to itself" and is the definition of a mutually comparable type.

#### Item 28: Use bounded wildcards to improve API flexibility
* If a parameterized type represents a `T` producer, use `<? extends T>`; if a parameterized type represents a `T` consumer, use `<? super T>`.
* Do not use wildcard types as return types, or clients will be forced to use wildcard types in their code.
* Comparables and comparators are always consumers, so you should always use `Comparator<? super T>` in preference to `Comparator<T>`.
* If a type parameter appears only once in a method declaration, replace it with a wildcard, using a private helper method to capture the type if necessary.

#### Item 29: Consider typesafe heterogeneous containers
* A type token is a class literal is passed among methods to communicate both compile-time and runtime type information.
* The cast method of the `Class` type is the dynamic analog of Java's cast operator, throwing a `ClassCastException` if the operation fails.
* The "checked" collection wrappers in `java.util.Collections` use this method with type tokens to enforce, at runtime, that invalid types are not added to a collection through its raw type.

### Chapter 6: Enums and Annotations

#### Item 30: Use `enum` instead of `int` constants
* You can add or reorder constants in an `enum` type without recompiling its clients because the constant values are not compiled into the clients as they are with the int enum pattern.
* Enums are by their nature immutable, and so all their fields should be final.
* If you override the `toString` method of an `enum` type, consider writing a `fromString` method, similar to how the static `valueOf` method would perform if you had not overridden `toString`.
* To share a constant specific method implementations between `enum` values, move each implementation into a private nested `enum`, and pass an instance of this strategy enum to the constructor of the top-level `enum`.

#### Item 32: Use `EnumSet` instead of bit fields
* An `EnumSet` is represented by one or more `long` values, and so many of its operations are effiiciently performed with bitwise arithmetic.

#### Item 33: Use `EnumMap` instead of ordinal indexing
* When you access an array that is indexed by the ordinal value of an `enum`, it is your responsibility to use the correct `int` value, as no type safety is afforded.
* An `EnumMap` contains an array internally, offering the speed of an ordinal-indexed array with the type safety and richness of the `Map` interface.
* Instead of using an array of arrays to define a mapping from two enum values, use `EnumMap<..., EnumMap<...>>`, which is internally represented as an array of arrays.

#### Item 34: Emulate extensible enums with interfaces
* If two enums implement the same interface, both their classes adhere to the type `<T extends Enum<T> & InterfaceName>`.
* Since implementations cannot be inherited from one `enum` type to another, the functionality must be encapsulated in a helper class or a static helper method.

#### Item 35: Prefer annotations to naming patterns
* Annotations like `Retention` and `Target` for annotation type declarations are called meta-annotations.
* A marker annotation is one with no parameter and simply serves to mark some class, method, or field for interpretation by some other method or program.

#### Item 36: Consistently use the `Override` annotation
* There is no need to use the `@Override` annotation when a concrete class overrides an abstract method, i.e. implements it, because a differing signature will be caught by the compiler anyway.
* In an abstract class or interface, annotate all methods you believe to override superclass or superinterface methods, whether concrete or abstract, to ensure that you don't accidentally introduce any new methods.

#### Item 37: Use marker interfaces to define types
* Use a marker interface instead of an annotation if you want to write one or more methods that accept only objects that have this marking, or implement the interface.
* Use a marker interface instead of an annotation if you want to limit the use of the marker to elements of a particular interface, by having the marker interface extend that interface.
* A marker annotation, however, allows marking elements other than classes and interfaces, and allows adding more information while retaining backwards compatibility through type elements with defaults.

### Chapter 7: Methods

#### Item 38: Check parameters for validity
* Nonpublic methods should check their parameters using assertions, which are enabled with the `-enableassertions` command line flag, instead of explicitly throwing exceptions.
* Skip checking a method's parameters before performing the computation if the validity check would be expensive or impractical and the validity check is performed implicitly during the computation.

#### Item 39: Make defensive copies when needed
* When making defensive copies of constructor parameters, check the validity of the copies instead of the originals to guard against malicious changes to the parameters by another thread.
* Do not use the `clone` method to make a defensive copy of a constructor parameter whose type is subclassable by untrusted parties.
* The defensive copy can be replaced by documented transfer of ownership if copying the object would be costly and the class trusts its clients not to modify the components inappropriately.

#### Item 40: Design method signatures carefully
* If you go over four parameters, try to break up the method into orthogonal methods with fewer parameters, introduce helper classes that bundle related parameters together, or use a builder pattern.
* Prefer two-element `enum` types to boolean parameters.

#### Item 41: Use overloading judiciously
* Beware that selection among overloaded methods is static, while selection among overridden methods is dynamic.
* Avoid cases where the same set of parameters can be passed to different overloadings of a method by the addition of casts.
* If a method with a primitive parameter overloads a method with a generic parameter, the two can become conflated from autoboxing.

#### Item 42: Use varargs judiciously
* Don't blindly retrofit every method that has a final array parameter to use varargs; use varargs only when a call operates on a variable-length sequence of values.

#### Item 43: Return empty arrays or collections, not nulls
* Returning `null` from an array or collection-valued method complicates the caller logic, and usually the callee's.
* To eliminate the overhead from creating an empty collection or array, return the immutable empty collections in `java.util.Collections`, or create a static empty array which is necessarily immutable.

#### Item 44: Write doc comments for all exposed API elements
* To include a multiline code example in a doc comment, use a Javadoc `{@code}` tag wrapped inside an HTML `<pre>` tag.
* The `{@literal}` tag is like the `{@code}` tag in that it eliminates the need to escape HTML metacharacters, but doesn't render the contents in monospaced font.
* No two members or constructors should have the same summary description, which is the first (sometimes incomplete) sentence of a doc comment.

### Chapter 8: General Programming

#### Item 45: Minimize the scope of local variables
* Declare a variable at the latest point possible, typically right before it is first used, and strive to provide an initializer.
* If the bound for a loop variable is expensive to compute on every iteration, store it in a loop variable that will fall out of scope with the counter variable.

#### Item 46: Prefer for-each loops to traditional for loops
* If you are writing a type that represents a group of elements, have it implement the `Iterable` interface even if it does not implement `Collection`.
* The enhanced `for` loop cannot be used if you need to remove elements through an iterator, reassign elements through a list iterator, or iterate over collections in parallel.

#### Item 47: Know and use the libraries
* Every programmer should be familiar with the contents of `java.lang` and `java.util` (particularly the collections framework), and to a lesser extent `java.io` and `java.util.concurrent`.

#### Item 48: Avoid `float` and `double` if exact answers are required
* The `BigDecimal` can contain decimal values of arbitrary size and provides eight rounding modes, which is ideal for business calculations with legally mandated rounding behavior.
* If you choose to keep track of the decimal point yourself, `int` provides up to nine decimal digits, while `long` provides up to eighteen.

#### Item 49: Prefer primitive types to boxed primitives
* Never use `==` on two boxed primitives, because this always performs identity comparison, while the `<` and `>` operators compare the underlying primitive values.
* When you mix primitives and boxed primitives in a single operation, the boxed primitive is auto-unboxed, which can result in `NullPointerExceptions`.
* Beware of boxed primitives being implicitly unboxed and then re-boxed, which can cause performance problems.

#### Item 50: Avoid strings where other types are appropriate
* Instead of using a key to represent an aggregate type, write a private static member class -- even if it only has two fields.

#### Item 51: Beware the performance of string concatenation
* A consequence of strings being immutable is that the time to concatenate *n* strings is quadratic in *n*.
* An alternative to using a `StringBuilder` is to try processing the strings one at a time to avoid all concatenation.

#### Item 52: Refer to objects by their interfaces
* If you depend on any properties of an implementation not specified by its interface, such as its synchronization policy, use the class as a type and document the requirements.

#### Item 53: Prefer interfaces to reflection
* If a class is unavailable at compile time but there exists an appropriate interface, create instances reflectively and access them normally through their interface.
* If writing a package that runs against multiple versions of some other package, you can compile it against the minimal environment required to support it, and access any newer classes or methods reflectively.

#### Item 55: Optimize judiciously
* Strive for encapsulation so that a part of the system can be rewritten for performance without changing its other parts.
* You need to measure attempted optimization carefully on the Java platform, because the language does not have a strong performance model, or well-defined relative costs.

#### Item 56: Adhere to generally accepted naming conventions
* Components of a package name should be short, generally eight or fewer characters, where abbreviations are encouraged and acronyms are acceptable.
* Names of `static final` fields whose values are immutable should be in uppercase with words separated by underscores.
* Methods that convert a type have the format `toType`, while methods that return a view have the format `asType`, and methods that return a primitive representation have the format `typeValue`.

### Chapter 9: Exceptions

#### Item 57: Use exceptions for exceptional conditions
* Do not force clients to use exceptions for ordinary control flow, and instead offer methods to test whether an exception could be thrown, such as `hasNext` for class `Iterator`.
* Use a distinguished return value, like `null`, if the object is accessed by multiple threads or if the state-testing method duplicates the work of the state-dependent method.

#### Item 58: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
* Use runtime exceptions to indicate programming errors, typically precondition violations.
* Don't implement any new `Error` subclasses, and don't define a throwable that does not subclass `Exception` or `RuntimeException`.

#### Item 59: Avoid unnecessary use of checked exceptions
* When designing an API, only throw a checked exception if it can be prevented by a proper use of the API, and the programmer can take some useful action once thrown.

#### Item 60: Favor the use of standard exceptions
* Throw a `NullPointerException` instead of an `IllegalArgumentException` if a caller passes in a `null` parameter where prohibited.
* Throw an `IndexOutOfBoundsException` instead of an `IllegalArgumentException` if the caller passes in an invalid index for a sequence.

#### Item 61: Throw exceptions appropriate to the abstraction
* Use exception translation, where low-level exceptions are caught and exceptions appropriate to the higher-level abstraction are thrown.
* If an exception does not have a chaining-aware constructor, use the `initCause` method of `Throwable`.
* Try to avoid low-level exceptions by checking the higher-level method's parameters upfront.

#### Item 62: Document all exceptions thrown by each method
* Document the unchecked exceptions a method can throw, thereby documenting its preconditions.
* Do not use the `throws` keyword to include unchecked exceptions in a method declaration.

#### Item 63: Include failure-capture information in detail messages
* The `toString` method, or "detail message," should contain the values of all the parameters and fields contributing to the exception.
* To capture this information easily, make them parameters to the constructor, and internally generate the detail message from them.

#### Item 64: Strive for failure atomicity
* A failed method invocation should leave the object in the state it was prior to the invocation.
* Typically you can easily achieve failure atomicity by checking the parameters' validity before the operation.

### Chapter 10: Concurrency

#### Item 66: Synchronize access to shared mutable data
* Synchronization is not just for mutual exclusion, but ensuring that a value written by one thread is seen by another.
* Both read and write operations on mutable data must be `synchronized` or visibility is not guaranteed.
* Beware that the increment and decrement operators are not atomic on volatile integers.

#### Item 67: Avoid excessive synchronization
* Inside a `synchronized` region, do not invoke a method that is provided by the client as a function object, or can be overriden.
* Reentrant locks simplify the construction of multi-threaded object oriented programs, but can allow foreign methods to access an object in an inconsistent state.
* Only make a mutable class thread-safe if intended for concurrent use and you can achieve better concurrency with internal locking; otherwise, punt locking to the client.
