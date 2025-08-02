# 자바 리플렉션 (Reflection)
자바 리플렉션(Reflection)은 JVM에 로드된 클래스의 메타정보(클래스명, 필드, 메서드, 생성자 등)를 
런타임에 동적으로 조회하고 조작할 수 있는 기능입니다 
java.lang.Class 클래스와 java.lang.reflect 패키지(예: Field, Method)의 클래스들을 통해 기능을 활용할 수 있습니다 
리플렉션은 프레임워크 개발, 동적 코드 생성, 디버깅 및 모니터링 등의 상황에서 사용되지만, 
잘못 사용할 경우 성능 저하와 보안 취약점을 발생할 수 있으므로 사용의 주의가 필요합니다 

# 리플렉션 관련 주요 클래스
자바 리플렉션은 클래스 및 그 구성 요소의 메타정보를 나타내는 여러 클래스를 제공합니다
핵심적인 세 가지 클래스는 다음과 같습니다:
## Class 클래스
java.lang.Class는 JVM에 로드된 클래스 또는 인터페이스 자체를 표현하는 메타클래스입니다
모든 자바 객체는 런타임에 자신의 Class 객체를 가지고 있으며,
Object.getClass() 메소드를 통해 정보를 얻을 수 있습니다

### 주요 메소드
- String getName(): 클래스의 패키지명을 포함한 이름을 반환합니다 `java.util.ArrayList`
- String getSimpleName(): 클래스의 단순 이름을 반환합니다. `ArrayList`
- Field[] getFields(): 해당 클래스의 모든 public 필드를 반환합니다 (상속받은 필드 포함).
- Field[] getDeclaredFields(): 해당 클래스에 선언된 모든 필드(private/protected 포함, 상속 제외)를 반환합니다.
- Method[] getMethods(): 해당 클래스의 모든 public 메서드를 반환합니다 (상속받은 메소드 포함)
- Method[] getDeclaredMethods(): 해당 클래스에 선언된 모든 메소드(private/protected 포함, 상속 제외)를 반환합니다

### 특징
Class 객체는 JVM에 클래스가 로드될 때 한 번 생성되어 유지되는 싱글톤 객체입니다
obj.getClass()를 호출할 때마다 새로운 객체가 만들어지는 것이 아니라, 
해당 객체의 클래스 메타객체를 참조로 돌려주는 것입니다


# Method 클래스
java.lang.reflect.Method는 클래스의 메소드 정보를 표현하는 객체입니다. 
Class.getMethods() 또는 Class.getDeclaredMethod()를 통해 정보를 얻을 수 있습니다

## 주요 메소드 
- Class getDeclaringClass(): 메소드가 선언된 클래스의 Class 객체를 반환합니다
- Class getReturnType(): 메소드의 반환 타입을 Class 객체로 반환합니다
- Class[] getParameterTypes(): 메소드의 매개변수 타입 목록을 Class 배열로 반환합니다.
- Object invoke(Object obj, Object... args): 해당 메소드를 지정된 객체(obj)에 대해 호출합니다. 

## 특징 
JUnit 같은 테스트 프레임워크나 ORM 라이브러리에서 동적으로 메소드를 호출하는 데 사용됩니다

# Field 클래스
java.lang.reflect.Field는 클래스의 멤버 변수(필드) 정보를 표현하는 객체입니다 
Class.getField() 또는 Class.getDeclaredFields() 등을 통해 정보를 얻을 수 있습니다

## 주요 메소드
- String getName() 필드 이름을 반환합니다
- Class getType(): 필드 타입의 Class 객체를 반환합니다

## 특징 
JSON 직렬화/역직렬화 라이브러리, 의존성 주입(DI) 프레임워크 등에서 런타임에 객체의 속성 값을 읽거나 수정하는 데 활용됩니다

## 주의할 점
Field.set(obj, value) 등을 통해 private 필드를 수정하는 행위는 캡슐화를 깨뜨리므로, 
특별한 경우가 아니면 권장되지 않습니다
Java 9+에서는 다른 모듈의 private 필드 접근은 기본적으로 금지되어 있습니다
그 이전에는 Field.set을 통해서 수정을 하면 경고 문구가 출력되는 정도였다면, 
해당 버전 이상부터는 다른 모듈일 경우 IllegalAccessException 예외를 통해 접근을 차단해버립니다

# 리플렉션 클래스를 잘못 사용하는 경우
객체의 클래스 이름을 문자열로 얻어 비교하려고 할 때, 
리플렉션의 getName() 메소드를 사용한다면 호출 시마다 문자열 객체가 새로 생성되고 비교하는 과정이 발생합니다
`src.getClass().getName()`
이것 자체는 큰 문제가 되지 않지만, 
다음과 같이 다른 문자열을 비교할 경우 불필요한 성능저하가 발생할 수 있습니다
`src.getClass().getName().equls("java.util.ArrayList")`
이 조건문이 반복문에서 동작한다면, 큰 차이는 발생하지만 `instanceof`를 사용할 때보다는 느려집니다
`src instanceof java.util.ArrayList`
책에서는 대략 6배 정도의 차이가 발생한다고는 했는데 초 단위가 마이크로 초 단위라서 
성능의 큰 영향을 주는 정도는 아닙니다

하지만 그럼에도 이런부분이 모여 큰 차이를 발생할 수는 있기 때문에 지양하는 것이 좋습니다

### 왜 이런 차이를 발생시킬까?
instanceof 연산자는 JVM 수준에서 최적화된 바이트코드로 수행되므로 
"getClass().getName()처럼 별도 객체 생성이나 문자열 비교를 하지 않기 떄문에 훨씬 효율적입니다

# 클래스 메타정보 저장 영역, PermGen과 Metaspace

자바의 클래스 메타정보는 JVM의 특정 메모리 영역에 저장됩니다
## PermGen
Java 7까지는 JVM Heap 내의 PermGen(Permanent Generation)이라는 고정 크기 영역에 클래스 메타데이터를 저장했습니다
PermGen은 크기가 고정되어 있어, 동적 클래스 로딩이 잦거나 대규모 애플리케이션 서버 환경에서 `OutOfMemoryError: PermGen space` 오류가 자주 발생했습니다

## Metaspace
Java 8부터 PermGen이 사라지고 Metaspace라는 새로운 영역으로 대체되었습니다
Metaspace는 JVM 힙 바깥의 네이티브(native) 메모리 영역에 할당되며, 
기본적으로 필요 시 자동으로 크기를 확장합니다
`-XX:MaxMetaspaceSize` 옵션으로 최대 크기를 제한할 수 있지만, 
기본 설정에서는 PermGen 시절처럼 OOM이 빈번하게 발생하지 않습니다

# Java 21기준 최신 리플렉션 정보
Java 18에서는 `Method.invoke()`, `Constructor.newInstance()`, `Field.get/set()` 등의 
핵심 리플렉션 구현이 MethodHandle 기반으로 재구현되었습니다
리플렉션 객체를 static final로 보관하여 상수로 취급될 경우,
JIT 최적화를 통해 기존 대비 약 43~57% 빨라졌습니다.

## 활용처
프레임워크의 활용: Spring과 같은 프레임워크에서 리플렉션을 활용하는데, 
성능 저하를 완화하기 위해 메타정보 캐싱이나 바이트코드 생성을 통한 프록시 호출(CGLIB)을 사용합니다

# 정리

자바 리플렉션은 런타임 시의 프로그램의 구조를 탐색하고 조작할 수 있게 해주는 도구입니다 Class, Method, Field 등의 클래스를 통해 손쉽게 구현할 수 있습니다

AOP와 같은 기능을 이해하거나 프레임워크를 개발하지 않는다면 자주 사용하는 일은 없겠지만
사용한다면 아래 경우를 고려해서 사용해야합니다

1. 빈번한 호출 경로에 리플렉션을 사용하는 것은 피해야 하며,
	instanceof와 같은 기본 언어 기능같은 대안을 사용해야합니다
2. setAccessible(true)를 통한 private 요소 접근은 캡슐화를 깨뜨리고, 
	Java 9 이후 다른 모듈에서의 접근이 제한되므로 조심해서 사용해야 합니다


# 참고 
- 자바 성능 튜닝 이야기 - 7챕터
