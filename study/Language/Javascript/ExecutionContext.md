# Execution Context

## 실행 컨텍스트

- 실행 컨텍스트를 통해 자바스크립트가 왜 그렇게 동작하는 지 알 수 있다.

```Javascript
var name = 'zero'; // (1)변수 선언 (6)변수 대입
function wow(word) { // (2)변수 선언 (3)변수 대입
  console.log(word + ' ' + name); // (11)
}
function say () { // (4)변수 선언 (5)변수 대입
  var name = 'nero'; // (8)
  console.log(name); // (9)
  wow('hello'); // (10)
}
say(); // (7)
```

- lexical scoping을 알고 있다면 nero, hello zero가 출력된다는 것을 알 수 있다.

> lexical scope : 함수가 생성되는 환경의 스코프 

- 처음 코드를 실행하는 순간 모든 것을 포함하는 전역 컨텍스트가 생긴다. 
	- 전역 컨텍스트 : 모든 것을 관리하는 환경. 페이지가 종료될 때까지 유지. 
	- 함수 컨텍스트 : 함수를 호출할 때마다 함수 컨텍스트가 하나씩 더 생긴다. 

### 컨텍스트의 원칙 네 가지

1. 전역 컨텍스트 하나 생성 후, 함수 호출 시마다 컨텍스트가 생긴다.
2. 컨텍스트 생성 시 컨텍스트 안에 변수객체(arguments, variable), scope chain, this가 생성된다.
3. 컨텍스트 생성 후 함수가 실행되는데, 사용되는 변수들은 변수 객체 안에서 값을 찾고, 없다면 스코프 체인을 따라 올라가며 찾는다.
4.함수 실행이 마무리되면 해당 컨텍스트는 사라집니다.(클로저 제외) 페이지가 종료되면 전역 컨텍스트가 사라진다.

- 아까 코드를 이 원칙들에 따라서 실행해보면

### 전역 컨텍스트

- 전역 컨텍스트가 생성된 후 두 번째 원칙에 따라 변수객체, scope chain, this가 들어온다. 
- 전역 컨텍스트는 arguments가 없고 variable은 해당 스코프의 변수들이다.
	- 여기서 variable은 name, wow, say

- scope chain(스코프 체인, 자신과 상위 스코프들의 변수객체)은 자기 자신인 전역 변수객체. 
- this는 따로 설정되어 있지 않으면 window.
	> ##### this를 바꾸는 방법 
	> - new 호출
	> - 함수에 다른 this 값을 bind

- this는 기본적으로 window고 new나 bind같은 상황에서 this가 바뀐다.

- 이것을 객체 형식으로 표현해보면
```Javascript
'전역 컨텍스트': {
  변수객체: {
    arguments: null,
    variable: ['name', 'wow', 'say'],
  },
  scopeChain: ['전역 변수객체'],
  this: window,
}
```

- 이제 코드를 위에서부터 실행해 보자. 
- wow랑 say는 호이스팅 때문에 선언과 동시에 대입 된다.
- 그 후 variable의 name에 'zero'가 대입된다.

```
variable: [{ name: 'zero' }, { wow: Function }, { say: Function }]
```

### 함수 컨텍스트

- 그 후 (7)번에서 say();를 하는 순간 새로운 컨텍스트인 say 함수 컨텍스트가 생긴다. 
	- 이때 이전의 전역 컨텍스트는 그대로 있다. 
- arguments는 없고, variable은 name. 
- scope chain은 say 변수객체와 상위의 전역 변수객체이다. 
- this는 따로 설정해준 적이 없으니까 window입니다.

```Javascript
'say 컨텍스트': {
  변수객체: {
    arguments: null,
    variable: ['name'], // 초기화 후 [{ name: 'nero' }]가 됨
  },
  scopeChain: ['say 변수객체', '전역 변수객체'],
  this: window,
}
```

- say를 호출한 후 위에서부터 차례대로 (8)~(10)이 실행된다. 
- variable의 name에 nero를 대입해주고 나서 console.log(name);이 있다. 
	- name 변수는 say 컨텍스트 안에서 찾으면 된다. 
	- variable에 name이 nero라고 되어 있으므로 nero가 콘솔에 찍힌다. 
- 그 다음엔 wow('hello')가 있는데 say 컨텍스트 안에서 wow 변수를 찾을 수 없다.
	- 찾을 수 없다면 scope chain을 따라 올라가 상위 변수객체에서 찾는다. 
	- 그래서 전역 변수객체에서 찾고 전역 변수객체의 variable에 wow라는 함수가 있으므로 이걸 호출한다.

- (10)번에서 wow함수가 호출되었으니 wow 컨텍스트도 생긴다.
	- arguments는 word = 'hello'고, scope chain은 wow 스코프와 전역 스코프
	- 여기서 중요한 것은 lexical scoping에 따라 wow 함수의 스코프 체인은 선언 시에 이미 정해져 있다. 따라서 say 스코프는 wow 컨텍스트의 scope chain이 아니다. 
		-variable은 없고, this는 window

```Javascript
'wow 컨텍스트': {
  변수객체: {
    arguments: [{ word : 'hello' }],
    variable: null,
  },
  scopeChain: ['wow 변수객체', '전역 변수객체'],
  this: window,
}
```

- 이제 컨텍스트가 생긴 후 함수가 실행 되는데 나중에 생긴 wow가 가장 먼저 실행된다. 
- wow 함수 안에서 console.log(word + ' ' + name);이 있는데 word랑 name 변수는 wow 컨텍스트에서 찾으면 된다. 
	- word는 arguments에서 찾을 수 있고, name은 wow 변수객체에는 값이 없으니, scope chain을 따라 전역 스코프에서 찾을 수 있다. 
	- 전역 변수객체로 올라가니 variable에 name이 zero라고 되어 있고 따라서 출력이 hello zero가 된다. 
		- wow 컨텍스트에 따르면 wow 함수는 애초에 say 컨텍스트와 일절 관련이 없다.

- 이제 wow 함수 종료 후 wow 컨텍스트가 사라지고, say 함수의 실행이 마무리된다.
- 따라서 say 컨텍스트도 사라지고, 마지막에 전역 컨텍스트도 사라진다. 

- 함수 실행, 변수 선언 등 모든 게 다 논리적! 
- 그래서 컨텍스트 개념을 이해하면 자바스크립트의 모든 문제들을 풀 수 있다. 

### 호이스팅

#### 호이스팅이란?

- 변수를 선언하고 초기화했을 때 선언 부분이 최상단으로 끌어올려지는 현상 
	- 초기화 또는 대입 부분은 그대로 남아있다
	- 아래처럼 sayWow처럼 함수 표현식이 아니라 함수 선언식일 때는 식 자체가 통째로 끌어올려진다.

```Javascript
console.log(zero); // 에러가 아니라 undefined
sayWow(); // 정상적으로 wow
function sayWow() {
  console.log('wow');
}
var zero = 'zero';
```

- 선언보다 호출을 먼저 하기 때문에 얼핏 보기에 말이 안 되는 것처럼 보인다. 
- 하지만 에러 없이 정상 작동한다. 변수 선언과 함수 선언식이 최상단으로 끌어올려졌기 때문. 

- 다음과 같이 나타낼 수 도 있다.

```Javascript
function sayWow() {
  console.log('wow');
}
var zero;
console.log(zero);
sayWow();
zero = 'zero';
```

- 하지만 같은 함수여도 함수 표현식으로 선언한 경우에는 에러가 발생한다.

```Javascript
sayWow(); // (3)
sayYeah(); // (5) 여기서 대입되기 전에 호출해서 에러
var sayYeah = function() { // (1) 선언 (6) 대입
  console.log('yeah');
}
function sayWow() { // (2) 선언과 동시에 초기화(호이스팅)
  console.log('wow'); // (4)
}
```

- 일단 처음 실행 시는 전역 컨텍스트가 먼저 생성되고 sayWow 함수는 함수 선언식이므로 컨텍스트 생성 후 바로 대입된.

```Javascript
'전역 컨텍스트': {
  변수객체: {
    arguments: null,
    variable: [{ sayWow: Function }, 'sayYeah'],
  },
  scopeChain: ['전역 변수객체'],
  this: window,
}
```

- 컨텍스트 생성 및 코드가 순차적으로 실행되는데 sayYeah는 대입되기 전에 호출해서 에러가 발생한다.

### 클로저

- 비공개 변수를 가질 수 있는 환경에 있는 함수가 클로저. 
- 비공개 변수는 클로저 함수 내부에 생성한 변수도 아니고, 매개변수도 아닌 변수. 
- 클로저를 말할 때는 스코프/컨텍스트/비공개 변수와 함수의 관계를 항상 같이 말해주어야 한다.

```Javascript
var makeClosure = function() {
  var name = 'zero';
  return function () {
    console.log(name);
  }
};
var closure = makeClosure(); // function () { console.log(name); }
closure(); // 'zero';
```

- closure 함수 안에 console.log(name)이 있는데 name은 closure 함수의 매개변수도 아니고, closure 함수 내부에서 생성한 변수도 아니다. 바로 이런 것이 비공개 변수이다. 
- function() { console.log(name) }은 name 변수나, name 변수가 있는 스코프에 대해 클로저라고 부를 수 있다. 수학적으로는 자유변수라고도 한다.

- 이걸 컨텍스트로 분석해보자. 
- 전역 컨텍스트 생성 후, makeClosure 함수 호출 시 makeClosure 컨텍스트도 만들어진다.

```javascript
"전역 컨텍스트": {
  변수객체: {
    arguments: null,
    variable: [{ makeClosure: Function }, 'closure'],
  },
  scopeChain: ['전역 변수객체'],
  this: window,
}
"makeClosure 컨텍스트": {
  변수객체: {
    arguments: null,
    variable: [{ name: 'zero' }],
  },
  scopeChain: ['makeClosure 변수객체', '전역 변수객체'],
  this: window,
}
```

- 주목할 점은 closure = makeClosure()할 때의 상황이다. 
- function을 return하는데 그 function 선언 시의 scope chain은 lexical scoping을 따라서 ['makeClosure 변수객체', '전역 변수객체']를 포함한다. 
  - 따라서 closure을 호출할 때 컨텍스트는 다음과 같다.

```Javascript
"closure 컨텍스트":  {
  변수객체: {
    arguments: null,
    variable: null,
  scopeChain: ['closure 변수객체', 'makeClosure 변수객체', '전역 변수객체'],
  this: window,
}
```

- 따라서 closure 함수에서 scope chain을 통해 makeClosure의 name 변수에 접근할 수 있다. 
- 다음은 클로저를 활용한 유명한 카운터 예제 이다.

```Javascript
var counter = function() {
  var count = 0;
  function changeCount(number) {
    count += number;
  }
  return {
    increase: function() {
      changeCount(1);
    },
    decrease: function() {
      changeCount(-1);
    },
    show: function() {
      alert(count);
    }
  }
};
var counterClosure = counter();
counterClosure.increase();
counterClosure.show(); // 1
counterClosure.decrease();
counterClosure.show(); // 0
```

- counter 함수는 호출 시 return을 통해 counterClosure 컨텍스트에 비공개 변수인 count에 접근할 수 있는 scope chain을 반환한다. 
- 그렇게 되면 이제 counterClosure에서 계속 count로 접근할 수 있다. return 안에 들어 있는 함수들은 count 변수나, changeCount 함수 또는 그것들을 포함하고 있는 스코프에 대한 클로저라고 부를 수 있다.
 
- 단점으로는 잘못 사용했을 시 성능 문제와 메모리 문제가 발생할 수 있다. 
- closure의 비공개 변수는 자바스크립트에서 언제 메모리 관리를 해야할 지 모르기 때문에 자칫 메모리 낭비로 이어질 수 있다. 
- 또한 scope chain을 거슬러 올라가는 행동을 하기 때문에 조금 느립니다.

[실행 컨텍스트](https://www.zerocho.com/category/JavaScript/post/5741d96d094da4986bc950a0)