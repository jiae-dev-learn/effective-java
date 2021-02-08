# [Effective Java] Item 17. 변경 가능성을 최소화하라

### 불변 클래스

- 인스턴스의 내부 값을 수정할 수 없는 클래스
- 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다.
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.
- 예) String, 기본 타입의 박싱된 클래스들, BigInteger, BigDecimal

### 불변 클래스로 만들기 위한 다섯 가지 규칙

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
    - 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아준다. 상속을 막는 대표적인 방법은 클래스를 final로 선언하는 것이지만, 다른 방법도 뒤에 살펴볼 예정이다.
- 모든 필드를 final로 선언한다.
    - 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법이다. 새로 생성된 인스턴스를 동기화 없이 다른 스레드로 건너도 문제없이 동작하게끔 보장하는 데도 필요하다.
- 모든 필드를 private로 선언한다.
    - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아준다. 기술적으로는 기본 타입 필드나 불변 객체를 참조하는 필드를 public final로만 선언해도 불변 객체가 되지만, 이렇게 하면 다음 릴리즈에서 내부 표현을 바꾸지 못하므로 권하지는 못한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
    - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 한다. 이런 필드는 절대 클라이언트가 제공한 객체 참조를 가리키게 해서는 안되며, 접근자 메서드가 그 필드를 그대로 반환해서도 안된다. 생성자, 접근자, readObject 메서드 모두에서 방어적 복사를 수행하라.

---

### 불변 클래스 예시

#### 불변 복소수 클래스

```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}

```

사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환하고 있다. 이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 함수형 프로그래밍이라 한다.

이와 달리, 절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다.

메서드의 이름을 동사(add 같은) 대신 전치사(plus 같은)를 사용한 것도 해당 메서드가 객체 의 값을 변경하지 않는다는 사실을 강조하려는 의도다.

---

`불변 객체는 단순하다. 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.`

모든 생성자가 클래스 불변식을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다. 반면 가변 객체는 임의의 복잡한 상태에 놓일 수 있다.

`불변 객체는 근본적으로 스레드 안전하며 따로 동기화할 필요도 없다.`


여러 스레드가 동시에 사용해도 절대 훼손되지 않는다. 사실 클래스를 스레드 안전하게 만드는 가장 쉬운 방법이기도 하다. 불변 객체는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 `불변 객체는 안심하고 공유할 수 있다.`

따라서 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하는 것이 좋다. 가장 쉬운 재활용 방법은 자주 쓰이는 값들을 상수 (public static final)로 제공하는 것이다.

예컨대 Complex 클래스는 다음 상수들을 제공할 수 있다.

```java
    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);
```

`불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.`

박싱된 기본 타입 클래스 전부와 BigInteger가 여기 속한다. 이런 정적 팩터리를 사용하면 여러 클래스가 인스턴스를 공유하여 메모리 사용량과 가비지 컬렉션 비용이 줄어든다. 새로운 클래스를 설계할 때 public 생성자 대신 정적 팩터리를 만들어주면 클라이언트를 수정하지 않고도 필요에 따라 캐시 기능을 나중에 덧붙일 수 있다.

`불변 객체를 자유롭게 공유할 수 있다는 점은 방어적 복사도 필요 없다는 결론으로 자연스럽게 이어진다. 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없다. 그러니 불변 클래스는 clone 메서드나 복사 생성자를 제공하지 않는게 좋다.`

`불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다.`

예컨대 BigInteger 클래스는 내부에서 값의 부호(sign)와 크기(magnitude)를 따로 표현한다. 부호에는 int 변수를, 크기(절댓값)에는 int 배열을 사용한다는 것이다. 한편 negate 메서드는 크기가 같고 부호만 반대인 새로운 BigInteger를 생성하는데, 이때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다. 그 결과 새로 만든 BigInteger 인스턴스도 원본 인스턴스가 가리키는 내부 배열을 그대로 가리킨다.

`객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.` 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문이다. 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문이다.

좋은 예로, 불변 객체는 맵의 키와 집합(Set)의 원소로 쓰기에 안성맞춤이다. 맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다.

`불변 객체는 그 자체로 실패 원자성을 제공한다.` 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다.

`불변 클래스에도 단점은 있다. 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.` 값의 가지수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다. 예컨대 백만 비트짜리 BigInteger에서 비트 하나를 바꿔야 한다고 해보자.

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

flipBit 메서드는 새로운 BigInteger 인스턴스를 생성한다. 원본과 단지 한 비트만 다른 백만 비트짜리 인스턴슬르 말이다. 이 연산은 BigInteger의 크기에 비례해 시간과 공간을 잡아먹는다 BigSet도 BigInteger처럼 임의 길이의 비트 순열을 표현하지만, BigInteger와는 달리 '가변'이다. BigSet 클래스는 원하는 비트 하나만 상수 시간 안에 바꿔주는 메서드를 제공한다.

```java
BitSet moby = ...;
moby.flip(0);
```

원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 더 불거진다. 이 문제에 대처하는 방법은 두 가지다. 첫 번째는 흔히 쓰일 다단계 연산들을 예측하여 기본 기능으로 제공하는 방법이다. 이러한 다단계 연산을 기본으로 제공한다면 더 이상 단계마다 객체를 생성하지 않아도 된다. 불변 객체는 내부적으로 아주 영리한 방식으로 구현할 수 있기 때문이다. 예컨대 BigInteger는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스를 package-private으로 두고 있다. 앞서 이야기한 이유들로, 이 가변 동반 클래스를 사용하기란 BigInteger를 쓰는 것보다 훨씬 어렵다. 그래도 우리는 운이 좋다. 그 어려운 부분을 모두 BigInteger가 대신 처리해주니 말이다.

클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있다면 package-private의 가변 동반 클래스만으로 충분하다. 그렇지 않다면 이 클래스를 public으로 제공하는 게 최선이다. 자바 플랫폼 라이브러리에서 이에 해당하는 대표적인 예가 바로 String 클래스다. 그렇다면 String의 가변 동반 클래스는? 바로 StringBuilder(와 구닥다리 전임자 StringBuffer)다.

---

이상으로 불변 클래스를 만드는 기본적인 방법과 불변 클래스의 장단점을 알아보았다. 그 다음으로 불변 클래스를 만드는 또 다른 설계 방법 몇 가지를 알아볼 차례다. 클래스가 불변임을 보장하려면 자신을 상속하지 못하게 함을 기억하는가? 자신을 상속하지 못하게 하는 가장 쉬운 방법은 final 클래스로 선언하는 것이지만, 더 유연한 방법이 있다. `모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공하는 방법이다.`

#### 생성자 대신 정적 팩터리를 사용한 불변 클래스

```java
public final class Complex {
    private final double re;
    private final double im;
    
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    // Static factory, used in conjunction with private constructor (Page 85)
    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
    ...
```

신뢰할 수 없는 하위 클래스와 인스턴스라고 확인되면, 이 인수들은 가변이라 가정하고 방어적으로 복사해 사용해야 한다.

```java
public static BigInteger safeInstance(BigInteger val) {
    return val.getclass() == BigInteger.class ?
        val : new BigInteger(val.toByteArray());
}
```

이번 아이템의 초입에서 나열한 불변 클래스의 규칙 목록에 따르면 모든 필드는 final이고 어떤 메서드도 그 객체를 수정할 수 없어야 한다. 사실 이 규칙은 좀 과한 감이 있어서, 성능을 위해 다음처럼 살짝 완화할 수 있다.

`어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다.`

어떤 불변 클래스는 계산 비용이 큰 값을 나중에 (처음 쓰일 때) 계산하여 final이 아닌 필드에 캐시해놓기도 한다. 똑같은 값을 다시 요청하면 캐시해둔 값을 반환하여 계산 비용을 절감하는 것이다. 이 묘수는 순전히 그 객체가 불변이기 때문에 부릴 수 있는데, 몇 번을 계산해도 항상 같은 결과가 만들어짐이 보장되기 때문이다.

`게터(getter)가 있다고 해서 무조건 세터(setter)를 만들지는 말자. 클래스는 꼭 필요한 경우가 아니면 불변이어야 한다.` 불변 클래스는 장점이 많으며, 단점이라곤 특정 상황에서의 잠재적 성능 저하 뿐이다. PhoneNumber와 Complex 같은 단순 값 객체는 항상 불변으로 만들자.

String와 BigInteger처럼 무거운 값 객체도 불변으로 만들 수 있는지 고심해야 한다. 성능 때문에 어쩔 수 없다면 불변 클래스와 쌍을 이루는 동반 클래스를 public 클래스로 제공하돌고 하자.

한편, 모든 클래스를 불변으로 만들 수는 없다. `불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분은 최소한으로 줄이자.` 객체가 가질 수 있는 상태의 수를 줄이면 그 객체를 예측하기 쉬워지고 오류가 생길 가능성이 줄어든다. 그러니 꼭 변경해야할 필드를 뺀 나머지 모두를 final로 선언하자.

#### 1. 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
#### 2. 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다. 

확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public으로 제공해서는 안된다. 객체를 재활용할 목적으로 상태를 다시 초기화하는 메서드도 안된다. 복잡성만 커지고 성능 이점은 거의 없다.

java.util.concurrent 패키지의 CountDownLatch 클래스가 이상의 원칙을 잘 방증한다. 비록 가변 클래스지만 가질 수 있는 상태의 수가 많지 않다. 인스턴스를 생성해 한 번 사용하고 그걸로 끝이다. 카운트가 0에 도달하면 더는 재사용할 수 없는 것이다.