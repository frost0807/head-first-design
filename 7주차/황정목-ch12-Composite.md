# 헤드 퍼스트 디자인 패턴 - Chapter 12 요약 및 정리

## 복합 패턴: 여러 패턴을 조합해 유연하고 확장 가능한 시스템을 만들자

이번 챕터에서는 **복합 패턴**(**Compound Pattern**)을 배웠다.

복합 패턴이란 **여러 디자인 패턴을 조합하여 해결책을 구성하는 방식**이다.  
각 패턴의 강점을 살려 **서로 협력하게 만들어 시스템을 더 유연하고 유지보수하기 쉽게 설계**할 수 있다.

이 챕터에서는 오리 시뮬레이션 시스템을 예제로 **복합 패턴을 실전 적용**한다.

> 사용된 패턴들: **팩토리 패턴, 어댑터 패턴, 데코레이터 패턴, 컴포지트 패턴, 옵저버 패턴**

---

## 1. 오리 시뮬레이션 시스템

오리들이 꽥꽥거리는 시뮬레이션을 만들고 싶다.  
단순히 `quack()`만 출력하면 될 것 같지만, 다음과 같은 요구사항이 있다:

- 오리 외에 거위도 꽥꽥거리는 것처럼 시뮬레이션에 포함하고 싶다
- 모든 꽥꽥 소리를 카운팅하고 싶다
- 오리 무리를 구성하고, 함께 행동시키고 싶다
- 오리의 행동을 관찰할 수 있도록 옵저버를 붙이고 싶다

이제 패턴들을 조합해서 요구사항을 해결해보자!

---

## 2. 핵심 인터페이스: Quackable

```java
public interface Quackable extends QuackObservable {
    void quack();
}
```

모든 오리와 유사 객체(거위, 장난감 등)는 `quack()` 메서드를 가진다.  
또한 옵저버 패턴을 적용하기 위해 `QuackObservable` 인터페이스도 확장한다.

---

## 3. 어댑터 패턴: 거위를 오리처럼 사용하기

거위는 `quack()`이 아니라 `honk()`를 한다. 이를 `Quackable`로 어댑팅해야 한다.

```java
public class GooseAdapter implements Quackable {
    Goose goose;
    Observable observable;

    public GooseAdapter(Goose goose) {
        this.goose = goose;
        this.observable = new Observable(this);
    }

    public void quack() {
        goose.honk();
        notifyObservers();
    }

    public void registerObserver(Observer observer) {
        observable.registerObserver(observer);
    }

    public void notifyObservers() {
        observable.notifyObservers();
    }
}
```

---

## 4. 데코레이터 패턴: 꽥꽥 카운팅 기능 추가

```java
public class QuackCounter implements Quackable {
    Quackable duck;
    static int numberOfQuacks;
    Observable observable;

    public QuackCounter(Quackable duck) {
        this.duck = duck;
        this.observable = new Observable(this);
    }

    public void quack() {
        duck.quack();
        numberOfQuacks++;
    }

    public static int getQuacks() {
        return numberOfQuacks;
    }

    public void registerObserver(Observer observer) {
        duck.registerObserver(observer);
    }

    public void notifyObservers() {
        duck.notifyObservers();
    }
}
```

모든 `Quackable` 객체를 `QuackCounter`로 감싸면 **데코레이터처럼 기능을 추가**할 수 있다.

---

## 5. 컴포지트 패턴: 오리 무리(Flock) 만들기

```java
public class Flock implements Quackable {
    List<Quackable> quackers = new ArrayList<>();

    public void add(Quackable quacker) {
        quackers.add(quacker);
    }

    public void quack() {
        for (Quackable quacker : quackers) {
            quacker.quack();
        }
    }

    public void registerObserver(Observer observer) {
        for (Quackable quacker : quackers) {
            quacker.registerObserver(observer);
        }
    }

    public void notifyObservers() {
        // 위임
    }
}
```

`Flock`은 여러 오리를 그룹으로 묶고, 한 번에 행동하게 만든다.

컴포지트 패턴 덕분에 `Quackable` 인터페이스만 알면 오리든 무리든 동일하게 다룰 수 있다.

---

## 6. 옵저버 패턴: 오리 관찰하기

### 옵저버 인터페이스

```java
public interface Observer {
    void update(QuackObservable duck);
}
```

### 옵저버 구현체

```java
public class Quackologist implements Observer {
    public void update(QuackObservable duck) {
        System.out.println("관찰자: " + duck + "가 방금 꽥 소리를 냈습니다.");
    }
}
```

### 옵저버 등록 및 알림

모든 `Quackable` 클래스는 `Observable` 객체를 포함하고 있고, 이 객체가 옵저버 등록/알림을 처리한다.

```java
public interface QuackObservable {
    void registerObserver(Observer observer);
    void notifyObservers();
}

public class Observable implements QuackObservable {
    List<Observer> observers = new ArrayList<>();
    QuackObservable duck;

    public Observable(QuackObservable duck) {
        this.duck = duck;
    }

    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(duck);
        }
    }
}
```

---

## 7. 클라이언트 코드: 오리 시뮬레이터

```java
public class DuckSimulator {
    public static void main(String[] args) {
        AbstractDuckFactory factory = new CountingDuckFactory();

        Quackable mallard = factory.createMallardDuck();
        Quackable redhead = factory.createRedheadDuck();
        Quackable duckCall = factory.createDuckCall();
        Quackable rubber = factory.createRubberDuck();
        Quackable goose = new GooseAdapter(new Goose());

        Flock flock = new Flock();
        flock.add(mallard);
        flock.add(redhead);
        flock.add(duckCall);
        flock.add(rubber);
        flock.add(goose);

        Quackologist observer = new Quackologist();
        flock.registerObserver(observer);

        simulate(flock);

        System.out.println("꽥 소리 횟수: " + QuackCounter.getQuacks());
    }

    static void simulate(Quackable duck) {
        duck.quack();
    }
}
```

---

## 정리

**복합 패턴**은 여러 디자인 패턴을 조합하여 문제를 유연하게 해결하는 방식이다.
이 장에서는 아래의 패턴들이 함께 사용되었다:

- **팩토리 패턴**: 오리 객체 생성을 캡슐화
- **어댑터 패턴**: 거위를 오리처럼 사용
- **데코레이터 패턴**: 꽥꽥 소리 횟수 측정
- **컴포지트 패턴**: 오리 무리(Flock) 구성
- **옵저버 패턴**: 오리 행동을 감시

실무에서는 여러 패턴을 **상황에 맞게 조합**하여 사용하는 것이 일반적이다.  
복합 패턴을 통해 더 **강력하고 유지보수 가능한 시스템**을 설계할 수 있다!
