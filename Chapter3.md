# Chapter3

# 1. 람다란 무엇인가?

---

람다표현식은 익명클래스처럼 이름이 없는 함수이면서, 메서드를 인수로 전달할 수 있다.

람다표현식은 메서드로 전달할 수 있는 익명함수를 단순화한 것이다.

람다표현식에는 이름은 없지만, `파라미터 리스트, 바디, 반환형식, 발생할 수 있는 예외리스트`를 가질 수 있다.



람다의 특징

-   **익명**
    -   일반적인 메서드와 다르게 이름이 없으므로 익명이라고 한다.
-   **함수**
    -   람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다.
    -   하지만 메서드처럼 메서드처럼 `파라미터 리스트, 바디, 반환형식, 가능한 예외리스트`를 포함한다.
-   **전달**
    -   람다표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
-   **간결성**
    -   익명 클래스처럼 자질구레한 코드를 구현할 필요가 없다.



람다표현식이 중요한 이유는 코드를 전달하는 과정에서 자질구레한 코드를 간결한 방식으로 나타낼 수 있다는 것이다.

기술적으로 자바 8 이전의 자바로 할 수 없었던 일을 제공하는 것은 아니지만 동적 파라미터를 이용할 때 익명 클래스 등 판에 박힌 코드를 구현할 필요가 없다.

**결과적으로 코드가 간결하고 유연해진다.**

예를들어 Comparator 객체를 람다로 구현해보자.

```java
// 익명클래스를 이용한 기존 코드
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
};
```

```java
// 람다를 이용한 코드
Comparator<Apple> byWeight =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

이처럼 코드가 훨씬 간단해졌다.



람다는 세 부분으로 이루어진다.

`(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());`

-   **파라미터 리스트**

    Comparator의 compare 메서드 파라미터(사과 두 개)

-   **화살표**

    화살표(`->`)는 람다의 파라미터 리스트와 바디를 구분한다.

-   **람다 바디**

    두 사과의 무게를 비교한다. 람다의 `반환값`에 해당하는 표현식이다.



다섯가지 람다 표현식의 예제를 보자.

-   ```java
    (String s) -> s.length()
    ```

​		String 형식의 파라미터 하나를 가지며 int를 반환한다.
​		람다 표현식에는 `return`문이 함축되어 있으므로 `return`문을 명시적으로 사용하지 않아도 된다.

-   ```java
    (Apple a) -> a.getWeight() > 150
    ```

​		Apple 형식의 파라미터 하나를 가지며 boolean을 반환한다.

-   ```java
    (int x, int y) -> {
        System.out.println("Result:");
        System.out.println(x + y);
    }
    ```

​		int 형식의 파라미터 두 개를 가지며 리턴값이 없다.(void 리턴)
​		람다 표현식은 여러 행의 문장을 포함할 수 있다.

-   ```java
    () -> 42
    ```

​		파라미터가 없으며 int 42를 반환한다.

-   ```java
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
    ```

​		Apple 형식의 파라미터 두 개를 가지며 int(두 사과의 무게 비교 결과)를 반환한다.



람다의 기본 문법은

`(parameters) -> expression` 
`(parametres) -> { statements; }`

로 표현할 수 있다.



이 때 expression은 표현식 이므로 block안에 올 수 없고,
statements는 구문이므로 block안에서 존재해야 한다.

>   **잘못된표현** : `(Integer i) -> return "Alan" + i`
>
>   **올바른표현** : `(Integer i) -> {return "Alan" + 1;}`
>
>   
>
>   **잘못된표현** : `(String s) -> {"Iron Man";}`
>
>   **올바른표현** : `(String s) -> "Iron man"`, `(String s) -> {return "Iron Man";})\`



# 2. 어디에, 어떻게 람다를 사용할까?

---

그래서 람다표현식을 어디에 사용할 수 있는걸까?

함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

```java
List<Apple> greenApples =
    filter(inventory, (Apple a) -> GREEN.equals(a.getColor()));
```

위 예제에서는 함수형 인터페이스인 **`Predicate<T>`**를 기대하는 `filter` 메서드와 두 번째 인수로 람다표현식을 전달했다.

## 함수형 인터페이스란?

2장에서 `Predicate<T>` 인터페이스로 필터 메서드를 파라미터화 할 수 있었는데 여기서 이 `Predicate<T>`가 함수형 인터페이스이다. 

함수형 인터페이스인 `Predicate<T>`는 오직 하나의 추상 메서드만 지정하기 때문이다.

```java
public interface Predicate<T> {
	boolean test (T t);
}
```

**다시 말해 함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스이다.**



지금 까지 살펴본 자바 API의 함수형 인터페이스로는 `Comparator, Runnable` 등이 있다.

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Runnable {
    void run();
}

public interface ActionListener extends EventListener {
    void actionPerformed(ActionEvent e);
}

public interface Callable<V> {
    V call() throws Exception;
}

public interface PrivilegedAction<T> {
    T run();
}
```

>   9장에서 더 보겠지만 인터페이스는 **디폴트 메서드(default method): 인터페이스의 메서드를 구현하지 않은 클래스를 고려하여 기본 구현을 제공하는 바디를 포함하는 메서드** 를 포함할 수 있다.
>
>   많은 디폴트 메서드가 있더라도 추상 메서드가 오직 하나면 함수형 인터페이스이다.



>   **Quiz**
>
>   ```java
>   public interface Adder {
>       int add(int a, int b);
>   }
>   
>   public interface SmartAdder extends Adder {
>       int add(double a, double b);
>   }
>   
>   public interface Nothing {
>       
>   }
>   ```
>
>   여기서 오로지 `Adder`만 함수형 인터페이스이다.
>
>   SmartAdder는 두 추상 add메서드(하나는 Adder에서 상속받음)를 포함하므로 함수형 인터페이스가 아니다.
>
>   Nothing은 추상 메서드가 없으므로 함수형 인터페이스가 아니다.



이 함수형 인터페이스로 무엇을 할 수 있을까?

람다 표현식으로 함수형 인터페이스의 추상메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**할 수 있다.

함수형 인터페이스보다는 덜 깔끔하지만 익명 내부클래스로도 같은 기능을 구현할 수 있다.

```java
// 람다 사용
Runnable r1 = () -> System.out.println("Hello World 1");

// 익명 클래스 사용
Runnable r2 = new Runnable() {
    public void run() {
        System.out.println("Hello World 2");
    }
}

public static void process(Runnable r) {
    r.run();
}
process(r1); // Hello World 1
process(r2); // Hello World 2
process(() -> System.out.println("Hello World 3")); // Hello World 3
```



## 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.

이 때 **람다표현식의 시그니처를 서술하는 메서드를 함수 디스크립터**라고 부른다.



예를 들어 Runnable 인터페이스의 유일한 추상 메서드 run은 인수와 반환값이 없으므로 Runnable 인터페이스는 인수와 반한값이 없는 시그니처로 생각할 수 있다.



```java
// 이는 정상적인 람다 표현이다.
process(() -> System.out.println("This is awesome"));

process(() -> { System.out.println("This is awesome"); });
// 한 개의 void 메소드 호출은 중괄호로 감쌀 필요가 없다.
```



***@FunctionalInterface***

새로운 자바 API를 살펴보면 함수형 인터페이스에 `@FunctioinalInterface` 어노테이션이 추가되어 있다.

이는 함수형인터페이스를 가리키는 어노테이션이다.

`@FunctioinalInterface`로 인터페이스를 선언했지만 실제로 함수형 인터페이스가 아니면 컴파일러가 에러를 발생시킨다.

예를 들어 추상메서드가 한 개 이상이라면 "Multiple nonoverriding abstract methods found in interface Foo"와 같은 에러가 발생할 수 있다.

# 3. 람다 활용: 실행 어라운드 패턴

---

람다와 동작 파라미터화로 유연하고 간결한 코드를 구현하는 데 도움을 주는 실용적인 예제를 보자.

예를 들어 데이터베이스의 파일처리에 사용하는 순환패턴(`recurrent Pattern`)은 자원을 열고 처리한 다음에 자원을 닫는 순서로 이루어진다.

설정과 정리 과정은 대부분 비슷하기 때문에 실제 자원을 처리하는 코드를 **설정과 정리 두 과정이 둘러싸는 형태**를 갖는데 이를 **실행 어라운드 패턴**이라고 한다.

이를 사용하면 자원을 명시적으로 닫을 필요가 없으므로 간결한 코드를 구현하는데 도움을 준다.

```java
public String processFile() throws IOException {
    try (BufferedReader br = 
        	new BufferedReader(new FiledReader("data.txt"))) {
        return br.readLine(); // 실제 필요한 작업을 하는 행
    }
}
```



## 1단계: 동작 파라미터화를 기억하라

현재 위의 코드에서는 파일에서 한번에 한줄만 읽을 수 있다.

한 번에 두줄을 읽거나 가장 자주 사용되는 단어를 반화하려면 어떻게 해야 할까?

기존의 **설정**과 **정리** 과정은 재사용하고 `processFile`메서드만 다른 동작을 수행하도록 명령할 수 있다면 좋을 것이다.

`processFile`의 동작을 파라미터화 하는 것이다.

BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile 메서드로 동작을 전달해야 하는데 이 때 람다를 이용할 수 있다.

```java
// BufferedReader에서 두 행을 출력하는 코드
String result = processFile((BufferedReader br -> br.readLine() + br.readLine()));
```



## 2단계: 함수형인터페이스를 이용해서 동작 전달

함수형 인터페이스 자리에 람다를 사용할 수 있다.

따라서 `BufferedReader -> String`과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

이 인터페이스를 BufferedReaderProcessor라고 정의하자.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(Bufferedreader b) throws IOException;
}
// 이렇게 정의한 인터페이스를 processFile의 메서드 인수로 전달할 수 있다.

public String processFile(BuffereReaderProcessor p) throws IOException {
    ...
}
```



## 3단계: 동작 실행

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
        return p.process(br);
    }
}
```



## 4단계: 람다 전달

이제 람다를 이용해서 다양한 동작을 `processFile()` 메서드로 전달할 수 있다.

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```



# 4. 함수형 인터페이스 사용

---

다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다.

3.2절에서 본 것처럼 이미 자바 API는 `Comparable, Runnable, Callable` 등의 다양한 함수형 인터페이스를 포함하고 있다.

자바 8라이브러리에서는 `java.util.function` 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다. 

이 절에서는 **`Preicate, Consumer, Function`** 인터페이스를 설명한다.



## Predicate

`java.util.function.Predicate<T>` 인터페이스는 `test`라는 추상메서드를 정의하며 test는 제네릭 형식 T의 객체를 인수로 받아 boolean을 반환한다.

따로 정의할 필요없이 바로 사용할 수 있는 것이 특징이다.

T형식의 객체를 사용하는 boolean 표현식이 필요한 상황에서 `Predicate` 인터페이스를 사용할 수 있다.

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for (T t : list) {
        if (p.test(T)) {
            results.add(t);
        }
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```



## Consumer

`java.util.function.Consumer<T>` 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 `accept`라는 추상 메서드를 정의한다.

T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스를 사용할 수 있다.

예로 Integer 리스트를 인수로 받아서 각 항목에 동작하는 어떤 동작을 수행하는 `forEach` 메서드를 정의할 때 Consumer를 활용할 수 있다.

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accpet(t);
    }
}
forEach(
    Arrays.asList(1, 2, 3, 4, 5),
    (Integer i) -> System.out.println(i) // Consumer의 accept()을 구현하는 람다
)
```



## Function

`java.util.function.Function<T, R>` 인터페이스는 제네릭 형식 T를 인수로 받아서 제네릭 형식 R 객체를 반환하는 추상 메서드 `apply`를 정의한다.

입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있다.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T t : list) {
        result.add(f.apply(t));
    }
    return result;
}

List<Integer> l = map(
    Arrays.asList("lambdas", "in", "action"),
    (String s) -> s.length() // Function의 apply()를 구현하는 람다
)
```



***`예외, 람다, 함수형 인터페이스의 관계`***

함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다.

즉 예외를 던지는 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 `try / catch` 블록으로 감싸야 한다.

다음 예제처럼 명시적으로 Checked Exception을 잡을 수 도 있다.

```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    }
    catch (IOException e) {
        throw new RuntimeException(e);
    }
};
```



# 5. 형식검사, 형식추론, 제약

## 형식검사

람다가 사용되는 콘텍스트를 이용해 람다의 형식을 추론할 수 있다.

```java
List<Apple> heavierThan150g =
    filter(inventory, (Apple, apple) -> apple.getWeight() > 150);
```

다음과 같은 순서로 형식 확인 과정이 진행된다.

1.   filter 메서드의 선언을 확인한다.
2.   filter 메서드는 두 번째 파라미터로 `Prdicate<Apple>` 형식(대상 형식)을 기대한다.
3.   `Predicate<Apple>`은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스이다.
4.   test메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.
5.   filter메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 한다.



## 다른 함수형 인터페이스

>   ***다이아몬드 연산자***
>
>   다이아몬드 연산자(`<>`)로 콘텍스트에 따른 제네릭 형식을 추론할 수 있다.
>
>   주어진 클래스 인스턴스 표현식을 두 개 이상의 다양한 콘텍스트에 사용할 수 있고 이 때 인스턴스 표현식의 형식 인수는 콘텍스트에 의해 추론된다.
>
>   `List<String> listOfStrings = new ArrayList<>();`
>   `List<Integer> listOfIntegers = new ArrayList<>();`

>   ***특별한 void 호환 규칙***
>
>   람다의 바디에 일반 표현식이 있으면 `void`를 반환하는 함수 디스크립터와 호환된다.
>
>   예를 들어 다음 두 행의 예제에서 List의 add메서드는 `Consumer` 콘텍스트(`T -> void`)가 기대하는 void 대신 boolean을 반환하지만 유효한 코드이다.
>
>   `Predicate<String> p = s -> list.add(s);` // *Predicate는 boolean을 반환*
>   `Consumer<String> b = s -> list.add(s);` // *Consumersms void를 반환*



## 형식추론

자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.

즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.

결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.

```java
// 컴파일러는 람다 파라미터 형식을 추론할 수 있다.
List<Apple> greenApples = 
    filter(inventory, apple -> GREEN.equals(apple.getColor()));

// 여러 파라미터를 포함하는 람다포현식에서는 코드 가독성 향상이 더욱 두드러진다.
Comparator<Apple> c = 
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
Comparator<Apple> c = 
    (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```



## 지역변수 사용

지금 까지의 람다 표현식은 인수를 자신의 바디안에서만 사용했다. 하지만 람다 표현식에서는 익명함수가 하는 것처럼 **자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)**를 활용할 수 있다. 이와 같은 동작을 **람다 캡처링**(capturing lambda)라고 부른다.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

하지만, 자유 변수에도 약간의 제약이 있다.

**인스턴스 변수와 정적변수를 자유롭게 캡쳐(자신의 바디에서 참조할 수 있도록) 할 수 있다.**

하지만 그러려면 **지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.**



즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.

```java
// portNumber라는 변수에 값을 두 번 할당하므로 컴파일할 수 없는 코드이다.
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337; 
```



>   ***지역 변수의 제약***
>
>   내부적으로 인스턴스변수와 지역변수는 태생부터 다르다.
>
>   **인스턴스 변수는 힙에 저장**되는 반명 **지역변수는 스택에 위치**한다.
>
>   람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당변수에 접근하려 할 수 있다.
>
>   따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 **복사본을 제공**한다.
>
>   따라서 이 복사본의 값이 바뀌지 않아야 하므로 지역변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.



# 6. 메서드 참조

메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.

때로는 람다 표현식보다 메서드 참조를 사용하는 것이 더 가독성이 좋으며 자연스러울 수 있다.

```java
// 람다 표현식
inventory.sort((Apple a1, Apple a2) ->
              	a1.getWeight().compareTo(a2.getWeight()));
// 메서드 참조
inventory.sort(comparing(Apple::getWeight));
```

특정 메서드만은 호출하기 위해서는 메서드를 어떻게 호출하는지 설명을 참조하기보다는 **메서드명을 직접 참조**하는 것이 편리하다.

메서드 참조는 메서드명 앞에 구분자(`::`)를 붙이는 방식으로 메서드 참조를 활용할 수 있다.

결과적으로 메서드 참조는 람다표현식 `(Apple a) -> a.getWeight()`를 축약한 것이다.



```java
// 헬퍼 메서드 정의
private boolean isValidName(String string) {
    return Character.isUpperCase(string.charAt(0));
}

// Predicate<String>을 필요로 하는 적당한 상황에서 메서드 참조 사용 가능
filter(words, this::isValidName);
```



## 생성자 참조

`ClasName::new` 처럼 클래스명과 `new` 키워드를 이용해 기존 생성자의 참조를 만들 수 있다.

```java
// 인수가 없는 생성자
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get(); // Supplier의 get메서드를 호출해서 새로운 Apple객체를 만들 수 있다.

//* 위 코드는 다음과 같다. *//

Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();
```

Apple(Integer weight)라는 시그니처를 갖는 생성자는 Function 인터페이스의 시그니처와 같다.

```java
Function<Integer, Apple> c2 = Apple::new; // Apple(Integer weight)의 생성자 참조
Apple a2 = c2.apply(110); // Function의 apply 메서드에 무게를 인수로 호출해서 새로운 Apple객체를 만들 수 있다.

//* 위 코드는 다음과 같다. *//

Function<Integer. Apple> c2 = (weight) -> new Apple(weight);
Apple a2 = c2.apply(110);
```



```java
// Integer를 포함하는 리스트의 각 요소를 우리가 정의한 map 메서드를 이용해 Apple 생성자로 전달한다.
List<Integer> weights = Arrays.asList(7, 3, 4, 10);
List<Apple> apples = map(weights, Apple::new); // map 메서드로 생성자 참조 전달

public List<Apple> map(List<Integer> list, Function<Integer, Apple> f) {
    List<Apple> result = new ArrayList<>();
    for (Integer i : list) {
        result.add(f.apply(i));
    }
    return result;
}
// 결과적으로 다양한 무게를 포함하는 사과 리스트가 만들어진다.
```



Apple(String color, Integer weight)처럼 두 인수를 갖는 생성자는 `BiFunction` 인터페이스와 같은 시그니처를 가지므로 다음처럼 할 수 있다.

```java
BiFunction<Color, Integer, Apple> c3 = Apple::new;
Apple a3 = c3.apply(GREEN, 110);

//* 위 코드는 다음과 같다. *//
BiFunction<Color, Integer, Apple> c3 = (color, weight) -> new Apple(color, weight);
Apple a3 = c3.apply(GREEN, 110);
```

위처럼 인스턴스화하지 않고도 생성자에 접근할 수 있는 기능을 다양한 상황에 쓸 수있다.

예를 들어 `Map`으로 생성자와 문자열값을 관련시키고 String과 Integer가 주어졌을 때 다양한 무게를 갖는 여러 종류의 과일을 만드는 giveMeFruit라는 메서드를 만들 수 있다.

```java
static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
static {
    map.put("apple", Apple::new);
    map.Put("orange", Orange::new);
    ...
}

public static Fruit giveMeFruit(String fruit, Integer weight) {
    return map.get(fruit.toLowerCase()) // Function<Integer, Fruit>
        	  .apply(weight); // 무게 파라미터로 Fruit를 만든다.
}
```



# 7. 람다, 메서드참조 활용하기

사과 리스트 정렬 문제를 해결하면서 지금까지 배운 동작을 총동원한다.

## 1단계: 코드전달

`void sort(Comparator<? super E> c)`

이 코드는 `Comparator` 객체를 인수로 받아 두 사과를 비교한다. 이제 `sort`의 **동작은 파라미터화**되었다고 할 수 있다.

```java
public class AppleComparator implements Comparator<Apple> {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}
inventory.sort(new AppleComparator());
```

## 2단계: 익명클래스 사용

한 번만 사용할 Comparator를 구현하는 것 보다는 **익명클래스**를 이용하는 것이 더 좋다.

```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
})
```

## 3단계: 람다표현식 사용

**함수형 인터페이스**를 기대하는 곳 어디에서나 람다 표현식을 사용할 수 있다고 했다.

함수형 인터페이스는 오직 하나의 추상 메서드를 정의하는 인터페이스이다.

여기서 추상메서드의 시그니처(**함수 디스크립터**)는 람다표현식의 시그니처를 정의한다.

따라서 Comparator의 함수 디스크립터는 `(T, T) -> int` 이다. 우리는 사과를 사용할 것이므로 더 정확히는 `(Apple, Apple) -> int`이다.

```java
inventory.sort((Apple a1, Apple a2) ->
                a1.getWeight().compareTo(a2.getWeight()));
```

자바 컴파일러는 람다표현식이 사용된 콘텍스트를 활용해 파라미터 형식을 추론한다.

```java
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

`Comparator`는 Comparable 키를 추출해서 Comparator 객체로 만드는 Function 함수를 인수로 받는 정적 메서드 **comparing**을 포함한다.

```java
Comparator<Apple> c = Comapartor.comparing((Apple a) -> a.getWeight());

import static java.util.Comparator.comparing;
inventory.sort(comparing(apple -> apple.getWeight()));
```

## 4단계: 메서드 참조 사용

```java
inventory.sort(comparing(Apple::getWeight));
```

이렇게 최적의 코드를 만들고 자바8 이전의 코드와 비교해보면, **단지 코드만 짧아진 것이 아니라 코드의 의미도 명확해졌다.**

코드 자체로 "Apple을 weight별로 비교해서 inventory를 sort하라"는 의미를 전달할 수 있다.



# :bell: Quiz

>   **Quiz**
>
>   올바른 람다표현식 사용법은?
>
>   ```java
>   // 1.
>   excute(() -> {});
>   public void execute(Runnable r) {
>      r.run();
>   }
>   
>   // 2.
>   public Callable<String> fetch() {
>      return () -> "Tricky example ;-)";
>   }
>   
>   // 3.
>   Predicate<Apple> p = (Apple a) -> a.getWeight();
>   ```
>
>   
>
>   1번에서 람다표현식 `() -> {}`의 시그니처는 `() -> void` 이며 Runnable의 추상 메서드 run의 시그니처와 일치하므로 유효한 람다 표현식이다.
>   람다의 바디가 비어 있으므로 이 코드를 실행하면 아무 일도 일어나지 않는다.
>
>   2번에서 fetch 메서드의 반환 형식은 Callable<String> 인데 T를 String으로 대치했을 때 Callable<String>  메서드의 시그니처는 `() -> String`이 된다.
>   `() -> "Tricky example ;-)"`는 `() -> String` 시그니처이므로 문맥상 유효하다.
>
>   3번에서 람다 표현식 `(Apple a) -> a.getWeight()`의 시그니처는 `(Apple) -> Integer` 이므로 `Predicate<Apple>: (Apple) -> boolean` 의 test메서드의 시그니처와 일치하지 않는다. 따라서 유효하지 않다.



>   **Quiz**
>
>   다음과 같은 함수형 디스크립터가 있을 때 어떤 함수형 인터페이스를 사용할 수 있는가? 또, 이 인터페이스에 사용할 수 있는 유효한 람다 표현식을 제시하시오.
>
>   1.   `T -> R`
>   2.   `(Int, int) -> int`
>   3.   `T -> void`
>   4.   `() -> T`
>   5.   `(T, U) -> R`
>
>   
>
>   1번은 Function<T, R>이 대표적이다 T형식의 객체를 R형식의 객체로 변환할 때 사용한다.
>
>   2번은 IntBinaryOperator의 `(int, int) -> int` 형식의 시그니처를 갖는 추상 메서드 applyAsInt를 정의한다.
>
>   3번은 Consumer<T> 는 `T -> void` 형식의 시그니처를 갖는 추상메서드 accept을 정의한다.
>
>   4번은 Supplier<T>는 `() -> T` 형식의 시그니처를 갖는 추상메서드 get을 정의한다.
>    또, Callable<T>도 `() -> T` 형식의 시그니처를 갖는 추상메서드 call을 정의한다.
>
>   5번은 BiFunction<T, U, R>은 `(T, U) -> R` 형식의 시그니처를 갖는 추상메서드 apply를 정의한다.

# :pencil: 마치며

-   람다표현식은 메서드로 전달할 수 있는 _를 단순화한 것이다.
-   함수형 인터페이스는 정확히 _의 _를 지정하는 인터페이스
-   람다표현식의 시그니처를 서술하는 메서드를 _라고 한다.
-   인스턴스 변수는 _에 저장되는 반면 지역변수는 _에 위치한다.
-   람다에서 지역변수를 사용할 때 지역 변수는 명시적으로 _로 선언되어 있어야 하거나 실질적으로 _로 선언된 변수와 똑같이 사용되어야 한다.
