# 🗂️ 4장: 어디에 담아야 하는지 (Java Collection Framework)

> 데이터의 특징(중복, 순서, 키-값)에 따라 가장 효율적인 저장소를 선택하는 방법을 알아봅니다.

---

## 🏛️ 1. Collection 인터페이스 톺아보기

Java의 Collection Framework는 크게 `Set`, `List`, `Map`이라는 세 가지 핵심 인터페이스로 나뉩니다.

### `Set<E>` - 순서 없고, 중복도 없는 데이터의 집합

- **`HashSet<E>`**: **순서를 보장하지 않는** 가장 기본적인 Set. 성능이 우수합니다.
- **`LinkedHashSet<E>`**: **삽입된 순서**를 기억하는 Set.
- **`TreeSet<E>`**: 데이터를 **정렬된 상태**로 유지하는 Set. 내부적으로 레드-블랙 트리(Red-Black Tree)로 구현되어 있습니다.

### `List<E>` - 순서 있고, 중복도 있는 데이터의 목록

- **`ArrayList<E>`**: 내부적으로 배열로 구현되어 **조회 성능**이 빠릅니다. (동기화 X)
- **`LinkedList<E>`**: 데이터 추가/삭제 시 성능이 뛰어납니다. `Queue` 인터페이스의 구현체이기도 합니다.
- **`Vector<E>`**: `ArrayList`와 유사하지만, 모든 메서드가 동기화(Thread-Safe) 처리되어 있습니다. (현재는 잘 사용되지 않음)

### `Map<K, V>` - Key와 Value 쌍으로 이루어진 데이터의 모음

- **`HashMap<K, V>`**: **순서를 보장하지 않는** 가장 일반적인 Map. (동기화 X)
- **`LinkedHashMap<K, V>`**: **삽입된 순서**를 기억하는 Map.
- **`TreeMap<K, V>`**: **Key를 기준으로 정렬된 상태**를 유지하는 Map. (레드-블랙 트리)
- **`Hashtable<K, V>`**: `HashMap`과 유사하지만, 모든 메서드가 동기화(Thread-Safe) 처리되어 있습니다. (현재는 잘 사용되지 않음)

---

## ⛓️ 2. 동기화 (Synchronization) 이슈

멀티 스레드 환경에서는 데이터의 일관성을 위해 동기화 처리가 필요할 수 있습니다.

- **레거시 클래스: `Vector`, `Hashtable`**
  - JDK 1.0부터 존재했던 클래스로, 자체적으로 동기화를 지원합니다.
  - 하지만 지금은 더 나은 대안이 많아 **새로운 코드에서는 사용을 권장하지 않습니다.**
- **`Collections`의 동기화 래퍼(Wrapper)**
  - `ArrayList`, `HashMap` 등 기본 컬렉션을 스레드에 안전한 컬렉션으로 만들고 싶을 때 사용합니다.
  - `Collections.synchronizedList(new ArrayList<>());`
  - `Collections.synchronizedMap(new HashMap<>());`
  - `Collections.synchronizedSet(new HashSet<>());`

> 🚀 **Pro Tip!**
>
> 더 높은 수준의 동시성 처리가 필요하다면 java.util.concurrent 패키지의 ConcurrentHashMap, CopyOnWriteArrayList 등의 클래스를 사용하는 것이 훨씬 효율적입니다.

---

## ✅ 3. 그래서 뭘 써야 할까? (일반적인 선택)

특별한 요구사항이 없다면, 대부분의 경우 아래의 조합으로 시작하는 것이 좋습니다.

| 인터페이스  | 추천 클래스  |
| ----------- | ------------ |
| **`Set`**   | `HashSet`    |
| **`List`**  | `ArrayList`  |
| **`Map`**   | `HashMap`    |
| **`Queue`** | `LinkedList` |
