---

title: "[이펙티브 자바 요약] Chapter6 - Enum과 Annotation"

date: "2020-07-07"

tags:

- 이펙티브자바

- effectivejava

- enumeration

- chapter6

- annotation

---

 이 글은 'Effective Java 3판'(출판사 : 프로그래밍 인사이트, 저자 : Joshua Bloch, 역자 : 개앞맵시(이복연)) 을 읽고 내용을 정리한 글입니다. 문제가 될 시 삭제하겠습니다.



## Item 34. int 상수 대신에 enum 타입을 사용하자

- 열거 타입을 사용하면 static final 정수를 나열하는 것 보다 타입 안전하고, 또 class의 일종인 enum 타입이기 때문에 인스턴스 필드를 활용할 수도 있다.
- 상수별로 다르게 동작해야하는 함수가 있다면 상수별 메서드 정의를 이용하면 되고, 이 경우 switch문을 통해 타입별 메서드의 로직을 정의하는 것보다 훨씬 더 안전하고 깔끔하게 구현할 수 있다.(switch문의 경우 상수 추가 시에 그에 맞춰 case 지정을 안해주면 예상치 못한 액션을 수행)
- 만약 열거 타입 상수 중 일부가 동작을 공유하면 전략 열거 타입 패턴을 사용하자!(private한 nested enum 타입 활용)

## Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하자

- ordinal은 사용하지 않는 편이 낫다. ordinal을 이용해 어떤 값을 추정해야한다면 차라리 enum 타입의 인스턴스에 필드를 지정하여 그 값을 넣자.

## Item 36. 비트 필드 대신 EnumSet을 사용하자

``` java
public class Text {
    public static final inst STYLE_BOLD = 1 << 0; // 1
    public static final inst STYLE_ITALIC = 1 << 1; // 2
    public static final inst STYLE_UNDERLINE = 1 << 2; // 4
    public static final inst STYLE_STRIKETHROUGH = 1 << 3; // 8
    
    public void applyStyles(int styled) { ... }
}
```

- 위 방법은 옛 방식으로 비트 필드를 열거한 상수들이다. (4bit 표현) 위 방법으로 합집합이나 교집합 같은 연산을 효율 적으로 할 수 있다.(비트 연산) 이 방법은 이 경우 몇 비트가 필요할 지 미리 알아야 하는 등의 단점이 있다. 
- EnumSet을 사용하면 훨신 더 간단하게 이런 비트필드를 표한할 수 있는데, 쉽게 이해하면 **특정 열거타입 상수만으로 구성된 Set**을 만드는 것이다. 이렇게 되면 Type안전하게 만들 수 있고 Set 인터페이스를 완벽하게 구현하기 때문에 다른 Set 구현체와도 함께 사용이 가능하다. 내부는 bit vector로 구현되기 때문에 비트필드와 비견되는 성능을 나타낸다. 

## Item 37. ordinal 인덱싱 대신 EnumMap을 사용하자

- EnumMap역시 쉽게 생각하면, 특정 Enum 타입의 상수들만을 Key로 하는 Map으로, Map inteface의 구현체이다.
- Ordinal을 배열 순회의 인덱스로 사용하는 경우 인덱스 값의 의미( 사실은 Enum의 각 상수를 표현하고 싶은 것)를 모르게 된다. 이 경우 Index 범위 에러가 나거나 최악의 경우 문제 없이 실행되고 이상한 결과를 내놓게 된다. 이 경우 출력시에 인덱스가 무얼 의미하는지 레이블도 달아야 하고 정확학 정숫값(인덱스로 사용할 수 있는)을 사용하지도 사용자가 보장해야한다. 정수 타입은 타입 안전하지 않기 때문이다.
- 내부적으로는 배열을 사용하므로 성능 또한 ordinal을 사용한 배열에 비견된다. 
- Stream을 이용하여 아래와 같이 구현이 가능하다. 이 경우 출력 시 따로 레이블링도 필요 없이 Map의 toString 메서드가 알아서 잘 구성된 문자열을 출력해준다. 또한 아래의 경우 만약 garden 에 

``` java
class Plant {
	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }    
    
    final String name;
    final LifeCycle lifeCycle;
   
    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }
    
    @Override public String toString() {
        return name;
    }
}

System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle,
                                                            () -> new EnumMap<>(LifeCycle.class),
                                                            toSet());
```



## Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하자

- 열거 타입의 단점 중 하나가 바로 확장할 수 없다는 점이다. 다시 말해, 열거한 값들을 그대로 가져와서 값을 더 추가하여 다른 목적으로 사용이 불가능하다는 것이다. 사실 이 생각 자체가 별로 좋은생각은 아니다. 확장한 타입은 기반 타입의 원소로 인정되는 반면 기반 타입은 확장한 타입의 원소로 취급이 안되니까.
- 연산 코드와 같이 가끔 열거 타입의 확장이 유용한 경우가 있는데, 이 경우 공통 interface를 정의하고 그 interface를 implements하는 열거 타입을을 정의하는 방법을 사용할 수 있다. 이 경우 열거 타입끼리는 구현을 상속할 수 없다.

 ## Item 39. 명명패턴보다 annotation을 사용하자



## Item 40. @Override anntation은 일관되게 사용하자

- 상위클래스의 메서드를 재정의하는 경우에는 항상 @OVerride를 사용하자. 그래야 시그니처를 부정확하게 사용했을 경우 컴파일 타임에서 에러를 잡을 수 있다. (실수하면 overloading되어 컴파일은 정상적으로 진행됨)
- 예외는 추상 메서드를 재정의하는 경우이다. 이 경우 추상 메서드를 구현 안하면 컴파일 에러가 나기 때문에 런타임 전에 잡을 수 있다.
- 대표적인 경우 Object의 equals를 확장 클래스에서 제대로 override하지 않으면 Object 의 equals가 적용돼버려  객체 주소값으로 equals를 판단하게 된다.(객체 identity로 판단)

## Item 41. 타입을 정의하려면 마커 인터페이스를 사용하자

- 메서드 없이 특정 속성을 가짐을 나타내주는 인터페이스를 마커인터페이스(marker interface)라고 한다 ( 예 : **Serializable** ).  
- 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 용도로 사용할 수 있고 이를 통해 런타임에서 잡힐 오류를 컴파일 타임에서 잡을 수 있다(마커 애너테이션이 비해 나은점)
- 적용 대상을 정밀하게 지정할 수 있다. 에너테이션의 경우 적용 대상(@target)을 element.TYPE으로 선언한 에너테이션은 모든 타입(인터페이스, 클래스, 열거 타입, 에너테이션)에 달 수 있다. 그에 반해 마커 인터페이스는 마킹하고 싶은 클래스에서만 마킹인터페이스를 구현하도록 하면 된다.(인터페이스라면 extends하도록)
- 마커 에너테이션의 장점은 에너테이션 시스템의 지원을 받을 수 있다는 점이다.



