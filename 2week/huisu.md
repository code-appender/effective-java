# 객체 생성과 파괴

## ITEM 7 다 쓴 객체 참조를 취소하라

### 다 쓴 객체의 참조 취소

- 다 쓴 참조란 다시는 쓰지 않을 참조를 의미
- 활성 영역 밖의 참조들이 모두 여기 해당
- 활성 영역 밖의 참조 = null로 메모리를 해제시켜 줄 수 있음
    - 잘못해서 해제한 참조 데이터를 사용하려고 했을 때 NullPointerException을 발생시켜 예외 처리도 가능
    - 객체 참조를 null 처리하는 것은 예외적인 경우
- 가비지 컬렉터가 자동으로 회수하지 않는 경우 참조를 해제해 주지 않으면 메모리 누수 발생

### 객체 참조 해제 시 null 처리하는 경우

- 사용하는 데이터의 추가/삭제 유무를 가비지 컬렉터가 알 길이 없고 스스로가 메모리를 관리할 경우
- 비활성 영역의 객체가 더는 사용되지 않는 것을 프로그래머만 알고 있을 때
- 자기 메모리를 직접 관리하는 클래스

### 기타 메모리 누수의 원인

- 캐시
    - 객체 참조를 캐시에 넣고 잊은 경우
- 리스너/콜백
    - 콜백만 등록하고 해지하지 않는 경우

## ITEM 8 finalize와 cleaner 사용을 피하라

### 소멸자 finalize와 cleaner 사용을 피해야 하는 이유

- finalizer는 예측할 수 없고 상황에 따라 위험할 수 있어서
- cleaner는 예측 불가능하고 느려서
- finalizer와 cleaner는 즉시 수행된다는 보장이 없음 → finalizer와 cleaner로는 제때 실행되어야 하는 작업을 할 수 없음
- finalizer와 cleaner를 얼마나 신속히 수행할지는 가비지 컬렉터 알고리즘에 따라 달렸으며 이는 천차만별
- finalizer와 cleaner는 수행 여부조차 보장하지 않음 → 상태를 영구적으로 수정하는 작업에서는 절대 사용 불가능
- finalizer 동작 중 발생한 예외는 무시되며 처리할 작업이 남았더라도 즉시 종료됨
- finalizer와 cleaner는 성능 자체도 느림
- finalizer는 finalizer 공격에 노출되어 심각한 보안 문제 발생
    - 생성자가 직렬화 과정에서 예외가 발생하면 생성되다 만 객체에서 악의적인 하위 클ㄹ래스의 finalizer 수행
    - finalizer는 가비지 컬렉터가 수집하지 못하게 막음
    - 객체 생성을 막거나 일그러진 객체를 만들고 허용되지 않을 작업 실행

### finalizer와 cleaner의 사용 목적

- 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
    - 즉시 호출되리라는 보장은 없지만 클라이언트가 하지 않은 자원 회수를 늦게라도 실행
- 네이티브 피어와 연결된 객체
    - 네이티브 메서드를 통해 기능을 위임한 네이티브 객체는 자바 객체가 아니니 가비지 컬렉터가 존재를 알지 못함
    - finalizer와 cleaner로 해결
- cleaner를 안전망으로 활용하는 AutoCloseable 클래스

    ```java
    public class Room implements AutoCloseable {
        private static final Cleaner cleaner = Cleaner.create();
        private static class State implements Runnable {
            int numJunkPiles;
            
            State(int numJunkPiles) {
                this.numJunkPiles = numJunkPiles;
            }
    
            @Override
            public void run() {
                System.out.println("방 청소");
                numJunkPiles = 0;
            }
        }
        
        private final State state;
        private final Cleaner.Cleanable cleanable;
        
        public Room(int numJunkPiles) {
            state = new State(numJunkPiles);
            cleanable = cleaner.register(this, state);
        }
        
        @Override
        public void close() throws Exception {
            cleanable.clean();
        }
    }
    ```

    - Room의 close를 호출할 때 또는 가비지 컬렉터가 Room을 회수할 때까지 클라이언트가 close를 호출하지 않는다면 cleaner가 State의 run 메서드르 바라건데 호출
    - State는 절대로 Room을 참조해서는 안 됨 → 순환 참조가 생겨 가비지 컬렉터가 Room을 회수할 기회가 오지 않음

## ITEM 9 try-finally보다는 try-with-resources를 사용하라

### try-with-resources를 사용해야 하는 이유

- 자원이 둘이상이면 try-finally 방식은 지나치게 지저분
- 선행되는 try-finally 구문에서 예외가 발생하면 두 번째 예외에 대한 디버깅 불가능
    - 스택 추적 내ㅔ역에 첫 번째 예외와 관련된 정보감 난믹 때문
- try-with-resource 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기에도 좋음
- try-catch를 통해 try를 중첩하지 않고도 다수의 예외를 처리할 있음

# 모든 객체의 공통 메서드

## ITEM 10 equals는 일반 규약을 지켜 재정의하라

### equals의 일반 규약

- 각 인스턴스가 본질적으로 고유
- 인스턴스의 논리적 동치성을 검사할 일이 없음
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 적용
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다

### equals를 재정의해야 하는 경우

- 객체 식별성이 아닌 논리적 동치성을 확인해야 할 때
- 클래스들 자체적인 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
- 주로 값 클래스들 해당
    - 같은 객체인지가 아닌 값이 같은지를 비교하고 싶을 때
    - 재정의를 해 두면 값 비교 + Map, Set 활용 가능

### equals를 재정의하지 않아도 되는 경우

- 같은 인스턴스가 둘 이상 만들어지지 않음이 보장될 때
- 인스턴스 통제 클래스, Enum 클래스

### equals는 동치 관계 (equivalence relation) 구현

- 반사성(reflexivity): null이 아닌 모든 참조값 x에 대해 x.equals(x)는 true
- 대칭성(symmetry): null이 아닌 모든 참조값 x, y에 대해 x.equals(y)가 true면
  y, equals(x) true
- 추이성(transitivity): null이 아닌 모든 참조값 x, y, z에 대해 x.equals(y)가 true이
  고 y.equals(z)도 true면 x.equals(z)도 true
- 일관성(consistency): null이 아닌 모든 참조값 x, y에 대해 x.equals(y)를 반복해서
  호출하면 항상 true를 반환하거나 항상 false를 반환
- nuLL-아님: null이 아닌 모든 참조값 x에 대해 x.equals(null)은 false

### equals 메서드 구현 단계

- == 연산자를 통해 입력이 자기 자신의 참조인지 확인 후 맞으면 true
- instanceof 연산자로 입력이 올바른지 확인
- 입력을 올바른 타입으로 형변환
- 입력 객체와 자기 자신의 대응되는 핵심 필드가 모두 일치하는지 확인
    - 기본 타입 필드 == 연산자
    - 참조 타입 필드 equals 메서드
    - float와 double은 부동소수점 계산을 위해 Float.compare(float, float), Double.compare(double, double) 사용

### equals 구현에서 주의할 점

- equals를 재정의할 때는 hashCode도 반드시 재정의
- Object 이외의 타입을 매개변수로 받는 equals 메서드는 선언 X
- 대칭적인지 추이성이 있는지 일관적인지 검사

## ITEM 11 equals를 재정의하려거든 hashCode도 재정의하자

### hashCode 관련 Object 명세

- equals 비교에 사용되는 정보가 변경되지 않았다면 애플리케이션이 실행되는 동안 그 객체의 hashcode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환
    - 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없음
- equals가 두 객체를 같다고 판단했다면 두 객체의 hashCode는 똑같은 값을 반환
- equals가 두 객체를 다르다고 판단했더라도 두 객체의 hashcode가 서로 다른 값을 반환할 필요는 없음.
    - 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 향상

### 최악의 hashCode

```java
@Override
public int hashCode() {
	return 42;
}
```

- 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법
- 하지만 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트처럼 동작
- 그 결과 평균 수행 기간이 O(n)으로 느려짐

### 좋은 hashCode 작성법

- int result = c로 초기화
    - c는 객체의 첫 번째 핵심 필드를 다음과 같이 계산한 것
    - 기본 타입 필드라면 Type.hashCode(c)
    - 참조 타입 필드라면 이 필드의 표준형을 만들어 그 표준형의 hashCode 호출
    - 필드가 배열이라면 핵심 원소 각각을 별도 필드처럼 다룸
        - 모든 원소가 핵심 원소라면 Arrays.hashCode 사용
- 해당 객체의 나머지 핵심 필드에 대해서도 위 작업을 적용해 다음과 같이 갱신한다
    - result = 31 * result + c;
- result를 반환한다
- 예시

    ```java
    public final class PhoneNumber {
        private final short areaCode, prefix, lineNum;
    
        public PhoneNumber(short areaCode, short prefix, short lineNum) {
            this.areaCode = areaCode;
            this.prefix = prefix;
            this.lineNum = lineNum;
        }
        
        @Override
        public boolean equals(Object o) {
            if (o == this) return true;
            if (!(o instanceof PhoneNumber)) return false;
            PhoneNumber phoneNumber = (PhoneNumber) o;
            return (phoneNumber.areaCode == areaCode) &&
                    (phoneNumber.prefix == prefix) &&
                    (phoneNumber.lineNum == lineNum);
        }
        
        @Override
        public int hashCode() {
            int result = Short.hashCode(areaCode);
            result = 31 * result + Short.hashCode(prefix);
            result = 31 * result + Short.hashCode(lineNum);
            return result;
        }
    }
    ```


### hashCode 작성 주의점

- 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 . 안됨
- hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세하게 공표하지 말아야 함
- 그래야 추후에 계산 방식을 바꿀 있음

## ITEM 12 toString을 항상 재정의하라

### toString

- toString의 규약은 모든 하위 클래스에서 재정의하라
- toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고 디버깅 간편
- 그 객체가 가진 주요 정보 모두를 반환하는 것이 바람직
- 포맷을  명시하든 아니든 프로그래머의 의도는 명확하게 밝혀야 함
- 포맷에 의한 변동이 걱정된다면 포맷 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어 올 . 수있는 API 제공