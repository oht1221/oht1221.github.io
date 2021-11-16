---
title: "[이펙티브 자바 요약] Chapter12 - Serialization"

date: "2020-08-18"

tags:

- 이펙티브자바

- effectivejava

- 직렬화

- chapter12

- serialization
---

 이 글은 'Effective Java 3판'(출판사 : 프로그래밍 인사이트, 저자 : Joshua Bloch, 역자 : 개앞맵시(이복연)) 을 읽고 내용을 정리한 글입니다. 문제가 될 시 삭제하겠습니다.



## Item 85 : 자바 직렬화의 대안을 찾으라

- ObjectInputStream의 readObject 메서드는 바이트 스트림을 역직렬화하는 과정에서 그 타입들 안의 모든 코드를 실행할 수 있고, 이는 그 타입들의 코드 전체가 악의적인 공격 범위에 들어간다는 뜻이다.
- 신뢰할 수 없는 바이트 스트림은 역직렬화 하면 안된다. 
- 아무것도 역직렬화하지 않으면 직렬화 위험은 피할 수 있다. 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘이 많이 있다.
- 자바 직렬화의 단점을 회피하면서 많은 이점을 제공하는 직렬화 시스템들이 있는데 자바 직렬화와 구분하여 cross-platforsm structured-data representation 이라 한다. 대표적으로 JSON, 프로토콜 버퍼(줄여서 protobuf) 이 있다.
- 직렬화를 피할 수 없고 역직렬화한 데이터 역시 안전한지 확실할 수 없다면 객체 역직렬화 필터링(java.io.ObjectInputFilter)를 사용하자

- 자바 직렬화 말고 크로스 플랫폼 구조화된 데이터 표현으로 마이그레이션 하는 것을 고민해보자



## Item 86 : Serializable 을 구현할지는 신중히 생각하라

- implements seriazliable 을 붙이는 것은 매우 간단한 일이지만 길게 보면 아주 값비싼 작업이다
- Serializable을 구현하면 릴리스 후에 수정하기 어렵다. 클래스가 Serializable을 구현하면 직렬화된 바티으 스트림 인코딩(직렬화 형태)도 하나의 공개 API가 된다. 기본 직렬화 형태에서는 클래스의 privater과 package-private 인스턴스 필드들마저 API로 공개하는 꼴이 된다.
- 뒤늦게 내부 구현을 손 본다고 해도 원래 직렬화의 형태와 달라지게 된다.
- Serializable을 구현하면 버그와 보안 구멍이 생길 위험이 높다
- Serializable을 구현하면 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.
- Serializable 구현 여부는 가볍게 결정할 사안이 아니다. 역사적으로 BigInteger와 Instant같은 '값' 클래스와 컬렉션 클래스들은 Serializable을 구현하고, 스레드 풀처럼 '동작'하는 객체를 표현하는 캘르스들은 대부분 그렇지 않았다.



## Item 87 : 커스텀 직렬화 형태를 고려해보라

- 먼저 고민해보고 괜찮다고 판단될 때에만 기본 직렬화 형태를 사용하라.

- 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다. 이상적인 직렬화 형태는 물리적 모습과 독립된 논리적 모습만 표현해야 한다. 예컨대 아래의 사람의 성명을 간략하게 표현한 클래스는 기본 직렬화 형태를 써도 괜찮다. 성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성되고 아래 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확하게 반영했다.

  ``` java
  public class Name implements Serializable {
    /**
     * 성. non-null
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. non-null
     * @serial
     */
    private final String firstName;
    
    /**
     * 미들네임. 미들네임이 없다면 null
     @serial
     */
    private final String middleName;
   
    .../// 생략
  }
  ```

  



## Item 88 : readObject 메서드는 방어적으로 작성하라

- readObject 실질적으로 또다른 public 생성자이다. 따라서 생성자와 똑같은 수준으로 주의를 기울여야 한다. 일반 생성자와 마찬가지로 인수가 유효한지 검사하고 필요하다면 방어적 복사도 수행해야 한다.
- 바이트 스트림을 매개변수로 받는 생성자인 readObject에 악의적으로 잘못 만든 바이트 스트림을 넣으면 클래스 불변식이 깨질 수 있다.

- 객체를 역직렬화할 때는 클라리언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.
- private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속한다.
- 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어즉 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
- 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라.
- 직접적이든 간접적이든 재정의할 수 있는 메서드는 호출하지 말자.



## Item 89 : 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거타입이 낫다

- 아래 클래스는 바깥에서 생성자를 호출하지 못하게 막는 방식으로 인스턴스가 오직 하나만 만들어짐을 보장했다. (싱글턴)

  ``` java
  public class Elvis {
    public static final Elvis Instance = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
  }
  ```

- 안타깝게도 위 클래스는 implements Seiralizable 을 추가하는 순간 싱글턴이 아니게 된다. 기본 직렬화던, 명시적인 readObject이건 어떤 readObject를 사용하든 이 클래스가 초기화될 때와는 다른 인스턴스를 반환 하게 된다.

- 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 한 열거 타입을 사용하자. 여의치 않은 상황에서 직렬화와 인스턴드 통제가 모두 필요하담녀 readResolve 메서드를 작성해야 하고 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야 한다.



## Item 90 : 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

- readObject를 통한 비정상적 인스턴스 생성 위험을 크게 줄여줄 기법이 하나 있는데 바로 직렬화 프록시 패턴 (serialization proxy  pattern)이다.

- 아래와 같은 클래스를 기존 클래스 안에 중첩 클래스로 추가하여 직렬화 프록시 패턴을 적용할 수 있다.

  ``` java
  private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;
    
    SerializationProxy(Period p) {
      this.start = p.start;
      this.end = p.end;
    }
    private static final long serialVersionUID = 12341234123414L;
  }
  ```

- 다음으로 바깥 클래스(Period)에 아래 writeReplace 메서드를 추가한다.

  ``` java
  private Object writeReplace() {
    return new SerializationProxy(this);
  }
  ```

- 아래의 메서드 추가를 통해 공격자가 불변식을 훼손하고자 하는 시도도 막을 수 있다.

  ``` java
  private void readObject(ObjectInputStream stream) {
    throws InvalidObjectException {
      throw new InvalidObjectException("프록시가 필요합니다.")
    }
  }
  ```

- 마지막으로 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가하면 된다. 이 메서드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.