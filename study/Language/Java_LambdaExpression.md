# Lambda Expression

- 식별자 없이 실행 가능한 함수
- Javascript의 Anonymous function와 매우 흡사한 형태이다.
- Java를 Functional programming을 가능하게 해준다!

## Funtional Programming이란

- 함수의 입력에만 의존하여 출력
- 외부 상태 변경을 지양
	- side effect의 최소화

### Funtional Programming 조건

- 순수함수
- 익명함수
- 고계함수
	- 함수를 다루는 상위의 함수로 함수형 언어에서는 함수도 하나의 값으로 취급하고, 함수의 인자로 함수를 전달할 수 있는 특성이 있다. 이러한 함수를 일급 객체 (a.k.a 일급 함수) 로 간주한다.

## Java는 함수의 개념이 없다

### Method ⊂ Funtion

#### Function 

- 특정 작업을 수행하는 코드조각
- 독립된 기능을 수행하는 단위

#### Method

- 클래스, 구조체, 열거형에 포함되어있는 함수
- 다른말로 클래스 함수라고도 한다.

- Java는 모든 것이 객체이므로 기존의 언어 체계에서는 함수형 언어를 코드레벨에서 구현이 불가능했다.

### Java v8에서의 변화

#### 함수형 인터페이스

- 단 하나의 메서드만이 선언된 인터페이스
- 람다식으로 표현 가능하게 되었다.

> 람다식 표현을 통해 입력에 의해서만 출력이 결정되도록 순수 함수를 표현할 수 있게 되었고, 람다식으로 표현함으로써 익명 함수를 정의할 수 있게 되었고, 함수형 인터페이스의 메소드에서 또다른 함수형 인터페이스를 인자로 받을 수 있도록 하여 고계 함수를 정의할 수 있게 되었다. 즉, 함수형 프로그래밍 언어의 조건을 만족시킬 수 있게 되었다고 할 수 있다.

## Example(Runnable interface)

```
// Thread - traditional
new Thread(new Runnable() {
	@Override
	public void run() {
		System.out.println("Hello World.");
	}
}).start();
```

```
// Thread - Lambda Expression
new Thread(()->{
	System.out.println("Hello World.");
}).start();
```

## Stream API

```
List<String> names = Arrays.asList("jeong", "pro", "jdk", "java");
// 기존의 코딩 방식
long count = 0;
for (String name : names) {
    if (name.contains("o")) {
        count++;
    }
}
System.out.println("Count : " + count); // 2
 
// 스트림 이용한 방식
count = 0;
count = names.stream().filter(x -> x.contains("o")).count();
System.out.println("Count : " + count); // 2
```

- 컬렉션, 배열등의 저장 요소를 하나씩 참조하며 함수형 인터페이스(람다식)를 적용하며 반복적으로 처리할 수 있도록 해주는 기능
- 불필요한 코딩(for, if 문법)을 걷어낼 수 있고 직관적이기 때문에 가독성이 좋아진다.

### Stream은 어떤 것들에 적용할 수 있을까?

- Stream은 주로 Collection, Arrays에서 쓰인다.
- 물론 두 개 뿐만 아니라 I/O resources(ex. File), Generators, Stream ranges, Pattern 등에서도 사용할 수 있다.

### 스트림 사용법

#### 스트림의 구조

1. 스트림생성
2. 중개 연산
3. 최종 연산

> 실제 사용법으로 표기하면 "Collections같은 객체 집합.스트림생성().중개연산().최종연산();" 이런식이다.

#### 중개연산

- filter, map, reduce, forEach등 Javascript에서 자주 보던 친구들을 쓸 수 있다.
- 이 외 iterator, noneMatch, allMatch 등도 있다.

### 스트림 주의 사항

- 재사용이 불가능하다
- 병렬 스트림은 여러 쓰레드가 작업한다.

```
names.parallelStream().filter(x -> x.contains("o")).count();
```

	- stream()으로 스트림을 생성하지 않고 위 처럼 parallelStream()으로 병렬 스트림을 만들 수 있다.
	- 이렇게하면 여러 쓰레드가 스트림에서 요소를 필터링하고 나온 요소 수를 계산하고 쓰레드끼리 다시 한 번 각자 계산한 count 값들을 더해서 리턴해준다.
	- 단순하게 생각하면 여러쓰레드가 처리해주니 병렬스트림이 항상 성능면에서 유리해보일 수 있지만 애플리케이션에서 사용하는 쓰레드가 많거나 스트림의 요소 수가 많지 않다면 오히려 쓰레드를 사용하는데 드는 오버헤드가 더 클 수도 있다.


[ref1](https://skyoo2003.github.io/post/2016/11/09/java8-lambda-expression)
[ref2](https://zeddios.tistory.com/233)
[ref3](https://jdm.kr/blog/181)
[ref4](https://jeong-pro.tistory.com/165)