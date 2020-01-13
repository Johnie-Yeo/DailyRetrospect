# Memory management

## Overview

C와 같은 언어에는 malloc()나 free()와 같은 저수준의 메모리 관리를 위한 원시함수(primitive)가 존재합니다. 이 원시함수들은 개발자에 의해 운영체제에서 명시적으로 메모리를 할당받고 해제됩니다. Javascript는 어떤 것(객체, 문자열 등)이 생성 될 때 메모리를 할당하고 더이상 사용하지 않을 때 자동으로 메모리가 해제되는데 이를 garbage collection이라고 한다.
이렇게 리소스가 자동으로 해제되는것 처럼 보이는 것은 Javascript 개발자들(그리고 다른 high level language 개발자들)에게 더이상 메모리 관리를 신경 쓸 필요가 없다고 착각하게 하는데, 이것은 큰 실수입니다.
High level language로 작업을 할 때도 개발자라면 메모리 관리에 대한 이해가 반드시 있어야 하며 최소 기본이라도 알아야 합니다. 가끔 이런 자동 메모리 관리로 인한 문제가 발생되는데(버그나 garbage collector에서 구현상의 제약 등) 개발자들은 이런 문제를 해결하기 위해 메모리 관리에 대한 이해가 필요합니다. 

고수준 언어를 사용할 때도 개발자들은 메모리 관리에 대해 이해하고 있어야 합니다. 비록 기본적인 수준이어도 좋습니다. 자동 메모리 관리는 구현상의 제한이나 버그 때문에 문제가 있을 수 있으며 이때 개발자들이 이런 문제를 적절히 해결하거나 최소한의 트레이드오프와 기술부채로 적절한 우회법을 구현하려면 메모리 관리에 대해 이해해야 합니다. (그게 안되면 최소한의 트레이드오프와 코드 부채로 해결법을 찾아야 합니다.)

## Memory life cycle

- 어떤 프로그래밍 언어이던 메모리 생명주기는 대부분 똑같습니다.

![](https://miro.medium.com/max/1464/1*slxXgq_TO38TgtoKpWa_jQ.png)

- 다음은 싸이클의 각 단계에서 어떤 일이 일어나는 지에 대한 개요입니다.
	
	- Allocate memory — 메모리는 우리가 사용하는 프로그램이 사용 할 수 있도록 운영체제에 의해 할당됩니다. low level 언어(C와 같은)에서는 이것은 명백히 개발자가 관리해야 하는 작업입니다. high level 언어는 이러한 작업을 우리 대신 처리해 줍니다.
	- Use memory — 이제 우리 프로그램이 실제로 할당받은 메모리를 사용할 차례입니다. 우리가 코드에서 할당받은 변수를 사용합으로써 읽기와 쓰기 작업이 이루어 집니다.
	- Release memory — 우리가 사용하지 않는 메모리를 해제함으로써 메모리를 다시 사용 가능한 상태로 만드는 단계입니다. 메모리 할당과 같이 low level 언어에서만 필요한 작업입니다.

> call stack 과 memory heap에 관한 내용은 여기서 다루지 않습니다.
> [여기](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)를 참조하세요.

### What is memory?

Javascript의 메모리에 대해 이야기 하지 전 일반적으로 메모리가 뭔지 어떻게 동작하는 지 간단히 살펴 보겠습니다. 하드웨어 레벨에서 컴퓨터 메모리는 많은 flip flop들로 이뤄져 있습니다. 각각의 flip flop은 몇개의 transistor와 저장가능한 한 비트로 구성됩니다. 개별적인 flip flop들은 고유 식별자를 통해 주소를 찾을 수 있고 이를 통해 우리는 읽고 overwrite할 수 있습니다. 그러면 개념적으로 우리는 우리의 컴퓨터 전체 메모리를 하나의 거대한 비트의 array라고 볼 수 있고 읽고 쓰기도 가능합니다.
인간이기 때문에 우리는 모든 사고와 연산에 대한 것을 비트로 하는 것에 익숙하지 않습니다. 우리는 그것들이 숫자로 표현되도록 큰 그룹으로 정리할 필요가 있습니다.
8비트는 1바이트라 불립니다. 바이트가 모이면 워드가 됩니다.(16비트일 때도 있고 32비트일때도 있음)

많은 것들이 이 메모리에 저장됩니다.

1. 모든 프로그램에서 사용 될 모든 변수와 기타 데이터들
2. 프로그램의 코드, 운영체제도 포함.

컴파일러와 운영체제는 대부분의 메모리 관리를 위해 같이 동작합니다. 하지만 우리는 그것에 대해 자세히 알아볼 필요가 있습니다.

코드가 컴파일 될 때, 컴파일러는 원시데이터(primitive data) 타입을 검사하고 미리 얼마나 많은 메모리가 필요할 지 계산할 수 있습니다. 그러면 필요한 양이 call stack space에서 프로그램에 할당됩니다. 이런 변수들이 할당되는 공간을 stack space라 하는 이유는 함수가 호출 되면서 해당 함수의 메모리가 기존 메모리의 위에 추가되기 때문입니다. 함수가 종료되면 LIFO 순서로 제거됩니다.

선언문의 예에 대해 보겠습니다.

```
int n; // 4 bytes
int x[4]; // array of 4 elements, each 4 bytes
double m; // 8 bytes
```

컴파일러는 곧바로 코드가 ```4+4*4+8=28 bytes```가 필요하다는 것을 알 수 있습니다.

> 현재 int와 double에서 동작하는 방식입니다.
> 20년 전에는 int는 보통 2바이트, double은 4바이트가 사용되었습니다.
> 우리 코드는 이제 기본 데이터 타입의 크기에 대해서 의존하면 안됩니다.

컴파일러는 변수가 저장될 스택에 필요한 크기를 요청하기 위해 운영체제와 상호작용할 코드를 삽입합니다.

위의 예어서 컴파일러는 각각의 변수에 정확한 메모리 주소를 알고 있습니다.
사실 언제든지 우리가 변수 n을 적을 때마다 내부적으로는 “memory address 4127963”와 같은 형태로 치환됩니다.

그런데 우리는 x[4]에 접근하려고 할 때, 우리는 m과 연결된 데이터에 접근다는 것을 알아야 합니다. 이유는 우리는 존재하지 않는 배열 요소에 접근하는 것이기 때문입니다. 이 요소는 실제로 마지막에 할당된 x[3]로부터 4바이트 떨어져 있고 m의 비트를 일부 읽거나 overwrite할 수 있습니다. 이것은 프로그램 나머지에서 원하지 않는 결과를 불러 올 것입니다.

![](https://miro.medium.com/max/1464/1*5aBou4onl1B8xlgwoGTDOg.png)

함수가 다른 함수를 호출할 때 각각은 자신의 스택 꾸러미를 갖게 됩니다. 그것은 모든 로컬 변수를 보관하고 있고 program counter가 실행 중인지도 보관합니다. 함수가 끝날때 함수의 메모리 블럭은 다른 목적으로 사용할 수 있게 됩니다.

### Dynamic allocation

안타깝지만 컴파일 시 얼마나 많은 메모리가 요구될지 모를때는 꽤나 어렵습니다. 다음과 같은 상황을 보겠습니다.

```
int n = readInput(); // reads input from the user
...
// create an array with "n" elements
```

이런 예에서 컴파일시 컴파일러는 배열이 얼마나 많은 메모리를 필요로 할지 알 수 없습니다. 사용자의 입력에 의해 결정되기 때문이죠.

그러므로 컴파일러는 스택에 변수를 위한 공간을 할당해 줄 수 없습니다. 대신 우리의 프로그램은 런타임에서 운영체제에게 적절한 양의 공간을 요구해야 합니다. 이 메모리는 heap space에서 할당 받습니다. 정적 메모리 할당과 동적 메모리 할당은 다음과 같은 차이가 있습니다.

![](https://miro.medium.com/max/1464/1*qY-yRQWGI-DLS3zRHYHm9A.png)

동적 메모리 할당이 어떻게 동작하는지 완전히 이해하기 위해 우리는 포인터에 대해 더 많이 알아 볼 필요가 있습니다. 하지만 이것은 우리 주제와 너무 벗어난 이야기이므로 다루지 않습니다.

### Allocation in JavaScript

Javascript에서 메모리할당을 위한 첫번째 과정을 보겠습니다.
Javascripts는 개발자들에게 메모리 할당 관리에 대한 책임감을 덜어주었습니다. Javascript는 스스로 변수 할당 시점에서 메모리 할당을 수행합니다.

```Javascript
var n = 374; // allocates memory for a number
var s = 'sessionstack'; // allocates memory for a string 
var o = {
  a: 1,
  b: null
}; // allocates memory for an object and its contained values
var a = [1, null, 'str'];  // (like object) allocates memory for the
                           // array and its contained values
function f(a) {
  return a + 3;
} // allocates a function (which is a callable object)
// function expressions also allocate an object
someElement.addEventListener('click', function() {
  someElement.style.backgroundColor = 'blue';
}, false);
```

다음과 같이 몇몇 함수들은 객체 할당 결과를 가져오기도 합니다.

```Javascript
var d = new Date(); // allocates a Date object
var e = document.createElement('div'); // allocates a DOM element
```

메서드는 새로운 값이나 객체를 할당할 수 있습니다.

```Javascript
var s1 = 'sessionstack';
var s2 = s1.substr(0, 3); // s2 is a new string
// Since strings are immutable, 
// JavaScript may decide to not allocate memory, 
// but just store the [0, 3] range.
var a1 = ['str1', 'str2'];
var a2 = ['str3', 'str4'];
var a3 = a1.concat(a2); 
// new array with 4 elements being
// the concatenation of a1 and a2 elements
```

### Using memory in JavaScript

Javascript에서 할당된 메모리를 상용하는 것은 해당 메모리에 대해 읽기와 쓰기를 하는 것을 의미합니다.

이러한 것은 변수의 값 혹은 객체 속성에 대해 읽고 쓰거나 함수에 인자를 전달할 때도 일어납니다. 

### Release when the memory is not needed anymore

대부분의 메모리 관리 이슈는 이 단계에서 일어납니다.

가장 어려운 부분은 할당된 메모리가 필요하지 않다고 판단되는 시점을 파악하는 데 있습니다.
프로그램에서 메모리가 필요치 않고 해제되야 하는 부분을 개발자가 직접해야 할 때 도 가끔 생깁니다.

High-level 언어는 garbage collector라는 소프트웨어가 내장되는데 메모리 할당을 추적하고 할당된 메모리가 더이상 사용되지 않을 때 자동으로 해제해주는 역할입니다.

안타깝지만 이러한 과정은 추정인데 일부 메모리가 필요한지 아는 것은 일반적으로 결정불가능한 문제이기 때문입니다.(알고리즘으로 해결할 수 없습니다.)

대부분의 garbage collector은 메모리가 더이상 접근될 수 없을 때, 예를 들어 모든 변수가 해당 스코프를 가리키지 않을 때, 메모리를 수집하여 동작합니다. 그러나 그러한 것은 수집될 수 있는 메모리 공간을 과소평가한 것입니다. 메모리 주소를 가리키는 어떤 지점은 스코프 내에 존재하더라도 절대 접근하지 않는 경우도 생기기 때문입니다.

### Garbage collection

몇몇 메모리가 더이상 필요치 않는지 판단하는 것이 undecidable이므로 garbage collector은 일반적 문제에 대한 솔루션을 제한하여 수행합니다. 이 섹션에서는 주요 garbage collection 알고리즘과 제한점을 이해하기 위한 필수 개념을 소개합니다.

#### Memory references

garbage collection 알고리즘의 주요 개념이 의존하는 것은 reference 입니다.

메모리 관리 관점에서는 한 객체가 다른 객체에 접근할 수 있을 때(명시적 혹은 암묵적으로) 해당 객체를 참조한다고 합니다. 예를 들어 Javascript 객체는 자신의 prototype에 대한 reference(암묵적 참조)와 자신의 속성의 값(명시적 참조)의 reference를 가지고 있습니다.
이런 관점에서 객체의 개념은 보통 Javascript 객체를 벗어난 함수 스코프, 혹은 글로벌 lexical scope를 포함한다고 까지 말할 수 있습니다.

> 렉시컬 스코핑은 중첩된 함수에서 어떻게 변수 이름이 해석된는 지를 정의합니다. 내부 함수는 부모 함수가 종료되었더라도 부모스코프를 포함하고 있습니다.

#### Reference-counting garbage collection

가장 단순한 형태의 garbage collection 알고리즘입니다.
한 객체는 자신을 가리키는 포인터가 없을 때 garbage collectible 하다고 여겨집니다.
코드로 예를 보겠습니다.

```Javascript
var o1 = {
  o2: {
    x: 1
  }
};
// 2 objects are created. 
// 'o2' is referenced by 'o1' object as one of its properties.
// None can be garbage-collected

var o3 = o1; // the 'o3' variable is the second thing that 
            // has a reference to the object pointed by 'o1'. 
                                                       
o1 = 1;      // now, the object that was originally in 'o1' has a         
            // single reference, embodied by the 'o3' variable

var o4 = o3.o2; // reference to 'o2' property of the object.
                // This object has now 2 references: one as
                // a property. 
                // The other as the 'o4' variable

o3 = '374'; // The object that was originally in 'o1' has now zero
            // references to it. 
            // It can be garbage-collected.
            // However, what was its 'o2' property is still
            // referenced by the 'o4' variable, so it cannot be
            // freed.

o4 = null; // what was the 'o2' property of the object originally in
           // 'o1' has zero references to it. 
           // It can be garbage collected.
```


#### Cycles are creating problems

순환 될 때는 제약이 하나 생기게 됩니다. 다음 예어서 두 객체는 생성되고 상대방을 참조하게 되고 순환참조가 생깁니다. 함수 호출 이후 스코프를 벗어날 것이라서 쓸모 없게 되고 해제 될 수 있습니다. 그러나 reference-counting 알고리즘은 두 객체가 모두 하나 이상에서 참조되고 있다고 생각하므로 두 객체 모두 garbage-collect 될 수 없습니다.
```Javascript
function f() {
  var o1 = {};
  var o2 = {};
  o1.p = o2; // o1 references o2
  o2.p = o1; // o2 references o1. This creates a cycle.
}

f();
```

![](https://miro.medium.com/max/552/1*GF3p99CQPZkX3UkgyVKSHw.png)

#### Mark-and-sweep algorithm

객체가 필요한지 결정하기 위해 이 알고리즘은 객체가 reachable한지 판단합니다.
Mark-and-sweep algorithm 3가지 과정으로 진행됩니다.

1.Roots: 일반적으로 루트는 코드에서 참조되는 전역변수입니다. Javascript에서의 예로 루트로 동작할 수 있는 전역 변수는 window 객체가 있습니다. 같은 객체로 Nodejs에서는 global이라고 부릅니다. 모든 루트에 대한 완전한 리스트는 garbage collector에 의해 만들어집니다.

2. 알고리즘은 모든 루트와 그 자식들에 대해 검사하고 active와 같이 표시합니다.(garbage가 아님을 의미) 루트가 접근할 수 없는 모든 것은 garbage입니다.

3. 마지막으로 garbage collector은 모든 active로 표시되지 않은 모든 메모리 조각을 해제하고 운영체제에서 메모리를 돌려줍니다.

![](https://miro.medium.com/max/1390/1*WVtok3BV0NgU95mpxk9CNg.gif)

이 알고리즘은 전에 소개한 것보다 더 좋습니다. "한 객체를 참조하는 것이 0개"는 이 객체를 unreachable로 만들기 때문입니다. 그 반대는 우리가 순환참조에서 본 것처럼 참이 될 수 없습니다.
2012년을 기준으로 모든 최신 브라우저는 mark-and-sweep garbage-collector을 가지고 있습니다.  Javascript garbage collection의 모든 개선사항(generational/incremental/concurrent/parallel garbage collection)은 몇년동안 mark-and-sweep 알고리즘에 관한 구현과 향상이었습니다. (garbage collection 알고리즘 자체에 대한 향상이나 객체가 reachable 인지 결정하는 것이 아닌)

- [garbage collection 추적 mark-and-sweep의 최적화에 관한 더 자세한 이야기](https://en.wikipedia.org/wiki/Tracing_garbage_collection)

#### Cycles are not a problem anymore

첫번째 예에서 모든 함수 호출이 반환되고 난 후 두 객체가 global 객체로 부터 reachable한 객체에 대해 참조되지 않았습니다. 결과적으로 garbage collector로 부터도 unreachable이 됩니다.

![](https://miro.medium.com/max/1464/1*FbbOG9mcqWZtNajjDO6SaA.png)

객체 사이에 참조가 있지만 그들은 root로부터 reachable하지 않습니다.

#### Counterintuitive(직관적이지 않은) behavior of Garbage Collectors

Garbage Collector은 매우 편리하지만 그것 대로 트레이트 오프는 존재합니다. 그중 한가지는 비결정주의(non-determinism)입니다. 다른말로 GC는 예측할 수 없습니다. 언제 collection이 수행될지 알 수 없습니다. 이건 곧 어떤 경우에는 프로그램이 실제 필요한 것 보다 더 많은 메모리를 소모할 수도 있다는 말입니다. 다른 경우는 민감한 어플리케이션에서는 짧은 정지가 보일수 도 있습니다. non-determinism이 collection이 수행될 때를 알 수 없다는 의미이긴 하지만 대부분 GC 구현은 메모리 할당에서 collection을 넘겨주는 패턴을 가지고 있습니다. 만약 메모리 할당이 수행되는 경우가 없다면 대부분 GC는 가만히 있습니다. 다음 예시상황을 보겠습니다.

1. 상당히 많은 다수의 allocation이 일어남
2. 대부분 요소(혹은 모두 다)는 unreachable로 표시 될 것입니다.(필요하지 않은 캐시에 대한 참조포인터를 null로 가정)
3. 더이상 어떤 allocation도 일어나지 않음

이 예에서 대부분 GC는 더이상 collection을 실행하지 않습니다. 다른 말로 collection을 위한 unreachable reference가 있지만 collector는 반환하지 않습니다. 이것은 엄밀히는 누수가 아니지만 보통보다 많은 메모리 사용을 초래합니다.

### What are memory leaks?

메모리 누수는 어플리케이션에서 과거에는 사용되었지만 더이상 필요하지 않은 메모리들이 OS 혹은 free memory pool로 반환되지 않는 것을 말합니다.

![](https://miro.medium.com/max/643/1*0B-dAUOH7NrcCDP6GhKHQw.jpeg)

프로그래밍 언어는 각자 다른 방식으로 방식으로 메모리 관리하는 것을 좋아합니다. 그러나 특정 메모리가 사용되었는지 아닌지는 undecidable한 문제입니다. 다른말로 개발자만이 메모리가 OS로 반환될수 있는지 분명하게 나타낼 수 있습니다.
어떤 프로그래밍 언어는 개발자를 위해 이런 것을 도와주는 기능을 제공합니다.
그 외는 메모리가 사용되지 않는 상황을 개발자에게 전적으로 의지합니다. (위키피디아 참조)

### The four types of common JavaScript leaks

#### 1: Global variables

Javascript는 선언되지 않는 변수를 재밌는 방법으로 다룹니다.
선언되지 않은 변수가 차보되었을때 새로운 변수가 global object에 생성됩니다. 브라우저상에서는 global object는 window가 되고 다음을 의미합니다.

```Javascript
function foo(arg) {
    bar = "some text";
}
```

위와 아래는 동일합니다.

```Javascript
function foo(arg) {
    window.bar = "some text";
}
```

bar의 목적이 foo 함수의 어떤 변수를 참조하는 것이라고 가정해 보겠습니다. var을 통해 선언하지 않았다면 불필요한 전역변수가 생성될 것입니다. 위의 예에서 이것은 큰 문제가 되지는 않습니다. 그렇지만 분명히 위험한 예도 떠올릴 수 있을 것입니다.

또 this를 이용한 전역 변수가 실수로 생길 수 도 있습니다.

```Javascript
function foo() {
    this.var1 = "potential accidental global";
}
// Foo called on its own, this points to the global object (window)
// rather than being undefined.
foo();
```

> Javascript 파일의 도입부에 'use strict'를 추가함으로써 이런 상황을 피할수 있습니다.
> 이 모드에서는 예상하지 못한 전역 변수 생성을 막아주는 엄격한 모드의 파싱으로 바뀝니다.

예상하지 못한 전역변수는 당연히 문제가 되지만 많은 경우 Garbage Collector에서 정의에 의해 수집할 수 없는 명시적 전역변수가 넘쳐나는 경우가 훨씬 많습니다. 전역 변수를 통해 임시적인 데이터를 저장할 때와 많은 정보를 처리할 때는 특별한 주의가 필요합니다. 반드시 전역변수를 통해 데이터를 저장해야 한다면 사용 후에는 반드시 null을 넣거나 reassign하는 과정을 수행해야 합ㄴ니다.

#### 2: Timers or callbacks that are forgotten

Javascript에서 자주 사용되는 setInterval을 예로 보겠습니다.

콜백을 허용하고 Observer와 기타 도구를 제공하는 라이브러리는 대부분의 경우 인스턴스에 대해 unreachable 일 때 콜백에 대한 모든 reference가 unreachable 되게 합니다. 하지만 아직도 종종 아래와 같은 코드가 보이곤 합니다.

```Javascript
var serverData = loadData();
setInterval(function() {
    var renderer = document.getElementById('renderer');
    if(renderer) {
        renderer.innerHTML = JSON.stringify(serverData);
    }
}, 5000); //This will be executed every ~5 seconds.
```

필요하지 않은 노드 혹은 데이터를 참조한 타이머를 사용한 결과입니다.

renderer 객체는 inteval handler에 의해 캡슐화된 블럭이 필요없게 만드는 어떤 시점에서 대체되거나 제거될 수 있습니다. 이런 일이 일어나면 interval이 먼저 멈춰야하는데 interval은 아직 active 상태이므로 핸들러나 핸들러에 의존하는 것들 모두 collect되지 못합니다. 결국 모든 데이터를 확실히 저장하고 처리하는 serverDat도 collect 되지 못합니다.

observer을 사용할 때, 사용이 끝나면 명시적으로 제거를 호출해야하는 과정이 반드시 필요합니다. observer가 더이상 사용되지 않을 때와 객체가 unreachable이 될 때 모두에 해당합니다.

다행히 대부분 최신 브라우저는 이러한 작업을 알아서 처리해 줍니다. 개발자가 리스너를 제거하는 것을 잊어도 관측되는 객체가 unreachable이 되면 자동으로 observer handler을 가져 갑니다. 과거에는 어떤 브라우저에서는 이런 경우는 불가능했습니다.(IE6)

그럼에도 여전히 객체가 쓸모 없게 되면 제거하는 것이 best practice 입니다.
해당 예시입니다.

```Javascript
var element = document.getElementById('launch-button');
var counter = 0;
function onClick(event) {
   counter++;
   element.innerHtml = 'text ' + counter;
}
element.addEventListener('click', onClick);
// Do stuff
element.removeEventListener('click', onClick);
element.parentNode.removeChild(element);
// Now when element goes out of scope,
// both element and onClick will be collected even in old browsers // that don't handle cycles well.
```

노드를 unreachable로 만들기 전에 removeEventListener을 더이상 호출해줄 필요가 없어졌습니다. 최신 브라우저들이 이런 싸이클에 대한 탐지와 적절히 처리해주는 garbage collector을 지원해주기 때문입니다. 
만약 jQuery API를 사용한다면(혹은 이런 것을 지원하는 다른 라이브러리나 프레임워크를 사용한다면) 노드가 쓸모없어지기 전에 리스너를 제거할 수 있습니다. 라이브러리는 어플리케이션이 구형 브라우저 위에서 돌아갈 때도 메모리 누수를 막을 수 있게 해줍니다.

#### 3: Closures

Javascript 개발의 중요한 요소는 클로져 입니다.
Javascript 런타임 구현의 특성 때문에 다음과 같은 메모리 누수가 일어날 수 있습니다.

```Javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing) // a reference to 'originalThing'
      console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log("message");
    }
  };
};
setInterval(replaceThing, 1000);
```

replaceThing이 호출되면 theThing은 큰 배열과 클로져로 이뤄진 객체를 받게 됩니다. 아직 originalThing은 unused에 걸린 클로져에 의해 참조되고 있습니다. 여기서 originalThing은 이전의 replaceThing 호출에서의 theThing을 말합니다.
주목할 점은 클로져를 위한 스코프가 생성되면 같은 부모 스코프에 클로저는 그 스코프가 공유된다는 점입니다.(말 드럽게 어렵네. 원문 : The thing to remember is that once a scope for closures is created for closures in the same parent scope, the scope is shared.)

이러한 경우 someMetho를 위해 생성된 스코프는 unused와 공유됩니다. unused는 originalThing에 대한 참조가 있습니다. unused가 한번도 사용되지는 않아도 someMethod는 theThing을 통해 replaceThing 스코프 바깥에서 사용될 수 있습니다.(전역공간과 같은)
그리고 someMethod가 unused와 클로져 스코프를 공유함으로써 unused reference가 originalThing에 대해 갖고 있는 참조 때문에 강제로 active 상태가 유지됩니다(두 클로져 사이에 공유된 전체 스코프). 이게 collection을 막을 수 있습니다.

이런 모든 것이 메모리 누수를 초래할 수 있습니다. 이 코드가 실행 될 때마다 메모리 사용량이 급증하는 것을 볼 수 있습니다. garbage collector가 동작해도 사이즈는 줄어들지 않을 것입니다. 클로저의 링크드 리스트가 생성되고(여기서의 루트는 theThing 변수) 각각의 클로저 스코프는 큰 배열에 대한 간접 참조가 전달되게 됩니다. 

이 문제는 Meteor 팀에서 발견되었습니다.

#### 4: Out of DOM references

개발자들이 DOM 노드를 자료구조 내부에 저장하는 경우가 있습니다. 테이블 내의 몇몇 열에 대해 업데이트를 빠르게 하고 싶다고 가정해 보겠습니다. 만약 각각의 DOM 열에 대해 dictionary 혹은 array로 가지고 있다면 각각의 DOM element 마다 두가지 reference가 존재할 것입니다. (하나는 DOM tree에 하나는 dictionary에) 만약 이 열ㅇ르 제거하고 싶다면 두가지 reference 모두 unreachable로 만들어 줘야 합니다.

```Javascript
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image')
};
function doStuff() {
    elements.image.src = 'http://example.com/image_name.png';
}
function removeImage() {
    // The image is a direct child of the body element.
    document.body.removeChild(document.getElementById('image'));
    // At this point, we still have a reference to #button in the
    //global elements object. In other words, the button element is
    //still in memory and cannot be collected by the GC.
}
```

DOM 트리 내부의 리프노드 혹은 inner 노드를 참조할 때 고려해야 할 사항이 또 있습니다. td태그와 같이 테이블 셀에 대한 참조를 코드에 가지고 있고 테이블을 DOM에서 삭제하기로 결정했는데도 해당 셀의 참조를 가지고 있으면 큰 메모리 누수가 따를 수 있습니다. garbage collector가 해당 셀 외의 모든 것을 해제할 것이라 생각한 수는 있습니다. 하지만 이건 다른경우입니다. 셀이 테이블의 자식노드이고 자식은 부모의 reference를 가지고 있기 때문에 이 하나의 테이블에 대한 reference가 메모리에 테이블 전체를 계속 가지고 있게 되는 것입니다.

((대충 자기 자랑)
Sessionstack의 개발자들은 메모리 할당을 적절히 처리하는 best practice를 따라 코드를 작성합니다. 아래는 그 이유 입니다.

SessionStack을 프로덕션 web app에 추가하게 되면 모든 것을 기록하기 시작합니다.(모든 DOM 변화, user interactions, Javascript exception, 스택 순회, 네트워크 요청 실패, 디버그 메세지 등등)
SessionStack에서는 web app의 문제를 동영상으로 재생할 수 있고 유저에게 생긴 모든 일을 볼 수 있습니다. 그리고 이런 모든 것은 web app의 성능에 전혀 지장을 주지 않으면서 일어나야 합니다.
유저가 페이지를 reload 하거나 앱 내부를 돌아다니기 때문에 모든 observer, interceptor, 변수 할당 등이 적절히 처리되야 합니다. 그래서 어떠한 메모리 누수도 발생도 유발하않고 메모리 소모를 늘리지 않습니다.
)

## Reference

- [How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)
- [자바스크립트는 어떻게 작동하는가 메모리 관리 4가지 흔한 메모리 누수 대처법](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B4%80%EB%A6%AC-4%EA%B0%80%EC%A7%80-%ED%9D%94%ED%95%9C-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EB%88%84%EC%88%98-%EB%8C%80%EC%B2%98%EB%B2%95-5b0d217d788d)