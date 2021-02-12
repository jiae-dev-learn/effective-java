# [Effective Java] Item 6. 불필요한 객체 생성을 피하라

---

### 예시

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
             + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral (String s) {
        return ROMAN.matcher(s).matches();
    }
    )
}
```

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다. 재사용은 빠르고 세련되어 특히 불변 객체는 언제든 재사용할 수 있다.

다음 코드는 하지 말아야할 극단적인 예시이다.

`String s = new String("bikini");` **// 따라 하지 말 것**

이 문장은 실행될 때마다 String 인스턴스를 새로 만든다. 완전히 쓸데없는 행위다. 생성자에 넘겨진 "Bikini" 자체가 이 생성자로 만들어내려는 String과 기능적으로 완전히 똑같다. 이 문장이 반복문이나 빈번히 호출되는 메서드 안에 있다면 쓸데없는 String 인스턴스가 수백만 개 만들어질 수도 있다.

개선된 코드를 보자.

`String s = "bikini"`

이 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 나아가 이 방식을 사용한다면 같은 가상머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.

생성자 대신 정적 팩터리 메서드를 제고하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다. 예컨대 Boolean(String) 생성자 대신 Boolean.valueOf(String) 팩터리 메서드를 사용하는 것이 좋다. (그래서 이 생성자는 자바 9에서 사용 자제 API로 지정되었다). 생성자는 호출될 때마다 새로운 객체를 만들지만, 팩터리 메서드는 전혀 그렇지 않다. 불변 객체만이 아니라 가변 객체라 해도 사용 중에 변경되지 않을 것임을 안다면 재사용할 수 있다.
 생성 비용이 아주 비싼 객체도 더러 있다. 이런 '비싼 객체'가 반복해서 필요하면 캐싱하여 재사용하길 권한다. 안타깝게도 자신이 만드는 객체가 비싼 객체인지를 매번 명확히 알 수는 없다. 예를 들어 주어진 ***문자열이 유효한 로마 숫자인지를 확인하는 메서드***를 작성한다고 해보자. 다음은 정규표현식을 활용한 가장 쉬운 해법이다.

 ```java
 static boolean isRomanNumeral(String s) {
     return s.matchs("^(?=.)M*(C[MD]|D?C{0,3})"
             + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
 }
 ```
<details>
<summary>String.matchs 구조</summary>
<div markdown="1">

**String.java**
```java
public boolean matches(String regex) {
        return Pattern.matches(regex, this);
    }
```

**Pattern.java**
```java
public static boolean matches(String regex, CharSequence input) {
        Pattern p = Pattern.compile(regex);
        Matcher m = p.matcher(input);
        return m.matches();
    }
```

**Pattern.java**
```java
public static Pattern compile(String regex) {
        return new Pattern(regex, 0);
    }
```

</div>
</details>

 이 방식의 문제는 String.matchs 메서드를 사용한다는 데 있다. `String.matchss는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않다.` 이 메서드가 내부에서 만드는 정규표현식용 Pattern 인스턴스는, 한 번 쓰고 버려져서 곧바로 가비지 컬렉션 대상이 된다. Pattern은 입력받은 정규표현식에 해당하는 유한상태 머신(finite state marchine)을 만들기 때문에 인스턴스 생성 비용이 높다.

성능을 개선하려면 필요한 정규표현식을 표현하는 (불변인) Pattern 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용한다.

**값비싼 객체를 재사용하는 개선된 코드**
```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
             + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral (String s) {
        return ROMAN.matcher(s).matches();
    }
    )
}
```

이렇게 개선하면 isRomanNumeral이 빈번히 호출되는 상황에서 성능을 상당히 끌어올릴 수 있다. 내 컴퓨터에서 길이라 8인 문자열을 입력햇을 때 개선 전에는 1.1us, 개선 후에는 0.17us가 걸렸다. 6.5배 정도 빨라진 것이다. 성능만 좋아진 것이 아니라 코드도 더 명확해졌다. 개선 전에는 존재조차 몰랐던 Pattern 인스턴스를 static final 필드로 끄집어내고 이름을 지어주어 코드의 의미가 훨씬 잘 드러난다.

개선된 isRomanNumeral 방식의 클래스가 초기화된 후 이 메서드를 한 번도 호출하지 않았다면 ROMAN 필드는 쓸데없이 초디화된 꼴이다. isROmanNumeral 메서드가 처음 호출될 때 필드를 초기화하는 지연 초기화 (lazy initialization)로 불필요한 초기화를 없앨 수 있지만, 권하지는 않는다. 지연 초기화는 코드를 복잡하게 만드는데, 성능은 크게 개선되지 않을 때가 많기 때문이다.

객체가 불변이라면 재사용해도 안전함이 명백하다. 하지만 훨씬 덜 명확하거나, 심지어 직관에 반대되는 상황도 있다. 어댑터를 생각해보자. (어댑터를 뷰라고도 한다) 어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체다. 어댑터는 뒷단 객체만 관리하면 된다. 즉, 뒷단 객체 외에는 관리할 상태가 없으므로 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분한다. 

예컨대 Map 인터페이스의 KeySet 메서드는 Map 객체 안의 키 전부를 담은 Set 뷰를 반환한다. KeySet을 호출할 때마다 새로운 set 인스턴스가 만들어지리라고 순진하게 생각할 수도 있지만, 사실은 매번 같은 Set 인스턴스를 반환할지도 모른다. 반환한 Set 인스턴스가 일반적으로 가변이더라도 반환된 인스턴스들은 기능적으로 모두 똑같다. 즉, 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀐다. 모두가 똑같은 Map 인스턴스를 대변하기 때문이다. 따라서 KeySet이 뷰 객체를 여러 개 만들어도 상관 없지만, 그럴 필요도 없고, 이득도 없다.

불필요한 객체를 만들어내는 또 다른 예로 오토박싱(auto boxing)을 들 수 있다. `오토박싱은 프로그래머가 기본타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술`이다. `오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다. 의미 상으로는 별다를 것 없지만 성능에서는 그렇지 않다.` 다음 메서드를 보자. 모든 양의 정수의 총합을 구하는 메서드로, int는 충분히 크지 않으니 long을 사용해 개선하고 있다.

**끔찍하게 느린 코드**
```java
private static long sum(){
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

이 프로그램이 정확한 답을 내기는 하지만, 제대로 구현했을 때보다 훨씬 느리다. 겨우 문자 하나 때문에 말이다. sum 변수를 long이 아닌 Long으로 선언해서 불필요한 Long 인스턴스가 231개나 만들어진 것이다. (대략, long 타입인 i가 Long 타입인 sum에 더해질 때마다). 단순히 sum의 타입을 long으로만 바꿔주면 내 컴퓨터에서는 6.3초에서 0.59초로 빨라진다. 교훈은 명확하다.

`박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.`

이번 아이템을 "객체 생성은 비싸니 피해야 한다"로 오해하면 안 된다. 특히 요즘의 JVM에서는 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않는다. 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일이다.

거꾸로, 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 여러분만의 객체 풀(pool)을 만들지는 말자. 물론 객체 풀을 만드는게 나은 예가 있긴 하다. 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸니 재사용하는 편이 낫다. 하지만 일반적으로 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨린다. 요즘 JVM의 가비지 컬렉터는 상당히 잘 최적화되어서 가벼운 객체용을 다룰 때는 직접 만든 객체 풀보다 훨씬 빠르다.

이번 아이템은 `방어적 복사(defensive copy)`를 다루는 아이템 50과는 대조적이다. 이번 아이템이 `기본 객체를 재사용해야 한다면 새로운 객체를 만들지 말자`라면, 아이템 50은 `새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라`다. 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자. 방어적 복사에 실패하면 언제 터져 나올지 모르는 버그와 보안 구멍으로 이어지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 준다.

---

### 관련 자료

1. 불변객체 - item 17
2. 지연 초기화 (lazy initialization) - item 83
3. 어댑터 - gramma 95


### 참고 자료
- Effective Java 3/E