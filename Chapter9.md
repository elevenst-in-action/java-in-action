# Chapter9

---

이 챕터에서는 기존 코드를 이용해 즉, 람다 표현식을 이용해 가독성과 유연형을 높이기 위해 어떻게 리팩터링해야하는지 설명한다.

또한, 람다표현식으로 **전략, 템플릿 메서드, 옵저버, 의무 체인, 팩토리** 등의 다양한 객체지향 디자인 패턴을 어떻게 간소화할 수 있는지 살펴본다.



# 1. 가독성과 유연성을 개선하는 리팩터링

일반적으로 코드 가독성이 좋다는 것은 **어떤 코드를 다른 사람도 쉽게 이해할 수 있음**을 의미한다.

즉, 코드 가독성을 개선한다는 것은 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수 할 수 있게 만드는 것을 의미한다.



## - 익명 클래스를 람다 표현식으로 리팩터링

람다 표현식을 이용하면 간결하고, 가독성이 좋은 코드를 구현할 수 있다.

그러나 모든 익명 클래스를 람다표현식으로 변환할 수 있는 것은 아닌데, 

1.   익명 클래스에서 사용한 `this`와 `super`는 람다 표현식에서 다른 의미를 갖는다. **익명클래스에서 `this`는 익명 클래스 자신을 가리키지만 람다에서 `this`는 람다를 감싸는 클래스를 가리킨다.**

2.   익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수(**섀도변수(shadow variable)**) 있지만 람다 표현식으로는 변수를 가릴 수 없다.

     ```java
     int a = 10;
     Runnable r1 = () -> {
         int a = 2;						// 컴파일 에러
         System.out.println(a);
     }
     Runnable r2 = new Runnable() {
         public void run() {
             int a = 2;					// 작동한다.
             System.out.println(a);
         }
     }
     ```

3.   익명 클래스를 람다 표현식으로 바꾸면 `Context Overoadling`에 따른 모호함이 초래될 수 있다. 익명 클래스는 인스턴스화 할 때 명식적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달리지기 때문이다.

     ```java
     // Task라는 Runnable과 같은 시그니처를 갖는 함수형 인터페이스를 선언한다.
     interface Task {
         public void execute();
     }
     public static void doSomething(Runnable r) { r. run(); }
     public static void doSomething(Task a) { r. execute(); }
     
     // Task를 구현하는 익명 클래스를 전달할 수 있다.
     doSomething(new Task() {
         public void execute() {
             System.out.println("Danger danger!!");
         }
     })
     ```

     하지만 익명 클래스를 람다 표현식으로 바꾸면 메서드를 호출할 때 Runnable과 Task 모두 대상 형식이 될 수 있으므로 문제가 생긴다.

     ```java
     doSomething(() -> System.out.println("Danger danger!!"));
     ```

     `doSomething(Runnable)`과 `doSomething(Task)` 중 어느 것을 가리키는지 알 수 없는 모호함이 발생한다.

     그래서 명시적 형변환 `(Task)`를 이용해서 모호함을 제거할 수 있다.

     ```java
     doSomething((Task)() -> System.out.println("Danger danger!!"));
     ```



## - 람다 표현식을 메서드 참조로 리팩터링

람다 표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다. 메서드명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.

```java
// 칼로리 수준으로 요리를 그룹화하는 코드
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream()
    	.collect(
			groupingBy(dish -> {
                if (dish.getCalories() <= 400) return CaloricLevel.DIET:
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                else return CaloricLevel.FAT;
            })
);

// 람다 표현식을 별도의 메서드로 추출한 다음 groupingBy에 인수로 전달할 수 있다.
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream().collect(groupingBy(Dish::getCaloricLevel));	// 람다표현식을 메서드로 추출

// Dish 클래스에 getCaloricLevel 메서드를 추가해야 한다.
public class Dish {
    ...
	public CaloricLevel getCaloricLevel() {
        if (this.getCalories() <= 400) return CaloricLevel.DIET:
        else if (this.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
    }
}
```

또한, `comparing`과 `maxBy` 같은 정적 헬퍼 메서드를 활용할 수도 있다.

`sum`, `maximum` 등 자주 사용하는 리듀싱 연산은 메서드 참조와 함게 사용할 수 있는 내장 헬퍼 메서드를 제공한다.

최댓값이나 합계를 계산할 때 람다 표현식과 저수준 리듀싱 연산을 조합하는 것보다 `Collectors API`를 사용하면 코드의 의미가 더 명확해진다.

```java
// 저수준 리듀싱 연산
int totalCalories = 
    menu.stream().map(Dish::getCalories)
    			 .reduce(0, (c1, c2) -> c1 + c2);

// Collectors API
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```



## - 명령형 데이터 처리를 스트림으로 리팩터링

이론적으로는 반복자를 이용한 기존의 모든 컬렉션 처리 코드를 스트림 API로 바꿔야 한다.

```java
// 전체 구현을 자세히 살펴본 이후에야 코드의 의도를 이해할 수 있다.
List<String> dishNames = new ArrayList<>();
for (Dish dish : menu) {
    if (dish.getCalories() > 300) {
        dishNames.add(dish.getName());
    }
}

// 스트림 API를 이용하면 문제를 더 직접적으로 기술할 수 있을 뿐 아니라 쉽게 병렬화 할 수 있다.
menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList());
```





# 2. 람다로 객체지향 디자인 패턴 리팩터링

이 절에서는 다섯가지 디자인 패턴을 살펴본다.

## - 전략

>   전략패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법이다.
>
>   다양한 기준을 갖는 입력값을 검증하거나, 다양한 파싱 방법을 사용하거나, 입력 형식을 설정하는 등 다양한 시나리오에 활용할 수 있다.

전략패턴은 세 부분으로 구성된다.

-   알고리즘을 나타내는 인터페이스(Strategy 인터페이스)
-   다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
-   전략 객체를 사용하는 한 개 이상의 클라이언트



예를 들어 오직 소문자 또는 숫자로 이루어져야 하는 등 텍스트 입력이 다양한 조건에 맞게 포맷되어 있는지 검증한다고 가정하자.

먼저 String 문자열 검증하는 인터페이스부터 구현한다. 

```java
public interface ValidationStrategy {
    boolean execute(String s);
}

// 위에서 정의한 인터페이스를 구현하는 클래스를 하나 이상 정의한다.
public class IsAllLowerCase implements ValidationStrategy {
    public boolean execute(String s) {
        return s.mathces("[a-z]+");
    }
}

public class IsNumeric implements ValidationStrategy {
    public boolean execute(String s) {
        return s.mathces("\\d+");
    }
}
```

지금까지 구현한 클래스를 다양한 검증 전략으로 활용할 수 있다.

```java
public class Validator {
    private final ValidationStrategy strategy;
    public Validator(ValidationStrategy v) {
        this.strateghy = v;
    }
    public boolean validate(Strin s) {
        return strategy.execute(s);
    }
}
Validator numericValidator = new Validator(new IsNumeric());
boolean b1 = numericValidator.validate("aaaa");				// false
Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
boolean b2 = lowerCaseValidator.validate("bbbb");			// true
```



**람다 표현식 사용**

`ValidationStrategy`는 함수형 인터페이스이며 `Predicate<String>`과 같은 함수 디스크립터를 갖고 있음을 파악했을 것이다.

따라서 다양한 전략을 구현하는 새로운 클래스를 구현할 필요없이 람다표현식을 직접 전달하면 코드가 간결해진다.

```java
Validator numericValidator = 
    new Validator((String s) -> s.mathces("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa");
Validator lowerCaseValidator = 
    new Validator((String s) -> s.mathces("\\d+"));
boolean b2 = lowerCaseValidator.validate("bbbb");
```

위 코드에서 확인할 수 있듯이 람다 표현식을 이용하면 전략 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있다.

람다표현식은 코드 조각을 캡슐화하기 때문에 람다 표현식으로 전략 디자인 패턴을 대신할 수 있다.



## - 템플릿 메서드

>   알고리즘의 개요를 제시한 다음, 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 템플릿 메서드 디자인 패턴을 사용한다.
>
>   다시 말해, "이 알고리즘을 사용하고 싶은데 그대로는 안 되고 조금 고쳐야 하는" 상황에 적합하다.

간단한 온라인 뱅킹 애플리케이션을 구현한다고 가정하자.

사용자가 고객 ID를 애플리케이션에 입력하면 은행 데이터베이스에서 고객 정보를 가져오고 고객이 원하는 서비스를 제공할 수 있다.

예를 들어 고객 계좌에 보너스를 입금한다고 가정하자.

은행마다 다양한 온라인 뱅킹 앱을 사용하며 동작방법도 다르다.

```java
// 온라인 뱅킹 앱의 동작을 정의하는 추상클래스
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }
    abstract void makeCustomerHappy(Customer c);
}
```

processCustomer는 온라인 뱅킹 알고리즘이 해야 할 일이다. 각 지점은 OnlineBanking 클래스를 상속받아 `makeCustomerHappy` 메서드가 원하는 동작을 수행하도록 구현할 수 있다.



**람다 표현식 사용**

람다나 메서드 참조로 알고리즘에 추가할 다양한 컴포넌트를 구현할 수 있다.

```java
// 이전에 정의한 makeCustomerHappy의 메서드 시그니처와 일치하도록 Consumer<Customer> 형식을 갖는 두번째 인수를 processCustomer에 추가한다.
public void processCustomer(int id, Consumer<Customer> amkeCustomerHappy) {
    Customer c = Database.getCustoemrWithId(id);
    makeCustomerHappy.accept(c);
}
// 이제 onlineBanking 클래스를 상속받지 않고 직접 람다 표현식을 전달해서 다양한 동작을 추가할 수 있다.
new OnlineBankingLambda().processCustomer(1337, (Customer c) -> 
                                         System.out.println("Hello " + c.getName()));
```

람다 표현식을 이용하면 템플릿 메서드 디자인 패턴에서 발생하는 자잘한 코드도 제거할 수 있다.



## - 옵저버



