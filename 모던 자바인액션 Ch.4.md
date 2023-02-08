# Ch.4 스트림 소개

## 스트림의 유용한 기능

스트림은 아주 간단하게 말해 데이터 컬렉션 반복을 멋지게 처리하는 기능이다. 이러한 스트림엔 어떠한 유용한 기능이 있을까?
아래의 코드는 저칼로리의 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬하는 자바7 코드다.

```java
List<Dish> lowCalorcDishes = new ArrayList<>();
for (Dish dish: menu) {
	if(dish.getCalories() <400) {
    	lowCaloricDishes.add(dish);
    }
}
Collections.sort(lowCaloricDishes, new Comparator<Dish> () { //익명 클래스로 요리 정렬
	public int compare(Dish dish1, Dish dish2) {
    	return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
});
List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish: lowCaloricDishes) {
	lowCaloricDishes.add(dish.getName()); //정렬된 리스트를 순회하며 요리 이름 선택
}
```

위 코드에서 lowCalricDishes 는 잠시 컨테이너 역할만 하는 중간 변수다. 즉 '가비지 변수'이다.
이러한 세부 구현을 자바 8에선 아래처럼 라이브러리 내에서 모두 처리 가능하다.

```java
List<String> lowCaloricDishesName = 
			menu.stream()
            	.filter(d -> d.getCalories() <400) 
                .sorted(comparing(Dish:getCalories))
                .map(Dish::getName)
                .collect(toList());
```

그리고 이 stream()을 parellStream()으로 바꾸면 멀티코어 아키텍처에서 병렬로 실행도 가능하다.
(이 부분은 7장에서 더 자세히 설명하겠다)
위와 같은 스트림을 이용한 코드로 스트림의 새로운 기능을 정리해보자.

**1. 선언형 코드를 구현할 수 있다.**
루프와 if 조건문등의 제어 블록을 사용하여 어떤 동작을 구현할지 지정할 필요 없이, '저칼로리의 요리만 선택하라'와 같은 동작의 수행을 지정할 수 있다. 만약 '고칼로리의 요리만 선택하라'로 동작이 바뀐다면 코드를 복사 붙여넣기 할 필요 없이, 람다 표현식을 이용해서 구현할 수 있다

**2. 여러 연산을 데이터 파이프라인으로 만들어 가독성과 명확성을 유지한다.**
filter, sorted, map, collect와 같은 고수준 빌딩 블록 (high-level building block) 연산을 연결하여 스트림 파이프 라인을 형성한다. 고수준 빌딩 블록은 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있다. 결과적으로 데이터 처리 과정을 병렬화하면서 스레드와 락을 걱정할 필요가 없다.

즉, 스트림 API 특징을 요약해보자면

- 선언형: 더 간결하고 가독성이 좋아짐
- 조립할 수 있음: 유연성이 좋아진다.
- 병렬화: 성능이 좋아진다.

## 스트림 시작하기

스트림이란 '데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소'로 정의할 수 있다.
스트림을 컬렉션과 비교하여 설명해보자.

- 연속된 요소
  스트림은 연속된 값 집합의 인터페이스를 제공한다. 컬렉션은 자료구조이므로 요소의 저장 및 접근 연산이 주를 이룬다. ex) add, get..
  반면, 스트림은 filter, sorted, map처럼 표현 계산식이 주를 이룬다. 즉, 컬렉션의 주제는 **데이터**이고, 스트림의 주제는 **계산**이다.
- 소스
  스트림은 컬렉션, 배열, I/O 자원등의 데이터 제공 소스로부터 데이터를 소비한다. 정렬된 컬렉션으로 스트림을 생성하면? 정렬은 그대로 유지된다. 즉, 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지한다.
- 데이터 처리 연산
  스트림은 filter, map, reduce, find, match, sort 등으로 데이터를 조작할 수 있다.

### 스트림의 중요한 두가지 특징

- 파이프라이닝
  스트림 연산은 스트림 연산끼리 연결하여 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
- 내부 반복
  컬렉션은 반복자를 이용하여 명시적으로 반복한다.but, 스트림은 내부반복을 지원한다.

```java
List<String> threeHighCaloricDishesNames = 
			menu.stream()
            	.filter(dish -> dish.getCalories() > 300) 
                .map(Dish::getName)
                .limit(3)
                .collect(toList());
System.out.println(threeHighCaloricDishesNames);
```

1. menu에서 stream 메서드를 호출하여 스트림을 얻었다. 이때, **데이터 소스** 는 요리 리스트(메뉴)이다. 데이터 소스는 **연속된 요소**를 스트림에 제공한다.
2. 스트림에 filter, map, limit, collect로 이어지는 일련의 **데이터 처리연산**을 적용한다. collect를 제외한 모든 연산은 서로 **파이프라인**을 형성하도록 스트림을 반환한다.
3. collect연산으로 파이프라인을 처리해서 결과를 반환한다.(List를 반환)
   - filter: 람다를 인수로 받아 스트림에서 특정 요소를 제외한다. (300칼로리 이하 요리 제외)
   - map: 람다를 이용하여 한 요소를 변환하거나 정보를 추출한다. (이름 추출)
   - limit: 정해진 개수로 스트림의 크기를 축소한다.
   - collect: 스트림을 다른 형식으로 변환한다.

이러한 스트림 라이브러리에서 필터링, 추출, 축소 기능을 제공하므로 직접 기능을 구현할 필요가 없다!

## 스트림과 컬렉션

컬렉션과 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. 하지만 이 둘의 차이점을 알아보자.

1. 데이터를 언제 계산하느냐?
   컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조이다. 즉, 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다.
   반면, 스트림은 요청할 때만 계산하는 자료구조이다. 사용자가 요청한 값만 스트림에서 추출한다는 것이 핵심이다.
2. 스트림은 딱 한번만 탐색할 수 있다.
   탐색된 스트림의 요소는 소비된다. 따라서 한번 탐색한 요소를 재탐색하려면 초기 데이터 소스에서 새롭게 스트림을 다시 만들어야한다.
3. 데이터 반복 처리 방법
   컬렉션은 **외부반복**, 스트림은 **내부반복**을 이용한다.

- 외부 반복: 사용자가 직접 요소를 반복하는 것 ex)for-each

```java
List<String> names = new ArrayList<>();
for(Dish dish:menu) {
	names.add(dish.getName());
```

- 내부 반복: 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해주는 것

```java
List<String> names = menu.stream()
					     .map(Dish::getName)
                         .collect(toList());
```

내부 반복은 반복과정을 신경 쓸 필요가 없고 작업을 투명하게 병렬로 처리하거나 더 최적화된 다양한 순서로 처리할 수 있다.

## 스트림 연산

```java
List<String> threeHighCaloricDishesNames = 
			menu.stream()
            	.filter(dish -> dish.getCalories() > 300) 
                .map(Dish::getName)
                .limit(3)
                .collect(toList());
```

아까 본 이 예제에서 연산은 두 그룹으로 구분할 수있다.

- filter, map, limit은 서로 연결되어 파이프라인을 형성한다
- collect로 파이프라인을 실행하고 닫는다.

#### 중간 연산

연결할 수 있는 스트림 연산을 중간연산이라고 한다. filter, sorted와 같은 중간 연산은 다른 스트림을 반환한다. 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다. 중간 연산을 합친 다음, 합쳐진 중간 연산을 최종 연산으로 한번에 처리한다.

#### 최종 연산

스트림 파이프라인에 결과를 도출한다. 최종 연산에 의해 스트림이 아닌 List, Integer, void 등의 결과가 반환된다.