# [Effective Java] item 29. 이왕이면 제네릭 타입으로 만들라

## 핵심 정리

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.
- 그러니 `새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있게 하라.` 그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다.
- `기존 타입 중 제네릭이었어야 하는 개체가 있다면 제네릭 타입으로 변경하자.` 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.

### 제네릭 타입을 왜 사용해야 하는가?

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다. 그래도 배워두면 그만한 값어치는 충분히 한다.

아이템 7에서 다룬 단순한 스택 코드를 다시 살펴보자.

#### Object 기반 스택 - 제네릭이 절실한 강력 후보!
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

   // Corrected version of pop method (Page 27)
   public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // Eliminate obsolete reference
       return result;
   }
}
```

이 클래스는 원래 제네릭 타입이어야 마땅하다. 그러나 제네릭으로 만들어보자. 이 클래스를 제네릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다. 오히려 지금 상태에서의 클라이언트는 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 위험이 있다.

일반 클래스를 제네릭 클래스로 만드는 첫 단계는 `클래스 선언에 타입 매개변수를 추가하는 일이다.`. 위의 예에서는 스택이 담을 원소의 타입 하나만 추가하면 된다. 이때 타입 이름으로는 보통 E를 사용한다.

그런 다음 코드에 쓰인 Object를 적절한 타입 매개변수로 바꾸고 컴파일해보자.

#### 제네릭 스택으로 가는 첫 단계 - 컴파일되지 않는다.
```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }
}
```

이 단계에서 대체로 하나 이상의 오류나 경고가 뜨는데, 이 클래스도 예외는 아니다. 여기서는 다행히 오류가 하나만 발생했다.

![item29_1](https://user-images.githubusercontent.com/37948906/107220849-30007480-6a56-11eb-93c4-9ffad3082bfd.PNG)

아이템 28에서 설명한 것처럼, `E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.` 배열을 사용하는 코드를 제네릭으로 만들려 할 때는 이 문제가 항상 발목을 잡을 것이다.

적절한 해결책은 두 가지다.

### 1. Object 배열을 생성한 다음 제네릭 배열로 형변환을 한다.
이 방법은 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다. 이제 컴파일러는 오류 대신 경고를 내보낼 것이다. 이렇게도 할 수는 있지만 (일반적으로) 타입 안전하지 않다.
![item29_2](https://user-images.githubusercontent.com/37948906/107221249-b9b04200-6a56-11eb-8436-4e5edc46415b.PNG)

컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없지만 우리는 할 수 있다. 따라서 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 우리 스스로 확인해야 한다. 문제의 배열 elements는 private 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 확실히 안전하다.

비검사 형변환이 안전함을 증명했다면 범위를 최소로 좁혀 @SuppressWarning 애너테이션으로 해당 경고를 숨긴다. 이 예에서는 생성자가 비검사 배열 생성 말고는 하는 일이 없으니 생성자 전체에서 경고를 숨겨도 좋다. 애너테이션을 달면 Stack은 깔끔히 컴파일되고, 명시적으로 형변환하지 않아도 ClassCastException 걱정 없이 사용할 수 있다.

![item29_3](https://user-images.githubusercontent.com/37948906/107222043-c5e8cf00-6a57-11eb-8c1d-7acb1a9c5faf.PNG)

### 2. elements 필드의 타입을 E[]에서 Object[]로 바꾼다.

제네릭 배열 생성 오류를 해결하는 두 번째 방법은 elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것이다. 이렇게 하면 첫 번째와는 다른 오류가 발생한다.

![item29_4_1](https://user-images.githubusercontent.com/37948906/107223116-24627d00-6a59-11eb-90cf-cc949e06d3db.PNG)

배열이 반환한 원소를 E로 형변환하면 오류 대신 경고가 뜬다.

![item29_4](https://user-images.githubusercontent.com/37948906/107223023-04cb5480-6a59-11eb-9a51-8048852d9a67.PNG)

E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 
이번에도 마찬가지로 우리가 직접 증명하고 경고를 숨길 수 있다. pop 메서드 전체에서 경고를 숨기지 말고, 아이템 27의 조언을 따라 비검사 형변환을 수행하는 할당문에서만 숨겨보자.

![item29_5](https://user-images.githubusercontent.com/37948906/107223522-9e930180-6a59-11eb-9f26-b16c062ada14.PNG)

---

제네릭 배열 생성을 제거하는 두 방법 모두 나름의 지지를 얻고 있다.

첫 번째 방법은 가독성이 더 좋다. 배열의 타입을 E[]로 선언하여 오직 E타입 인스턴스만 받음을 확실히 어필한다. 코드도 더 짧다. 보통의 제네릭 클래스라면 코드 이곳저곳에서 이 배열을 자주 사용할 것이다.

첫 번째 방식에서는 형변환을 배열 생성 시 단 한번만 해주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다. 따라서 현업에서는 첫 번째 방식을 더 선호하며 자주 사용한다. 하지만 (E가 Object가 아닌 한) 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(heap pollution)을 일으킨다. 힙 오염이 맘에 걸리는 프로그래머는 두 번째 방식을 고수하기도 한다. (이번 예시에서는 힙 오염이 해가 되지 않았다.)

---

다음은 명령줄 인수들을 역순으로 바꿔 대문자로 출력하는 프로그램으로, 방금 만든 제네릭 Stack 클래스를 사용하는 모습을 보여준다. Stack에서 꺼낸 원소에서 String의 toUpperCase 메서드를 호출할 때 명시적으로 형변환을 수행하지 않으며, (컴파일러에 의해 자동 생성된) 이 형변환이 항상 성공함을 보장한다.

#### 제네릭 Stack을 사용하는 맛보기 프로그램
```java
public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for (String arg : args)
        stack.push(arg);
    while (!stack.isEmpty())
        System.out.println(stack.pop().toUpperCase());
}
```

지금까지 설명한 Stack 예는 "배열보다는 리스트를 우선하라"는 아이템 28과 모순돼 보인다. 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다.

자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList와 같은 제네릭 타입도 결국 기본 타입인 배열을 사용해 구현해야 한다. 또한 HashMap같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

Stack 예처럼 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다. `Stack<Object>, Stack<int[]>, Stack<List<String>>, Stack` 등 어떤 참조 타입으로도 Stack을 만들 수 있다. 단, 기본 타입은 사용할 수 없다. `Stack<int>`나 `Stack<double>`을 만들려고 하면 오류가 난다. 이는 자바 제네릭 타입 시스템의 근본적인 문제이나, 박싱된 기본 타입을 사용해 우회할 수 있다.

타입 매개변수에 제약을 두는 제네릭 타입도 있다. 예컨대 java.util.concurrent.DelayQueue는 다음처럼 선언되어 있다.

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

타입 매개변수 목록인 `<E extends Delayed>`는 java.util.concurrent.Delayed의 하위 타입만 받는다는 뜻이다. 이렇게 하여 DelayQueue 자신과 DelayaQueue를 사용하는 클라이언트는 DelayQueue의 원소에서 (형변환 없이) 곧바로 Delayed 클래스의 메서드를 호출할 수 있다. ClassCastException 걱정은 할 필요가 없다. 이러한 타입 매개변수 E를 한정적 타입 매개변수(bounded type parameter)라 한다. 그리고 모든 타입은 자기 자신의 하위 타입이므로 `DelayQueue<Delayed>`로도 사용할 수 있음을 기억해두자.

---

### 참고 자료
- Effective Java 3/E