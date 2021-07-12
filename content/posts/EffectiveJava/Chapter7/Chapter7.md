---
title: "[이펙티브 자바 요약] Chapter7 - Lambda와 Stream"

date: "2020-07-07"

tags:

- 이펙티브자바

- effectivejava

- stream

- chapter7

- labdma

---

 이 글은 'Effective Java 3판'(출판사 : 프로그래밍 인사이트, 저자 : Joshua Bloch, 역자 : 개앞맵시(이복연)) 을 읽고 내용을 정리한 글입니다. 문제가 될 시 삭제하겠습니다.

## Item 42. 익명 클래스 보다는 람다를 사용하자 

예전 자바에서는 함수 타입을 표현할 때 함수 객체(function object, 추상 메서드를 하나만 담은 인터페이스)를 사용하며 특정 함수나 동작을 표현하는데 사용하였다. 그후에 함수 객체를 만드는 주요 수단으로 익명 클래스(anonymous class)가 도입되었다. 

```
<figcaption style="text-align:center; font-size:15px;>
    익명 클래스 사용 예
  </figcaption>
```

```java
Collections.sort(words, new Comparator<String>(){
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});

```

## Item 43. 람다보다는 메서드 참조를 사용하자

 