# 아이템 1. 생성자 대신 정적 팩토리 메서드를 고려하라

- 클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자다.
- 클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩토리 메서드를 제공할 수 있다.

## 정적 팩토리 메서드의 장점

- 이름을 가질 수 있다.
    - 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.
    - 반면 정적 팩토리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
    - 생성자는 시그니처에 제약이 있다. 똑같은 타입을 파라미터로 받는 생성자 두 개를 만들 수 없으니까 그런 경우에도 public static 팩토리 메서드를 사용하는 것이 유용하다.
- 호출될 때 마다 인스턴스를 새로 생성하지는 않아도 된다.
    - 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
    - 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다.
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    - API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

## 정적 팩토리 메서드의 단점

- 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.
- 프로그래머가 static 팩토리 메서드를 찾는게 어렵다.
    - 생성자처럼 API 설명에 명확히 드러나지 않으니 팩토리 메서드에 대한 문서를 제공하는 것이 좋겠다.

---

# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

- static 팩토리 메서드와 public 생성자 모두 매개변수가 많이 필요한 경우에 불편해진다.
- NutritionFacts라는 클래스를 예로 들고있다.

## 해결책 1. 생성자

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

- 이런 생성자를 쓰다보면 필요 없는 매개변수도 넘거야 하는 경우도 발생한다. 보통 0같은 기본값을 넘긴다.
- 이런 코드는 작성하기도 어렵고 읽기도 어렵다.

## 해결책 2. 자바빈

- 매개변수를 받지 않는 생성자를 사용해서 인스턴스를 만들고, 세터를 사용해서 필요한 필드만 설정한다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

- 단점은 최종적인 인스턴스를 만들기까지 여러번의 호출을 거쳐아 하기 때문에 자바빈이 중간에 사용되는 경우 안정적이지 않은 상태로 사용될 여지가 있다.
- 또한 (게터와 세터가 있어서) 불변 클래스(아이템17)로 만들지 못한다는 단점이 있다.
- 쓰레드 안정성을 보장하려면 추가적인 수고(locking 같은)가 필요하다.

## 해결책 3. 빌더

- 생성자의 안정성과 자자빈을 사용할 때 얻을 수 있었던 가독성을 모두 취할 수 있는 대안이다.
- 빌더 패턴은 만드려는 객체를 바로 만들지 않고 클라이언트는 빌더 빌더(생성자 또는 static 팩토리)에 필수적인 매개변수를 주면서 호출해 Builder 객체를 얻은 다음 빌더 객체가 제공하는 세터와 비슷한 메서드를 사용해서 부가적인 필드를 채워넣고 최종적으로 build라는 메서드를 호출해서 만드려는 객체를 생성한다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```

- 빌더는 가변 인자(vargars) 매개변수를 여러 개 사용할 수 있다는 장점도 있다.
- 여러 메서드 호출을 통해 전달받은 매개변수를 모아 하나의 필드에 담는 것도 가능하다.
- 매번 생성하는 객체를 조금씩 변화를 줄 수 있다.
- 단점으로는 객체를 만들기 전에 먼저 빌더를 만들어야 하는데 성능에 민감한 상황에서는 문제가 될 수 있다.
- 생성자를 사용하는 것보다 코드가 더 장황하다.
- 따라서 빌더 패턴은 매개변수가 많거나(4개 이상?) 또는 앞으로 늘어날 가능성이 있는 경우에 사용하는 것이 좋다.

---

# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 오직 한 인스턴스만 만드는 클래스를 싱글톤이라고 부른다. 보통 함수 같은 Stateless 객체(아이템24) 또는 본질적으로 유일한 시스템 컴포넌트를 그렇게 만든다.
- 싱글톤을 사용하는 클아이언트 코드를 테스트하는게 어렵다. 싱글톤이 인터페이스를 구현한게 아니라면 mock으로 교체하는게 어렵기 때문이다.
- 싱글톤으로 만드는 두 가지 방법이 있는데, 두 방법 모두 생성자를 private으로 만들고 public static 멤버를 사용해서 유일한 인스턴스를 제공한다.

## final 필드

첫 번째 방법은 final 필드로 제공한다.

```java
public class Elvis {

    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

}
```

리플렉션을 사용해서 private 생성자를 호출하는 방법을 제외하면 (그 방법을 막고자 생성자 안에서 카운팅하거나 flag를 이용해서 예외를 던지게 할 수도 있지만) 생성자는 오직 최초 한번만 호출되고 Elvis는 싱글톤이 된다.

### 장점

- 이런 API 사용이 static 팩토리 메소드를 사용하는 방법에 비해 더 명확하고 더 간단하다.

## static 팩토리 메서드

```java
public class Elvis {

    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

}
```

### 장점

API를 변경하지 않고로 싱글톤으로 쓸지 안쓸지 변경할 수 있다. 처음엔 싱글톤으로 쓰다가 나중엔 쓰레드당 새 인스턴스를 만든다는 등 클라이언트 코드를 고치지 않고도 변경할 수 있다.

필요하다면 `Generic 싱글톤 팩토리`([아이템 30](https://github.com/keesun/study/blob/master/effective-java/item30.md))를 만들 수도 있다.

static 팩토리 메소드를 `Supplier<Elvis>`에 대한 `메소드 레퍼런스`로 사용할 수도 있다.

## 직렬화 (Serialization)

위에서 살펴본 두 방법 모두, 직렬화에 사용한다면 역직렬화 할 때 같은 타입의 인스턴스가 여러개 생길 수 있다. 그 문제를 해결하려면 모든 인스턴스 필드에 `transient`를 추가 (직렬화 하지 않겠다는 뜻) 하고 `readResolve` 메소드를 다음과 같이 구현하면 된다. (객체 직렬화 API의 비밀 참고)

```java
private Object readResolve() {
		return INSTANCE;
}
```

## Enum

직렬화/역직렬화 할 때 코딩으로 문제를 해결할 필요도 없고, 리플렉션으로 호출되는 문제도 고민할 필요없는 방법이 있다.

```java
public enum Elvis {
    INSTANCE;
}
```

코드는 좀 불편하게 느껴지지만 싱글톤을 구현하는 최선의 방법이다. 하지만 이 방법은 Enum 말고 다른 상위 클래스를 상속해야 한다면 사용할 수 없다. (하지만 인터페스는 구현할 수 있다.)

---

# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

static 메서드와 static 필드를 모아둔 클래스를 만든 경우 해당 클래스를 abstract로 만들어도 인스턴스를 만드는 걸 막을 수 없다. 상속 받아서 인스턴스를 만들 수 있기 때문이다.

그리고 아무런 생성자를 만들지 않은 경우 컴파일러가 기본적으로 아무 인자가 없는 public 생성자를 만들어주기 때문에 그런 경우에도 인스턴스를 만들 수 있다.

명시적으로 private 생성자를 추가해야 한다.

```java
public class UtilityClass {
		
		// 기본 생성자가 만들어지는 것을 막는다. (인스턴스화 방지용)
		private UtilityClass() {
				throw new AssertionError();
		}
}
```

AssertionError는 꼭 필요하진 않지만, 의도치 않게 생성자를 호출한 경우에 에러를 발생시킬 수 있고 private 생성자기 때문에 상속도 막을 수 있다.

생성자를 제공하지만 쓸 수 없기 때문에 직관에 어긋나는 점이 있는데, 위에 코드처럼 주석을 추가하는 것이 좋다.

부가적으로 상속도 막을 수 있다. 상속한 경우에 명시적이든 암묵적이든 상위 클래스의 생성자를 호출해야 하는데, 이 클래스의 생성자가 private이라 호출이 막혔기 때문에 상속할 수 없다.

---

# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스가 하나 이상의 자원에 의존한다. 가령 맞춤법 검사기는 사전(dictionary)에 의존하는데, 이런 클래스를 정적 유틸리티 클래스(아이템 4)로 구현한 모습을 드물지 않게 볼 수 있다.

## 부적절한 구현

### static 유틸 클래스 (아이템4)

```java
// 부적절한 static 유틸리티 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {
		
		private static final Lexicon dictionary = new KoreanDictionary();

		private SpellCheck() {
		}

		public static boolean isValid(String word) {
				throw new UnsupportedOperationException();
		}

		public static List<String> suggestions(String type) {
				throw new UnsupportedOperationException();
		}
}

interface Lexicon {}
		
class KoreanDicationary implements Lexicon {}
```

### 싱글톤으로 구현하기 (아이템3)

```java
// 부적젏 싱글톤 사용 예 - 유연하지 않고 테스트 할 수 없다.
public class SpellChecker {
		
		private final Lexicon dictionary = new KoreanDictionary();

		private SpellCheck() {
		}

		public static final SepllChecker INSTANCE = new SpellChecker() {};

		public static boolean isValid(String word) {
				throw new UnsupportedOperationException();
		}

		public static List<String> suggestions(String type) {
				throw new UnsupportedOperationException();
		}
}
```

사전을 하나만 사용할 거라면 위와 같은 구현도 만족스러울 수 있지만, 실제로는 각 언어의 맞춤법 검사기는 사용하는 사전이 각기 다르다.
또한 테스트 코드에서는 테스트용 사전을 사용하고 싶을 수 있다.

어떤 클래스가 사용하는 리로스에 따라 행동을 달리 해야 하는 경우에는 스태틱 유틸리티 클래스와 싱글톤을 사용하는 것은 부적절하다.

그런 요구 사항을 만족할 수 있는 간단한 패턴으로 생성자를 사용해서 새 인스턴스를 생성할 때 사용할 리소스를 넘겨주는 방법이 있다.

## 적절한 구현

```java
public class SpellChecker {
		
		private final Lexicon dictionary;

		public SpellChecker(Lexicon dectionary) {
				this.dictionary = Objects.requireNonNull(dictionary);
		}

		public static boolean isValid(String word) {
				throw new UnsupportedOperationException();
		}

		public static List<String> suggestions(String type) {
				throw new UnsupportedOperationException();
		}
}
```

위와 같은 의존성 주입은 생성자, 스태틱 팩토리(아이템1) 그리고 빌더(아이템2)에도 적용할 수 있다.

의존하는 리소스에 따라 행동을 달리하는 클래스를 만들 때 싱글톤이나 스태틱 유틸 클래스를 사용하지 말자.
그런 경우에는 리소스를 생성자나 팩토리로 전달하는 의존성 주입을 사용하여 유연함, 재사용성, 테스트 용이성을 향상 시키자.

---

# 아이템 6. 불필요한 객체 생성을 피하라

기능적으로 동일한 객체를 새로 만드는 대신 객체 하나를 재사용하는 것이 대부분 적절하다.
재사용하면 더 빠르고 스타일리쉬하다. 불변객체(아이템17)는 항상 재사용할 수 있다.

## 문자열 객체 생성

자바의 문자열, String을 new로 생성하면 항상 새로운 객체를 만들게 된다.

```java
String s = "bikini";
```

문자열 리터럴을 재사용한다.

## static 팩토리 메서드 사용하기

자바 9에서 deprecated 된 Boolean(String) 대신 Boolean.valueOf(String) 같은 static 팩토리 메서드(아이템1)를 사용할 수 있다.

생성자는 반드시 새로운 객체를 만들어야 하지만 팩토리 메서드는 그렇지 않다.

## 무건 객체

만드는데 메모리나 시간이 오래 걸리는 객체 즉 “비싼 객체”를 반복저으로 만들어야 한다면 캐시해두고 재사용할 수 있는지 고려하는 것이 좋다.

정규 표현식으로 예제를 살펴보자. 문자열이 로마 숫자를 표현하는지 확인하는 코드는 다음과 같다.

```java
static boolean isRomanNumeral(String s) {
		return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|.....;
```

String.matches가 가장 쉽게 정규 표현식에 매치가 되는지 확인하는 방법이다.

하지만 성능이 중요한 상황에서 반복적으로 사용하기에 적절하지 않다.

String.matches는 내부적으로 Pattern 객체를 만들어 쓰는데 그 객체를 만들려면 정규 표현식으로 유한 상태 기계로 컴파일 하는 과정이 필요하다. 즉 비싼 객체다.

성능을 개선하려면 Pattern 객체를 만들어 재사용하는 것이 좋다.

```java
public Class RomanNumber {

		private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|.....);

		static boolean isRomanNumeral(String s) {
				return ROMAN.matcher(s).matches();
		}
}
```

## 어댑터

불변 객체인 경우에 안전하게 재사용하는 것이 매우 명확하다. 하지만 몇몇 경우에 분명하지 않은 경우가 있다. 오히려 반대로 보이기도 한다.

어댑터를 예로 들면, 어댑터는 인터페이스를 통해서 뒤에 있는 객체로 연결해주는 객체라 여러개 만들 필요가 없다.

Map 인터페이스가 제공하는 keySet은 Map이 뒤에 있는 Set인터페이스의 뷰를 제공한다.
keySet을 호출할 때마다 새로운 객체가 나올 거 같지만 사실 같은 객체를 리턴하기 때문에 리턴 받은 Set타입의 객체를 변경하면, 결국 그 뒤에 있는 Map 객체를 변경하게 된다.

```java
public class UsingKeySet {
		
		public static void main(String[] args) {
				Map<String, Integer> menu = new HashMap<>();
				menu.put("Burger", 8);
				menu.put("Pizza", 9);

				Set<String> names1 = menu.keySet();
				Set<String> names2 = menu.keySet();

				names1.remove("Burger");
				System.out.println(names2.size()); // 1
				System.out.println(menu.size()); // 1
		}
}
```

## 오토박싱

불필요한 객체를 생성하는 또 다른 방법은 오토박싱이 있다.

오토박싱은 프로그래머가 프리미티브 타입과 박스 타입을 섞어 쓸 수 있게 해주고 박싱과 언박싱을 자동으로 해준다.

오토박싱은 프리미티브 타입과 박스 타입의 경계가 안보이게 해주지만 그렇다고 그 경계를 없애주진 않는다.

```java
public class AutoBoxingExample {
		
		public static void main(String[] args) {
				long start = System.currentTimeMillis();
				Long sum = 0l;
				for (long i = 0; i <= Integer.MAX_VALUE; i++) {
						sum += i;
				}

				System.out.println(sum);
				System.out.println(System.currentTimeMillis() - start);
		}
}
```

위 코드에서 sum변수의 타입을 Long으로 만들었기 때문에 불필요한 Long 객체를 2의 31제곱개 만큼 만들게 된다. 타입을 프리미티브 타입으로 바꾸면 훨씬 빠르다.

불필요한 오토박싱을 피하려면 박스 타입 보다는 프리미티브 타입을 사용해야 한다.