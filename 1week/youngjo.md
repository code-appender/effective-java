# 1주차 (item1 ~ 6)

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

### 정적 팩터리 메서드의 장점 5가지
1. 메소드 이름을 가질 수 있다. -> 명확한 이름으로 값을 생성할 수 있다.
2. 호출 때 마다 인스턴스를 새로 생성하지 않을 수 있다.
3. 하위 클래스를 반환하는 유연성을 얻을 수 있다.
4. 매개 변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적 팩토리 메소드를 작성하는 시점에는 해당 객체의 클래스가 존재하지 않아도 된다.

### 정적 팩터리 메서드의 단점 2가지
1. 정적 팩터리 메서드만으로는 하위 클래스를 만들 수 없다. -> 상속을 위해선 생성자가 필요하기 때문
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다. -> 이를 위해 메서드 네이밍 규약을 따라짓는 것으로 완화

## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

생성자의 매개변수가 많아지면 코드를 직관적으로 이해하기 어렵다.
-> Setter를 사용하면 여러 개의 Setter가 호출되며 객체의 생성 전까지 일관성이 무너지게 되고, Open-Closed 법칙을 위배하게 된다.
-> Builder를 사용하면 쓰기 쉽고, 읽기 쉬워 가독성을 높일 수 있으며 가변 매개변수를 활용함으로써 유연함을 얻을 수 있다.

Builder는 계층적으로 설계된 클래스와 함께 쓰기 좋다. (추상 클래스 -> 추상 빌더 / 구체 클래스 -> 구체 빌더)

단점: 빌더 패턴을 사용하기 전에 빌더를 생성해야함. (생성 비용이 크진 않으나 성능적인 면에서 문제가 될 수도 있음)

## 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴: 인스턴스를 오직 하나만 생성할 수 있는 클래스

싱글턴 만드는 방법은 보통 둘 중에 하나다. -> 두 방식 모두 private 생성자로 감춰두고 public static 멤버를 하나 만들어서 접근한다.

1. public static final 필드 방식의 싱글턴
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    
    public void leaveTheBuilding() {}
}
```

2. 정적 팩터리 방식의 싱글턴
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() {
        return INSTANCE;
    }
    
    public void leaveTheBuilding() {}
}
```

3. 열거 타입 방식의 싱글턴 (가장 바람직함)
```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() {}
}
```
원소가 하나뿐인 열거 타입 -> 싱글턴 만들기에 가장 적합하다!
단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면? 이 방법 사용 불가

## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라.

정적 메서드나 정적 필드만을 갖는 클래스를 생성하는 경우가 있다.
객체 지향적 관점에서 그렇게 좋지는 않은 방식이지만 분명 필요한 경우가 있는데, 예를 들면 유틸성 클래스이다.
이러한 클래스들은 인스턴스를 생성하기 위해 만든 것이 아니다.
하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들 것이고, 다른 사용자는 이것이 자동 생성된 것인지 구분할 수 없을 것이다.
그러므로 이를 방지하기 위해 private 생성자를 추가해주도록 하자.

## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

하나 이상의 자원에 의존하는 클래스는 정적 유틸리티나 싱글턴을 사용하는 것이 효과적이지 않다. (하나의 자원만 사용하는 것이 아니기 때문)
-> 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주자.

의존 객체 주입이 유연성, 테스트 용이성을 개선해주지만 의존성이 수천 개나 되는 프로젝트에서는 어지럽게 만들기도 한다.
-> 대거, 주스, 스프링과 같은 의존 객체 주입 프레임워크를 사용하자.

## 아이템 6. 불필요한 객체 생성을 피하라.

같은 기능을 하는 객체를 매번 생성 (X) -> 객체 하나 재사용(O)

### 예시 1) 문자열 리터럴 사용으로 하나의 String 인스턴스 재사용
```java
String s = new String("bikini"); // 따라하지 말 것!

String s = "bikini";
```

### 예시 2) 정적 팩터리 메서드 사용 -> 생성자 사용 X
```java
Boolean(String) // X

Boolean.valueOf(string) // O
```

### 예시 3) 생성 비용이 큰 객체 -> 캐싱하여 재사용
```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
// String.matches 내부 Pattern 인스턴스는 유한 상태 머신을 만들기 때문에 생성 비용이 높다.
// 그럼 해당 부분을 빼내서 직접 생성해 캐싱하자!

public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

만약 개선된 RomanNumerals 클래스를 생성하고 사용하지 않는다면 ROMAN 필드는 쓸데없이 초기화된다.
isRomanNumeral 메서드가 처음 호출될 때 필드를 초기화하는 지연 초기화로 불필요한 초기화를 없앨 수는 있지만, 코드가 복잡해져 권장하지 않는다.

### 예시 4) 오토박싱으로 인한 객체 생성
```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```
sum 변수가 long 타입이 아닌 Long 타입으로 생성되어 불필요한 Long 인스턴스가 2<sup>31</sup>개 만들어진다.
박싱된 기본 타입 (X) -> 기본 타입 (O)