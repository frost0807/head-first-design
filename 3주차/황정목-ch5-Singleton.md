# 헤드 퍼스트 디자인 패턴 - Chapter 5 싱글톤 패턴

## 🏆 싱글톤 패턴이란?

**싱글톤 패턴**(**Singleton Pattern**)은 **클래스의 인스턴스를 하나만 유지하도록 보장**하는 패턴이다. 
애플리케이션 전역에서 공유해야 하는 객체를 만들 때 유용하게 사용된다.

✅ **사용 예시**
- 데이터베이스 연결 (DB Connection)
- 로깅 (Logging)
- 설정 정보 관리 (Configuration Management)
- 스레드 풀 (Thread Pool)

---

## 🎯 싱글톤 패턴의 핵심 원칙

1. **클래스 인스턴스를 하나만 유지한다.**
2. **외부에서 직접 객체를 생성할 수 없도록 한다.**
3. **글로벌 접근 지점을 제공한다.**

---

## 🛠️ 싱글톤 패턴 구현 방법

### 1️⃣ 기본적인 싱글톤 구현

가장 기본적인 싱글톤 패턴을 구현하는 방법은 **정적(static) 변수를 활용하는 것**이다.

```java
public class Singleton {
    private static Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

✅ **문제점**: 멀티스레드 환경에서 `uniqueInstance`가 여러 번 생성될 가능성이 있다.

---

### 2️⃣ 동기화로 해결하는 싱글톤

멀티스레드 환경에서도 안전하게 싱글톤을 보장하려면 `synchronized`를 추가할 수 있다.

```java
public class Singleton {
    private static Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static synchronized Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

✅ **문제점**: `synchronized` 키워드는 성능 저하를 유발할 수 있다.

---

### 3️⃣ 더블 체크 락킹(Double-Checked Locking)

성능 문제를 해결하기 위해 **Double-Checked Locking**을 사용할 수 있다.

```java
public class Singleton {
    private static volatile Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

✅ **장점**: `synchronized`가 한 번만 실행되므로 성능 저하를 최소화할 수 있다.

---

### 4️⃣ 정적 초기화(Static Initialization)

JVM의 클래스 로딩 특성을 활용하여 인스턴스를 미리 생성하는 방법이다.

```java
public class Singleton {
    private static final Singleton uniqueInstance = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return uniqueInstance;
    }
}
```

✅ **장점**: 스레드 안전(Thread Safe)하며, 구현이 간단하다.
✅ **단점**: 클래스가 로딩될 때 바로 인스턴스가 생성되므로 필요하지 않을 때도 메모리를 차지할 수 있다.

---

### 5️⃣ 내부 정적 클래스(Holder 방식)

JVM이 클래스 로딩을 지연시키는 특성을 활용하는 방법이다.

```java
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

✅ **장점**:
- Lazy Initialization(필요할 때 생성됨)
- Thread-safe(멀티스레드 환경에서 안전함)
- 성능 저하 없음 (synchronized 사용 X)

---

## 🚀 싱글톤 패턴을 사용할 때 주의할 점

✅ **싱글톤이 적합한 경우**
- 전역적으로 하나의 인스턴스만 존재해야 할 때
- 설정, 로그 관리, 데이터베이스 연결 등에서 사용

⚠️ **싱글톤을 남용하면 안 되는 경우**
- 객체 간의 강한 결합도를 초래할 수 있음 (테스트 어려움)
- 멀티스레드 환경에서 설계 실수 시 문제 발생 가능
- 객체 생명주기를 명확히 관리해야 할 때 (ex. 웹 애플리케이션)

---

## 🎯 싱글톤 패턴 정리

✅ **싱글톤 패턴의 목적**: 인스턴스 하나만 생성하고 이를 전역적으로 공유한다.
✅ **구현 방법**
- 기본 싱글톤 (`synchronized` X, 멀티스레드 문제 O)
- 동기화(`synchronized`) 추가 (성능 저하 문제 O)
- **더블 체크 락킹(Double-Checked Locking)** (권장)
- **정적 초기화(Static Initialization)** (성능 최적화, 단 메모리 문제 O)
- **내부 정적 클래스(Holder 방식)** (가장 안전하고 권장됨)

💡 **가장 권장되는 방법**: **내부 정적 클래스(Holder 방식)**