---
title: "[이펙티브 자바 요약] Chapter8 - method"

date: "2020-07-21"

tags:

- 이펙티브자바

- effectivejava

- method

- chapter8


---

 이 글은 'Effective Java 3판'(출판사 : 프로그래밍 인사이트, 저자 : Joshua Bloch, 역자 : 개앞맵시(이복연)) 을 읽고 내용을 정리한 글입니다. 문제가 될 시 삭제하겠습니다.



## Item 49 : 매개변수가 유효한지 검사하라

-  메서드의 매개변수의 조건은 반드시 문서화하고, 메서드 본체가 시작되기 전에 조건을 검사해야 오류가 발생한 위치를 정확하게 잡을 수 있고, 이를 통해 즉각적으로 깔끔한 방식으로 예외를 던질 수 있다.
- public이나 proteced는 매개변수 값이 잘못됐을 때 던질 예외를 문서화해아 한다.(@throws 자바독 태그 사용)
- 자바 7에서 추가된 java.util.Objects.requireNotNull 메서드를 사용하면 null 검사를 수동으로 할 필요가 없고, 원하는 예외 메세지도 지정할 수 있다.
- Objects에는 checkFromIndexSize나 checkFromToIndex, checkIndex 같은 범위 검사 기능도 더해졌는데 예외 메세지를 지정할 수 없고 리스트와 배열 전용인 점이 있어 아주 유연하지는 않다.
- publicdd이 아닌, 공개되지 않은 메서드는 assert를 통해 매개변수 유효성을 검증할 수 있다.
- 메서드가 직접 사용하지 않고 나중에 쓰기 위해 저장하는 매개변수는 더 신경써서 검사해야하는데 이는 차후 사용시에 예외가 발생하기 때문에 최초 에러가 심어진 지점과 발생 지점이 달라지기 때문이다. 예를 들어 정적 팩터리 메서드에서 인자를 받아 멤버 객체로 저장해둔 다음 나중에 다른 메서드에서 이 객체를 사용하는 경우 최초 팩터리 메서드에서 null 검사를 해주지 않으면 나중에 다른 메서드에서 객체 사용시 nullPointerException이 발생하게 된다.
- 메서드는 매개변수에 최대한 제한을 두지 말고 범용적으로 설계하돼, 적용된 제한 조건은 잘 검사해줘야한다.



## Item 50 : 적시에 방어적 복사본을 만들어라

- 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하지만, 주의를 기울이지 않으면 자기도 모르게 내부를 수정하도록 허락하는 경우가 생긴다.

  ``` java
  public final class Period {
    private final Date start;
    private final Date end;
   
    public Period(Date start, Date end) {
      if (start.compareTo(end) > 0) {
      		throw new IllegalArgumentException (start + "가 " + end + "보다 늦다.");  
      }	
      this.start = start;
      this.end = end;
    }
    
    public Date start() {
      return start;
    }
    public Date end() {
      return end;
    }
    
    ...// 나머지 생략
  }
  ```

   위와 같은 클래스가 있을 때, 우리는 위 클래스가 불변이고 날짜 조건도 잘 지켜질 것이라고 기대하지만

  ``` java
  Date start = new Date();
  Date end = new End();
  Period = new Period(start, end);
  end.setYear(123);
  ```

  위와 같은 방법으로 불변성이 깨질 수 있다. 자바 8 이후로는 Date 대신 불변인 Instant를 사용하면 간단히 해결되는 문제다.(혹은 LocalDateTime이나 ZonedDateTime)

- Date는 낡은 API니 가급적 사용하지 말자

- 외부 공격으로 부터 내부를 보호하려면 생성자에게서 받은 <strong>가변</strong> 매개변수 각각을 <strong>방어적으로 복사</strong>(defensive copy)해야한다.

- 위 예에서는 아래와 같이 방어적 복사를 수행할 수 있다.

  ``` java
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    
    if (this.start.compareTo(this.end) > 0) {
      	throw new IllegalArgumentException(this.start + " 가" + this.end + "보다 늦다.");
      ...//생략
    }
  }
  ```

- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용하면 안된다. 왜냐하면 생성자에 넘겨주는 Date는, 즉 외부에서 넘겨주는 값은 그것이 Date를 악의적으로 확장한 클래스인지 알 수 없기 때문이다.

- 생성자 뿐만 아니라 접근자 메서드를 이용한 객체 변경 또한 아래와 같이 막아야 한다.

  ``` java
  public Date start() {
    return new Date(start.getTime());
  }
  public Date end() {
    return new Date(end.getTime());
  }
  ```

- 위의 예에서, 즉 접근자를 통한 방어적 복사에서는 clone을 사용해도 되는데, 이는 생성자에서 이미 방어적 복사를 통해 Date 객체임을 보장했기 때문이다. 

- 항상 클라이언트가 넘겨주는 '객체'가 불변인지, 불변이 아니라면 변경되어도 객체에 영향이 없는지를 잘 생각해야한다. 따라서 불변 객체 만으로 클래스를 구성하면 방어적 복사를 할 일이 줄어든다.

- 방어적 복사에는 성능 저하가 항상 따르게 돼있으므로, 불필요하다고 판단된다면 사용하지 않을 수도 있다.

## Item 51 : 메서드 시그니처를 신중하게 설계해라

- 메서드 이름을 신중히 짓자
  - 표준 명명 규칙을 따를 것
  - 널리 받아들여지는 이름 사용할 것
  - 가능하면 짧은 것
  - 애매하면 자바 라이브러리의 API 가이드 참조
- 편의 메서드를 너무 많이 만들지 말자
  - 메서드가 너무 많은 클래스를 사용, 유지 보수하기는 어렵다
  - 메서드가 너무 많으면 구현자와 사용자 모두 고통스럽다
  - 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 한다.
  - 아주 자주 쓰일 경우에만 별도의 양칭 메서드를 두자.
  - <strong> 확신이 없으면 만들지 마라</strong>
- 매개변수 목록은 짧게 하자
  - 4개 이하가 좋다
  - 같은 타입의 매개변수가 연속해서 나오는 경우는 해롭다
  - 매개변수 줄이는 방법
    - 여러 메서드로 쪼갠다. 매서드 수가 너무 많아질 수있지만 잘 쪼갠 매서드는 서로 직교성이 높고, 그러면 오히려 메서드 수를 줄일 수 있다. 직교성이 높은 메서드를 잘 조합하면 여러 기능을 수행할 수 있기 때문이다.
    - 매개변수 여러개를 묶은 도우미 클래스를 만들자. 일반적으로는 정적 멤버 클래스로 만든다.
    - 생성자의 빌더 패턴을 사용해본다. 
  - 매개변수의 타입으로는 클래스보다는 인터페이스를 써야 더 범용성이 있다.
- boolean보다는 값 2개 짜리 열거 타입이 더 의미를 잘 나타낸다. boolean만으로 의미를 잘 살리는 경우라면 상관 없다.

## Item 52 : 다중 정의는 신중하게 사용하라

- 재정의(오버라이딩) 경우에는 어떤 메서드가 호출될 지 런타임에(실제 어떤 객체가 호출하느냐에 따라) 결정되지만, 중복정의(오버로딩)의 경우에는 컴파일 타임에 결정된다. 즉 실제 객체에 따라 동적으로 결정되지 않는다. 따라서 아래 코드의 결과는 기대했던 "집합", "리스트", "그 외" 가 아니고 "그 외", "그 외", "그 외" 가 된다. 컴파일 타임에는 for 문의 c는 항상  Collection<?> 이기 때문이다.

  ``` java
    public static String classify(Set<?> s) {
      return "집합";
    }
    
    public static String classify(List<?> lst) {
      return "리스트";
    }
    
    public static String classify(Collection<?> c) {
      return "그 외";
    }
    
    public static void main(String[] argd) {
      Collection<?>[] collections = {
        new HashSet<String>(),
        new ArrayList<BigInteger>(),
        new HashMap<String, String>().values()
      };
      
      for (Collection<?> c : collections) {
        System.out.println(classify(c)_)
      }
    }
  }
  ```

- 위의 경우 기대했던 값을 얻으려면 classify함수를 하나로 줄이고 그 안에서 instanceof를 이용해 처리하면 된다.

- 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자. 다중정의 대신 메서드 이름을 다르게 지어줄 수 있다. ObjectOutputStream 클래스의 readBoolean(), readInt(), readLong()과 같은 함수들이 대표적이다. 

- 생성자는 메서드 이름을 같게할 수 밖에 없으니 오버로딩밖에 안되는데, 정적팩터리라는 대안이 있다.

- 매개변수 수가 같은 다중정의라도, 매개변수들 중에 하나 이상이 '근본적으로 다르다(radicallly different)'면 헷갈리지 않는다. 이는 서로 형변환 되지 않는다는 뜻이고, 형변환되지 않는 매개변수끼리는 잘못 넣어도 어차피 exception이 터지게 된다.

- 자바 4까지는 원시 타입과 객체 타입이 근본적으로 달랐지만, 이후 오토박싱이 나오면서 이게 깨졌다 ㅠㅠ. 따라서 주의 하자

  

## Item 53 : 가변 인수는 신중히 사용하라

- 냉무

## Item 54 : null이 아닌 빈 컬렉션이나 배열을 반환하라

``` java
private final List<Cheese> cheesesInStock = ...;

public List<Cheese> getCheese() {
  return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock);
}
```

- 위와 같은 코드는 클라이언트로 하여금 null에 대한 처리를 추가로 해야하게 만든다.
- 대부분의 상황에서는 그냥 빈 컬렉션을 반환하는 것이 낫다.
- 드물게 빈 컬렉션 할당이 성능을 심각하게 저하시키는 경우도 있을 수 있는데 이런 경우 매변 똑같은 '불변' 컬렉션을 반환하면 된다. 불변 객체는 자유롭게 공유해도 안전하니까.

``` java
public List<Cheese> getCheese() {
  return cheesesInStock.isEmpty() ? Collection.emptyList() : new ArrayList<>(cheeseInStock);
}
```

- 배열인 경우도 마찬가지로 null을 쓰지 말고 길이가 0인 배열을 반환하자. 0는 그냥 배열 길이 값의 한 사례일 뿐 특별히 처리하지 않아도 된다. 만약 성능이 걱정되면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다.(추천하지는 않음)



## Item 55 : Optional은 신중히 반환하라

- 특정 조건에서 값을 반환할 수 없을 때 는 두 가지 선택지가 있고, 둘 모두 나름의 문제점을 가지고 있다.
  - 예외 던지기
    - 예외는 진짜 예외적인 상황에서만 사용해야한다
    - 스택 추적 전체를 캡쳐하므로 비용도 비싸다.
  - Null 반환
    - 별도의 null인 경우 처리 로직을 추가해야한다.
    - 처리 안해주면 언제가 숨이었다가 전혀 다른 코드에서 NullPointerException을 터뜨린다.
- Optional은 Optional\<T\>와 같이 제네릭으로 사용이 가능한데,  이는 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않는 다는 뜻이다. 아무 것도 담지 않은 것을 '비었다' 라고 한다.
- Optional은 원소를 최대 1개 가지는 '불변' 컬렉션이다. (Collection을 구현했다는게 아니라 원칙적으로 그렇다는 뜻)
  - 보통은 T를 반환하지만 특정조건에서는 마우것도 반환하지 않아야할 때, T대신 Optioanl\<T\>를 반환하도록하면 된다.
- Optional을 반환하는 메서드에서는 절대 null을 반환하지 말자
- null 값도 허용하는 Optional을 만드려면 Optional.ofNullable(value)와 같은 방법을 사용하면 된다.
- Optional은 반환 값이 없을 수도 있음을 명확하게 알려준다.
- 반환값으로 Optional을 쓴다고 무조건 득은 아니다. <strong>컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 Optional로 감싸면 안된다</strong>.
  - 빈 Optional\<List\<T\>\> 를 반환하기 보다는 빈 List\<T\>를 반환하는게 좋다.

  - <strong>결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional을 반환하면 된다.</strong>
- 제네릭에서 타입 T는 객체타입만 가능한데, 박싱된 기본 타입을 담으면 기본 타입보다 무거울 수 밖에 없기 때문에 OptioanlInt, OptionalLong, OptionalDouble과 같은 것을 쓰면 된다.

## Item 56 : 공개된 API 요소에는 항상 문서화 주석을 달자

- API를 올바로 문서화 하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
- 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야한다.
  - how가 아닌 what(무엇을 하는지)
  - 호출 전제조건
  - 사후 조건
  - 부작용 (명확하진 않지만 시스템의 상태에 어떤 변화를 가져올 때)
- 한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버가 둘 이상이면 안된다.
- 각 주석의 첫 번째 문장은 요약설명인데, 마침표(.)에 주의해야 한다.

