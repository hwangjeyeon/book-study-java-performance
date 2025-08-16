# 스레드 기본 개념
## 멀티스레드 환경 이해: 
자바 애플리케이션(WAS 등)을 실행하면 운영체제 프로세스가 하나 생성되고, 
그 프로세스 내에서 여러 개의 스레드가 동작할 수 있습니다 
하나의 프로세스가 다수의 스레드를 포함할 수 있으므로 프로세스와 스레드는 **1 대 N**의 관계로 볼 수 있습니다
스레드는 Light Weight Process (LWP)라고 불리는데, 가벼운 프로세스로서 
부모 프로세스의 메모리를 공유하여 실행됩니다 
이러한 메모리 공유 특성 때문에, 여러 스레드가 하나의 자원을 동시에 접근하면 충돌이 발생할 수 있어 동기화(synchronization)가 필요합니다

# 스레드 구현 방법: Thread vs Runnable
자바에서 스레드를 구현하는 방법은 두 가지가 있습니다
## Thread 클래스 상속 
`Thread` 클래스를 상속받아 `run()` 메서드를 오버라이드합니다
이 방법을 사용할 경우, 스레드 객체를 생성한 뒤 `start()` 메서드를 호출하면 새로운 스레드에서 `run()` 메서드가 실행됩니다

## Runnable 인터페이스 구현 
`Runnable` 인터페이스를 구현하고 `run()` 메서드를 정의합니다. 
그런 다음 `new Thread(runnable객체).start()`로 해당 Runnable을 실행합니다

`Thread` 클래스 자체가 이미 `Runnable`을 구현하고 있기 때문에 동작 면에서는 Thread를 상속한 것과 거의 동일합니다. 
하지만 Runnable 구현 방식은 이미 다른 클래스를 상속 중인 경우에도 사용할 수 있다는 **유연성**이 있습니다

그럼에도 Runnable 구현을 사용하면 매번 Thread를 만들어주어야 하는 **불편함**이 있고, 
별도의 스레드 객체 생성이 필요하다는 **단점**도 있습니다

아래는 두 가지 구현 방법을 비교한 간단한 코드 예시입니다:

```
// Thread 클래스를 상속하여 스레드 구현
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("MyThread 실행");
    }
}

// Runnable 인터페이스를 구현하여 스레드 구현
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("MyRunnable 실행");
    }
}

// Thread와 Runnable 사용 예시
public class ThreadExample {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start(); // 새로운 스레드에서 MyThread.run() 실행

        MyRunnable runnable = new MyRunnable();
        Thread thread2 = new Thread(runnable);
        thread2.start(); // 새로운 스레드에서 MyRunnable.run() 실행
    }
}

```


# 스레드 대기 메서드: sleep, wait, join
멀티스레드 프로그래밍에서는 특정 스레드를 일시 정지하거나 대기시키는 메소드들이 제공됩니다

## sleep(time)
현재 실행 중인 스레드를 지정된 시간동안 (밀리초 단위) 일시 정지시킵니다 

## `wait(time)` 
대상 객체의 모니터 락을 가진 상태에서 호출하여, 해당 객체에서 다른 스레드의 `notify`/`notifyAll` 신호가 올 때까지 지정된 시간동안 현재 스레드를 대기시킵니다
`wait()`은 반드시 동기화 블록 또는 메소드(모니터 락을 소유한 상태)에서 호출해야 합니다
파라미터를 지정하지 않으면 `notify()` 또는 `notifyAll()`로 깨워줄 때까지 무기한 대기합니다.

## `join(time)` 
다른 스레드가 종료될 때까지 현재 스레드의 실행을 **일시 정지**시킵니다
예를 들어 `t.join();`은 현재 스레드를 멈추고 스레드 `t`가 끝날 때까지 기다립니다 
시간 제한을 지정하면 최대 그 시간만큼 기다린 후 이어서 실행을 계속합니다. 

주로 **메인 스레드가 작업 스레드들의 종료를 기다릴 때** 사용됩니다
```
public static void main(String[] args) {
	SumTask task1 = new SumTask(1, 50);
	SumTask task2 = new SumTask(51, 100);
	
	Thread thread1 = new Thread(task1, "thread-1");
	Thread thread2 = new Thread(task2, "thread-2");
	
	thread1.start();
	thread2.start();
	
	thread1.join();
	thread2.join();

	int sumAll = task1.result + task2.result;
}
static class SumTask implements Runnable {
	int startValue;
	int endValue;
	int result = 0;
	public SumTask(int startValue, int endValue) {
		this.startValue = startValue;
		this.endValue = endValue;
	}
	@Override
	public void run() {		
		int sum = 0;
		for (int i = startValue; i <= endValue; i++) {
			sum += i;
		}
		result = sum;
		
	}
}

```


## 스레드 제어 메서드: interrupt, notify, notifyAll
스레드의 동작을 제어하거나 상태를 확인하기 위한 메th드들도 있습니다
## `interrupt()`
다른 스레드의 차단(block) 상태를 해제합니다. 
예를 들어 어떤 스레드가 `sleep()`이나 `wait()` 등으로 대기하고 있을 때, 
그 스레드의 `interrupt()`를 호출하면 해당 스레드에서 `InterruptedException`이 발생하며 대기 
상태에서 빠져나옵니다
## `isInterrupted()`
특정 스레드가 interrupt 되었는지 **확인**합니다 

## `notify()` 
`wait()`으로 기다리던 스레드 중 하나가 다시 실행될 수 있도록 신호를 보내는 것입니다. 
어떤 스레드가 깨어날지는 지정할 수 없으며, 임의의 한 스레드가 깨어납니다

## `notifyAll()` 
여러 스레드가 `wait()` 중이라면 모두 깨워 실행 대기 상태로 전환시킵니다

# synchronized 키워드와 동기화
## synchronized의 역할: 
`synchronized` 키워드는 임계 구역(Critical Section)을 지정하여 동시에 한 스레드만 해당 코드를 실행하도록 보장합니다
즉, 공유 자원에 여러 스레드가 접근할 때 한 번에 하나의 스레드만 접근하도록 
상호 배제(Mutual Exclusion)를 구현하는 장치입니다. 
`synchronized`는 두 가지 형태로 사용할 수 있습니다

## 메소드 선언: 
메소드 전체에 락을 거는 방식입니다 
한 스레드가 메소드 실행 중이면 같은 객체의 그 메서드에 다른 스레드가 동시에 진입하지 못합니다.

## 임의의 블록에 사용: 
메소드 내의 일부 블록을 지정하여 락을 거는 방식입니다 
명시된 객체를 모니터 락으로 사용하여 그 블록을 보호합니다. 
원하는 객체로 락을 걸 수 있으므로 보다 세밀한 동기화가 가능합니다.

> **생성자에는 `synchronized`를 사용할 수 없습니다.** 생성자 실행 중에는 해당 객체를 다른 스레드가 볼 수 없으므로 동기화 키워드를 허용하지 않습니다.


##  static 객체 동기화 주의사항
동기화 대상이 static으로 선언된 객체(클래스 변수)인 경우에는 주의해야 합니다
만약 여러 스레드에서 공유하는 변수가 `static`이라면, 
그 변수를 조작하는 메서드도 `static synchronized`로 선언해야 올바르게 동기화됩니다

```
class StaticExample { 
private static int staticCount = 0;     

public static synchronized void incrStatic() {         
		staticCount++;     
	} 
}
```

위 코드처럼 `static synchronized` 메소드는 해당 클래스의 **Class 객체**에 락을 겁니다.
이렇게 해야 모든 스레드가 클래스 단위로 **하나의 동일한 락**을 공유하게 됩니다. 만약 `staticCount`를 증가시키는 메서드를 단순히 `synchronized` (인스턴스 메서드)로 선언하면, 각 객체별로 다른 락이 적용됩니다
그 결과 같은 `StaticExample` 클래스의 여러 인스턴스가 있을 때 
`staticCount`의 동시 접근이 제대로 통제되지 않는 오류가 생길 수 있습니다. 
따라서 클래스 변수는 클래스 자체의 락으로 보호해야 합니다.


### 동기화가 필요한 경우와 성능 고려
`synchronized` 키워드를 **언제 사용해야 할지** 판단하는 것이 중요합니다. 
다음 두 가지 경우에만 동기화를 고려하는 것이 좋습니다
- **하나의 객체를 여러 스레드가 동시에 접근할 경우**
- **`static`으로 선언된 객체를 여러 스레드가 동시에 사용할 경우** 


위의 상황에 해당하지 않는다면, 굳이 동기화를 사용할 필요는 많지 않습니다. 
다시 말해, 필요한 곳에만 최소한으로 synchronized를 써야 합니다. 
`synchronized`는 동시 접근을 막아주는 장점이 있지만, 그에 따라 성능 저하라는 단점이 따릅니다

락을 획득하고 해제하는 오버헤드, 그리고 락이 걸린 구간에서는 여러 스레드가 대기하며 직렬화되기 때문에 응답 시간이 늘어날 수 있습니다
따라서 무조건 `synchronized`를 남발해서는 안 됩니다

## Vector vs ArrayList
자바의 `Vector` 클래스는 모든 메소드가 synchronized로 구현되어 있어 쓰레드 안전(thread-safe)
하지만, 단일 스레드 상황에서는 불필요한 락 획득/해제로 인해 `ArrayList`보다 훨씬 느립니다

## 고급 동시성 도구 (java.util.concurrent)
자바 5부터는 개발자가 직접 `wait()`/`notify()` 또는 낮은 수준에서 `synchronized`를 관리하지 않고도 thread-safe한 프로그래밍을 할 수 있도록 다양한 **고수준 동시성 유틸리티**가 추가되었습니다
이러한 도구들은 `java.util.concurrent` 패키지에 포함되어 있으며, 성능과 사용 편의성 면에서 많은 이점을 제공합니다:

## Lock 및 조건 변수 (`java.util.concurrent.locks`):
기존 `synchronized` 키워드의 기능을 보다 확장한 락 객체들입니다. 

가장 많이 쓰이는 **`ReentrantLock`** 클래스는 `synchronized`와 동일한 메모리 가시성 및 상호배타 보장에 더해, **공정성(fairness) 옵션**이나 **시도 횟수 제한**(`tryLock`) 등 부가 기능을 제공합니다

### 스레드 풀 및 실행 서비스 (`Executors`): 
새로운 스레드를 직접 만들고 관리하는 대신, 스레드 풀(thread pool)을 활용하여 스레드 재사용과 작업 큐잉을 관리할 수 있습니다 
`Executors` 유틸리티 클래스를 통해 고정 크기 풀, 캐시된 풀 등 다양한 풀을 생성할 수 있으며, `ExecutorService` 인터페이스를 통해 스레드 풀에 작업을 제출(submit)하고 **비동기 작업 수행(Future 등)** 결과를 받을 수 있습니다

## Concurrent 컬렉션: 
자바 5 이후 다양한 동시성 컬렉션 클래스가 도입되었습니다. 
대표적으로 `ConcurrentHashMap`은 내부를 세분화된 락으로 관리하여 다중 스레드 환경에서 
고성능으로 Map을 동시 수정/조회할 수 있습니다.

## 원자적 연산 변수 (`AtomicInteger`, `AtomicReference` 등):
`java.util.concurrent.atomic` 패키지는 멀티스레드 환경에서 **락을 사용하지 않고도(thread-safe)** 변수의 원자적 연산을 가능하게 해줍니다
예를 들어 `AtomicInteger`는 `incrementAndGet()`과 같은 메서드로 내부적으로 CAS(compare-and-swap) 같은 원자적 CPU 연산을 활용하여 락 없이도 스레드 안전하게 정수 값을 증가시킬 수 있습니다

동시성 도구들을 적절히 활용하면, 기존에 `synchronized`와 `wait()/notify()`로 직접 구현해야 했던 패턴들을 보다 **안전하고 효율적으로** 구현할 수 있습니다. 
이런 고급 도구들은 내부 구현에서 이미 최적화와 검증이 이루어졌으므로, 
직접 락을 관리하는 것보다 버그를 줄이고 성능을 높일 수 있는 방법이라고 할 수 있습니다.

# 참고
- 자바 성능 튜닝 이야기 - 8챕터
