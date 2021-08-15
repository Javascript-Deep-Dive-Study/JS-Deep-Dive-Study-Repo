# 24장: 클로저

- 자바스크립트 고유의 개념이 아니므로 ECMAScript 정의에는 등장하지 않음
- 함수와 그 함수가 선언된 렉시컬 환경과의 조합

```js
const x = 1;

function outerFunc() {
  const x = 10;
  
  function innerFunc() {
    console.log("inner: ", x);
  }
  innerFunc();	// inner: 10
  globalFunc();		// global: 1
}

function globalFunc() {
  console.log("global: ", x);
}

outerFunc();
```

- 위와 같은 현상이 발생하는 이유는 자바스크립트가 렉시컬 스코프를 따르는 프로그래밍 언어이기 때문

<br>

## 01. 렉시컬 스코프

- **렉시컬 스코프(정적 스코프)**? 함수를 어디서 호출했는지가 아니라 함수를 어디에 정의했는지에 따라 상위 스코프를 결정하는 것

```js
const x = 1;

function foo() {
  const x = 10;
  bar();
}

function bar() {
  console.log(x);
}

// foo()와 bar()는 모두 전역에서 정의된 전역 함수이고,
// 함수의 상위 스코프는 함수를 호출한 위치와 상관없이 함수를 어디서 정의했느냐에 따라 결정되므로
// 두 함수의 상위 스코프는 전역임 => 상위 스코프가 정적으로 결정됨
foo();	// 1
bar();	// 1
```

- 렉시컬 환경의 "외부 렉시컬 환경에 대한 참조"에 저장할 참조값, 즉 상위 스코프에 대한 참조는 함수 정의가 평가되는 시점에 함수가 정의된 환경(위치)에 의해 결정

<br>

## 02. 함수 객체의 내부 슬롯 `[[Environment]]`

- 함수가 정의된 환경과 호출되는 환경은 다를 수 있으므로 렉시컬 스코프가 가능하려면 함수는 자신이 호출되는 환경과는 상관없이 자신이 정의된 환경, 즉 상위 스코프(함수 정의가 위치하는 스코프)를 기억해야함
  - 함수 정의가 평가되어 함수 객체를 생성할 때 자신이 정의된 환경에 의해 결정된 상위 스코프의 참조를 함수 객체 자신의 내부 슬롯 `[[Environment]]`에 저장
  - 자신의 내부 슬롯 `[[Environment]]`에 저장된 상위 스코프의 참조는 현재 실행 중인 실행 컨텍스트의 렉시컬 환경을 가리킴
- 함수 객체는 내부 슬롯 `[[Environment]]`에 저장한 렉시컬 환경의 참조, 즉 상위 스코프를 자신이 존재하는 한 기억

```js
const x = 1;

// foo 함수와 bar 함수는 모두 전역에서 함수 선언문으로 정의됨
// => 전역 코드가 평가되는 시점에 평가되어 함수 객체를 생성하고 전역 객체 window의 메서드가 됨
// 이때 생성된 함수 객체의 내부 슬롯 [[Environment]]에는 함수 정의가 평가된 시점,
// 즉 전역 코드가 평가 시점에 실행 중인 실행 컨텍스트의 렉시컬 환경인 전역 렉시컬 환경의 참조가 저장

function foo() {
  const x = 10;
  
  // 상위 스코프는 함수 정의 환경에 따라 결정
  // 함수 호출 위치와 상위 스코프는 아무런 관계가 없음
  bar();
}

// 함수 bar는 자신의 상위 스코프, 즉 전역 객체의 렉시컬 환경을 [[Environment]]에 저장하여 기억
function bar() {
  console.log(x);
}

foo();	// 1
bar();	// 1
```

- 함수 코드 평가는 아래와 같은 순서로 진행

```
1. 함수 실행 컨텍스트 생성
2. 함수 렉시컬 환경 생성
	2.1. 함수 환경 레코드 생성
	2.2. this 바인딩
	2.3. 외부 렉시컬 환경에 대한 참조 결정
```

- 외부 렉시컬 환경에 대한 참조에는 함수 객체의 내부 슬롯 `[[Environment]]`에 저장된 렉시컬 환경에 대한 참조가 할당
  - 함수 객체의 내부 슬롯 `[[Environment]]`에 저장된 렉시컬 환경의 참조는 바로 함수의 상위 스코프를 의미

<br>

## 03. 클로저와 렉시컬 환경

```js
const x = 1;

// ①
// outer 함수가 평가되어 함수 객체를 생성할 때
// 현재 실행 중인 실행 컨텍스트의 렉시컬 환경, 즉 전역 렉시컬 환경을
// outer 함수 객체의 [[Environment]] 내부 슬롯에 상위 스코프로서 저장
function outer() {      
  const x = 10;
  
  // ②
  // inner 함수는 함수 표현식으로 정의했기 때문에 런타임에 평가됨
  // 이때 중첩 함수 inner는 자신의 [[Environment]] 내부 슬롯에
  // 현재 실행 중인 실행 컨텍스트의 렉시컬 환경, 즉 outer 함수의 렉시컬 환경을 상위 스코프로서 저장
  const inner = function() {
    console.log(x);
  };
  
  return inner;
}
// ③
// outer 함수의 실행이 종료되면 inner 함수를 반환하면서 outer 함수의 생명 주기가 종료
// 즉, outer 함수의 실행 컨텍스트는 실행 컨텍스트 스택에서 팝되어 제거
// 이때 outer 함수의 렉시컬 환경까지 소멸하는 것은 아님
// outer 함수의 렉시컬 환경은 inner 함수의 [[Environment]] 내부 슬롯에 참조되고 있고
// inner 함수는 전역 변수 innerFunc에 의해 참조되고 있으므로 가비지 컬렉션의 대상이 아니기 때문
const innerFunc = outer();

// ④
// outer 함수가 반환한 inner 함수를 호출하면
// inner 함수의 실행 컨텍스트가 생성되고 실행 컨텍스트 스택에 푸시
// 렉시컬 환경의 외부 렉시컬 환경에 대한 참조에는
// inner 함수 객체의 [[Environment]] 내부 슬롯에 저장되어 있는 참조값이 할당
innerFunc();	// 10
```

- 외부 함수보다 중첩 함수가 더 오래 유지되는 경우 중첩 함수는 이미 생명 주기가 종료한 외부 함수의 변수를 참조할 수 있는데, 이러한 중첩 함수를 클로저라고 부름

- 자바스크립트의 모든 함수는 자신의 상위 스코프를 기억하며, 함수를 어디서 호출하든 상관없이 유지
  - 함수는 언제나 자신이 기억하는 상위 스코프의 식별자를 참조할 수 있으며, 식별자에 바인딩 된 값을 변경할 수도 있음

- 자바스크립트의 모든 함수는 상위 스코프를 기억하므로 이론적으로 모든 함수는 클로저지만, 일반적으로 모든 함수를 클로저라고 하지 않음

```html
<!-- 브라우저 환경에서 실행 -->
<!DOCTYPE html>
<html>
  <body>
    <script>
      function foo() {
        const x = 1;
        const y = 2;
        
        // 일반적으로 클로저라고 하지 않음
        // 외부 함수 foo보다 더 오래 유지되지만 상위 스코프의 어떤 식별자도 참조하지 않기 때문
        // 이런 경우 대부분의 모던 브라우저는 최적화를 통해 상위 스코프를 기억하지 않음
        function bar() {
          const z = 3;
          
          debugger;
          // 상위 스코프의 식별자를 참조하지 않음
          console.log(z);
        }
        
        return bar;
      }
      
      const bar = foo();
      bar();
    </script>
  </body>
</html>
```

```html
<!-- 브라우저 환경에서 실행 -->
<!DOCTYPE html>
<html>
  <body>
    <script>
      function foo() {
        const x = 1;
        
        // bar 함수는 상위 스코프의 식별자를 참조하므로 클로저였지만
        // 외부 함수 foo 함수보다 중첩 함수 bar의 생명주기가 짧으므로 곧바로 소멸함
        // 이러한 함수는 일반적으로 클로저라고 부르지 않음
        function bar() {
          debugger;
          console.log(x);
        }
        
        return bar;
      }
      
      foo();
    </script>
  </body>
</html>
```

```html
<!-- 브라우저 환경에서 실행 -->
<!DOCTYPE html>
<html>
  <body>
    <script>
      function foo() {
        const x = 1;
        const y = 2;
        
        // 중첩 함수 bar는 외부 함수보다 더 오래 유지되며
        // 상위 스코프의 식별자를 참조하기 때문에 클로저임
        function bar() {
          debugger;
          // 다만 상위 스코프의 x만 참조하고 있기 때문에
          // 대부분의 모던 브라우저는 최적화를 통해
          // 상위 스코프의 식별자 중에서 클로저가 참조하고 있는 식별자만을 기억
          console.log(x);
        }
        
        return bar;
      }
      
      const bar = foo();
      bar();
    </script>
  </body>
</html>
```

- 클로저는 중첩 함수가 상위 스코프의 식별자를 참조하고 있고 중첩 함수가 외부 함수보다 더 오래 유지되는 경우에 한정하는 것이 일반적

- 클로저에 의해 참조되는 상위 스코프의 변수를 자유 변수라고 함

<br>

## 04. 클로저의 활용

- 클로저는 상태가 의도치 않게 변경되지 않도록 상태를 안전하게 은닉하고 특정 함수에게만 상태 변경을 허용하기 위해 사용

```js
// 호출된 횟수(num)가 안전하게 변경되고 유지되어야 할 상태
let num = 0;

const increase1 = function() {
  return ++num;
};

console.log(increase1());	// 1
console.log(increase1());	// 2
console.log(increase1());	// 3

// 위 코드가 바르게 동작하려면
// 1. 카운트 상태(num)는 increase 함수가 호출되기 전까지 변경되지 않고 유지되어야 함
// 2. 이를 위해 카운트 상태(num)는 increase 함수만이 변경할 수 있어야 함
// => 그러나 카운트 상태는 전역 변수를 통해 관리되고 있기 때문에 언제든지 누구나 접근할 수 있고 변경할 수 있음
// 	따라서 increase 함수만이 num 변수를 참조하고 변경할 수 있게 해야함

const increase2 = function() {
  let num = 0;
  
  return ++num;
};

console.log(increase2());	// 1
console.log(increase2());	// 1
console.log(increase2());	// 1

// 카운트 상태를 안전하게 변경하고 유지하기 위해 전역 변수 num을 increase 함수의 지역 변수로 변경하였으나
// increase 함수가 호출될 때마다 지역 변수 num은 다시 선언되고 0으로 초기화되기 때문에
// 상태가 변경되기 이전 상태를 유지하지 못함. => 이전 상태를 유지할 수 있도록 클로저를 사용

const increase3 = (function() {
  let num = 0;
  
  return function() {
    return ++num;
  };
}());

console.log(increase3());	// 1
console.log(increase3());	// 2
console.log(increase3());	// 3

// 위 코드가 실행되면 즉시 실행 함수가 호출되고 즉시 실행 함수가 반환한 함수가 increase 변수에 할당
// 이는 자신이 정의된 위치에 의해 결정된 상위 스코프인 즉시 실행 함수의 렉시컬 환경을 기억하는 클로저
// 즉시 실행 함수가 반환한 클로저는 상위 스코프인 즉시 실행 함수의 렉시컬 환경을 기억하고 있음
// 즉시 실행 함수는 한 번만 실행되므로 increase가 호출될 때마다 num 변수가 초기화될 일은 없으며,
// num 변수는 외부에서 직접 접근할 수 없는 은닉된 private 변수이므로 의도치 않은 변경을 막을 수 있음
```

```js
// 즉시 실행 함수가 반환하는 객체 리터럴은 즉시 실행 함수의 실행 단계에서 평가되어 객체가 됨
const counter = (function() {
  let num = 0;
  
  // 클로저인 메서드를 갖는 객체를 반환
  // 객체 리터럴은 스코프를 만들지 않기 때문에 아래 메서드들의 상위 스코프는 즉시 실행 함수의 렉시컬 환경임
  return {
    // num: 0,	// 프로퍼티는 public하므로 은닉되지 않음
    increase() {
      return ++num;
    },
    decrease() {
      return num > 0 ? --num : 0;
    }
  };
}());

console.log(counter.increase());	// 1
console.log(counter.increase());	// 2

console.log(counter.decrease());	// 1
console.log(counter.decrease());	// 0
```

```js
// 위 예제를 생성자 함수로 표현하면 다음과 같음
const Counter = (function() {
  let num = 0;
  function Counter() {
    // this.num = 0;	// 프로퍼티는 public 하므로 은닉되지 않음
  }
  
  // increase, decrease 메서드는 모두 자신의 함수 정의가 평가되어 함수 객체가 생성될 때
  // 실행 중인 실행 컨텍스트인 즉시 실행 함수 실행 컨텍스트의 렉시컬 환경을 기억하는 클로저
  Counter.prototype.increase = function() {
    return ++num;
  };
  
  Counter.prototype.decrease = function() {
    return num > 0 ? --num : 0;
  };
  
  return Counter;
}());

const counter = new Counter();

console.log(counter.increase());	// 1
console.log(counter.increase());	// 2

console.log(counter.decrease());	// 1
console.log(counter.decrease());	// 0
```

- 외부 상태 변경이나 가변 데이터를 피하고 불변성을 지향하는 함수형 프로그래밍이서 부수 효과를 최대한 억제하여 오류를 피하고 프로그램의 안정성을 높이기 위해 클로저는 적극적으로 사용됨

```js
// 함수를 인수로 전달받고 함수를 반환하는 고차 함수
// 이 함수는 카운트 상태를 유지하기 위해 makeCounter가 생성됐을 때의 렉시컬 환경인
// makeCounter 함수의 스코프에 속한 counter 변수를 기억하는 클로저를 반환

function makeCounter(predicate) {
  // 카운트 상태를 유지하기 위한 자유 변수
  let counter = 0;
  
  //  counter 변수를 기억하는 클로저 반환
  return function () {
    // 인수로 전달받은 보조 함수에 상태 변경을 위임
    counter = predicate(counter);
    return counter;
  };
}

// 보조 함수
function increase(n) {
  return ++n;
}

function decrease(n) {
  return --n;
}

// 함수로 함수를 생성
// makeCounter 함수는 보조 함수를 인수로 전달받아 함수를 반환

// ①
// makeCounter 함수를 호출했으므로 makeCounter 함수의 실행 컨텍스트가 생성되고, makeCounter 함수는 함수 객체를 생성하여 반환한 후 소멸
// makeCounter 함수가 반환한 함수는 makeCounter 함수의 렉시컬 환경을 상위 스코프로서 기억하는 클로저이며, 전역 변수인 increaser에 할당
// makeCounter 함수의 실행 컨텍스트는 소멸되지만 makeCounter 함수 실행 컨텍스트의 렉시컬 환경은 makeCounter 함수가 반환한 함수의 [[Environment]] 내부 슬롯에 의해 참조되고 있기 때문에 소멸하지 않음
const increaser = makeCounter(increase);
console.log(increaser());	// 1
console.log(increaser());	// 2

// increaser 함수와는 별개의 독립된 렉시컬 환경을 갖기 때문에 카운터 상태가 연동되지 않음
// ②
// makeCounter 함수를 호출하면 새로운 makeCounter 함수의 실행 컨텍스트가 생성됨
// 동작은 위의 경우와 같음
const decreaser = makeCounter(decrease);
console.log(decreaser());	// -1
console.log(decreaser());	// -2

// 전역 변수 increaser와 decreaser에 할당된 함수는 각각 자신만의 독립된 렉시컬 환경을 갖기 때문에
// 카운트를 유지하기 위한 자유 변수 counter를 공유하지 않아 카운터의 증감이 연동되지 않음
// 따라서 렉시컬 환경을 공유하는 클로저를 만들기 위해 makeCounter를 즉시 실행 함수로 만들어야 한다.

const counter = (function () {
  // 카운트 상태를 유지하기 위한 자유 변수
  let counter = 0;
  
  //  counter 변수를 기억하는 클로저 반환
  return function (predicate) {
    // 인수로 전달받은 보조 함수에 상태 변경을 위임
    counter = predicate(counter);
    return counter;
  };
}());

console.log(counter(increase));	// 1
console.log(counter(increase));	// 2

console.log(counter(decrease));	// 1
console.log(counter(decrease));	// 0
```

<br>

## 05. 캡슐화와 정보 은닉

- 캡슐화: 객체의 상태를 나타내는 프로퍼티와 프로퍼티를 참조하고 조작할 수 있는 동작인 메서드를 하나로 묶는 것
- 캡슐화는 객체의 특정 프로퍼티나 메서드를 감출 목적으로 사용하기도 하는데 이를 정보 은닉이라 함
  - 정보 은닉은 외부에 공개할 필요가 없는 구현의 일부를 외부에 공개하지 않도록 감추어 적절치 못한 접근으로 부터 객체의 상태가 변경되는 것을 방지해 정보를 보호
  - 객체 간의 상호 의존성, 즉 결합도를 낮춰줌

```js
function Person(name, age) {
  this.name = name;	// public
  let _age = age;	// private
  
  // (1) 인스턴스 메서드이므로 Person 객체가 생성될 때마다 중복 생성
  // => 프로토타입 메서드로 변경하여 중복 생성 방지
  /*
  this.sayHi = function () {
    console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
  };
  */
}

// (2) Person 생성자 함수의 지역 변수 _age를 참조할 수 없음
// => 즉시 실행 함수를 사용해서 Person 생성자 함수와 Person.prototype.sayHi 메서드를 하나로 모음
Person.prototype.sayHi = function () {
  console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
};

// (3)
// 즉시 실행 함수가 반환하는 Person 생성자 함수와 Person 생성자 함수의 인스턴스가 상속받아 호출할 Person.prototype.sayHi 메서드는 즉시 실행 함수가 종료된 이후 호출되지만,
// Person 생성자 함수와 sayHi 메서드는 즉시 실행 함수가 종료되어도 지역 변수 _age를 참조할 수 있는 클로저임
const person (function () {
  let _age = 0;	// private
  
  // 생성자 함수
  function Person(name, age) {
    this.name = name;	// public
    let _age = age;	// private
  }
  
  Person.prototype.sayHi = function () {
    console.log(`Hi! My name is ${this.name}. I am ${_age}.`);
  };
  
  return Person;
}());

const me = new Person('Lee', 20);
me.sayHi();	// Hi! My name is Lee. I am 20.
console.log(me.name);	// Lee
console.log(me._age);	// undefiend

// 그러나 (3) 코드도 Person 생성자 함수가 여러 개의 인스턴스를 생성할 경우,
// Person.prototype.sayHi 메서드가 단 한 번 생성되는 클로저이기 때문에 동일한 하나의 상위 스코프를 사용하므로
// _age 변수의 상태가 유지되지 않는다는 문제가 있음
const you = new Person('Kim', 30);
you.sayHi();	// Hi! My name is Kim. I am 30.

// _age 값이 변경되었음
me.sayHi();	// Hi! My name is Lee. I am 30.
```

- 자바스크립트는 정보 은닉을 완전하게 지원하지 않음
  - 인스턴스 메서드를 사용한다면 자유 변수를 통해 private을 흉내 낼 수는 있지만 프로토타입 메서드를 사용하면 이마저도 불가능

<br>

## 06. 자주 발생하는 실수

```js
var funcs = [];

// var 키워드로 선언한 i 변수는 블록 레벨 스코프가 아닌 함수 레벨 스코프를 갖기 때문에 전역 변수임
for (var i = 0; i <  3; i++) {
  funcs[i] = function () { return i; };
}

// 따라서 해당 함수를 호출하면 전역 변수 i를 참조하여 3의 값이 출력됨
for (var j = 0; j < funcs.length; j++) {
  console.log(funcs[j]());	// 3 3 3
}
```

```js
// 클로저를 사용해 올바르게 동작하는 코드 만들기
var funcs = [];

for (var i = 0; i < 3; i++) {
  // 즉시 실행 함수는 현재 전역 변수 i에 할당되어 있는 값을 인수로 전달받아 매개변수 id에 할당한 후
  // 중첩 함수를 반환하고 종료되고, 즉시 실행 함수가 반환한 함수는 funcs 배열에 순차적으로 저장
  funs[i] = (function (id) {
    // 즉시 실행 함수의 매개변수 id는 즉시 실행 함수가 반환한 중첩 함수의 상위 스코프에 존재하며,
    // 즉시 실행 함수가 반환한 중첩 함수는 자신의 상위 스코프(즉시 실행 함수의 렉시컬 환경)를 기억하는 클로저이며
    // 매개변수 id는 즉시 실행 함수가 반환한 중첩 함수에 묶여있는 자유 변수가 되어 그 값이 유지됨
    return function () {
      return id;
    };
  }(i));
}

for (var j = 0; j < funcs.length; j++) {
  console.log(funcs[j]());	// 3 3 3
}
```

```js
// ES6의 let 키워드를 사용하면 깔끔하게 해결!~

const funcs = [];

// let 키워드로 선언한 변수는 for 문의 코드 블록이 반복 실행될 때마다 새로운 렉시컬 환경이 생성
// for 문의 내에 정의한 함수의 상위 스코프는 코드 블록이 반복 실행될 때마다 생성된 새로운 렉시컬 환경임
// 단, 이는 반복문 내에 함수를 정의할 때만 의미가 있으며,
// 함수 정의가 없는 반복문이 생성하는 새로운 렉시컬 환경은 반복 직후 아무도 참조하지 않으므로 가비지 컬렉션의 대상이 됨
for (let i = 0; i <  3; i++) {
  funcs[i] = function () { return i; };
}

for (let i = 0; i < funcs.length; i++) {
  console.log(funcs[i]());	// 0 1 2
}
```

```js
// 함수형 프로그래밍 기법인 고차 함수를 사용하는 방법

// 요소가 3개인 배열을 생성하고 배열의 인덱스를 반환하는 함수를 요소로 추가
// 배열의 요소로 추가된 함수들은 모두 클로저임
const funcs = Array.from(new Array(3), (_, i) => () => i);	// (3) [f, f, f]

// 배열의 요소로 추가된 함수들을 순차적으로 호출
funcs.forEach(f => console.log(f()));	// 0 1 2
```