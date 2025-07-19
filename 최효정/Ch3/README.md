# Ch3. 왜 자꾸 String을 쓰지 말라는거야?

# String을 쓰지 말라는 이유

## String 사용한 쿼리문

```sql
String strSQL = "";
strSQL += "select * ";
strSQL += "from ( ";
strSQL += "select A_column, ";
strSQL += "B_column, ";
// 중간 생략(약 400라인)
...
```

- String을 사용한 쿼리문 실행했을 시에 메모리 사용량

| 구분 | 결과 |
| --- | --- |
| 메모리 사용량 | 10회 평균 약 5MB |
| 응답 시간 | 10회 평균 약 5ms |

## StringBuilder 쿼리문

```sql
StringBuilder strSQL = new StringBuilder();
strSQL.append(" select * ");
strSQL.append(" from ( ");
strSQL.append(" select A_column, ");
strSQL.append(" B_column, ");
// 중간 생략(약 400라인)...
```

- 메모리 사용량

| 구분 | 결과 |
| --- | --- |
| 메모리 사용량 | 10회 평균 약 371KB |
| 응답 시간 | 10회 평균 약 0.3ms |

# StringBuffer 클래스, StringBuilder 클래스

- 제공하는 메서드는 동일함

## 📍 StringBuffer 클래스

- 멀티스레드 환경에서 사용 가능
    - 여러 개의 스레드에서 하나의 StringBuffer 객체를 처리해도 전혀 문제가 되지 않음
    - Synchronized(동기화) 연산 적용되어있음 → 비교적 느림

## 📍StringBuilder

- 단일 스레드에서의 안전성만을 보장한다.
    - 여러 개의 스레드에서 하나의 StringBuilder 객체를 처리하면 문제 발생
- 대신 비교적 빠름

# String vs StringBuffer vs StringBuilder

- 셋 중 가장 빠른 것은?
- 10,000회 반복하여 문자열을 더하고, 이러한 작업을 10회 반복한다.

### 프로파일링 툴의 결과

| 주요 소스 부분 | 응답 시간(ms) | 비고 |
| --- | --- | --- |
| a+=aValue; | 95,801.41ms | 95초 |
| b.append(aValue); | 247.48ms | 0.24초 |
| c.append(aValue); | 174.17ms | 0.17초 |

### 메모리 사용량

| 주요 소스 부분 | 메모리 사용량(bytes) | 생성된 임시 객체수 | 비고 |
| --- | --- | --- | --- |
| a+=aValue; | 100,102,000,000 | 4,000,000 | 약 95Gb |
| b.append(aValue); | 29,493,600 | 1,200 | 약 28Mb |
| c.append(aValue); | 29,493,600 | 1,200 | 약 28Mb |

## 결과

### 📍응답시간

- String < StringBuffer < StringBuilder 순으로 빠름

### 📍메모리

- Sting > StringBuffer, StringBuilder 순으로 메모리를 더 사용함

## 이유

### 1️⃣ String

`a += aValue; // a = a + aValue` 

> *주소= **100** [abcde]
주소= **150** [abcde] [abcde]
주소= **200** [abcde] [abcde] [abcde]*
> 

- a에 aValue를 더하면 **새로운** String 클래스의 **객체가 만들어짐**
- 이전에 있던 a 객체는 필요 없는 **쓰레기 값이 되어** GC 대상이 되어 버린다.
- 반복되면 메모리를 많이 사용→ 응답 속도 느려짐
- GC를 하면 할수록 시스템의 CPU 소모가 크고 시간도 많이 소요됨

### 2️⃣ StringBuffer, StringBuilder

> *주소= **100** [abcde]
주소= **100** [abcde] [abcde]
주소= **100** [abcde] [abcde] [abcde]*
> 
- 새로운 객체를 생성하지 않음
- 기존의 객체의 크기를 증가시키면서 값을 더함

# String 사용 구분

**`String`**

- 짧은 문자열을 더할 경우

**`StringBuffer`**

- 스레드에 안전한 프로그램이 필요할 때
- 클래스에 static으로 선언된 문자열을 변경
- singleton으로 선언된 클래스에 선언된 문자열일 경우

**`StringBuilder`**

- 스레드에 안전한지의 여부와 전혀 관계 없는 프로그램을 개발할 때
- 메서드 내에 변수를 선언했다면, 해당 변수는 그 메서드 내에서만 살아있으므로, StringBuilder를 사용하면 됨