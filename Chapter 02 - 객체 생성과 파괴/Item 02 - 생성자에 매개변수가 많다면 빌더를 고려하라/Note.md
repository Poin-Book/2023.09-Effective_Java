# 생성자에 매개변수가 많다면 빌더를 고려하라

> 작성자: 캐슬

## 목차
### 1. 점층적 생성자 패턴
### 2. 자바 빈즈 패턴
### 3. 빌더 패턴


<br>
시작하며

정적 팩터리와 생성자에는 똑같은 제약이 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다. 이 문제에 대해 세가지 대안이 있다.

**EX_)**

식품 포장의 영양정보를 표현하는 클래스를 생각해보면 영양 정보에는 많은 선택 항목들로 이루어지고 많은 제품에서 해당 선택항목 중 대다수의 값이 0이다. 이러한 경우 해당 클래스의 인스턴스를 만드려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출해 사용하면된다.

보통 이런 생성자는 사용자가 설정하기 원치 않는 매개변수까지 값을 지정해서 포함시켜야 하는 경우가 있다.

<aside>
💡 **이러한 경우 *점층적 생성자 패턴* 을** **사용할 수 있지만 매개변수의 개수가 많아지면 사용하기 어렵다.**

</aside>

---

### 1. 점층적 생성자 패턴

- 생성자를 필수 매개변수 1개만 받는 생성자, 필수 매개변수 1개와 선택 매개변수 1개를 받는 생성자, 선택 매개변수 2개를 받는 생성자를 받는 생성자 등에 형태로 **필요한 매개변수 개수만큼 늘리는 방식**이다.

> ***점층적 생성자 패턴 특징***
>
- 사용자가 원치 않는 매개변수까지 **어쩔 수 없이 값을 지정**해야한다.
- 매개변수 조합에 따라 **생성자 수가 무수히 많아질 수** 있다.
- 매개변수 수가 늘어나면 **코드의 가독성이 매우 떨어**진다.

**점층적 생성자 패턴** - **코드**

```java
//식품 영양정보 표현 클래스
public class NutritionFacts {	
	private final int servingSize;  // 필수
	private final int servings;     // 필수
	private final int calories;     // 선택
	private final int fat;          // 선택
	private final int sodium;       // 선택
	private final int carbohydrate; // 선택
	
	public NutritionFacts(int servingSize, int servings) {
		this(servingSize, servings, 0);
	}
	
	public NutritionFacts(int servingSize, int servings, int calories) {
		this(servingSize, servings, calories, 0);
	}
	
	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this(servingSize, servings, calories, fat, 0);
	}
	
	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
		this(servingSize, servings, calories, fat, sodium, 0);
	}
	
	public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = sodium;
		this.carbohydrate = carbohydrate;
	}
}

//호출 방식
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

**매개변수가 많을때 점층적 생성자 패턴을 사용하기 어려운 이유**

1. *코드를 읽을 때 각 값의 의미가 무엇인지 헷갈린다.*
2. *매개변수가 몇개인지도 주의해서 세어보아야 한다.*
3. *타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수 있다.*
4. *클라이언트가 실수로 매개변수의 순서를 바꿔 건네줘도 컴파일러는 알아채지 못하고 런타임에 엉뚱한 동작을 할 수있다.*

---

### 2. 자바빈즈 패턴(JavaBeans pattern)?

- 선택 매개변수가 많을 때 활용할 수 있는 패턴이다.
- 매개 변수가 없는 생성자로 객체들을 만든 후 , 세터(setter) 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식이다.

**자바빈즈 패턴 - 코드**

```java
//영양정보 표현 클래스
public class NutritionFacts {
 
	private int servingSize = -1; // 필수
	private int servings = -1; // 필수
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;
 
	public NutritionFacts() { }
 
	public void setServingSize(int servingSize) {
		this.servingSize = servingSize;
	}
	public void setServings(int servings) {
		this.servings = servings;
	}
	public void setCalories(int calories) {
		this.calories = calories;
	}
	public void setFat(int fat) {
		this.fat = fat;
	}
	public void setSodium(int sodium) {
		this.sodium = sodium;
	}
	public void setCarbohydrate(int carbohydrate) {
		this.carbohydrate = carbohydrate;
	}
}

//호출 방법
NutritionFacts cocalCola = new NutritionFacts();
cocalCola.setServingSize(240);
cocalCola.setServings(8);
cocalCola.setCalories(100);
cocalCola.setSodium(35);
cocalCola.setCarbohydrate(27);
```

> **자바빈즈 패턴 특징**
>
- 코드가 길어졌지만 인스턴스를 만들기 쉽다.
- 읽기 쉬운 코드( 가독성이 좋다)

> **자바빈즈 패턴 단점**
>
- 객체 하나를 만드려면 매서드를 여러개 호출해야 한다.
- 객체가 완전히 생성되기 전까지는 일관성(consistency)가 무너진 상태에 놓이게 된다. (객체 생성시 모든 값이 주입되지 않아 일관성과 불변성의 문제가 발생)
- 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드의 안전성을 얻으려면 프로그래머가 추가 작업을 해주어야한다.

*위와 같은 단점을 완화하고자 수동으로 객체를 얼리는 방법도 있지만 거의 사용하지 않는다.*

---

### 3. 빌더 패턴(Builder pattern)

- 복잡한 객체의 생성과정과 표현과정을 분리하여 다양한 구성의 인스턴스를 만드는 생성 패턴이다.
- 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비했다.
- 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(or 정적 팩터리)를 호출해 빌더 객체를 얻는다. 그 후 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 마지막으로 매개변수가 없는 build 매서드를 호출해 필요한 객체를 얻는다.

**빌더 패턴** - **코드**

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        // 필수 매개변수만을 담은 Builder 생성자
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택 매개변수의 setter, Builder 자신을 반환해 연쇄적으로 호출 가능
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        // build() 호출로 최종 불변 객체를 얻는다.
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}

// 호출 방식
NutritionFacts cocaCola = new NutriFacts.Builder(240, 8)
      .calories(100).sodium(35).carbohydrate(30).build();
```

- 빌더의 setter 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 이런 방식을 플루언트 API or 메서드 연쇄 라고 한다.

<aside>
💡 **빌더 패턴과 자바 빈즈 패턴의 차이점.**

**자바 빈즈 패턴** : 객체를 생성한 후에 값을 setter로 넣는다. 객체 사용도중 실수 혹은 악의적 목적으로 setter 메서드를 통해 유효하지 않은 값, null, 혹은 정확하지 않은 값이 들어갈 수 있다.

**빌더 패턴** : 객체를 생성하기 전에 값을 setter로 넣는다. 값을 모두 넣었다면 build 메서드를 호출해 객체를 생성한다. ⇒ 객체 사용중 값이 변경 될 우려가 없고 불변성과 안전성이 올라간다.

</aside>

> **빌더 패턴 장점**
>
1. 객체 생성 과정을 일관된 프로세스로 표현한다.
    1. 직관적으로 어떤 데이터에 어떤 값이 설정되는지 한눈에 파악할 수 있게 된다.
2. 디폴트 매개변수 생략을 간접적으로 지원한다.
    1. 디폴트 매개변수가 설정된 필드를 설정하는 매서드를 호출하지 않는 방식으로 사용가능하다.
3. 필수 맴버 변수와 선택적 맴버 변수를 분리할 수 있다.
    1. 초기화가 필수인 맴버는 빌더의 생성자로 받게 하여 필수 맴버를 설정해주어야 빌더 객체가 생성되도록 유도한다.
4. 객체 생성 단계를 지연할 수 있다.

> **빌더 패턴 단점**
>
1. 코드 복잡성 증가
    1. N개의 클래스에 대해 N개의 새로운 빌더 클래스를 만들어야 해서, 클래스 수가 기하급수적으로 늘어나 관리해야할 클래스가 많아지고 구조가 복잡해질 수 있다.
2. 생성자보다 성능 감소
    1. 매번 매서드를 호출하여 빌더를 거쳐 인스턴스화 하기때문에 성능이 떨어진다.
3. 지나친 빌더 남용은 안 좋다.
    1. 클래스의 필드의 개수가 4개보다 적고 필드의 변경 가능성이 없는 경우라면 생성자나 정적 팩토리 메소드를 이용하는 것이 좋을 수 있다. (하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있으므로 빌더로 시작하는 것이 좋을때가 많다)

> **빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.**
>
1. 각 계층의 클래스에 관련 빌더를 맴버로 정의하자.
2. 추상클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

**계층적으로 설계된 클래스와 잘 어울리는 빌더 패턴 - 코드**

```java
public abstract class Pizza{
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   // 추상 클래스는 추상 Builder를 가진다. 서브 클래스에서 이를 구체 Builder로 구현한다.
   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();

      // 하위 클래스는 이 메서드를 overriding하여 this를 반환하도록 해야 한다.
      protected abstract T self();
   }

   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone();
   }
}

//뉴욕 피자
public class NyPizza extends Pizza {
   public enum Size { SMALL, MEDIUM, LARGE }
   private final Size size;

   public static class Builder extends Pizza.Builder<Builder> {
      private final Size size;

      public Builder(Size size) {
         this.size = Objects.requireNonNull(size);
      }

      @Override public NyPizza build() {
         return new NyPizza(this);
      }

      @Override protected Builder self() { return this; }
   }

   private NyPizza(Builder builder) {
      super(builder);
      size = builder.size;
   }
}

//칼초네 피자
public class Calzone extends Pizza {
   private final boolean sauceInside;

   public static class Builder extends Pizza.Builder<Builder> {
      private boolean sauceInside = false;

      public Builder sauceInside() {
         sauceInside = true;
         return this;
      }

      @Override public Calzone build() {
         return new Calzone(this);
      }

      @Override protected Builder self() { return this; }
   }

   private Calzone(Builder builder) {
      super(builder);
      sauceInside = builder.sauceInside;
   }
}

// 계층적 빌더 사용 코드
public class Main {
    public static void main(String[] args) {
        NYPizza pizza = new NYPizza.Builder(SMALL)
                .addTopping(SAUSAGE)
                .addTopping(ONION)
                .build();

        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM)
                .sauceInside()
                .build();
    }
}
```

- 각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다.
- NyPizza.Builder는 NyPizza를 반환하고, Calzone.Builder는 Calzone를 반환한다.

⇒ 하위클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 공변 반환 타이핑이라고 한다.

<aside>

### 핵심정리
생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다. 
매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바 빈즈보다 훨씬 안전하다.

</aside>