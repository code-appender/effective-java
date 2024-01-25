# 아이템 1. 생성자 대신 정적 팩토리 메서드를 고려하라

클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다. 
하지만 클래스는 생성자와 별도로 정적 팰토리 메서드를 제공할 수 있다.

## 정적 팩토리 메서드의 장점
1. **이름을 가질 수 있다.**
   - 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 
   - 반면 정적 팩토리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
     - ex) `BigInteger.probablePrime`
2. **호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**
   - 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
   - 또한 불변 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다.
3. **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**
    - 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있다.
4. **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**
    - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
    - 즉, 성능을 더 개선한 구현체로 바꿔도 클라이언트는 코드를 변경할 필요가 없다.
5. **정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**
    - <a href="https://sihyung92.oopy.io/java/service-provider-framework"> 이해를 위한 참고 링크 </a>
    
## 정적 팩토리 메서드의 단점
1. **상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.**
    - 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다.
2. **정적 팩토리 메서드는 프로그래머가 찾기 어렵다.**

# 아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라

## 인스턴스 생성 방법
1. 점층적 생성자 패턴
    - 필수 매개변수만 받는 생성자를 만든 후, 선택 매개변수를 추가하는 방식
2. 자바빈즈 패턴
    - 매개변수가 없는 생성자를 호출하여 객체부터 만든 후, setter 메서드를 호출하여 필요한 매개변수를 설정하는 방식
3. 빌더 패턴 
    - 점층적 생성자 패턴의 안정성과 자바 빈즈 패턴의 가독성을 겸비했다.

## 빌더 패던의 단점
1. 객체를 생성하려면 먼저 빌더 객체를 생성해야 한다.
2. 매개변수가 4개 이상은 되어야 값어치를 한다.

# 아이템3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

싱글턴을 만드는 방식은 보통 둘 중 하나다.
1. 생성자를 private으로 감추고 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련하는 방식
   - ex) `public static final Elvis INSTANCE = new Elvis();`
   - 하지만 리플렉션 API를 통해서 private 생성자를 호출할 수 있다.
   - 이를 방지하기 위해서는 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 된다.
2. 정적 팩토리 메서드를 public static 멤버로 제공하는 방식
   - ex) `public static Elvis getInstance() { return INSTANCE; }`
   - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다.
   - 제네릭 싱글턴 팩토리로 만들 수 있다는 점이다. 
   - 메서드 참조물 공급자로 사용할 수 있다.

두 방식 모두 Serializable을 구현하고 선언하는 것만으로 직렬화를 처리할 수 없다.
모든 인스턴스 필드를 transient로 선언하고 readResolve 메서드를 제공해야 한다.
그렇지 않으면 역직렬화할 때마다 새로운 인스턴스가 만들어진다.

# 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라

추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없지만
private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

```java
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

이렇게 private 생성자를 만들면 클래스 바깥에서 접근할 수 없다. 또한, 에러를 굳이 던지지 않아도 되지만 클래스 안에서 실수로 생성자를 호출하는 것을 방지할 수도 있다.

이 방식은 상속을 불가능하게 하는 효과도 있다. 

# 아이템5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

대신 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다.

이 방법을 `의존 객체 주입`이라 한다.

의존 객체 주입은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

# 아이템6. 불필요한 객체 생성을 피하라


1. 문자열 리터럴을 사용하면 같은 가상 머신 안에서는 동일한 객체가 재사용된다.

 ```java
// String 생성자
 String s = new String("bikini"); // 따라 하지 말 것

// 문자열 리터럴
 String s = "bikini";
 ```

2. 생성자 대신 정적 팩토리 메서드를 제공하는 불변 클래스에서는 정적 팩토리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.

```java
Boolean.valueOf(String)
```

3. 생성 비용이 아주 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하는 것이 좋다.

```java
// 생성 비용이 비싼 객체를 재사용하는 잘못된 예
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
            + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

// 개선된 코드
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile( // 캐싱
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
    
// 만약 개선된 RomanNumerals 클래스를 생성하고 사용하지 않는다면 ROMAN 필드는 쓸데없이 초기화된다.
// 이런 경우에는 아래와 같이 지연 초기화를 사용할 수 있지만 코드가 복잡해져 권장하지 않는다.
}
```

4. 오토박싱은 불필요한 객체를 만들어 낸다.

```java
// 불필요한 Long 객체를 만드는 오토박싱
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}

// sum 변수를 long이 아닌 Long으로 선언함으로써 불필요한 Long 인스턴스가 231개나 만들어진다.
// 이를 해결하기 위해서는 sum 변수를 long으로 선언해야 한다.
// 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.
```




