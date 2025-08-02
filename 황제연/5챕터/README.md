# 최신 switch 문에 대한 간략 소개

## Java 8버전의 switch문 
```
int day = 3;
String name;
switch (day){
	case 1:
		name = "Mon";
		break;
	case 2:
		name = "Tue";
		break;
	...
}

```

break문은 필수로 사용해야하며, 값을 직접 반환할 수 없습니다
## 14버전 이후 switch문
```
int day = 3;
String name = switch (day){
	case 1 -> "Mon";
	case 2 -> "Tue";
	....
}
```
arrow label이 등장하며, break문 사용이 필수가 아니고, 값을 직접 반환할 수 있습니다

```
int day = 3;
String name = switch (day){
	case 1 -> "Mon";
	case 2 -> "Tue";
	case 9, 10, 11 -> {
		doEffect();
	}
}
```
위와같이 다중 케이스를 묶어서 처리할 수도 있습니다

```
int day = 3;
String name = switch (day){
	case 1 -> "Mon";
	case 2 -> "Tue";
	case 9, 10, 11 -> {
		int k = day.toString();
		yield k;
	}
}
```
블록을 지정해서 단일 수행이 아닌 다중 수행도 가능하며, 
yield를 선언해서 return과 동일하게 활용할 수 있습니다

다만 yield 키워드는 블록 내부에서만 사용할 수 있습니다

```
int volatile = 3
int yield = 3;
```
일반적으로 예약어는 변수명으로 사용이 불가능하지만 yield는 사용 가능합니다!


# for 루프의 성능 문제와 개선 방법
##  size() 메서드 반복 호출 문제 
컬렉션을 반복할 때 루프 조건에서 list.size()와 같은 size() 메서드를 직접 호출하면, 반복할 때마다 해당 메서드가 계속 실행되어 성능 저하를 초래할 수 있습니다

 이 문제로 성능 저하가 발생한다면 컬렉션 크기를 루프 시작 전에 한 번만 계산하여 변수에 저장하고 사용하면 됩니다

 컬렉션의 내용을 배열로 가져오는 toArray() 메서드를 루프 안에서 매번 호출하는 것도 성능 오버헤드를 발생시킵니다. 
 매 반복마다 불필요한 배열 생성 및 복사가 발생하기 때문입니다.

이것 역시 루프 전에 toArray()를 한 번만 호출하여 결과를 캐싱하고, size도 미리 저장하여 사용하면 됩니다

# for-each vs 전통적인 for문
for-each문은 Java 5부터 도입되었으며, 컬렉션을 순회하는 간결한 문법을 제공합니다 
내부적으로 Iterator를 사용하여 순회합니다
## 장점 
- 코드 가독성과 편의성이 높습니다.

## 단점 
Iterator 생성으로 인한 약간의 오버헤드가 있을 수 있습니다 
인덱스 제어나 역순 반복 등에는 사용할 수 없습니다.

## 전통적인 for문방식 
평소 사용하는 for문 방식입니다 `for(int i=0; i<length; i++){...}`
for-each보다는 약간 더 빠를 수 있는데... 사실상 마이크로 초 수준이라서... 큰 의미는 없습니다

차라리 성능 차이보다 코드 가독성과 안전성이 중요하므로, 컬렉션을 처음부터 끝까지 순회할 때는 for-each문을 많이 사용합니다 
다만, 성능이 특히 중요한 경우에는 상황에 맞게 전통적인 for문을 사용하는 방식도 쓰거나 람다 스트림을 선택하는 방식을 고려해볼만 합니다

# Java 8 이후의 반복문 방법 정리

## Collection.forEach() 메서드
Java 8부터 도입된 Iterable 컬렉션의 forEach 메소드입니다

많은 컬렉션 구현체(예: ArrayList.forEach())에서 내부적으로 전통적인 루프로 구현되어
Iterator 생성 없이 빠르게 동작합니다

람다를 활용한 간결한 코드 작성이 가능하며, 
JIT 컴파일러가 잘 최적화해주므로 성능상 손실 없이 깔끔한 반복 처리를 구현할 수 있습니다

## Iterator 객체 내부구조
```
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```
내부적으로 for-each로 동작하고 있습니다
Iterator 객체를 생성한다는 것은 ArrayList든 LinkedList든 결국, `hasNext()`나 `next()`를 호출해서
처음부터 모두 탐색하기 때문에 ArrayList를 조회로 사용할 때의 이점을 얻을 수 없습니다

## ArrayList.forEach 내부구조
```
@Override
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    @SuppressWarnings("unchecked")
    final E[] elementData = (E[]) this.elementData;
    final int size = this.size;
    for (int i = 0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```
내부 구조는 Iterator 생성 없이 동작하고 있습니다
그렇기에 앞선 오버헤드의 문제를 고민하지 않아도 됩니다

## 사용법
```
List<String> list = Arrays.asList("Java", "Kotlin", "Scala");
list.forEach(item -> System.out.println("람다: " + item));

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream()
    .filter(n -> n.length() <= 3)
    .forEach(n -> System.out.println("짧은 이름: " + n));
```


## 스트림 (Stream)
Java 8의 Stream API는 함수 조합 방식으로 `stream()`으로 스트림을 만들고 `filter`·`map` 같은 중간연산을 연결한 뒤 `forEach`·`collect` 등 최종연산으로 한 번에 처리해, 간결하면서도 효율적으로 반복 작업을 수행합니다

## 사용법
```
List<String> words = Arrays.asList("Java", "Kotlin", "Scala", "Python", "Go");

        // - 짧은 단어(길이 ≤ 4)만 필터링
        // - 모두 대문자로 변환
        // - 결과를 리스트로 수집
List<String> shortUpper = words.stream()
                 .filter(w -> w.length() <= 4) // 필터: 조건문 역할
                 .collect(Collectors.toList()); // 수집: 리스트로 조회한 문자열 반환

// Stream을 활용한 그룹화
Map<Integer, List<String>> groupedByLength =
            words.stream()
                 .collect(Collectors.groupingBy(String::length));
```

## JAVA Stream API 학습 도움 사이트
 - [Java Stream API](https://www.baeldung.com/java-8-streams)
 

# 참고
- 자바 성능 튜닝 이야기 - 5챕터
