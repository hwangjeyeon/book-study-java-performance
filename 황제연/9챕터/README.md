# I/O 병목 현상
웹 애플리케이션에서 I/O는 성능을 좌우하는 가장 중요한 요소 중 하나입니다 
I/O 작업은 CPU 연산과 달리 디스크나 네트워크 같은 외부 장치의 응답을 기다리는 대기 시간이 대부분을 차지하기 때문에, 
비효율적인 I/O 처리는 시스템 전체의 응답 속도를 저하시키는 주된 병목점이 됩니다

# 작은 단위 I/O 처리에서의 비효율성
가장 기본적인 I/O 병목은 데이터를 처리하는 단위가 너무 작을 때 발생합니다

#### 원인: 잦은 System Call 오버헤드
Java에서 `FileReader`의 `read()` 메소드로 파일을 한 글자씩 읽는 것은 가장 직관적인 방법이지만, 성능은 매우 비효율적입니다.

Java

```
// 비효율적인 방식: 한 글자씩 읽기
while ((data = fileReader.read()) != -1) {
    // ... 처리 로직 ...
}
```

이 코드는 루프가 한 번 돌 때마다 JVM에서 운영체제(OS)를 거쳐 디스크에 접근하는 **시스템 호출**을 유발할 수 있습니다. 
10MB 크기의 파일을 읽는다면 수백만 번의 호출이 발생하며, 
이 과정에서 발생하는 컨텍스트 스위칭(Context Switching) 비용과 
데이터 처리 오버헤드로 인해 응답 시간이 늘어납니다. (예시: 약 **2.5초** 소요)

#### 해결 방안: 버퍼(Buffer)를 활용해서 한 번에 처리

이 문제를 해결하는 방법이 버퍼링(Buffering)입니다 
데이터를 한 번에 일정 크기(Chunk)만큼 모아서 처리하여 System Call 횟수를 줄이는 것입니다

## char 배열 버퍼 직접 사용 
`read(char[] cbuf)` 메소드를 사용하면 지정된 배열 크기만큼 한 번에 데이터를 읽어올 수 있습니다 
1MB 크기의 버퍼를 사용하면 시스템 호출 횟수가 10번 내외로 줄어들어 성능이 향상됩니다. (예시: 약 **400ms** 소요)
```
    // 개선된 방식: 1MB 버퍼 사용
    try (FileReader fr = new FileReader(fileName)) {
        char[] buffer = new char[1024 * 1024]; // 1MB 버퍼
        while (fr.read(buffer) != -1) {
            // ... 버퍼에 담긴 데이터 처리 ...
        }
    }
```

## BufferedReader 사용 
Java는 `BufferedReader`라는 고수준 스트림 클래스를 제공합니다 
이 클래스는 내부적으로 버퍼를 가지고 있어 `readLine()` 메서드를 호출하면, 알아서 효율적으로 데이터를 읽어 한 줄씩 반환해 줍니다 
코드가 간결해지고 성능 또한 매우 우수합니다 (예시: 약 **350ms** 소요)
```
   // 권장 방식: BufferedReader와 try-with-resources 사용
    ArrayList<String> lines = new ArrayList<>();
    try (BufferedReader br = new BufferedReader(new FileReader(fileName))) {
        String line;
        while ((line = br.readLine()) != null) {
            lines.add(line);
        }
    }
```

### try-with-resources
Java 7부터 도입된 `try-with-resources` 구문은 `finally` 블록에서 
`close()`를 명시적으로 호출할 필요 없이 리소스를 자동으로 해제해주므로 반드시 사용해야 합니다 
단, try() 안에 AutoCloseable 또는 Closeable을 구현한 객체를 선언해야합니다

또한, `java.nio.file.Files` 클래스는 더 편리한 API를 제공합니다
- `Files.newBufferedReader(path)`: `BufferedReader`를 더 쉽게 생성합니다.
- `Files.readAllLines(path)`: 파일의 모든 줄을 읽어 `List<String>`으로 반환합니다. 단, 파일 전체를 메모리에 올리므로 대용량 파일에는 `OutOfMemoryError`를 유발할 수 있어 주의해야 합니다.
- `Files.lines(path)`: 파일 내용을 `Stream<String>`으로 제공하여, 대용량 파일도 메모리 효율적으로 처리할 수 있습니다.


# 불필요하게 반복되는 I/O의 성능저하
성능 저하는 I/O 처리 단위뿐만 아니라, 불필요한 I/O 호출 빈도 때문에도 발생합니다

## 원인: 데이터 재사용의 부재
웹 요청이 있을 때마다 설정 파일을 디스크에서 읽고 파싱하는 코드는 대표적인 안티패턴입니다

```
// 안티패턴: 요청마다 설정 파일을 읽는 구조
public void processRequest() {
    // 매번 파일을 열고 파싱하는 로직
    Config config = readConfigFile("path/to/config.xml");
    // ...
}
```
이러한 구조는 동시 사용자가 많아질수록 서버에 디스크 I/O 부하를 주며, 
이는 애플리케이션 전체의 성능 저하로 이어집니다.

## 해결 방안: 캐싱(Caching)을 통한 I/O 호출 최소화
자주 바뀌지 않는 데이터는 최초 한 번만 읽어 메모리에 캐싱하고 재사용하는 것이 정답입니다

### Static 변수 또는 싱글톤 활용: 
애플리케이션 시작 시 설정 파일을 로드하여 `static` 변수나 싱글톤 객체에 저장해두고, 
이후 모든 요청은 메모리에 있는 이 객체를 사용합니다

만약 파일 내용이 변경될 수 있다면, 
매 요청마다 확인하는 대신 주기적으로 갱신하는 전략을 사용해야 합니다

### 파일 변경 감지 및 갱신 방법
과거에는 파일 변경을 감지하기 위해 별도의 스레드가 주기적으로 
파일의 `lastModified()` 시간을 체크하는 비효율적인 폴링(Polling) 방식을 사용했습니다 
 Java 7부터 도입된 `WatchService` API를 사용하면 OS의 파일 시스템 이벤트 기능을 활용하여, 
 파일이 실제로 변경되었을 때만 알림을 받아 캐시를 갱신할 수 있습니다 
 
# 동기 블로킹 I/O로 인한 성능 저하
Java I/O는 스레드가 I/O 작업을 요청하면 결과가 올 때까지 기다리는 **동기 블로킹(Synchronous Blocking)** 방식입니다 
이로 인해 스레드가 대기 상태에 머물며 자원을 낭비하는 문제가 발생합니다.

#### NIO (New I/O)의 등장
Java 1.4에서 도입된 NIO는 이러한 문제를 해결하기 위해 
채널(Channel), 버퍼(Buffer), 셀렉터(Selector) 개념을 도입했습니다
- 채널 & 버퍼: OS의 커널 버퍼와 직접 상호작용하여 데이터 복사 단계를 줄이고 효율을 높입니다
- 셀렉터: 하나의 스레드가 여러 개의 채널(네트워크 연결 등)을 동시에 관리하며, 
	I/O 작업이 준비된 채널만 골라서 처리하는 논블로킹(Non-Blocking) I/O를 가능하게 합니다


NIO는 성능상 이점이 크지만, 콜백 기반의 비동기 코드는 복잡하고 디버깅이 어렵다는 단점이 있었습니다

## 가상 스레드 (Virtual Threads)
Java 21의 가상 스레드는 I/O 처리의 패러다임을 바꾸었습니다
가상 스레드를 사용하면, 개발자는 단순한 동기 블로킹 코드를 그대로 작성하면서도, 
내부적으로는 JVM이 I/O 대기 시 해당 스레드를 차단하지 않고 다른 작업을 처리하도록 만들어 
논블로킹과 같은 높은 동시성을 얻을 수 있습니다

Java

```
// 가상 스레드에서는 이렇게 단순한 코드로도 수만 개의 동시 요청을 처리할 수 있다.
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100_000; i++) {
        executor.submit(() -> {
            // 간단한 동기 블로킹 I/O 코드
            URL url = new URL("https://example.com");
            try (InputStream in = url.openStream()) {
                byte[] data = in.readAllBytes();
                // ...
            }
        });
    }
}
```

이제 복잡한 NIO 셀렉터 프로그래밍 없이도, 읽기 쉬운 코드로 높은 I/O 처리를 할 수 있게 되었습니다 
가상 스레드는 I/O 대기 시간을 없애는 것이 아니라, 
그 시간에 다른 일을 할 수 있게 만들어 시스템 자원 활용률을 극대화하는 방식입니다

# `DirectByteBuffer`의 올바른 사용법
NIO의 `ByteBuffer` 중 `allocateDirect()`로 생성하는 **DirectByteBuffer**는 JVM 힙이 아닌 OS 네이티브 메모리를 사용합니다 
이는 OS의 I/O 작업 시 데이터 복사를 줄여주어 성능에 유리하지만, 
잘못 사용하면 오히려 좋지 않은 결과를 유발할 수 있습니다

## 잦은 할당으로 인한 `System.gc()` 유발
`DirectByteBuffer`를 반복문 안에서 계속 `allocateDirect()`로 생성하면,
네이티브 메모리가 한계에 도달했을 때 JVM은 메모리 확보를 위해 내부적으로 `System.gc()`를 호출하여 **Full GC를 강제로 유발**합니다 
이는 애플리케이션 전체를 멈추게(Stop-the-world) 만들어 심각한 성능 저하를 일으킵니다

## 해결 방안: 재사용과 풀링(Pooling)
DirectByteBuffer는 생성 비용이 비싸므로 한 번 생성해서 재사용하는 것이 원칙입니다

애플리케이션 전역에서 사용할 버퍼를 미리 할당해두고 돌려쓰거나,
Netty와 같은 프레임워크가 제공하는 버퍼 풀(Pool)을 사용하는 것이 좋습니다.

# 파일 변경 감지: `lastModified()` 폴링 vs. `WatchService`

파일의 변경 여부를 확인하는 고전적인 방법은 `File.lastModified()`를 주기적으로 호출하는 것입니다.

## **원인: 비효율적인 폴링(Polling)**

`lastModified()` 호출 자체는 매우 빠르지만(, 수많은 파일을 짧은 주기로 계속 확인하는 것은 
**변경이 없더라도 불필요한 디스크 I/O를 지속해서 발생**시킵니다 
이는 리소스 낭비이며, 시스템에 부하를 계속 누적시킵니다.


## **해결 방안: 이벤트 기반의 `WatchService` 활용 (권장)**
Java 7의 NIO.2에 도입된 `WatchService`는 이 문제에 대한 해결책입니다 
OS가 제공하는 파일 시스템 이벤트 알림 기능을 사용하여, 
지정된 디렉터리에 파일 생성, 수정, 삭제 등의 변경이 발생했을 때만 이벤트를 받아 처리할 수 있습니다

```
// WatchService를 사용한 이벤트 기반 파일 변경 감지
Path dir = Paths.get("/path/to/config");
WatchService watcher = FileSystems.getDefault().newWatchService();
dir.register(watcher, StandardWatchEventKinds.ENTRY_MODIFY);

while (true) {
    WatchKey key = watcher.take(); // 변경이 있을 때까지 대기
    for (WatchEvent<?> event : key.pollEvents()) {
        // 파일이 수정되었을 때만 이 로직이 실행됨
        System.out.println(event.context() + " is modified.");
    }
    key.reset();
}
```

이 방식은 CPU와 I/O 자원을 거의 사용하지 않고 대기하다가, 필요할 때만 동작하므로 매우 효율적입니다.


# 참고
- 자바 성능 튜닝 이야기 - 9챕터
