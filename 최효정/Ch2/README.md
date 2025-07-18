# Ch2. 내가 만든 프로그램의 속도를 알고싶다

# 프로그램의 속도를 알고 싶을 땐?

시스템의 성능이 느릴 때 가장 먼저 해야 하는 작업은 병목 지역을 파악하는 것이다.

애플리케이션의 속도에 문제가 있을 때 분석 프로그램을 사용하면 병목 지점을 파악할 수 있다. 

대표적으로 프로파일링 툴, APM 툴이 있다

# 프로파일링 툴 vs APM

- 시스템 문제 분석 툴
- 자주 사용하는 툴로는 APM(Application Performance Monitoring / Management)이 있다.

## 프로파일링 툴

> **개발자용 툴**
주로 느린 메서드, 느린 클래스를 찾는 것을 목적으로 함
> 
- 소스 레벨의 분석을 위한 툴
- 애플리케이션의 세부 응답 시간까지 분석할 수 있다
- 메모리 사용량을 객체나 클래스, 소스의 라인 단위까지 분석할 수 있다.
- 가격이 APM 툴에 비해서 저렴하다
- 보통 사용자수 기반으로 가격이 정해진다
- 자바 기반의 플라이언트 프로그램 분석을 **할 수 있다.**

## APM 툴

> **운영 환경용 툴**
목적에 따라 용도가 상이함
> 
- 애플리케이션의 장애 상황에 대한 모니터링 및 문제점 진단이 주 목적이다.
- 서버의 사용자 수나 리소스에 대한 모니터링을 할 수 있다
- 실시간 모니터링을 위한 툴이다
- 가격이 프로파일링 툴에 비하여 비싸다
- 보통 CPU 수를 기반으로 가격이 정해진다
- 자바 기반의 클라이언트 프로그램 분석이 **불가능**하다

## 프로파일링 툴이 기본적으로 제공하는 기능

> 응답 시간 프로파일링과 메모리 프로파일링 기능을 기본적으로 제공함
> 

## 응답 시간 프로파일링 기능

- 주로 응답 시간을 측정하기 위해 사용
- 하나의 클래스 내에서 사용되는 메서드 단위의 응답 시간 측정
- (툴에 따라) 소스 라인 단위로 응답 속도 측정 가능
- CPU 시간, 대기 시간 두 가지 시간이 제공됨

<aside>


### **CPU 시간과 대기 시간의 차이**

- 하나의 메서드, 한 라인을 수행하는 데 소요되는 실제 시간
- 실제 소요 시간 = CPU 시간 + 대기 시간

**CPU 시간**

- CPU를 점유한 시간

**대기 시간**

- CPU를 점유하지 않고 대기하는 시간
</aside>

# 더 간단하게 프로그램 속도를 측정하는 방법

## 1️⃣ System 클래스에서 제공하는 메서드를 활용하기

- 프로그램 시작 전 시간과 끝난 후 시간의 차를 구해 프로그램 실행 시간을 구하기

### currentTimeMillis()

> static long currentTimeMillis()
> 
- 현재의 시간을 ms로 리턴한다(1/1000초)
- ms(밀리초)를 리턴해준다.
- UTC 기준 1970년 1월 1일부터의 시간을 long타입으로 리턴한다. → 호출할 때마다 다름
- ms
    
    ms = 1/1000초
    
    - 20ms = 0.02초
    - 1,200ms = 1.2초
    - 6,000,000ms = 6000초 = 1시간 40분
    

```java
package com.example.tikicktaka.apiPayload.code.status;

import java.util.ArrayList;
import java.util.HashMap;

public class CompareTimer {
    public static void main(String[] args) {
        CompareTimer timer = new CompareTimer();
        for(int loop=0; loop<10; loop++) {
            timer.checkNanoTime();
            timer.checkCurrentTimeMillis();
        }
    }
    
    private DummyData dummy;
    
    public void checkCurrentTimeMillis() {
        long startTime = System.currentTimeMillis();
        //  System.currentTimeMillis()를 사용하여 시작 밀리초를 startTime에 기록한다
        dummy= timeMakeObjects();
        //HashMap, ArrayList를 생성
        long  endTime = System.currentTimeMillis();
        //종료된 시점의 밀리초를 endTime에 할당
        long elapsedTime = endTime-startTime;
        //그 차이 값인 응답 시간을 확인
        System.out.println("milli="+elapsedTime);
    }
    
    public void checkNanoTime() {
        long startTime = System.nanoTime();
        //System.nanoTime으로 측정
        dummy = timeMakeObjects();
        long endTime = System.nanoTime();
        double elapsedTime= (endTime-startTime)/100000.0;
        System.out.println("nano="+elapsedTime);
    }
    
    public DummyData timeMakeObjects(){
        HashMap<String, String> map = new HashMap<String, String> (1000000);
        ArrayList<String> list = new ArrayList<String>(1000000);
        return new DummyData(map,list);
        
    }
    
    public class DummyData{
        HashMap<String,String> map;
        ArrayList<String> list;
        
        public DummyData(HashMap<String, String>map, ArrayList<String> list){
            this.map = map;
            this.list = list;
        }
    }
    
}

```

### nanoTime()

> statice long nanoTime()
> 
- 현재의 시간을 ns로 리턴함 (1/1,000,000,000초)
- JDK 5.0부터 추가된 메서드

## 2️⃣ 전문 라이브러리 이용(JMH)

> JMH(Java Microbenchmark Harness) : JDK를 오픈 소스로 제공하는 OpenJDK에서 만든 성능 측정용 라이브러리
>