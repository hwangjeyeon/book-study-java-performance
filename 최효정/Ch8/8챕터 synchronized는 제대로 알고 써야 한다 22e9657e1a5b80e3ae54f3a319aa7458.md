# 8챕터: synchronized는 제대로 알고 써야 한다

# 8. synchronized는 제대로 알고 써야 한다

---

## 프로세스와 스레드

- **프로세스**
    
    실행 중인 하나의 프로그램. 예를 들어, 서버에서 WAS를 기동하면 자바 프로세스 하나가 생성된다.
    
- **스레드**
    
    프로세스 내에서 실행되는 작업 단위.
    
    하나의 프로세스에는 하나 이상의 스레드가 존재할 수 있다.
    
- 관계: **1 프로세스 : N 스레드**

---

## 스레드 구현 방법

### 1) Thread 클래스 상속

```java
public class ThreadExtends extends Thread {
  public void run() {
    System.out.println("This is ThreadExtends.");
  }
}

public class RunThreadExtends {
  public static void main(String[] args) {
    ThreadExtends te = new ThreadExtends();
    te.start();  // 새로운 스레드 실행
  }
}

```

### 2) Runnable 인터페이스 구현

```java
public class RunnableImpl implements Runnable {
  public void run() {
    System.out.println("This is RunnableImpl.");
  }
}

public class RunRunnableImpl {
  public static void main(String[] args) {
    RunnableImpl ri = new RunnableImpl();
    new Thread(ri).start();  // Thread에 Runnable 넣고 실행
  }
}

```

- `Thread` 상속과 `Runnable` 구현 모두 가능
- 이미 다른 클래스를 상속받고 있다면 `Runnable` 구현 방식을 사용해야 함
- 주의: `run()`을 직접 호출하면 단순 메서드 실행일 뿐, 새로운 스레드가 생성되지 않음 → 반드시 `start()` 사용

---

## 스레드 제어 메서드

- **sleep(ms)**
    
    지정한 시간(ms) 동안 해당 스레드를 대기시킴
    
- **wait()**
    
    `notify()` 또는 `notifyAll()`이 호출될 때까지 대기
    
    (동기화 블록 안에서만 사용 가능)
    
- **join()**
    
    다른 스레드가 종료될 때까지 대기
    
- **interrupt()**
    
    스레드에 인터럽트 신호를 보내 강제로 깨움 → `InterruptedException` 발생
    
- **notify() / notifyAll()**
    
    `wait()` 중인 스레드를 깨움
    
    - `notify()` : 한 개만
    - `notifyAll()` : 모두 깨움

---

## synchronized 키워드

- **용도**: 여러 스레드가 동시에 하나의 객체(또는 static 자원)에 접근할 때 데이터 불일치를 막기 위해 사용
- **사용 방법**
    
    ```java
    public synchronized void method() {
      // 메서드 전체 동기화
    }
    
    public void method2() {
      synchronized(this) {
        // 특정 블록만 동기화
      }
    }
    
    ```
    
- **필요한 경우**
    - 하나의 객체를 여러 스레드가 동시에 사용할 때
    - static 자원에 여러 스레드가 동시에 접근할 때

---

## java.util.concurrent (JDK 5 이상)

동기화를 더 안전하고 편리하게 처리하기 위한 패키지

- **Lock**
    
    명시적으로 `lock()` / `unlock()` 제어 가능 → 데드락 방지에 유용
    
- **Executors**
    
    스레드 풀 관리 → 불필요한 스레드 생성/제거를 줄이고 효율적 운영 가능
    
- **Concurrent Collections**
    
    `ConcurrentHashMap`, `CopyOnWriteArrayList` 등 동기화된 컬렉션 제공
    
- **Atomic 변수**
    
    `AtomicInteger`, `AtomicLong` 등 원자적 연산 보장
    
    → `synchronized` 없이도 안전한 연산 가능
    

---

## JVM 내부에서의 synchronized 동작

- JVM은 **모니터(monitor)** 라는 구조를 통해 동기화를 관리
- 한 시점에 오직 하나의 스레드만 모니터를 획득하여 동기화 블록에 진입 가능
- 스레드가 블록을 빠져나오면 모니터는 다른 대기 중인 스레드로 넘어감

### 최적화 기법 (JDK 5 이후)

- **Biased Locking (편향 락)**
    
    락을 특정 스레드에 치우치게 하여 재진입 비용을 줄임
    
- **Adaptive Spinning**
    
    곧 락이 풀릴 것으로 예상되면, 스레드를 재우지 않고 잠시 바쁘게 기다리도록 하여 성능 향상
    
- **fast-path / slow-path**
    - 일반적인 경우: 빠른 경로(fast-path)로 락 처리
    - 경합 발생: 느린 경로(slow-path)로 전환 (C++ 기반 구현)

---

## 핵심 정리

1. 프로세스와 스레드는 1:N 관계이며, 스레드는 `Thread` 상속 또는 `Runnable` 구현으로 생성
2. `start()` 메서드를 통해서만 새로운 스레드가 실행됨
3. `synchronized`는 공유 자원에 대한 동시 접근을 막아 데이터 불일치를 방지
4. JDK 5 이후에는 `java.util.concurrent` 패키지를 활용해 더 효율적인 동기화 가능
5. JVM은 모니터 기반의 락 관리와 다양한 최적화 기법으로 동기화 성능을 개선