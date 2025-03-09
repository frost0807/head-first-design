# 헤드 퍼스트 디자인 패턴 - Chapter 4 요약 및 정리

## 🏭 팩토리 패턴: 객체를 생성하는 방법을 개선하자!

이번 챕터에서는 **팩토리 패턴**(**Factory Pattern**)을 배웠다.

객체를 생성할 때 `new` 키워드를 직접 사용하면 **코드가 유연하지 않고 변경이 어려워진다.**  
이를 해결하기 위해 **팩토리 패턴**을 사용하여 객체 생성을 캡슐화할 수 있다.

팩토리 패턴을 이해하기 위해 **피자 주문 시스템**을 예제로 살펴보자.

---

## 🍕 1. 피자 주문 시스템 (Pizza Store)

우리는 피자를 주문하는 애플리케이션을 만든다고 가정하자.  
각각의 피자는 **뉴욕 스타일 피자**(**NYStylePizza**), **시카고 스타일 피자**(**ChicagoStylePizza**)처럼 지역별로 다를 수 있다.

### 💡 기본적인 피자 클래스 설계

```java
public abstract class Pizza {
    String name;
    String dough;
    String sauce;
    
    public void prepare() {
        System.out.println("Preparing " + name);
    }
    
    public void bake() {
        System.out.println("Baking " + name);
    }
    
    public void cut() {
        System.out.println("Cutting " + name);
    }
    
    public void box() {
        System.out.println("Boxing " + name);
    }

    public String getName() {
        return name;
    }
}
```

각 지역별 피자를 만든다고 가정하면, 아래와 같이 구현할 수 있다.

```java
public class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        name = "New York Style Cheese Pizza";
        dough = "Thin Crust Dough";
        sauce = "Marinara Sauce";
    }
}

public class ChicagoStyleCheesePizza extends Pizza {
    public ChicagoStyleCheesePizza() {
        name = "Chicago Style Deep Dish Cheese Pizza";
        dough = "Extra Thick Crust Dough";
        sauce = "Plum Tomato Sauce";
    }
    
    @Override
    public void cut() {
        System.out.println("Cutting the pizza into square slices");
    }
}
```

이제 피자를 주문하는 `PizzaStore` 클래스를 만들어 보자.

---

### 🚨 2. 문제 발생: 객체 생성이 중복된다!

현재 `PizzaStore`에서 피자를 주문하는 코드가 있다고 가정해 보자.

```java
public class PizzaStore {
    public Pizza orderPizza(String type) {
        Pizza pizza;
        
        if (type.equals("cheese")) {
            pizza = new NYStyleCheesePizza();  // 뉴욕 스타일 피자만 주문 가능
        } else {
            pizza = null;
        }

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
```

✅ 문제점:
1. `PizzaStore`가 구체적인 피자 클래스(`NYStyleCheesePizza`)를 직접 생성한다.
2. 새로운 피자 종류를 추가하려면 `PizzaStore`를 수정해야 한다.
3. 지역별 피자를 지원하려면 클래스가 계속 증가한다.

이 문제를 해결하기 위해 **팩토리 패턴**을 적용해 보자.

---

## 🏭 3. 팩토리 패턴 적용하기

팩토리 패턴을 사용하면 객체 생성의 책임을 별도의 클래스(팩토리)로 이동할 수 있다.

### 💡 Simple Factory (간단한 팩토리)

```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new NYStyleCheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new ChicagoStyleCheesePizza();
        }

        return pizza;
    }
}
```

이제 `PizzaStore`에서 직접 객체를 생성하는 것이 아니라, **팩토리에게 생성 책임을 위임**할 수 있다.

```java
public class PizzaStore {
    SimplePizzaFactory factory;

    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public Pizza orderPizza(String type) {
        Pizza pizza = factory.createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
```

✅ 장점
- `PizzaStore`는 객체 생성 코드에서 완전히 독립된다.
- 새로운 피자 종류를 추가할 때 **팩토리만 수정하면 된다.**
- **OCP(개방-폐쇄 원칙, Open-Closed Principle)**을 지킬 수 있다.

---

## 🏢 4. 팩토리 메서드 패턴: 상속을 활용한 객체 생성

팩토리 메서드 패턴을 적용하면 각 피자 가게에서 **고유한 피자를 생산할 수 있도록 캡슐화**할 수 있다.

```java
public abstract class PizzaStore {
    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }

    // 서브클래스에서 구현할 팩토리 메서드
    protected abstract Pizza createPizza(String type);
}
```

```java
public class NYPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else {
            return null;
        }
    }
}
```

---

## 🏭 5. 추상 팩토리 패턴: 관련 제품군을 위한 객체 생성

추상 팩토리 패턴은 **서로 관련있는 객체들의 집합(제품군)**을 구체적인 클래스에 의존하지 않고 생성하는 패턴이다.

```java
public interface PizzaIngredientFactory {
    Dough createDough();
    Sauce createSauce();
    Cheese createCheese();
}
```

```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    public Dough createDough() { return new ThinCrustDough(); }
    public Sauce createSauce() { return new MarinaraSauce(); }
    public Cheese createCheese() { return new ReggianoCheese(); }
}
```

---

🎯 **팩토리 패턴들의 장점**
- 객체 생성 로직을 분리하여 **코드의 유지보수성을 높인다.**
- **OCP(개방-폐쇄 원칙)**을 만족하여 **새로운 객체 추가가 용이하다.**
- 팩토리 메서드 패턴을 사용하면 **지역별로 다른 객체를 생성할 수 있다.**
- 추상 팩토리 패턴을 사용하면 **일관된 제품군을 제공할 수 있다.**