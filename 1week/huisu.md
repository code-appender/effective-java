# 객체 생성과 파괴

## ITEM 1 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩터리 메소드의 장점

- 이름을 가질 수 있다
    - 생성자에 넘기는 매개 변수와 생성자 자체만으로는 반환될 객체의 특성을 잘 알 수 없음
    - BigInteger(int, int, Random) → BigInteger.probablePrime
- 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다
    - Boolean.valueOf(boolean) 사용할 때마다 Boolean 객체 사용하지 않음
    - FlyWeight Pattern: 동일하거나 유사한 객체들 사이에 가능한 많은 데이터를 공유해 사용하도록 하여 메모리 사용량을 최소화하는 패턴
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
    - 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API 작게 유지
    - java.util.Collections
    - 프로그래머가 API을 사용하기 위해 익혀야 하는 개념의 수와 난이도도 하강
    - 인터페이스 문서만 참조하고 구현 클래스는 보지 않아도 됨
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
    - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환해도 됨
    - EnumSet → RegularEnumSet, JumboEnumSet
    - 클라이언트는 팩터리가 건네 주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없음
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
    - 서비스 인터페이스: 구현체의 동작 정의
    - 제공자 등록 API: 제공자가 구현체를 등록
    - 서비스 접근 API: 클라이언트가 서비스의 인스턴스를 얻을 때 사용
    - 제공자는 서비스의 구현체이고 클라이언트에 제공하는 구현체들의 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리
    - 브리지 패턴: 구현부에서 추상층을 분리해 각자 독립적으로 변경 가능하게 함

### 정적 팩터리 메소드의 단점

- 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다
- 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
    - 생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법 모색 필요

## ITEM 2 생성자에 매개변수가 많다면 빌더를 고려하라

### 인스턴스 생성 방법

- 점층적 생성자 패턴
    - 선택 매개변수를 전부 다 받는 생성자까지 늘려 가는 방식
    - 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려움
- 자바빈즈 패턴
    - 매개변수가 없는 생성자로 객체를 만든 후 세터 메서드를 호출해 원하는 매개변수의 값을 설정
    - 객체 하나를 만들기 위해 메서드 여럿 호출
    - 객체가 완전히 생성되기 전까지 일관성 무너짐
- 빌더 패턴
    - 필수 매개변수만으로 생성자를 호출해 빌더 객체 얻음
    - 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수를 설정
    - 계층적 클래스에 유연하게 사용
- 빌더 패턴의 단점
    - 빌더 생성이 장황해서 매개변수가 작다면 다른 방법이 더 간편

## ITEM 3 private 생성자나 열거 타입으로 싱글텀임을 보증하라

### public static final 필드 방식

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}
}
```

- 싱글턴임이 API에 명백히 드러남

### 정적 팩터리 방식

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {...}
	public static Elvis getInstance() { return INSTANCE; }
}
```

- 마음이 바뀌면 API를 바꾸지 않고도 싱글턴이 아니도록 변경 가능
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 바꿀 수 있음
- 정적 팩터리 메소드 참조를 공급자로 사용할 수 있음

### 열거 타입

```java
public enum Elvis {
	INSTANCE; 
}
```

- 대부분의 상황에서 가장 좋은 방법

## ITEM 4 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때
- 기본 클래스를 생성해 주기 때문에 인스턴스화가 이루어지는 경우도 있음
- 추상 클래스는 하위 클래스를 만들어 인스턴스화할 수도 있음
- private 생성자를 추가해 클래스의 인스턴스화 차단

## ITEM 5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음
- 의존 객체 주입 패턴: 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨 주는 방식

## ITEM 6 불필요한 객체 생성을 피하라

### 객체 재사용

```java
String s = new String("bikini");
```

- 이 문장은 실행될 때마다 Stiring 인스턴스 생성

```java
String s = "bikini";
```

- 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용
- 불변 객체일 경우 굳이 매번 인스턴스를 사용하지 않고 재사용 권장

### 캐싱

- 생성 비용이 아주 비싼 객체의 경우 캐싱해서 사용하길 권장

```java
static boolean isRomanNumeral(String s) {
	return s.matches("^(?=.)M*(C[MD]|D?C{0,3}");
}
```

- 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져 바로 가비지 컬렉션 대상

```java
public class RomanNumerals {
	private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3}");
}
```

- 캐싱해서 사용
- 객체가 불변이라면 재사용해도 안전
- 박싱된 기본 타입보다는 기본 타입을 사용하고 의도치 않은 오토 박싱 점검