# Chapter6

>   이 장에서는 reduce가 그랬던 것처럼 collect 역시 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있음을 설명한다.
>
>   지금부터 **컬렉션(Collection), 컬렉터(Collector), `collect`**를 헷갈리지 말자.



# 1. 컬렉터란 무엇인가?

함수형 프로그래밍에서는 '무엇'을 원하는지 직접 명시할 수 있어서 어떤 방법으로 이를 얻을지는 신경 쓸 필요가 없다.

`Collector` 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정한다.



5장에서는 '각 요소를 리스트로 만들어라'를 의미하는 `toList`를 `Colletor` 인터페이스의 구현으로 사용했다,

여기서는 `groupingBy`를 이용해서 동작을 수행한다.

다수준(multilevel)으로 그룹화를 수행할 때 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 두드러진다.

-   명령형코드 : 문제해결과정에서 다중 루프와 조건문을 추가하며 가독성과 유지보수성이 크게 떨어짐
-   함수형코드 : 필요한 컬렉터를 쉽게 추가ㅏ하여 동작을 수행할 수 있음



## - 고급 리듀싱 기능을 수행하는 컬렉터

훌륭하게 설계된 함수형 API의 또 다른 장점으로 높은 수준의 조합성과 재사용성을 꼽을 수 잇다.

**`collect`로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다**는 점이 컬렉터의 최대 강점이다.

*> 스트림에 `collect`를 호출하면 스트림의 요소에 리듀싱 연산이 수행된다. 내부적으로 **리듀싱 연산** 이 일어난다.*
*> 명령형 프로그래밍에서 우리가 직접 구현해야 했던 작업이 자동으로 수행된다.*
*> `collect`에서는 리듀싱 연산을 이용해서 스트림의 각 요소를 방문하면서 컬렉터가 작업을 처리한다.*



`Collector` 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정된다.

가장 많이 사용되는 `toList` 메서드는 스트림의 모든 요소를 리스트로 수집한다.



## - 미리 정의된 컬렉터

미리 정의된 컬렉터, `groupingBy` 같이 `Collectors` 클래스에서 제공하는 팩토리메서드의 기능을 설명한다.

-   스트림 요소를 하나의 값으로 리듀스하고 요약
    -   먼저 리듀싱과 요약 관련 기능을 수행하는 컬렉터부터 살펴본다. 다양한 계산을 수행할 때 이들 컬렉터를 유용하게 활용할 수 있다.
-   요소 그룹화
    -   이전 예제를 다수준으로 그룹화하거나 각각의 결과 서브그룹에 추가로 리듀싱 연산을 적용할 수 있도록 컬렉터를 조합하는 방법을 배운다.
-   요소 분할
    -   또한 그룹화의 특별한 연산인 **분할**도 설명한다.
    -   **분할**은 한 개의 인수를 받아 boolean을 반환하는 함수, 즉 `Predicate`를 그룹화 함수로 사용한다.

# 2. 리듀싱과 요약

5장에서 사용한 맛있는 요리 리스트, 즉 메뉴 예제를 활용해서 `Collector` 팩토리 클래스로 만든 컬렉터 인스턴스로 어떤 일을 할 수 있는지 살펴보자.

컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다.



## - counting

첫번 째 예제로 `counting()`이라는 팩토리 메서드가 반환하는 컬렉터로 메뉴에서 요리 수를 계산한다.

```java
long howManyDishes = menu.stream().collect(Collectors.counting());

// 아래처럼 불필요한 과정을 생략할 수 있다.
long howManyDishses = menu.steram().count();
```

>   `import static java.util.stream.Collectors.*`
>
>   정적 팩토리 메서드를 모두 임포트했다고 가정하면 `Collectors.counting()`을 간단하게 `counting()`으로 표현할 수 있다.

## - 스트림값에서 최대값과 최솟값 검색

메뉴에서 칼로리가 가장 높은 요리를 찾는다고 가정하자.

`Collectors.maxBy`, `Collectors.minBy` 두 개의 메서드를 이용해 스트림의 최대값과 최소값을 계산할 수 있다.

두 컬렉터는 스트림의 요소를 비교하는 데 사용할 `Comparator`를 인수로 받는다.

```java
// 칼로리로 요리를 비교하는 Comparator를 구현한 다음,
// Collectors.maxBy로 전달하는 코드이다.
Comparator<Dish> dishCaloriesComparator = 
    Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = 
    menu.stream()
    	.collect(maxBy(dishCaloriesComparator));
```



## - 요약 연산

또한 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용되는 데 이러한 연산을 **요약 연산**이라 부른다.

`Collectors` 클래스는 `Collectors.summingInt`라는 특별한 요약 팩토리 메서드를 제공한다.

-   `summingInt`는 객체를 int로 매핑하는 함수를 인수로 받는다.
-   이 함수는 객체를 int로 매핑한 컬렉터를 반환한다.
-   그리고 `summingInt`가 `collect` 메서드로 전달되면 요약 작업을 수행한다.

```java
// 메뉴 리스트의 총 칼로리를 계산하는 코드이다.
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```



이러한 단순 합계 외에 평균값 계산 등의 연산도 요약 기능으로 제공된다

즉, `Collectors, averagingInt, averagingLong, averagingDouble` 등으로 당양한 형식으로 이루어진 숫자 집합의 평균을 계산할 수 있다.

```java
double avgCalories =
    menu.stream().collect(averagingInt(Dish::getCalories));
```



종종 두 개 이상의 연산을 한번에 수행해야 할 때도 있다.

이런 상황에서는 팩토리 메서드 `summarizingInt`가 반환하는 컬렉터를 사용할 수 있다.

```java
// 하나의 요약연산으로 메뉴에 있는 요소 수, 요리의 칼로리 합계, 평균, 최댓값, 최솟값 등을 계산하는 코드이다.
IntSummarayStatistics menuStatistics = 
    menu.stream().collect(summarizingInt(Dish::getCalories));

// 위 코드를 실행하면 IntSummarayStatistics 클래스로 모든 정보가 수집된다.
IntSummarayStatistics(count=9, sum=4300, min=120, average=477.7778, max=800) // 객체를 출력하면 이렇게 확인할 수 있다.
```



## - 문자열 연결

컬랙터에 `joining` 팩토리 메서드를 이용하면 스트림의 각 객채에 `toString` 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다.

```java
// 메뉴의 모든 요리명을 연결하는 코드이다.
String shortMenu = menu.stream().map(Dish::getName).collect(joining());
```

`joining` 메서드는 내부적으로 `StringBuilder`를 이용해서 문자열을 하나로 만든다.

Dish 클래스가 요리명을 반환하는 `toString` 메서드를 포함하고 있다면 다음 코드에서 보여주는 것처럼 `map`으로 각 요리의 이름을 추출하는 과정을 생략할 수 있다.

```java
// 위 코드와 같은 결과를 도출한다.
String shortMenu = menu.stream().collect(joining());
```



## - 범용 리듀싱 요약 연산

지금까지 살펴본 모든 컬렉터는 `reducing` 팩토리 메서드로도 정의할 수 있다.

즉, 범용 `Collectors.reducing`으로도 구현할 수 있다.

```java
int totalCalories = menu.stream
    					.collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```

reducing은 인수 세 개를 받는다.

-   **초깃값**: 첫 번째 인수는 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값이다.
-   **변환함수**: 두 번째 인수는 요리를 정수로 변환할 때 사용한 변환함수이다.
-   **합계함수**: 세 번째 인수는 같은 종류의 두 항목을 하나의 값으로 더하는 `BinaryOperator`다.  예제에서는 두 개의 int가 사용되었다.

```java
// 다음처럼 한 개의 인수를 가진 reducing 버전을 이용해서 가장 칼로리가 높은 요리를 찾는 방법도 있다.
Optional<Dish> mostCalorieDish =
    menu.stream().collect(reducing((d1, d2) -> d1.getCaloreis() > d2.getCalories() ? d1 : d2));
```



한 개의 인수를 갖는 reducing 팩토리 메서드는 세 개의 인수를 갖는 reducing 메서드에서 스트림의 첫 번째 요소를 시작 요소, 즉 첫 번째 인수로 받으며,

자신을 그대로 반환하는 **항등함수**를 두 번째 인수로 받는 상황에 해당한다.



한 개의 인수를 갖는 `reducing` 컬렉터는 시작값이 없으므로 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않는 상황이 벌어진다.

그래서 한 개의 인수를 갖는 `reducing`은 Optional<Dish> 객체를 반환한다.



## [ collect와 reduce의 차이 ]

이들 메서드로 같은 기능을 구현하기 때문에 어떤 차이를 갖는지 궁금할 것이다.

```java
// 예를 들어 toList 컬렉터를 사용하는 collect대신 reduce메서드를 사용할 수 있다.
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();

List<Integer> numbers = stream.reduce(
	new ArrayList<Integer>(),
    (List<Integer> l, Integere e) -> {
        l.add(e);
        return l;
    },
    (List<Integer> l1, List<Integer> l2) -> {
        l1.addAll(l2);
        return l1;
    }
);
```

위 코드에서는 의미론적인 문제와 실용성 문제 두 가지 문제가 발생한다.

1.   **`collect` 메서드는 도출하려는 결과를 누적하는 컨테이너늘 바꾸도록 설계된 메서드**인 반면
     **`reduce`는 두 값을 하나로 도출하는 불변형 연산**이라는 점에서 의미론적인 문제가 일어난다.
     즉, 위에서 reudce는 누적자로 사용된 리스트를 변환시키므로 reduce 메서드를 잘못 활용한 예에 해당한다.

2.   여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져버리므로 `reducing`연산을 병렬로 수행할 수 없다는 문제가 일어난다.
     이 문제를 해결하려면 매번 새로운 리스트를 할당해야 하고 그에 따른 성능이 저하될 것이다.
     7장에서 자세히 보겠지만 **가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 `collect`메서드로 리듀싱 연산을 구현하는 것이 바람직**하다.



>   ***컬렉션 프레임워크 유연성 : 같은 연산도 다양한 방식으로 수행할 수 있다.***
>
>   `reducing` 컬렉터를 사용한 이전 예제에서 람다 표현식 대신 Integer 클래스의 sum 메서드 참조를 이용하면 코드를 좀 더 단순화 할 수 있다,
>
>   ```java
>   int totalCalories = menu.stream().collect(reducing(0,					// 초깃값
>                                                      Dish::getCalories,	// 변환함수
>                                                      Integer::sum));		// 합계함수
>   ```
>
>   ```java
>   int toalCaloreis = 
>       menu.steram().map(Dish::getCalories).reduce(Integer::sum).get();
>   // Integer::sum 으로 빈 스트림과 관련한 널 문제를 피할 수 있도록 int가 아닌 Optional<Integer>를 반환한다.
>   // 요리스트림이 비어있지 않은 사실을 알고 있으므로 get을 자유롭게 사용할 수 있다.
>   // 일반적으로는 기본값을 제공할 수 있는 orElse, orElseGet 등을 이용해 Optional의 값을 얻어오는 것이 좋다.
>   ```
>
>   ```java
>   // 스트림을 IntStream으로 매핑한 다음에 sum메서드를 호출하는 방법으로도 결과를 얻을 수 있다.
>   int totalCalories = 
>       menu.stream().mapToInt(Dish::getCalories).sum();
>   ```
>
>   
>
>   ***자신의 상황에 맞는 최적의 해법 선택***
>
>   함수형 프로그래밍에서는 하나의 연산을 다양한 방법으로 해결할 수 있음을 알아보았다.
>
>   또한 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 비해 컬렉터를 이용하는 코드가 더 복잡하는 사실도 보여주었다.
>
>   코드가 좀 더 복잡한 대신 재사용성과 커스터마이즈 가능성을 제공하는 높은 수준의 추상화와 일반화를 얻을 수 있다.
>
>   문제를 해결할 수 있는 다양한 해결방법을 확인한 다음, 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다.
>
>   이렇게 하면 **가독성**과 **성능**이라는 두 마리 토끼를 잡을 수 있다.
>
>   예를 들어 이전 예제에서는 가장 마지막에 확인한 방법이 가독성이 가장 좋고 간결하다.
>
>   또한 `IntStream` 덕분에 **자동언박싱** 연산을 수행하거나 Integer를 int로 변환하는 과정을 피할 수 있으므로 성능까지 좋다.



# 3. 그룹화

데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 데이터베이스에서 많이 수행되는 작업이다.

명령형으로 그룹화를 구현하려면 까다롭고, 할일이 많으며, 에러도 많이 발생한다.

하지만 자바8의 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.

팩토리 메서드 **`Collectors.groupingBy`**를 이용해서 쉽게 메뉴를 그룹화 할 수 있다.

```java
Map<Dish.Type, List<Dish>> dishesByType = 
    menu.stream().collect(groupingBy(Dish::getType));

// Map에 포함된 결과
{
    FISH=[prawns, salmon],
    OTHER=[french fries, rice, season fruit, pizza],
    MEAT=[pork, beef, chicken]
}
```

각 요리에서 `Dish.Type`과 일치하는 모든 요리를 추출하는 함수를 `groupingBy`메서드로 전달했다.

이 함수를 기준으로 스트림이 그룹화되므로 이를 **분류 함수**라고 부른다.



하지만, 단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 분류함수로 사용할 수 없다.

이 때는 메서드 참조 대신 람다표현식으로 필요한 로직을 구현할 수 있다.

```java
public enum CaloricLevel { DIET, NORMAL, FAT }

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
	groupingBy(dish -> {
        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
    })
);
```



## - 그룹화된 요소 조작

요소르 그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요하다.

예를 들어 500칼로리가 넘는 요리만 필터한다고 가정하자.

```java
// 그룹화 하기 전 Predicate로 필터를 적용해 문제를 해결할 수 도 있다.
Map<Dish.Type, List<Dish>> caloricDishesByType = 
    menu.stream().filter(dish -> dish.getCalories() > 500)
    			 .collect(groupingBy(Dish::getType));
```

하지만 위 코드는 단점이 존재한다.

우리의 메뉴 요리에는 맵 형태로 되어 있으므로 위 기능을 사용하려면 맵에 코드를 적용해야 한다.

`{OTHER=[french fries, pizza], MEAT=[pork, beef]}`

우리의 filter `Preidcate`를 만족하는 FISH 종류 요리는 없으므로 결과 맵에서 해당 키 자체가 사라진다.



```java
// 두 번째 Collector 안으로 filter Predicate를 이동함으로 이 문제를 해결할 수 있다.
Map<Dish.Type, List<Dish>> caloricDishesBytpye =
    menu.stream()
    	.collect(groupingBy(Dish::getType,
                           filtering(dish -> dish.getCalories() > 500),
                           toList()));
```



**`filtering`** 메서드는 `Collectors`클래스의 또 다른 정적 팩토리 메서드로 Predicate를 인수로 받는다.

이 Predicate로 각 그룹의 요소와 필터링 된 요소를 재그룹화 한다. 이렇게 해서 아래 결과 맵에서 볼 수 있는 것처럼 목록이 비어있는 `FISH`도 항목으로 추가된다.

`{OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}`



또 다른 유용한 기능 중 하나로 **매핑함수를 이용해서 요소를 변환하는 작업이 있다.**

```java
// mapping 함수를 이용해 그룹의 각 요리 이름을 관렴 이름 목록으로 변환할 수 있다.
Map<Dish.Type, List<String>> dishNamesByType = 
    menu.stream()
    	.collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
```

이전 예제와 달리 결과 맵의 각 그룹은 요리가 아니라 이름으로 추출한 문자열 리스트이다.

또한, `groupingBy`와 연계해 세 번째 컬렉터를 사용해서 일반 맵이 아닌 **flatMap** 변환을 수행할 수 있다.

**`flatMapping`**컬렉터를 이용하면 각 형식의 요리의 태그를 간편하게 추출할 수 있다.

```java
// 각 그룹에 수행한 flatMapping 연산 결과를 수집해서 리스트가 아니라 집합으로 그룹화 해 중복 태그를 제거한다.
Map<Dish.Type, Set<String>> dishNaemsByType = 
    menu.stream().
    	.collect(groupingBy(Dish::getType,
                            flatMapping(dish -> dishTags.get( dish.getName() ).stream(),
                            toSet())));
```



지금까지는 칼로리 같은 한 가지 기준으로 메뉴의 요리를 그룹화했는 데 두 가지 이상의 기준을 동시에 적용할 수 있을까?



## - 다수준 그룹화

두 인수를 받는 팩토리 메서드인 `Collectors.groupingBy`를 이용해 항목을 다수준으로 그룹화할 수 있다.

`Collectors.groupingBy`는 일반적인 **분류함수와 컬렉터를 인수**로 받는다.

즉, 바깥쪽 `groupingBy`메서드에 스트림의 항목을 분류할 두 번째 기준을 정의하는 내부 `groupingBy`를 전달해서 두 수준으로 스트림의 항목을 그룹화 할 수 있다.

```java
Map<Dish.type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
menu.stream().collect(
	groupingBy(Dish::getType,
          groupingBy(dish -> {
              if (dish.getCalories() <= 400) {
                  return CaloricLevel.DIET;
              } else if (dish.getCalories() <= 700) {
                  return CaloricLevel.NORMAL;
              } else {
                  return CaloricLevel.FAT;
              }
          }))
);
```

이 그룹화의 결과로 두 수준의 맵이 만들어 진다.

`{`

`MEAT={DIET=[chicken], NORMAL=[beef], FAT=[prok]}, FISH={DIET=[prawns], NORAL=[salmon]}},`

`OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}`

`}`

외부 맵은 첫 번째 수준의 분류 함수에서 분류한 키값 `fish, meat, other`를 갖는다.

그리고 외부 맵의 값은 두 번째 수준의 분류 함수의 기준 `normal, diet, fat`을 키값으로 갖는다.



## - 서브그룹으로 데이터 수집

두 번째 `groupingBy` 컬렉터를 외부 컬렉터로 전달해서 다수준 그룹화 연산을 구현했다.

사실 첫 번째 `groupingBy`로 넘겨주는 컬렉터의 형식은 제한이 없다.

```java
// groupingBy 컬렉터에 두 번째 인수로 counting을 전달해서 메뉴에서 요리의 수를 종류별로 계산할 수 있다.
Map<Dish.Type, Long> typesCount = menu.stream().collect(
	groupingBy(Dish::getType, counting()));

// {MEAT=3, FISH=2, OTHER=4}
```

**분류 함수 한 개의 인수를 갖는 `groupingBy(f)` 는 사실 `groupingBy(f, toList())`의 축약형이다.**

요리의 **종류**를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 프로그램도 다시 구현할 수 있다.

```java
Map<Dish.Type, Optional<Dish>> mostCaloricByType = 
    menu.stream()
    	.collect(groupingBy(Dish::gettype,
                           	maxBy(comparingInt(Dish::getCalories))));

// {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```



>   ***컬렉터 결과를 다른 형식에 적용하기***
>
>   마지막 그룹화 연산에서 Optional로 감쌀 필요가 없으므로 Optional을 삭제할 수 있다.
>
>   즉, 다음처럼 팩토리 메서드 `Collectors.collectionAndThen`으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.
>
>   ```java
>   Map<Dish.Type, Dish> mostCaloricByType = 
>       menu.stream()
>       	.collect(groupingBy(Dish::getType,									// 분류함수
>                              collectingAndThen(
>                              		maxBy(comparingInt(Dish::getCalories)),		// 감싸인 컬렉터
>                              		Optional.get())));							// 변환 함수
>   
>   {FISH=salmon, OTHER=pizza, MEAT=prok}
>   ```
>
>   팩토리메서드 `collectingAndThen`은 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환한다.
>
>   반환되는 컬렉터는 기존 컬렉터의 래퍼역할을 하며 `collect`의 마지막 과정에서 변환함수로 자신이 반환하는 값을 매핑한다.
>
>   이 예제에서는 maxBy로 만들어진 컬렉터가 감싸지는 컬렉터이고, 변환함수 `Optional::get`으로 반환된 Optional에 포함된 값을 추출한다.
>
>   이미 언급했듯이 **리듀싱 컬렉터는 절대 Optional.empty()를 반환하지 않으므로 안전한 코드이다.**



>   ***groupingBy와 함께 사용하는 다른 컬렉터 예제***
>
>   일반적으로 스트림에서 같은 그룹으로 분류된 모든 요소에 리듀싱 작업을 수행할 때는 팩토리 메서드 `groupingBy`에 두 번째 인수로 전달한 컬렉터를 사용한다.
>
>   ```java
>   // 여기서는 메뉴에 있는 모든 요리의 칼로리 합계를 구하려고 만든 컬렉터를 재사용할 수 있다.
>   Map<Dish.Type, Integer> totalCaloriesBytype =
>       menu.stream().collect(groupingBy(Dish::getType,
>                                       summingInt(Dish::getCalories)));
>   ```
>
>   이 외에도 `mapping` 메서드로 만들어진 컬렉터도 `groupingBy`와 자주 사용된다.
>
>   ```java
>   Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType =
>       menu.stream().collect(
>           groupingBy(Dish::getType, mapping(dish -> {
>               if (dish.getCalories() <= 400) return CaloricLevel.DIET;
>               else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
>               else return CaloricLevel.FAT;
>           },
>           toSet()))
>       );
>   
>   {OTHER=[DIET, NORMAL], MEAT=[DIET, NORMAL, FAT], FISH=[DIET, NORMAL]}
>   ```
>
>   -   mapping 메서드에 전달한 변환 함수는 Dish를 CaloricLevel로 매핑한다.
>   -   그리고 CaloricLevel 결과 스트림은 toSet 컬렉터로 전달되면서 리스트가 아닌 집합으로 스트림의 요소가 누적된다.
>
>   위 예제에서는 Set의 형식이 정해져 있지 않았는데 **`toCollection`**을 이용하면 원하는 방식으로 결과를 제어할 수 있다.
>
>   ```java
>   // 메서드참조 HashSet::new를 toCollection에 전달할 수 있다.
>   Map<Dish.Type, Set<Caloriclevel>> caloricLevelsByType =
>       menu.stream().collect(
>   		groupingBy(Dish::getType, mapping(dish -> {
>   			if (dish.getCalories() <= 400) return CaloricLevel.DIET;
>               else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
>               else return CaloricLevel.FAT;
>           },
>   		toCollection(HashSet::new)))
>   	);
>   ```



# 4. 분할

분할은 **분할 함수**라 불리는 Predicate를 분류 함수로 사용하는 특수한 그룹화 기능이다.

분할함수는 boolean을 반환하므로 맵의 키 형식은 `Boolean`이다. 이 그룹화 맵은 최대 (참/거짓)을 갖는 두 개의 그룹으로 분류된다.

```java
Map<Boolean, List<Dish>> partitionedMenu = 
    menu.stream().collect(partitioningBy(Dish::isVegetarian)); // 분할함수

// 다음과 같은 맵이 반환된다.
{false=[pork, beef, chicken, prawns, salmon],
 true=[french fries, rice, saeson fruit, pizza]}

// 이제 참값의 키로 맵에서 모든 채식 요리를 얻을 수 있다.
List<Dish> vegetarainDishes = partitionedMenu.get(true);

// 아래 코드도 위와 같은 결과를 얻게 한다.
List<Dish> vegetarainDishes = 
    menu.stream().filter(Dish::isVegetarain).collect(toList());
```



## - 장점

분할 함수가 반환하는 `참, 거짓` 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다.

```java
// 컬렉터를 두번 째 인수로 전달할 수 있는 오버로드 된 버전의 partitioningBy 메서드도 있다.
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarainDishesByType = 
    menu.stream().collect(partitioningBy(Dish::isVegetarian,			// 분할함수
                                          groupingBy(Dish::getType)));	// 두 번째 컬렉터

{false={FISH=[prawns, salmon], MEAT=[pork,beef, chicken]},
 true={OTEHR=[french fries, rice, season fruit, pizza]}}

// 채식요리의 스트림과, 채식이아닌요리의 스트림을 각각
// 요리 종류로 그룹화 해서 두 수준의 맵이 반환되었다.
```

또,, 이전 코드를 활용하면 채식 요리와 채식이 아닌 요리 각각의 그룹에서 가장 칼로리가 높은 요리도 찾을 수 있다.

```java
Map<Boolean, Dish> mostCaloricPartitionedByVegetarain = 
    menu.stream().collect(
		partitioningBy(Dish::isVegetarain,
                      collectingAndThen(maxBy(comparingInt(Dish::getCalories)),
                                        Optional::get)));

{false=pork, true=pizza}
```

**partitioningBy**가 반환한 맵 구현은 참과 거짓 두 가지 키만 포함되므로 간결하고 효과적이다.



# 5. Collector 인터페이스

`Collector` 인터페이스는 리듀싱 연산(즉, 컬렉터)를 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

지금까지는 `toList`나 `groupingBy` 등 컬렉터 인터페이스를 구현하는 많은 컬렉터를 살펴봤다.

`toList`가 어떻게 구현되었는지 살펴보면서 **Collector**가 어떻게 정의되어있고 내부적으로 **`collect`**메서드가 `toList`가 반환하는 함수를 어떻게 활용하는지 이해하자.

```java
// Collector interface
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Cahracteristics> characteristics();
}
```

-   `T` : 수집될 스트림 항목의 제네릭 형식
-   `A` : 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
-   `R` : 수집 연산 결과 객체의 형식(대개 컬렉션)

예를 들어 `Stream<T>`의 모든 요소를 `List<T>`로 수집하는 `ToListCollector<T>`라는 클래스를 구현할 수 있다.

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
```



>   컬렉터 인터페이스에 정의된 다섯 개의 메서드 중 먼저 네 개의 메서드는 `collect`메서드에서 실행하는 함수를 반환하는 반면, 다섯번째인 `characteristics`는 `collect` 메서드가 어떤 최적화를 이용해서 리듀싱연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공한다.

## - supplier()

***새로운 결과 컨테이너 만들기***

`supplier` 메서드는 빈 결과로 이루어진 `Supplier`를 반환해야 한다.

즉, `supplier`는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수이다.

```java
// ToListCollector에서 supplier는 다음처럼 빈 리스트를 반환한다.
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}

// 생성자참조를 전달할 수도 있다.
public Supplier<List<T>> supplier() {
	return ArrayList::new;
}
```



## - accumulator()

***결과 컨테이너에 요소 추가하기***

`accumulator` 메서드는 리듀싱 연산을 수행하는 함수를 반환한다.

스트림에서 n번째 요소를 탐색할 때 두 인수, 즉 누적자와 n번째 요소를 함수에 적용한다.

함수의 반환값은 void, 즉 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부 상태가 바뀌므로 누적자가 어떤 값인지 단정할 수 없다.

```java
// ToListCollector에서 accumulator가 반환하는 함수는 이미 탐색한 항목을 포함하는 리스트에 현재 항목을 추가하는 연산을 수행한다.
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}

// 메서드 참조로 더 간결하게 표현가능하다.
public BiConsumer<List<T>, T> accumulator() {
	return List::add;
}
```



## - finisher()

***최종 변환값을 결과 컨테이너로 적용하기***

`finisher` 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환해야 한다.

때로는 누적자 객체가 이미 최종 결과인 상황도 있다.

이 때는 변환과정이 필요하지 않으므로 `finisher` 메서드는 항등함수를 반환한다.

```java
public Function<List<T>, List<T>> finisher() {
    return Function.identity();
}
```



>   **순차 리듀싱 과정의 논리적 순서**
>
>   ![image-20221228194903057](/Users/a1101716/Desktop/TIL_IMG/image-20221228194903057.png)



## - combiner()

`combiner` 메서드는 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의한다.

```java
// toList는 스트림의 두 번째 서브파트에서 수집한 리스트를 첫 번째 서브파트 결과 리스트의 뒤에 추가하면 된다.
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    }
}
```



>   **병렬화 리듀싱 과정에서 combiner 메서드 활용**
>
>   ![image-20221228195135130](/Users/a1101716/Desktop/TIL_IMG/image-20221228195135130.png)
>
>    **`combiner`** 메서드를 이용하면 **스트림의 리듀싱을 병렬로 수행할 수 있다.**
>
>   병렬 리듀싱 수행과정
>
>   1.   스트림을 분할해야 하는지 정의하는 조건이 거짓으로 바뀌기 전까지 원래 스트림을 재귀적으로 분할한다.
>        -   보통 분산된 작업의 크기가 너무 작아지면 병렬 수행의 속도는 순차 수행의 속도보다 느려진다.
>        -   즉 졍렬 수행의 효과가 상쇄된다.
>        -   일반적으로 프로세싱코어의 개수를 초과하는 병렬작업은 효율적이지 않다.
>   2.   순차 리듀싱 연산에서 보여주는 것처럼 서브스트림(substream)의 각 요소에 리듀싱 연산을 순차적으로 적용해 서브스트림을 병렬로 처리할 수 있다.
>   3.   마지막에는 컬렉터의 `combiner` 메서드가 반환하는 함수로 모든 부분결과를 쌍으로 합친다. 즉, 분할된 모든 서브스트림의 결과를 합치면서 연산이 완료된다.

## - characteristics()

`characteristics` 메서드는 컬렉터의 연산을 정의하는 `Characteristics` 형식의 불변 집합을 반환한다

`Characteristics`는 스트림을 병렬로 리듀스할 것인지, 그리고 병렬로 리듀스한다면 어떤 최적화를 선택할지 힌트를 제공한다.

불변 집합은 다음 Enum 항목을 포함한다.

-   **`UNORDERED`** : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
-   **`CONCURRENT`** : 다중 스레드에서 `accumulator` 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다.
    -   컬렉터의 플래그에 `UNORDERED`를 함께 설정하지 않았다면 데이터 소스가 정렬되어 있지 않은(집합 처럼 요소의 순서가 무의미한) 상황에서만 병렬리듀싱을 수행할 수 있다.
-   **`IDENTITY_FINISH`** : `finisher` 메서드가 반환하는 함수는 단순히 `identity`를 적용할 뿐이므로 이를 생략할 수 있다.
    -   따라서 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다.
    -   또한 누적자 A를 결과 R로 안전하게 형변환 할 수 있다.

>   지금까지 개발한 `ToListCollector`에서 스트림의 요소를 누적하는 데 사용한 리스트가 최종결과 형식이므로 추가 변환이 필요없다.
>
>   따라서 `ToListCollector`는 `IDENTITY_FINISH`이다.
>
>   하지만 리스트의 순서는 상관이 없으므로 `UNORDERED`이다.
>
>   `ToListCollector`는 `CONCURRENT` 이다.
>
>   하지만 요소의 순서가 무의미한 데이터 소스여야 병렬로 실행할 수 있다.



## -응용하기

위에서 살펴본 다섯 가지 메서드로 자신만의 커스텀 `ToListCollector`를 구현할 수 있다.

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {
    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new;					// 수집 연산의 시발점
    }
    
    @Override
	public BiConsumer<List<T>, T> accumulator() {
        return List::add;						// 탐색한 항목을 누적하고 바로 누적자를 고친다.
    }
    
    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity();				// 항등함수
    }
    
    @Override
    public BinaryOperator<List<T>> combiner() {
        return (list1, list2) -> {
            list1.addAll(list2);
            return list1;
        }
    }
    
    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(
        	IDENTITY_FINISH, CONCURRENT));		// 컬렉터의 플래그를 IDENTITY_FINISH, CONCURRENT로 설정한다.
    }
}
```

이 구현이 `Collectors.toList()` 메서드가 반환하는 결과와 완전히 같은 것은 아니지만 사소한 최적화를 제외하면 대체로 비슷하다.

이제 자바에서 제공하는 API 대신 우리가 만든 컬렉터를 메뉴 스트림의 모든 요리를 수집하는 예제에 사용할 수 있다.

```java
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());
```

기존 코드의 `toList`는 팩토리이지만 커스텀 컬렉터는 `new`로 인스턴스화한다는 점이 다르다.

>   ***컬렉터 구현을 만들지 않고도 커스텀 수집 수행***
>
>   Stream은 세 함수(발행, 누적, 합침)를 인수로 받는 `collect` 메서드를 오버로드하며 각각의 메서드는 `Collector` 인터페이스의 메서드가 반환하는 함수와 같은 기능을 수행한다.
>
>   ```java
>   // 아래처럼 스트림의 모든 항목을 리스트에 수집하는 방법도 있다.
>   List<Dish> dishes = menuStream.collect(
>   	ArrayList::new,		// 발행
>       List::add,			// 누적
>       List::addAll		// 합침
>   );
>   ```
>
>   위 코드는 간결하고 축약되어 있지만 가독성은 떨어진다. **적절한 클래스로 커스텀 컬렉터를 구현하는 편이 중복을 피하고 재사용성을 높이는 데 도움이 된다.**
>
>   또한, 위는 `Characteristics`를 전달할 수 없기 때문에 `IDENTITY_FINISH`와 `COCURRENT`이지만, `UNORDERED`는 아닌 컬렉터로만 동작한다.



# :pencil: 마치며

-   _는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터)을 인수로 갖는 **최종연산**이다.
-   미리 정의된 컬렉터인 _로 스트림의 요소를 **그룹화**하거나, _로 스트림의 요소를 **분할**할 수 있다.
