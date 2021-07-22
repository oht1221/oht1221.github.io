---
title: "[이펙티브 자바 요약] Chapter7 - Lambda와 Stream"

date: "2020-07-12"

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

 익명 클래스 방식이 코드가 너무 길기 때문에 자바가 함수형 프로그래밍에 적합하지 않았고, 자바 8에서는 함수형 인터페이스(명칭은 거창하지만 사실은 추상메서드 하나 짜리인 인터페이스)라는 것을 도입했고, 이 함수형 인터페이스의 인스턴스는 람다를 통해서 만들 수 있게 되었다. 사실상 람다식이 함수형 인터페이스의 생성자 중 하나인 것이다. 위 코드를 람다를 사용하면 아래와 같이 된다.

 ``` java
 Collections.sort(words,
                 (s1, s2) -> Integer.compare(s1.length(), s2.length()));
 ```

  람다를 사용할 때에는 타입을 명시해야 의미가 더 명확한 경우를 제외하고는 매개변수 타입은 생략하고 차후에 컴파일러가 타입 추론에 실패해 에러를 일으키면 명시해주자. 참고로 람다 자리에 비교자 생성 메서드를 이용하여 함수 참조를 사용하면 더 깔끔해진다.

 동작을 나타내는 모든 경우에 람다를 쓰면 될 것 같지만, 그렇지 않은 경우도 있다. 람다를 사용했을 때 코드가 세 줄 이상 넘어가거나, 코드 자체로 동작이 명확히 설명되지 않는 경우에는 람다를 사용하지 않는 것이 좋다. 또한 넘겨진 동작 매개변수는 인스턴스 맴버를 참조할 수 없기 때문에 각 인스턴스의 멤버 변수와 상호작용하는 경우 람다 뿐 아니라 동작을 매개변수로 넘기지 못한다.

 마지막으로, 람다는 자기 자신을 참조할 수 없기 때문에(람다에서 this는 바깥 클래스를 의미한다) 함수 객체가 자기 자신을 참조해야한다면 익명 클래스를 사용해야한다.



## Item 44. 표준 함수형 인터페이스를 사용하자

자바 8의 특징을 충분히 살려 프로그래밍을 하면 함수형 매개변수를 쓸 일이 많은데,  필요한 용도에 맞는게 있다면 이 때 자바에서 정의한 표준 함수형 인터페이스를 사용하는 것이 좋다.  자바 8에는 43가지의 표준 함수형 인터페이스들이 정의돼있고 많은 경우 이 표준형 인터페이스로 표현할 수 있는 함수 매개변수이다.

 함수형 인터페이스 대부분은 기본 타입만 지원하는데, 물론 박싱 타입을 넣어도 동작은 권장하지 않는다. 또한 대부분의 경우 표준을 사용하는게 좋지만, 구조적으로는 똑같은 표준 함수형 인터페이스가 있더라도 직접 구현하는게 좋은 경우가 있는데 ToIntBiFunction\<T, U\>와 구조적으로 동일한 Comparator\<T\>가 대표적이다. Compartor는 자주 쓰이며 반드시 따라야하는 규약이 있고, 유용한 디폴트 메서드들은 많이 제공하는데 이러한 특징을 가지게 될 경우 직접 함수형 인터페이스를 구현하는게 좋을 수 있다.



## Item 43. 람다보다는 메서드 참조를 사용하자

 람다는 익명 클래스보다 간결하다. 하지만 그 람다보다도 더 간결한 것이 메서드 참조이다. 메서드 참조는 함수의 매개변수가 많을 수록 줄일 수 있는 코드 양도 늘어난다. 하지만 경우에 따라 람다로 표현하는 것이 더 읽기 쉽고 유지 보수에 좋은 경우(매개변수의 이름이 의미가 명확하다던지)도 있으니 잘 구분해 사용해야한다.

 마지막으로, 람다로는 할 수 없는 것이 한가지 있는데 제네릭 함수를 표현하는 것이다. 제네릭 람다식이라는 문법 자체가 존재하지 않기 때문에 제니릭 함수 타입은 메서드 참조로만 받을 수 있다.





## Item 45. Stream은 주의해서 사용하자

 스트림을 언제 써야하는지에 대한 명확한 규정이 있지는 않지만 참고할만한 노하우는 있다. 다음의 예는 같은 아나그램으로 구성된 단어들의 set 중에서 특정 개수 이상의 set만 출력하는 코드인데 스트림을 이용하여 가독성 좋은 코드로 바꿀 수 있다.

``` java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computIfAbsent(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
            }
        }
        
        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize) 
                System.out.println(group.size()) + ": " + group);
    }
    
    private static String alphabetize(String s) {
        char[] = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```



```
public 
```

 스트림을 너무 과하게 사용해도 가독성이 떨어지므로, 적절한 스트림 사용으로 가독성을 떨어뜨리지 않는 선에서 코드의 양을 줄이는 것이 중요하다.

 반복문에서는 가능하지만 스트림에서는 불가능한 것도 있는데 예를 들면 다음과 같다.

- final이나 사실상 final인 변수 외에 지역 변수들을 수정하는 것은 불가능하다
- return 문을 이용해서 메서드에서 탈출하거나 break, continue등으로 블록 바깥의 반복문을 종료하거나 건너뛰는 것이 불가능하다. 또한 메서드 선언에 명시된 예외도 터뜨릴 수 있다.

 반대로 다음과 같은 경우에는 스트림이 매우 잘 맞다

- 원소 시퀀스를 일관되게 관리한다.
- 필터링
- 하나의 연산으로 결합
- 컬렉션에 모으기
- 특정 조건을 만족하는 원소 검색

 또한 값을 변경하는 일련의 과정에서 변경 이전의 값을 종단에서 다시 사용하는 경우에도 스트림을 사용하는 것은 불가능하다. 이럴 경우 종단에서 역산을 통해 다시 이전 값을 찾을 수 있는 경우에는 적용해볼 수도 있다.



##  Item 46. 스트림에서는 부작용 없는 함수를 사용하자

  스트림의 핵심은 계산을 일련의 변환(transformation)으로 재구성하는 것이다. 이 때 각 변환은 이전 단계의 결과를 받아 처리하는 순수 함수여야한다. 즉 부작용이 없는 함수여야한 한다.

 ```
 try (Stream<String> words = new Scanner(file).tokens()) {
 	words.forEach(word -> {
 		freq.merge(word.toLowerCase(), 1L, Long::sum);
 	});
 }
 ```

 위 예제 코드는 스트림을 가장한 반복문 코드나 다름 없다. 전혀 스트림의 장점을 살리지 못하며 반복문보다 길고 유지보수도 어렵다.  기본적으로 forEach는 스트림 종단 연산 중에서 기능도 제일 적고 기본적으로 반복이라 병렬화도 할 수 없다. 따라서 forEach는 스트림 계산 결과를 보고하는 경우에만 사용하는 것이 좋다.(물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 용도로 사용할 수는 있다.) 위 코드는 아래와 같이 바꾸는 것이 훨씬 스트림 스럽다.

``` java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

 위 코드의 collector는 스트림을 잘 사용하기 위해 꼭 이해해야하는데, 기본적으로는 reducer의 대용이라 생각하는 것이 쉽다. 이 collector가 생성하는 객체는 일반적으로 collection이다.

 collector를 toList(), toSet(), toCollection(collectionFactory)의 총 세 가지이며 스트림 원소를 손 쉽게 collection으로 모을 수 있다.



## Item.47 반환 타입은 스트림보다 컬렉션이 좋다

 원소 시퀀스를 반환할 때에는, 이를 스트림으로 처리해야하는 경우나 반복으로 처리해야하는 경우 모두 고려하는 것이 좋다. 이 경우 컬렉션을 반환하는 편이 좋은데, 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 collection에 담아 반환하는 편이 낫다. 

 하지만, 경우에 따라 컬력센 반환이 불가능한 경우가 있다. 이럴 경우에는 스트림과 Iterable중 더 자연스러운 것을 반환하면 된다. 



## Item.48 스트림 병렬화는 주의해서 적용하자

- 자바의 병렬 처리 자체를 먼저 공부하고 다시 정리...

