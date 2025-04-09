# 헤드 퍼스트 디자인 패턴 - Chapter 11 요약 및 정리

## 프록시 패턴: 대리인을 통해 객체에 대한 접근을 제어하자

이번 챕터에서는 **프록시 패턴**(**Proxy Pattern**)을 배웠다.

프록시 패턴은 **다른 객체에 대한 접근을 제어하기 위해 대리 객체(Proxy)를 사용하는 디자인 패턴**이다.
즉, 실제 객체에 직접 접근하는 대신 **프록시 객체를 통해 제어하거나 기능을 추가할 수 있다.**

---

## 1. 프록시 패턴의 핵심 아이디어

프록시는 **실제 객체와 동일한 인터페이스를 구현하고**, 요청을 가로채어 필요한 작업을 수행한 후 실제 객체에 전달하거나 자체적으로 처리할 수 있다.

프록시의 목적:

- **접근 제어 (Protection Proxy)**
- **지연 초기화 (Virtual Proxy)**
- **원격 통신 (Remote Proxy)**
- **캐싱, 로깅 등 부가 기능 (Smart Proxy)**

---

## 2. 예제 시나리오: 원격 자판기 시스템(Remote Gumball Machine)

멀리 떨어진 자판기 객체에 접근해야 한다면 어떻게 해야 할까?  
자바의 RMI(Remote Method Invocation)를 사용하면 객체를 네트워크를 통해 호출할 수 있다.

이때 프록시 객체는 **클라이언트와 실제 자판기 객체 사이에서 대리 역할**을 하며, **원격 호출을 담당한다.**

---

## 3. Remote Proxy 구조

### 공통 인터페이스 정의 (원격 인터페이스)

```java
public interface GumballMachineRemote extends Remote {
    String getLocation() throws RemoteException;
    int getCount() throws RemoteException;
    State getState() throws RemoteException;
}
```

### 실제 객체 (서버 측)

```java
public class GumballMachine extends UnicastRemoteObject implements GumballMachineRemote {
    State state;
    int count;
    String location;

    public GumballMachine(String location, int count) throws RemoteException {
        this.location = location;
        this.count = count;
        // 상태 초기화
    }

    public String getLocation() {
        return location;
    }

    public int getCount() {
        return count;
    }

    public State getState() {
        return state;
    }
}
```

### 원격에서 호출하는 클라이언트 측 코드

```java
public class GumballMonitor {
    GumballMachineRemote machine;

    public GumballMonitor(GumballMachineRemote machine) {
        this.machine = machine;
    }

    public void report() {
        try {
            System.out.println("자판기 위치: " + machine.getLocation());
            System.out.println("알맹이 개수: " + machine.getCount());
            System.out.println("현재 상태: " + machine.getState());
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}
```

### 클라이언트 실행 예시

```java
public class GumballMonitorTestDrive {
    public static void main(String[] args) {
        try {
            GumballMachineRemote machine = (GumballMachineRemote) Naming.lookup("rmi://127.0.0.1/gumballmachine");
            GumballMonitor monitor = new GumballMonitor(machine);
            monitor.report();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

클라이언트는 **원격 객체인지, 로컬 객체인지 구분할 필요 없이 동일한 인터페이스로 사용**할 수 있다.

네트워크를 통해 호출하되, **프록시가 그 복잡함을 숨겨준다.**

---

## 4. 다양한 프록시 유형

| 유형               | 설명 |
|--------------------|------|
| 원격 프록시 (Remote Proxy) | 원격 객체에 대한 접근 제어. 네트워크를 통한 호출을 캡슐화 |
| 가상 프록시 (Virtual Proxy) | 리소스가 무거운 객체의 생성을 지연시킴 (ex. 이미지 로딩) |
| 보호 프록시 (Protection Proxy) | 권한이 없는 사용자가 민감한 메서드를 호출하지 못하게 제한 |
| 스마트 프록시 (Smart Proxy) | 접근 시 부가 작업 수행 (캐싱, 로깅, 참조 횟수 등) |

---

## 정리

- 프록시 패턴은 **다른 객체에 대한 접근을 제어하거나 기능을 추가**하고 싶을 때 유용한 패턴이다.
- 클라이언트는 실제 객체와 프록시 객체를 구분하지 않고 사용할 수 있다 (같은 인터페이스 구현).
- 다양한 상황(원격 호출, 지연 로딩, 권한 제어 등)에서 유연하게 활용할 수 있다.

RMI, AOP, Spring Proxy, Lazy Loading 등 많은 기술이 프록시 패턴을 기반으로 동작한다.
