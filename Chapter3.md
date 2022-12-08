# Chapter3

# 1. 람다란 무엇인가?

---

람다표현식은 익명클래스처럼 이름이 없는 함수이면서, 메서드를 인수로 전달할 수 있다.

람다표현식은 메서도로 전달할 수 있는 익명함수를 단순화한 것이다.

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



>   **Quiz**
>
>   올바른 람다표현식 사용법은?
>
>   ```java
>   // 1.
>   excute(() -> {});
>   public void execute(Runnable r) {
>       r.run();
>   }
>   
>   // 2.
>   public Callable<String> fetch() {
>       return () -> "Tricky example ;-)";
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
>   3번에서 람다 표현식 `(Apple a) -> a.getWeight()`의 시그니처는 `(Apple) -> Integer` 이므로 `Predicate<Apple>: (Apple) -> boolean` 의 test메서드의 시그니처와 일치하지 않는다. 따라서 유효하지 않ㄴ다.



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
>   d
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

