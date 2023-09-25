# clone 재정의는 주의해서 진행하라

> 작성자: 워니

## 목차
### Cloneable과 clone
### clone의 문제
### 주의사항
### 추천: 복사 생성자와 복사 팩터리

---

### Cloneable과 clone
#### 1. Cloneable 인터페이스란?
복제해도 되는 클래스임을 명시하는 믹스인 인터페이스

객체를 복사하고싶다면 Cloneable 인터페이스를 구현해 clone 메서드를 재정의하는 방법이 일반적이다. <br>
하지만, clone 메서드가 정의된곳은 Cloneable이 아닌 Object에 접근제어자도 protected이다. <br>
그렇기에 Cloneable 인터페이스를 구현하는것만으로는 clone 메서드 호출이 안된다. <br>

``` java
/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * <p>
 * Invoking Object's clone method on an instance that does not implement the
 * <code>Cloneable</code> interface results in the exception
 * <code>CloneNotSupportedException</code> being thrown.
 * <p>
 * By convention, classes that implement this interface should override
 * {@code Object.clone} (which is protected) with a public method.
 * See {@link java.lang.Object#clone()} for details on overriding this
 * method.
 * <p>
 * Note that this interface does <i>not</i> contain the {@code clone} method.
 * Therefore, it is not possible to clone an object merely by virtue of the
 * fact that it implements this interface.  Even if the clone method is invoked
 * reflectively, there is no guarantee that it will succeed.
 *
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   1.0
 */
public interface Cloneable {
}
```
- Clonable 역할
  - 메서드가 없다.
  - Object의 clone 메서드의 동작방식을 결정한다.
  - Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, <br>
    그렇지 않은 클래스의 인스턴스에서 호출하면 ClassNotSupportedException을 던진다.

``` java
public class exam1 {
    public static void main(String[] args) throws CloneNotSupportedException {
        Fruit apple = new Fruit("사과", 1000);
        Fruit copiedApple = (Fruit) apple.clone();

        System.out.println("copiedApple = " + copiedApple);
    }

    static class Fruit {
        private String name;
        private long price;

        public Fruit(String name, long price) {
            this.name = name;
            this.price = price;
        }

        @Override
        public String toString() {
            return "Fruit {name='" + name + '\'' + ", price=" + price + '}';
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }
}
```
![image](https://github.com/kiwijomn/test-repo/assets/116738827/17743fa7-b36f-4b68-821e-53f12a73cef2)

❎ clone 메서드를 재정의하고 사용했지만 Cloneable을 구현하지 않았기에 예외가 발생한다.


#### 2. Object 클래스의 clone
``` java
/**
 * Creates and returns a copy of this object.  The precise meaning
 * of "copy" may depend on the class of the object. The general
 * intent is that, for any object {@code x}, the expression:
 * <blockquote>
 * <pre>
 * x.clone() != x</pre></blockquote>
 * will be true, and that the expression:
 * <blockquote>
 * <pre>
 * x.clone().getClass() == x.getClass()</pre></blockquote>
 * will be {@code true}, but these are not absolute requirements.
 * While it is typically the case that:
 * <blockquote>
 * <pre>
 * x.clone().equals(x)</pre></blockquote>
 * will be {@code true}, this is not an absolute requirement.
 * <p>
 * By convention, the returned object should be obtained by calling
 * {@code super.clone}.  If a class and all of its superclasses (except
 * {@code Object}) obey this convention, it will be the case that
 * {@code x.clone().getClass() == x.getClass()}.
 * <p>
 * By convention, the object returned by this method should be independent
 * of this object (which is being cloned).  To achieve this independence,
 * it may be necessary to modify one or more fields of the object returned
 * by {@code super.clone} before returning it.  Typically, this means
 * copying any mutable objects that comprise the internal "deep structure"
 * of the object being cloned and replacing the references to these
 * objects with references to the copies.  If a class contains only
 * primitive fields or references to immutable objects, then it is usually
 * the case that no fields in the object returned by {@code super.clone}
 * need to be modified.
 * <p>
 * The method {@code clone} for class {@code Object} performs a
 * specific cloning operation. First, if the class of this object does
 * not implement the interface {@code Cloneable}, then a
 * {@code CloneNotSupportedException} is thrown. Note that all arrays
 * are considered to implement the interface {@code Cloneable} and that
 * the return type of the {@code clone} method of an array type {@code T[]}
 * is {@code T[]} where T is any reference or primitive type.
 * Otherwise, this method creates a new instance of the class of this
 * object and initializes all its fields with exactly the contents of
 * the corresponding fields of this object, as if by assignment; the
 * contents of the fields are not themselves cloned. Thus, this method
 * performs a "shallow copy" of this object, not a "deep copy" operation.
 * <p>
 * The class {@code Object} does not itself implement the interface
 * {@code Cloneable}, so calling the {@code clone} method on an object
 * whose class is {@code Object} will result in throwing an
 * exception at run time.
 *
 * @return     a clone of this instance.
 * @throws  CloneNotSupportedException  if the object's class does not
 *               support the {@code Cloneable} interface. Subclasses
 *               that override the {@code clone} method can also
 *               throw this exception to indicate that an instance cannot
 *               be cloned.
 * @see java.lang.Cloneable
 */
@HotSpotIntrinsicCandidate
protected native Object clone() throws CloneNotSupportedException;
```

> **x.clone() != x 는 참이여야 한다.** <br>
> **x.clone().getClass() == x.getClass() 는 참이여야 한다.** <br>
> **x.clone().equals(x) 는 참이여야 한다.** <br>
>

➡️ clone() 메서드의 반환값이 복사될 객체를 가르키므로 생성자 연쇄(constructor chaining)와 유사한 패턴이다. 즉, clone 내부 로직이 생성자를 호출해 얻은 인스턴스를 반환해도 문제가 없다는 것. <br>
하지만, 이렇게 되면 해당 클래스의 하위클래스에서 super.clone()으로 호출할 때 상위 객체에서 잘못된 클래스가 생성될 수 있기에 위험하다. 

---

### clone의 문제

#### 1. 클래스의 모든 필드가 기본타입이거나 불변 객체를 참조할 때
- super.clone()만으로도 문제없이 동작한다. 

``` java
public class Fruit implements Cloneable{
    private String name;
    private long price;

    @Override
    public Fruit clone() throws CloneNotSupportedException {
        try{
            return (Fruit)super.clone();
        }catch(CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```

#### 2. 가변 객체를 참조하는 클래스의 clone을 재정의할 때
▫️ **문제** <br>
- super.clone()만 사용한다면 문제가 발생할 수 있다.

``` java
public class Fruits implements Cloneable{
    private static int BUFFER_SIZE = 16;
    private Fruit[] store = new Fruit[BUFFER_SIZE];
    private int size;

    public Fruits() {
    }

    public void add(Fruit fruit) {
        ensureCapacity();
        store[size++] = fruit;
    }

    private void ensureCapacity() {
        if (store.length == size) {
            store = Arrays.copyOf(store, 2 * size + 1);
        }
    }

    ...

    public Fruit[] getAll() {
        return store.clone();
    }

    @Override
    protected Object clone(){
        try{
            return (Fruits) super.clone();
        }catch(CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```

```java
public class App {
    public static void main(String[] args) throws CloneNotSupportedException {
        Fruits fruits = new Fruits();
        fruits.add(new Fruit("사과", 1000));
        fruits.add(new Fruit("바나나", 2000));
        fruits.add(new Fruit("오렌지", 3000));

        Fruits clonedFruits = fruits.clone();
        System.out.println("fruits: "+ fruits);
        System.out.println("clonedFruits: "+clonedFruits);

        clonedFruits.add(new Fruit("딸기", 4000));
        System.out.println("fruits: "+ fruits);
        System.out.println("clonedFruits: "+clonedFruits);
    }
}
```

![image](https://github.com/kiwijomn/test-repo/assets/116738827/b4ebd2d5-be33-416e-a747-272117dd91b6)

❎ clonedFruits에만 딸기 객체를 추가했지만 원본에도 추가되어 있다. <br>
*clone 메서드는 생성자와 같은 효과를 내는데 원본과 동일한 내용을 원본에 영향을 주지 않으며 복제된 객체의 불변식을 보장해야 한다.*

▫️ **해결** <br>
➡️ 객체의 참조 변수나 배열의 clone을 **재귀적으로 호출**해서 해결할 수 있다.

``` java
@Override
public Fruits clone(){
    try {
        Fruits result = (Fruits) super.clone();
				result.store = store.clone();
        return result;
    } catch(CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

#### 3. 재귀적 호출의 한계
▫️ **문제** <br>
내부적으로 배열을 재귀적 호출로 clone을 호출해줘서 해결을 할 수 있었지만, 이 배열이 객체 배열이고 연결리스트라면 원본과 복제본은 같은 연결 리스트를 참조하여 문제가 발생할 수 있다.

``` java
@Data
public class CustomTable implements Cloneable{
    private final int BUFFER_SIZE = 16;
    private int size;
    private Entry[] buckets = new Entry[BUFFER_SIZE];

    public CustomTable() {}

    public void put(Object key, Object obj) {
        ensureCapacity();
        Entry entry = new Entry(key, obj, null);
        linkEntry(entry);
        buckets[size++] = entry;
    }

    private void linkEntry(Entry entry) {
        if (size > 0) {
            buckets[size-1].setNext(entry);
        }
    }

    private void ensureCapacity() {
        if (buckets.length == size) {
            buckets = Arrays.copyOf(buckets, 2 * size + 1);
        }
    }

    @Data
    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    @Override
    public CustomTable clone() {
          try {
              CustomTable result = (CustomTable) super.clone();
              result.buckets = buckets.clone();
              return result;
          } catch(CloneNotSupportedException e) {
              throw new AssertionError();
          }
    }

    ...
}
```

``` java
public static void main(String[] args) {
    CustomTable ct = new CustomTable();

    ct.put("first", new Fruit("사과", 100));
    ct.put("second", new Fruit("바나나", 200));
    ct.put("third", new Fruit("포도", 300));

    CustomTable clonedCT = ct.clone();
    clonedCT.put("four", new Fruit("복숭아", 400));

    System.out.println("clonedCT = " + clonedCT);
    System.out.println("ct = " + ct);
}
```

![image](https://github.com/kiwijomn/test-repo/assets/116738827/b55353e8-86d4-4bfe-94ea-a213fa38d0ba)

❎ 복제된 CustomTable 객체에만 복숭 객체를 추가해줬는데, 결과는 예상과 다르다. 

▫️ **해결1** <br>
➡️ **깊은 복사**(deep copy)를 이용하여 해결한다.

(1) 이너 클래스 Entry에 deepCopy 메서드 추가
- 엔트리가 가르키는 연결리스트 노드를 재귀적으로 복사한다.

``` java
Entry deepCopy() {
    return new Entry(key, value, next == null
										                    ? null
										                    : next.deepCopy());
}
```

(2) clone 메서드 수정
``` java
@Override
public CustomTable clone() {
    try {
        CustomTable result = (CustomTable) super.clone();
        result.buckets = new Entry[buckets.length];
        Entry current = buckets[0].deepCopy();
        int count = 0;
        while (!current.hasNext()) {
            result.buckets[count++] = current;
            current = current.next;
        }

        result.buckets[count] = current;
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```
- 적절한 크기로 buckets 객체 배열을 생성 후 버킷을 순회하며 깊은 복사된 새로운 엔트리를 삽입한다. 
- 하지만, 이런 재귀호출 방식은 리스트의 원소 숫자만큼 스택 프레임을 낭비하기에 스택오버플로우 예외가 발생할 수 있다.

▫️ **해결2** <br>
➡️ 재귀 호출 대신 **반복자**를 사용한다.

``` java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```

---

### 주의사항

- 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 하며 이는 clone 메서드도 동일하다. <br>
⇒ 만약, clone이 하위 클래스에서 재정의한 메서드를 호출하면 하위 클래스는 복제 과정에서 자신의 상태를 바꿀 기회가 사라지며 복제본과 원본의 상태가 달라질 수 있다.
- 재정의한 clone 메서드는 throws 절을 없애야 한다.
- 상속용 클래스는 Cloneable을 구현해서는 안된다. <br>
또는 다음과 같이 구현하여 clone을 재정의하지 못하게 할 수 있다.
``` java
@Override
protected final Object clone() throws CloneNotSupportedException {
		throw new CloneNotSupportedException();
}
```
- Cloneable을 구현하는 모든 클래스는 clone을 재정의 해야 한다.

---
### 추천: 복사 생성자와 복사 팩터리

- 이미 Cloneable을 구현한 클래스라면 어쩔 수 없지만 그게 아니라면 복사 생성자와 복사 팩토리라는 객체 복사 방식을 고려하는 것이 좋다.

#### 복사 생성자
``` java
public CustomTable(CustomTable ct){ ... };
```

#### 복사 팩터리
``` java
public static CustomTable newInstance(CustomTable ct){ ... };
```

▫️ **복사 생성자 / 복사 팩터리의 장점**
- 언어 모순적이고 생성자를 쓰지않는 객체 생성 메커니즘을 사용하지 않는다.
- 정상적인 final 필드 용법과도 충돌하지 않는다.
- 불필요한 예외가 발생하지 않는다.
- 형변환도 필요하지 않다. 
- 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다.
  (ex. HashSet을 TreeSet 타입으로 복제할 수 있다.)

---

### 정리
1. 인터페이스(클래스)를 새로 만들때는 Cloneable을 확장하지 말아라. 
2. 복사가 필요하면 복제 생성자 및 팩터리를 사용하는게 좋다. 
3. 오직 배열만이 clone 메서드 방식의 사용이 권장된다. 
