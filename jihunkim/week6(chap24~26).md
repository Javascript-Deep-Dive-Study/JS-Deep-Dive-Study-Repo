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





# 25장: 클래스

<br>

## 01. 클래스는 프로토타입의 문법적 설탕인가?

- 자바스크립트는 프로토타입 기반 객체지향 언어
  - 생성자 함수와 프로토타입을 통해 객체지향 언어의 상속을 구현함

```js
var Person = (function() {
  function Person(name) {
    this.name = name;
  }
  
  Person.prototype.sayHi = function() {
    console.log(`Hi! My name is + ${this.name}`)
  };
  
  return Person;
}());

var me = new Person('Lee');
me.sayHi();	// My name is Lee
```

- 클래스와 생성자 함수는 모두 프로토타입 기반의 인스턴스를 생성하며, 클래스는 생성자 함수와 유사하게 동작하지만 차이점이 있음
  - 클래스를 `new` 연산자 없이 호출하면 에러가 발생하지만 생성자 함수를 `new` 연산자 없이 호출하면 일반 함수로서 호출
  - 클래스만 상속을 지원하는 `extends`와 `super` 키워드를 제공
  - 클래스는 호이스팅이 발생하지 않는 것처럼 동작하지만 함수 선언문으로 정의된 생성자 함수는 함수 호이스팅이, 함수 표현식으로 정의한 생성자 함수는 변수 호이스팅이 발생
  - 클래스 내의 모든 코드에는 암묵적으로 strict mode가 지정되어 실행되며, strict mode를 해제할 수 없음
  - 클래스의 constructor, 프로토타입 메서드, 정적 메서드는 모두 프로퍼티 어트리뷰트 `[[Enumerable]]`의 값이 `false`이므로 열거되지 않음

- 클래스는 새로운 객체 생성 매커니즘

<br>

## 02. 클래스 정의

- `class` 키워드를 사용하여 정의하며, 생성자 함수와 마찬가지로 파스칼 케이스를 사용하는 것이 일반적

```js
class Person {}
```

- 일반적이지는 않지만 표현식으로 클래스 정의도 가능
  - 함수와 마찬가지로 이름을 가질 수도 있고, 갖지 않을 수도 있음

```js
// 익명 클래스 표현식
const Person = class {};

// 기명 클래스 표현식
const Person = class MyClass {};
```

- 클래스를 표현식으로 정의할 수 있다는 것은 클래스가 값으로 사용할 수 있는 일급 객체라는 것을 의미
  - 무명의 리터럴로 생성 가능, 즉 런타임에 생성이 가능
  - 변수나 자료구조(객체, 배열 등)에 저장할 수 있음
  - 함수의 매개변수에 전달 가능
  - 함수의 반환값으로 사용 가능
- 클래스의 몸체에는 0개 이상의 메서드만 정의할 수 있음
- 클래스 몸체에서 정의할 수 있는 메서드는 constructor(생성자), 프로토타입 메서드, 정적 메서드 세 가지

```js
// 클래스 선언문
class Person {
  // 생성자
  constructor(name) {
    // 인스턴스 생성 및 초기화
    this.name = name;
  }
  
  // 프로토타입 메서드
  sayHi() {
    console.log(`Hi! My name is ${this.name}!`);
  }
  
  // 정적 메서드
  static sayHello() {
    console.log('Hello!');
  }
}

// 인스턴스 생성
const me = new Person('Lee');

// 인스턴스의 프로퍼티 참조
console.log(me.name);	// Lee
me.sayHi();	// Hi! My name is Lee
Person.sayHello();	// Hello!
```

<br>

## 03. 클래스 호이스팅

- 클래스는 함수로 평가됨

```js
class Person {}
console.log(typeof Person);	// function
```

- 클래스 선언문으로 정의한 클래스는 함수 선언문과 같이 소스 평가 과정, 즉 런타임 이전에 먼저 평가되어 함수 객체를 생성
  - 클래스가 평가되어 생성된 함수 객체는 생성자 함수로서 호출할 수 있는 함수, 즉 constructor
  - 생성자 함수로서 호출할 수 있는 함수는 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입도 더불어 생성
    - 프로토타입과 생성자 함수는 단독으로 존재할 수 없고 언제나 쌍으로 존재

- 단, 클래스는 클래스 정의 이전에 참조할 수 없음
  - `let`, `const` 키워드로 선언한 변수처럼 호이스팅되므로 클래스 선언문 이전에 일시적 사각지대에 빠지기 때문에 호이스팅이 발생하지 않는 것처럼 동작

```js
const Person = '';
{
  // 호이스팅이 발생하지 않는다면 ''이 출력되어야 함
  console.log(Person);
  // ReferenceError: Cannot access 'Person' before initialization
  
  class Person {}
}
```

- `var`, `let`, `const`, `function`, `function*`, `class` 키워드를 사용하여 선언된 모든 식별자는 호의스팅됨
  - 모든 선언문은 런타임 이전에 먼저 실행되기 때문

<br>

## 04. 인스턴스 생성

- 클래스는 생성자 함수이며 `new` 연산자와 함께 호출되어 인스턴스 생성
  - 함수는 `new` 연산자의 사용 여부에 따라 일반 함수로 호출되거나 인스턴스 생성을 위한 생성자 함수로 호출되지만 클래스는 인스턴스를 생성하는 것이 유일한 존재 이유이므로 반드시 `new` 연산자와 함께 호출해야 함

```js
class Person {}

const me = new Person();
console.log(me);	// Person {}

const you = Person();
// TypeError: Class constructor Person cannot be invoked without 'new'
```

- 클래스 표현식으로 정의된 클래스의 경우 클래스를 가리키는 식별자를 사용해 인스턴스를 생성하지 않고 기명 클래스 표현식의 클래스 이름을 사용해 인스턴스를 생성하면 에러가 발생
  - 기명 함수 표현식과 마찬가지로 클래스 표현식에서 사용한 클래스 이름은 외부 코드에서 접근 불가능

```js
const Person = class MyClass {};
const me = new Person();

// 클래스 이름 MyClass는 클래스 몸체 내부에서만 유효한 식별자
console.log(MyClass);	// ReferenceError: MyClass is not defined
const you = new MyClass();	// ReferenceError: MyClass is not defined
```

<br>

## 05. 메서드

- 클래스 몸체에서 정의할 수 있는 메서드는 constructor(생성자), 프로토타입 메서드, 정적 메서드가 있음

### `constructor`

- 인스턴스를 생성하고 초기화하기 위한 특수한 메서드
- 이름을 변경할 수 없음

```js
class Person {
  // 생성자
  constructor(name) {
    // 인스턴스 생성 및 초기화
    this.name = name;
  }
}
```

- 클래스는 평가되어 함수 객체가 되기 때문에 함수 객체 고유의 프로퍼티를 모두 가지고 있음
- 함수와 동일하게 프로토타입과 연결되어 있으며 자신의 스코프 체인을 구성
- 모든 함수 객체가 가지고 있는 `prototype` 프로퍼티가 가리키는 프로토타입 객체의 `constructor` 프로퍼티는 클래스 자신을 가리키고 있음
  - 클래스가 인스턴스를 생성하는 생성자 함수라는 것을 의미 => `new` 연산자와 함께 클래스를 호출하면 클래스는 인스턴스를 생성

- 생성자 함수와 마찬가지로 `constructor` 내부에서 `this`로 추가한 프로퍼티는 인스턴스 프로퍼티가 되며, `this`는 클래스가 생성한 인스턴스를 가리킴

- 클래스가 평가되어 생성된 함수 객체나 클래스가 생성한 인스턴스에 `constructor` 메서드가 없음 => 클래스 몸체에 정의한 `constructor`는 단순한 메서드가 아님
  - `constructor`는 메서드로 해석되는 것이 아니라 클래스가 평가되어 생성한 함수 객체 코드의 일부가 됨
  - 클래스 정의가 평가되면 `constructor`의 기술된 동작을 하는 함수 객체가 생성

- `constructor`는 클래스 내에 최대 한 개만 존재 가능
  - 2개 이상의 `constructor`를 포함하면 문법 에러 발생

```js
class Person {
  constructor() {}
  constructor() {}
}

// SyntaxError: A class may only have one constructor
```

- `constructor`를 생략하면 클래스에 빈 `constructor`가 암묵적으로 정의되고, 빈 `constructor`에 의해 빈 객체를 생성

```js
class Person {
  // constructor가 생략되었음
  // constructor() {}
}

const me = new Person();
console.log(me);	// Person {}
```

- 프로퍼티가 추가되어 초기화된 인스턴스를 생성하려면 `constructor` 내부에서 `this`에 인스턴스 프로퍼티를 추가

```js
class Person {
  constructor(name, address) {
    // 인수로 인스턴스 초기화
    this.name = 'Lee';
    this.address = 'Seoul';
  }
}

const me = new Person('Lee', 'Seoul');
console.log(me);	// Person {name: "Lee", address: "Seoul"}
```

- `constructro`는 별도의 반환문을 갖지 않음
  - `new` 연산자와 함께 클래스가 호출되면 생성자 함수와 동일하게 암묵적으로 `this`, 즉 인스턴스를 반환하기 때문
  - `this`가 아닌 다른 객체를 명시적으로 반환하면 `this`, 즉 인스턴스가 반환되지 못하고 `return` 문에 명시한 객체가 반환됨
  - 명시적으로 원시값을 반환하면 원시값 반환은 무시되고 암묵적으로 `this`가 반환

```js
class Person1 {
  constructor(name) {
    this.name = name;
    return {};
  }
}

class Person2 {
  constructor(name) {
    this.name = name;
    return 0;
  }
}

// 명시적으로 반환한 객체가 반환
const me = new Person1('Lee');
console.log(me);	// Person {}

// 그러나 원시값을 반환하면 무시됨
const you = new Person2('Lee');
console.log(you);	// Person {name: "Lee"}
```

### 프로토타입 메서드

- 생성자 함수를 사용하여 인스턴스를 생성하는 경우 프로토타입 메서드를 생성하기 위해서는 명시적으로 프로토타입에 메서드를 추가함
- 클래스 몸체에서 정의한 메서드는 생성자 함수에 의한 객체 생성 방식과는 다르게 클래스의 prototype 프로퍼티에 메서드를 추가하지 않아도 기본적으로 프로토타입 메서드가 됨

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  // 프로토타입 메서드
  sayHi() {
    console.log(`Hi! My name is ${this.name}.`);
  }
}

const me = new Person('leonardo dicaprio');
me.sayHi();	// Hi! My name is leonardo dicaprio.
```

- 생성자 함수와 마찬가지로 클래스가 생성한 인스턴스는 프로토타입 체인의 일원이 됨

```js
Object.getPrototypeOf(me) === Person.prototype;	// → true
me instanceof Person;	// → true

Object.getPrototypeOf(Person.prototype) === Object.prototype;	// → true
me instanceof Object;	// → true

me.constructor === Person;	// → true
```

### 정적 메서드

- 인스턴스를 생성하지 않아도 호출할 수 있는 메서드
- 생성자 함수의 경우 정적 메서드를 생성하기 위해서는 명시적으로 생성자 함수에 메서드를 추가
- 클래스에서는 메서드에 `static` 키워드를 붙이면 정적 메서드(클래스 메서드)가 됨

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  
  // 정적 메서드
  static sayHi() {
    console.log('Hi!');
  }
}
```

- 정적 메서드는 클래스에 바인딩된 메서드가 됨
- 클래스는 함수 객체로 평가되므로 자신의 프로퍼티/메서드를 소유할 수 있으며, 클래스는 클래스 정의가 평가되는 시점에 함수 객체가 되므로 정적 메서드는 클래스 정의 이후 인스턴스를 생성하지 않아도 호출 가능
- 단, 정적 메서드는 인스턴스로 호출할 수 없음
  - 정적 메서드가 바인딩된 클래스는 인스턴스의 프로토타입 체인 상에 존재하지 않기 때문

```js
Person.sayHi();	// Hi!
me.sayHi();	// TypeError: me.sayHi is not a function
```

### 정적 메서드와 프로토타입 메서드의 차이

- 메서드가 속해 있는 프로토타입 체인이 다름
- 정적 메서드는 클래스로 호출하고 프로토타입 메서드는 인스턴스로 호출
- 정적 메서드는 인스턴스 프로퍼티를 참조할 수 없지만 프로토타입 메서드는 인스턴스 프로퍼티를 참조할 수 있음

```js
class Square {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  
  // 프로토타입 메서드
  // 인스턴스 프로퍼티를 참조해야 한다면 프로토타입 메서드 사용
  // 메서드 내부의 this는 메서드를 호출한 객체에 바인딩
  area() {
    return this.width * this.height;
  }
  
  // 정적 메서드
  // this를 사용하지 않으며, 인스턴스 프로퍼티를 참조하지 않음
  // 애플리케이션 전역에서 사용할 유틸리티 함수를 전역 함수로 정의하지 않고 메서드로 구조화할 때 유리
  static area(width, height) {
    console.log(this);
    return width * height;
  }
}

const square = new Square(10, 10);
console.log(square.area());	// 100

console.log(Square.area(10, 10));	// 100
```

### 클래스에서 정의한 메서드의 특징

- `function` 키워드를 생략한 메서드 축약 표현을 사용
- 객체 리터럴과는 다르게 클래스에 메서드를 정의할 때는 콤마가 필요 없음
- 암묵적으로 strict mode로 실행
- `for ... in` 문이나 `Object.keys` 메서드 등으로 열거할 수 없음. 즉, 프로퍼티의 열거 가능 여부를 나타내는 불리언 값인 `[[Enumerable]]`의 값이 `false`
- 내부 메서드 `[[Constructor]]`를 갖지 않는 non-constructor임. 따라서 `new` 연산자와 함께 호출 불가

<br>

## 06. 클래스의 인스턴스 생성 과정

1. 인스턴스 생성과 `this` 바인딩
   - `new` 연산자와 함께 클래스를 호출하면 `constructor`의 내부 코드가 실행되기 전 암묵적으로 빈 객체 생성 => 클래스가 생성한 인스턴스
   - 클래스가 생성한 인스턴스의 프로토타입으로 클래스의 `prototype` 프로퍼티가 가리키는 객체가 설정
   - 암묵적으로 생성한 빈 객체인 인스턴스가 `this`에 바인딩 => `constructor` 내부 `this`는 클래스가 생성한 인스턴스를 가리킴
2. 인스턴스 초기화
   - `constructor`의 내부 코드가 실행되어 `this`에 바인딩되어 있는 인스턴스를 초기화
   - `this`에 바인딩되어 있는 인스턴스에 프로퍼티를 추가하고 `constructor`가 인수로 전달받은 초기값으로 인스턴스의 프로퍼티 값을 초기화
   - `constructor`가 생략되었다면 이 과정도 생략
3. 인스턴스 반환
   - 클래스의 모든 처리가 끝나면 완성된 인스턴스가 바인딩된 `this`가 암묵적으로 반환

<br>

## 07. 프로퍼티

### 인스턴스 프로퍼티

```js
class Person {
  constructor(name) {
    // 인스턴스 프로퍼티
    this.name = name;
  }
}

const me = new Person('Lee');
console.log(me);	// Person {name: "Lee"}
console.log(me.name);	// Lee
```

- `constructor` 내부 코드가 실행되기 이전에 `constructor` 내부의 `this`에는 이미 클래스가 암묵적으로 생성한 인스턴스인 빈 객체가 바인딩되어 있음
- `constructor` 내부에서 정의하면 클래스가 암묵적으로 생성한 빈 객체, 인스턴스에 프로퍼티가 추가됨
- 인스턴스 프로퍼티는 언제나 public

### 접근자 프로퍼티

- 자체적으로는 값(`[[Value]]` 내부 슬롯)을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 사용하는 접근자 함수로 구성된 프로퍼티
  - getter 함수와 setter 함수로 구성되어 있음

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  
  // getter 함수
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
  
  // setter 함수
  set fullName(name) {
    [this.firstName, this.lastName] = name.split(' ');
  }
}

const me = new Person('leonardo', 'dicaprio');
console.log(`${me.firstName} ${me.lastName}`);	// leonardo dicaprio

// 접근자 프로퍼티 fullName에 값을 저장하면 setter가 호출
me.fullName = 'leonardo dicaprio';
console.log(me);	// {firstName: "leonardo", lastName: "dicaprio"}
// 접근자 프로퍼티 fullName에 접근하면 getter가 호출
console.log(me.fullName);	// leonardo dicaprio

console.log(Object.getOwnPropertyDescriptor(Person.prototype, 'fullName'));
// {get: f, set: f, enumerable: false, configurable: true}
```

- getter와 setter는 호출하는 것이 아니라 프로퍼티처럼 참조하는 형식으로 사용

### 클래스 필드 정의 제안

- 클래스 필드란? 클래스 기반 객체지향 언어에서 클래스가 생성할 인스턴스의 프로퍼티를 가리키는 용어
- 자바스크립트의 클래스에서 인스턴스의 프로퍼티를 선언하고 초기화하려면 반드시 `constructor` 내부에서 `this`에 프로퍼티를 추가해야함
- 자바스크립트의 클래스에서 인스턴스 프로퍼티를 참조하려면 반드시 `this`를 사용하여 참조
- 자바스크립트의 클래스 몸체에는 메서드만 선언 가능
  - 클래스 몸체에 클래스 필드를 선언하면 문법 에러가 발생
  - 최신 브라우저는 표준 사양으로 클래스 필드를 클래스 몸체에 정의하 수 있도록 구현해놓음

### private 필드 정의 제안

- 자바스크립트는 다른 클래스 기반 객체지향 언어에서 지원하는 private, public, protected 키워드와 같은 접근 지원자를 지원하지 않음
  - 인스턴스 프로퍼티는 인스턴스를 통해 클래스 외부에서 언제나 참조 가능
- 그러나 TC39프로세스의 stage 3에는 private 필드를 정의할 수 있는 새로운 표준 사양이 제안됨
  - private 필드의 선두에 `#`을 붙이는 방식

### static 필드 정의 제안

- `static` 키워드를 사용하여 정적 메서드는 정의할 수 있지만, 정적 필드를 정의할 수는 없음
- 그러나 TC39 프로세스의 stage 3에 제안되어 있으며, 최신 브라우저에 구현되어 있음

<br>

## 08. 상속에 의한 클래스 확장

### 클래스 상속과 생성자 함수 상속

- 프로토타입 기반의 상속은 프로토타입 체인을 통해 다른 객체의 자산을 상속받는 개념이지만 상속에 의한 클래스 확장은 기존 클래스를 상속받아 새로운 클래스를 확장하여 정의하는 것
- 클래스는 상속을 통해 기존 클래스를 확장할 수 있는 문법이 기본적으로 제공되지만 생성자 함수는 그렇지 않음
  - 상속을 통해 다른 클래스를 확장할 수 있는 문법인 `extends` 키워드가 기본적으로 제공

```js
class Animal {
  constructor(age, weight) {
    this.age = age;
    this.weight = weight;
  }
  eat() { return 'eat'; }
  move() { return 'move'; }
}

// 상속을 통해 Animal 클래스를 확장한 Bird 클래스
class Bird extends Animal {
  fly() { return 'fly'; }
}

const bird = new Bird(1, 5);

console.log(bird);	// Bird {age: 1, weight: 5};
console.log(bird instanceof Bird);	// true
console.log(bird instanceof Animal);	// true

console.log(bird.eat());	// eat
console.log(bird.move());	// move
console.log(bird.fly());	// fly
```

### `extends` 키워드

- 상속을 통해 확장된 클래스를 서브클래스라 부르고, 서브클래스에게 상속된 클래스를 수퍼클래스라고 함
  - 서브클래스를 파생 클래스 또는 자식 클래스, 수퍼클래스를 베이스 클래스 또는 부모 클래스라고 부르기도 함
- `extends` 키워드의 역할은 수퍼클래스와 서브클래스 간의 상속 관계를 설정하는 것
- 수퍼클래스와 서브클래스는 인스턴스의 프로토타입 체인뿐 아니라 클래스 간의 프로토타입 체인도 생성
  - 프로토타입 메서드, 정적 메서드 모두 상속이 가능

### 동적 상속

- `extends` 키워드는 클래스뿐만 아니라 생성자 함수를 상속받아 클래스를 확장할 수도 있음
- `extends` 키워드 앞에는 반드시 클래스가 와야 하며, `extends` 키워드 다음에는 클래스 뿐만이 아니라 `[[Construct]]` 내부 메서드를 갖는 함수 객체로 평가될 수 있는 모든 표현식을 사용할 수 있음

```js
function Base1() {}
class Base2 {}

let condition = true;

// 조건에 따라 동적으로 상속 대상을 결정하는 서브클래스
class Derived extends (condition ? Base1 : Base2) {}

const derived = new Drived();
console.log(derived);	// Derived {}
console.log(derived instanceof Base1);	// true
console.log(derived instanceof Base2);	// false
```

### 서브클래스의 constructor

- `constructor`를 생략하면 클래스에서 비어있는 `constructor`가 암묵적으로 정의됨
- 서브클래스에서 `constructor`를 생략하면 클래스에 아래와 같은 `constructor`가 암묵적으로 정의됨
  - `args`는 `new` 연산자와 함께 클래스를 호출할 때 전달할 인수의 리스트
  - `super()`는 수퍼클래스의 `constructor`를 호출하며 인스턴스를 생성

```js
constructor(...args) { super(...args); }
```

### `super` 키워드

- `super` 키워드는 함수처럼 호출할 수도 있고 `this`와 같이 식별자처럼 참조할 수 있는 특별한 키워드
  - `super`를 참조하면 수퍼클래스의 메서드를 호출 가능

#### `super` 호출

- `super`를 호출하면 수퍼클래스의 `constructor`를 호출

```js
class Base {
  constructor(a, b) {
    this.a = a;
    this.b = b'
  }
}

class Derived extends Base {
  // 다음과 같이 암묵적으로 constructor가 정의됨
  // constructor(...args) { super(...args); }
}

const derived = new Derived(1, 2);
console.log(derived);	// Derived {a: 1, b: 2}
```

- 수퍼클래스에서 추가한 프로퍼티와 서브클래스에서 추가한 프로퍼티를 갖는 인스턴스를 생성한다면 서브클래스의 `constructor`를 생략할 수 없음
  - `new` 연산자와 함께 서브클래스를 호출하면서 전달한 인수 중에서 수퍼클래스의 `constructor`에 전달할 필요가 있는 인수는 서브클래스의 `constructor`에서 호출하는 `super`를 통해 전달

```js
class Base {
  constructor(a, b) {
    this.a = a;
    this.b = b'
  }
}

class Derived extends Base {
  constructor(a, b, c) {
    super(a, b);	// super 호출을 통해 Base 클래스의 constructor에게 일부가 전달
    this.c = c;
  }
}

// new 연산자와 함께 Derived 클래스를 호출하면서 전달한 인수는 Derived 클래스의 constructor에 전달됨
const derived = new Derived(1, 2, 3);
console.log(derived);	// Derived {a: 1, b: 2, c: 3}
```

- `super`를 호출할 때의 주의할 사항
  - 서브클래스에서 `constructor`를 생략하지 않는 경우 서브클래스의 `constructor`에서는 반드시 `super`를 호출해야 함
  - 서브클래스의 `constructor`에서 `super`를 호출하기 전에는 `this`를 참조할 수 없음
  - `super`는 반드시 서브클래스의 `constructor`에서만 호출. 서브클래스가 아닌 클래스의 `constructor`나 함수에서 `super`를 호출하면 에러가 발생

#### `super` 참조

- 메서드 내에서 `super`를 참조하면 수퍼클래스의 메서드를 호출할 수 있음
- 서브클래스의 프로토타입 메서드 내에서 `super.(메서드명)`는 수퍼클래스의 프로토타입 메서드를 가리킴

- 서브클래스의 정적 메서드 내에서 `super.(메서드명)`은 수퍼클래스의 정적 메서드를 가리킴

### 상속 클래스의 인스턴스 생성 과정

```js
// 수퍼클래스
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  
  getArea() {
    return this.width * this.height;
  }
  
  toString() {
    return `width = ${this.width}, height = ${this.height}`;
  }
}

// 서브클래스
class ColorRectangle extends Rectangle {
  constructor(width, height, color) {
    super(width, height);
    this.color = color;
  }
  
  // 메서드 오버라이딩
  toString() {
    return super.toString() + `, color = ${this.color}`;
  }
}

const colorRectangle = new ColorRectangle(2, 4, 'red');
console.log(colorRectangle); // ColorRectangle {width: 2, height: 4, color: 'red'}

// 상속을 통해 getArea 메서드를 호출
console.log(colorRectangle.getArea());	// 8
// 오버라이딩 된 toString 메서드를 호출
console.log(colorRectangle.toString());	// widht = 2, height = 4, color = red
```

#### 1. 서브클래스의 `super` 호출

- 자바스크립트 엔진은 클래스를 평가할 때 수퍼클래스와 서브클래스르 구분하기 위해 `"base"` 또는 `"derived"`를 값으로 갖는 내부 슬롯 `[[ConstructorKind]]`를 가짐
- 다른 클래스를 상속받지 않는 클래스와 생성자 함수는 내부 슬롯 `[[ConstructorKind]]` 값이 `"base"`로 생성되지만 다른 클래스를 상속받는 서브클래스는 내부 슬롯 `[[ConstructorKind]]` 값이 `"derived"`로 설정
  - 이를 통해 수퍼클래스와 서브클래스는 `new` 연산자와 함께 호출되었을 때의 동작이 구분됨
- 다른 클래스를 상속받지 않는 클래스는 `new` 연산자와 함께 호출되었을 때 암묵적으로 빈 객체, 즉 인스턴스를 생성하고 `this`에 바인딩하지만 서브클래스는 자신이 직접 인스턴스를 생성하지 않고 수퍼클래스에게 인스턴스 생성을 위임
  - 서브클래스의 `constructor`에서 `super`를 호출해야 하는 이유

#### 2. 수퍼클래스의 인스턴스 생성과 `this` 바인딩

- 수퍼클래스 `constructor` 내부의 코드가 실행되기 이전에 암묵적으로 빈 객체, 즉 인스턴스를 생성하고 수퍼클래스의 `constructor` 내부의 `this`는 생성된 인스턴스를 가리킴

```js
class Rectangle {
  constructor(width, height) {
    // 암묵적으로 빈 객체가 생성되고 this에 바인딩
    console.log(this);	// ColorRectangle {}
    // new 연산자와 함께 호출한 함수, 즉 new.target은 ColorRectangle
    console.log(new.target);	// ColorRectangle
  }
}
```

- `new` 연산자와 함께 호출된 함수를 가리키는 `new.target`은 서브클래스를 가리킴. 즉, 인스턴스는 `new.target`이 가리키는 서브클래스가 생성한 것으로 처리

#### 3. 수퍼클래스의 인스턴스 초기화

- `this`에 바인딩 되어 있는 인스턴스에 프로퍼티를 추가하고 `constructor`가 인수로 전달받은 초기값으로 인스턴스의 프로퍼티를 초기화

#### 4.  서브클래스 `constructor`로의 복귀와 `this` 바인딩

- `super`가 반환한 인스턴스가 `this`에 바인딩되며, 서브클래스는 별도의 인스턴스를 생성하지 않고 `super`가 반환한 인스턴스를 `this`에 바인딩하여 그대로 사용
- 서브클래스에서 `super`가 호출되지 않으면 인스턴스가 생성되지 않으며, `this` 바인딩도 할 수 없음

#### 5. 서브클래스의 인스턴스 초기화

- `super` 호출 이후, 서브클래스의 `constructor`에 기술되어 있는 인스턴스 초기화가 실행됨
- `this`에 바인딩되어 있는 인스턴스에 프로퍼티를 추가하고 `constructor`가 인수로 전달받은 초기값으로 인스턴스의 프로퍼티를 초기화

#### 6. 인스턴스 반환

- 모든 처리가 끝나면 완성된 인스턴스가 바인딩된 `this`가 암묵적으로 반환

### 표준 빌트인 생성자 함수 확장

- `extends` 키워드 다음에는 클래스뿐만 아니라 `[[Construct]]` 내부 메서드를 갖는 함수 객체로 평가될 수 있는 모든 표현식을 사용할 수 있음
  - `String`, `Number`, `Array` 같은 표준 빌트인 객체도 `[[Construct]]` 내부 메서드를 갖는 생성자 함수이므로 `extends` 키워드를 사용하여 확장 가능

```js
class MyArray extends Array {
  // 중복된 배열 요소를 제거하고 반환
  uniq() {
    return this.filter((v, i, self) => self.indexOf(v) === i);
  }
  
  // 모든 배열 요소의 평균을 구함
  average() {
    return this.reduce((pre, cur)> pre + cur, 0) / this.length;
  }
}

const myArray = new MyArray(1, 1, 2, 3);
console.log(myArray);		// MyArray(4) [1, 1, 2, 3]
console.log(myArray.uniq());	// MyArray(3) [1, 2, 3]
console.log(myArray.average());	// 1.75
```

# 26장. ES6 함수의 추가 기능

<br>

## 01. 함수의 구분

- ES6 이전의 함수는 동일한 함수라도 다양한 형태로 호출할 수 있음
  - 일반 함수로서 호출할 수 있는 것은 물론 생성자 함수로서 호출 가능 => `callable`이면서 `configurable`

```js
var foo = function() {
  return 1;
};

// 일반적인 함수로서 호출
foo();	// 1

// 생성자 함수로서 호출
new foo();	// foo {}

// 메서드로서 호출
var obj = { foo: foo };
obj.foo();	// 1

var bar = function() {};

bar();	// undefined
new bar();	// bar {}
```

- 객체에 바인딩된 함수나 콜백 함수도 일반 함수로서 호출할 수 있는 것은 물론 생성자 함수로서 호출할 수 있음
  - `constructor`이면 `prototype` 프로퍼티를 가지며, 프로토타입 객체도 생성하는 것이기 때문에 성능 면에서도 문제

- 이러한 문제를 해결하기 위해 ES6에서는 함수를 사용 목적에 따라 세 가지 종류로 구분

| ES6 함수의 구분    | `constructor` | `prototype` | `super` | `arguments` |
| ------------------ | ------------- | ----------- | ------- | ----------- |
| 일반 함수(Normal)  | O             | O           | X       | O           |
| 메서드(Method)     | X             | X           | O       | O           |
| 화살표 함수(Arrow) | X             | X           | X       | X           |

<br>

## 02. 메서드

- ES6 사양에서 메서드는 메서드 축약 표현으로 정의된 함수를 뜻함

- ES6 메서드는 인스턴스를 생성할 수 없는 non-constructor
  - 생성자 함수로서 호출할 수 없음
  - 인스턴스를 생성할 수 없으므로 `prototype` 프로퍼티가 없고 프로토타입도 생성하지 않음
  - 표준 빌트인 객체가 제공하는 프로토타입 메서드와 정적 메서드는 모두 non-constructor

```js
const obj = {
  x: 1,
  // foo는 메서드
  foo() {
    return this.x;
  },
  // bar는 일반 함수
  bar: function() {
    return this.x;
  }
};

console.log(obj.foo());	// 1
console.log(obj.bar());	// 1

new obj.foo();	// TypeError: obj.foo is not a constructor
new obj.bar();	// bar {}

obj.foo.hasOwnProperty('prototype');	// false
obj.bar.hasOwnProperty('prototype');	// true
```

- ES6 메서드는 자신을 바인딩한 객체를 가리키는 내부 슬롯 `[[HomeObject]]`를 가짐
  - `super` 참조는 내부 슬롯 `[[HomeObject]]`를 사용하여 수퍼클래스의 메서드를 참조하므로 ES6 메서드는 `super` 키워드 사용 가능
  - ES6 메서드가 아닌 함수는 내부 슬롯 `[[HomeObject]]`를 가지지 않기 때문에 `super` 키워드를 사용할 수 없음

```js
const base = {
  name: 'Lee',
  sayHi() {
    return `Hi! ${this.name}`;
  }
};

const derived  = {
  __proto__: base,
  // sayHi는 ES6 메서드이므로 [[HomeObject]]를 가짐
  // sayHi의 [[HomeObject]]는 dervied.prototype을 가리키고
  // super는 sayHi의 [[HomeObject]]의 프로토타입인 base.prototype을 가리킴
  sayHi() {
    return `${super.sayHi()}. How are you doing?`;
  },
  // sayHello는 ES6 메서드가 아님
  // 따라서 [[HomeObject]]를 가자지 않으므로 super 키워드 사용 불가능
  sayHello: function() {
    return `${super.sayHi()}. How are you doing?`;
  }
};

console.log(derived.sayHi());	// Hi! Lee. How are you doing?
```

- 메서드를 정의할 때 프로퍼티 값으로 익명 함수 표현식을 할당하는 ES6 이전 방식은 사용하지 않는 것이 좋음

<br>

## 03. 화살표 함수

- `function` 키워드 대신 화살표(=>, fat arrow)를 사용하여 기존의 함수 정의 방식보다 간략하게 함수를 정의 가능
  - 표현만 간략한 것이 아니라 내부 동작도 기존의 함수보다 간략함
- 콜백 함수 내부에서 `this`가 전역 객체를 가리키는 문제를 해결하기 위한 대안으로 유용

### 화살표 함수 정의

#### 함수 정의

- 화살표 함수는 함수 선언문으로 정의할 수 없고 함수 표현식으로 정의해야 함

```js
const multiply = (x, y) => x * y;
multiply(2, 3);
```

#### 매개변수 선언

- 매개변수가 여러 개인 경우 소괄호 () 안에 매개변수를 선언
- 매개변수가 한 개인 경우 소괄호 ()를 생략 가능
- 매개변수가 없는 경우에는 소괄호 () 생략 불가능

```js
const arrow1 = (x, y) => { ... };
const arrow2 = x = { ... };
```

#### 함수 몸체 정의

- 함수 몸체가 하나의 문으로 구성된다면 함수 몸체를 감싸는 중괄호 {}를 생략할 수 있음
  - 이때 함수 몸체 내부의 문이 값으로 평가될 수 있는 표현식인 문이라면 암묵적으로 반환됨

```js
// concise body
const power = x => x ** 2;
power(2);	// 4

// 위 표현은 다음과 동일
// block body
const power = x => { return x ** 2; }
```

- 함수 몸체를 감싸는 중괄호를 생략한 경우 함수 몸체 내부의 문이 표현식이 아닌 문이라면 에러가 발생
  - 표현식이 아닌 문은 반환할 수 없기 때문

```js
const arrow = () => const x = 1;	// SyntaxError: Unexpected token 'const'
```

- 객체 리터럴을 반환하는 경우 객체 리터럴을 소괄호 ()로 감싸 주어야 함

```js
const create = (id, content) => ({ id, content });	
create(1, 'JavaScript');	// {id: 1, content: "JavaScript"}
```

- 함수 몸체가 여러 개의 문으로 구성된다면 함수 몸체를 감싸는 중괄호 {}를 생략할 수 없고, 반환값이 있다면 명시해야 함

```js
const sum = (a, b) => {
  const result = a + b;
  return result;
}
```

- 화살표 함수도 즉시 실행 함수로 사용할 수 있음

```js
const person = (name => ({
  sayHi() { return `Hi? My name is ${name}`; }
}))('Lee');
```

- 화살표 함수도 일급 객체이므로 고차 함수에 인수로 전달 가능

```js
// ES5
[1, 2, 3].map(function(v) {
  return v * 2;
});

// ES6
[1, 2, 3].map(v => v * 2);
```

### 화살표 함수와 일반 함수의 차이

- 화살표 함수는 인스턴스를 생성할 수 없는 non-constructor
  - 화살표 함수는 인스턴스를 생성할 수 없으므로 `prototype` 프로퍼티가 없고 프로토타입도 생성하지 않음

```js
const Foo = () => {};
// 화살표 함수는 생성자 함수로서 호출할 수 없음
new Foo();
Foo.hasOwnProperty('prototype');	// false
```

- 중복된 매개변수 이름을 선언할 수 없음
  - 일반 함수는 중복된 매개변수 이름을 선언해도 에러가 발생하지 않음

```js
function normal(a, a) { return a + a; }
console.log(normal(1, 2));	// 4

const arrow = (a, a) => a + a;
// SyntaxError: Duplicate parameter name not allowed in this context
```

- 화살표 함수는 함수 자체의 `this`, `arguments`, `super`, `new.target` 바인딩을 갖지 않음
  - 따라서 함수 내부에서 `this`, `arguments`, `super`, `new.target`을 참조하면 스코프 체인을 통해 상위 스코프를 참조
  - 화살표 함수와 화살표 함수가 중첩되어 있다면 상위 화살표 함수에도 `this`, `arguments`, `super`, `new.target`바인딩이 없으므로 스코프 체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수를 참조

### `this`

- 콜백 함수 내부의 `this`가 외부 함수의 `this`와 다르기 때문에 발생하는 문제를 해결하기 위해 의도적으로 설계 
- `this` 바인딩은 함수의 호출 방식, 즉 함수가 어떻게 호출되었는지에 따라 동적으로 결정
  - 함수를 정의할 때 `this`에 바인딩할 객체가 정적으로 결정되는 것이 아니고, 함수를 호출할 때 함수가 어떻게 호출되었는지에 따라 `this`에 바인딩할 객체가 동적으로 결정됨

```js
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }
  
  add(arr) {
    // add 메서드는 인수로 전달된 배열 arr를 순회하며 배열의 모든 요소에 prefix를 추가
    // ①
    return arr.map(function (item) {
      return this.prefix + item;	// ②
      // TypeError: Cannot read property 'prefix' of undefined
    });
  }
}

const prefixer = new Prefixer('-webkit-');
console.log(prefixer.add(['transition', 'user-select']));

// 위 예제를 실행했을 때 기대하는 결과는 ['-webkit-transition', '-webkit-user-select']
// ①에서 this는 메서드를 호출한 객체를 가리키지만,
// Array.prototype.map의 인수로 전달한 콜백 함수 내부인 ②에서 this는 undefined를 가리킴
// 이는 Array.prototype.map 메서드가 콜백 함수를 일반 함수로서 호출하여
// 콜백 함수의 this와 외부 함수의 this가 서로 다른 값을 가리키고 있기 때문 
```

- 일반 함수로서 호출되는 모든 함수 내부의 `this`는 전역 객체를 가리킴
- 클래스 내부의 모든 코드에는 strict mode가 암묵적으로 적용 => 클래스 내의 콜백 함수에도 strict mode가 적용
  - strict mode에서 일반 함수로서 호출된 모든 함수 내부의 `this`에는 전역 객체가 아니라 `undefined`가 바인딩

- ES5 이전에는 아래의 방법을 사용

```js
// 1)
// this를 회피시킨 후 콜백 함수 내부에서 사용
add(arr) {
  const that = this;
  return arr.map(function (item) {
    return that.prefix + item;
  });
}

// 2)
// Array.prototype.map의 두 번째 인수로 콜백 함수 내부에서 this로 사용할 객체를 전달
add(arr) {
  return arr.map(function (item) {
    return this.prefix + item;
  }, this);	// this에 바인딩된 값이 콜백 함수 내부의 this에 바인딩
}

// 3)
// Function.prototype.bind 메서드를 사용하여 this를 바인딩
add(arr) {
  return arr.map(function (item) {
    return this.prefix + item;
  }.bind(this));	// this에 바인딩된 값이 콜백 함수 내부의 this에 바인딩
}
```

- ES6에서는 화살표 함수를 사용하여 콜백 함수 내부의 `this` 문제를 해결 가능

```js
class Prefixer {
  constructor(prefix) {
    this.prefix = prefix;
  }
  
  add(arr) {
    return arr.map(item => this.preifx + item);
  }
}

const prefixer = new Prefixer('-webkit-');
console.log(prefixer.add(['transition', 'user-select']));
// ['-webkit-transition', '-webkit-user-select']
```

- 화살표 함수는 함수 자체의 `this` 바인딩을 갖지 않기 때문에 화살표 함수 내부에서 `this`를 참조하면 상위 스코프의 `this`를 그대로 참조함 => lexical this: 렉시컬 스코프와 같이 화살표 함수의 `this`가 함수가 정의된 위치에 의해 결정된다는 것을 의미

- 화살표 함수 내부에서 `this`를 참조하면 일반적인 식별자처럼 스코프 체인을 통해 상위 스코프에서 `this`를 탐색
- 화살표 함수와 화살표 함수가 중첩되어 있다면 상위 화살표 함수에도 `this` 바인딩이 없으므로 스코프 체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수의 `this`를 참조

```js
// 중첩 함수 foo의 상위 스코프는 즉시 실행 함수
// 따라서 화살표 함수 foo의 this는 상위 스코프인 즉시 실행 함수의 this를 가리킴
(function() {
  const foo = () => console.log(this);
}).call({a: 1});	// { a: 1 }

// bar 함수는 화살표 함수를 반환하고, 반환된 함수의 상위 스코프는 화살표 함수 bar
// 그러나 화살표 함수는 함수 자체의 this 바인딩을 갖지 않으므로
// bar 함수가 반환한 화살표 함수 내부에서 참조하는 this는
// 화살표 함수가 아닌 즉시 실행 함수의 this를 가리킴
(function() {
  const bar = () => () => console.log(this);
  bar()();
}).call({a: 1});	// { a: 1 }
```

- 만약 화살표 함수가 전역 함수라면 화살표 함수는 `this`의 전역 객체를 가리킴
  - 전역 함수의 상위 스코프는 전역이고 전역에서의 `this`는 전역 객체를 가리키기 때문

- 프로퍼티에 할당한 화살표 함수도 스코프 체인 상에서 가장 가까운 상위 함수 중에서 화살표 함수가 아닌 함수의 `this`를 참조

```js
// increase 프로퍼티에 할당한 화살표 함수의 상위 스코프는 전역
// 따라서 해당 함수의 this는 전역을 가리킴
const counter = {
  num: 1,
  increase: () => ++this.num
};

console.log(counter.increase());	// NaN
```

- 화살표 함수는 함수 자체의 `this` 바인딩을 갖지 않기 때문에 `Function.prototype.call`, `Function.prototype.apply`, `Function.prototype.bind` 메서드를 사용해도 화살표 함수 내부의 `this`를 교체할 수 없음
  - 메서드를 호출할 수는 있지만 화살표 함수는 함수 자체의 `this` 바인딩을 갖지 않기 때문에 `this`를 교체할 수 없고 언제나 상위 스코프의 `this` 바인딩을 참조

```js
window.x = 1;

const normal = function () { return this.x; };
const arrow = () => this.x;

console.log(normal.call({ x: 10 }));	// 10
console.log(arrow.call({ x: 10 }));	// 1
```

- 메서드를 화살표 함수로 정의하는 것은 피해야 함
  - 메서드를 정의할 때는 ES6 메서드 축약 표현으로 정의한 ES6 메서드를 사용하는 것이 좋음

```js
// Bad
const person = {
  name: 'Lee',
  sayHi: () => console.log(`Hi, ${this.nama}!`);
};

// sayHi 프로퍼티에 할당된 화살표 함수 내부의 this는 상위 스코프인 전역의 this가 가리키는 전역 객체를 가리키므로
// 이 예제를 브라우저에서 실행하면 this.name은 빈 문자열을 갖는 window.name과 같음
person.sayHi();	// Hi

// Good
const person = {
  name: 'Lee',
  sayHi() {
    console.log(`Hi, ${this.name}!`);
  }
};

person.sayHi();	// Hi, Lee!
```

- 프로토타입 객체의 프로퍼티에 화살표 함수를 할당하는 경우도 동일한 문제가 발생하므로 일반 함수를 할당
  - 일반 함수가 아닌 ES6 메서드를 동적 추가하고 싶다면 객체 리터럴을 바인딩하고 프로토타입의 `constructor` 프로퍼티와 생성자 함수 간의 연결을 재설정

```js
function Person(name) {
  this.name = name;
}

Person.prototype = {
  constructor: Person,
  sayHi() { console.log(`Hi, ${this.name}!`) };
};

const person = new Person('Leo');
person.sayHi();	// Hi, Leo
```

- 클래스 필드 정의 제안을 사용하여 클래스 필드에 화살표 함수를 할당할 수도 있지만, 해당 함수는 프로토타입 메서드가 아니라 인스턴스 메서드가 되므로 ES6 메서드 축약 표현으로 정의한 ES6 메서드를 사용하는 것이 좋음

### `super`

- 화살표 함수는 함수 자체의 `super` 바인딩을 갖지 않음
- 화살표 함수 내부에서 `super`를 참조하면 `this`와 마찬가지로 상위 스코프의 `super`를 참조

```js
class Base {
  constructor(name) {
    this.name = name;
  }
  
  sayHi() {
    return `Hi! ${this.name}`;
  }
}

// sayHi 클래스 필드에 할당한 화살표 함수는 ES6 메서드는 아니지만 함수 자체의 super 바인딩을 갖지 않으므로
// super를 참조해도 에러가 발생하지 않고 상위 스코프인 constructor의 super 바인딩을 참조함
class Derived extends Base {
  sayHi = () => `${super.sayHi()} how are you doing?`;
}

const derived = new Derived('Lee');
console.log(derived.sayHi());	// Hi! Lee how are you doing?
```

### `arguments`

- 화살표 함수는 함수 자체의 `arguments` 바인딩을 갖지 않음
- 화살표 함수 내부에서 `arguments`를 참조하면 `this`와 마찬가지로 상위 스코프의 `arguments`를 참조

```js
(function () {
  // 화살표 함수 foo의 arguments는 상위 스코프인 즉시 실행 함수의 arguments를 가리킴
  const foo = () => console.log(arguments);	// [Arguments] { '0': 1, '1': 2 }
  foo(3, 4);
}(1, 2));

// 화살표 함수 foo의 arguments는 상위 스코프인 전역의 arguments를 가리킴
// 그러나 함수 내부에서만 유효한 arguments 객체는 전역에  없음
const foo = () => console.log(arguments);
foo(1, 2);	// ReferenceError: arguments is not defined
```

- 화살표 함수로 가변 인자 함수를 구현해야 할 때는 반드시 Rest 파라미터를 사용해야 함

<br>

## 04. Rest 파라미터

### 기본 문법

- Rest 파라미터는 매개변수 이름 앞에 세개의 점 `...`을 붙여서 정의한 매개변수로써, 함수에 전달된 인수들의 목록을 배열로 전달받음

```js
function foo(...rest) {
  console.log(rest);	// [1, 2, 3, 4, 5]
}

foo(1, 2, 3, 4, 5);
```

- 일반 매개변수와 함께 사용할 수 있으며, 이때 함수에 전달된 인수들은 매개변수와 Rest 파라미터에 순차적으로 할당

```js
function foo(param, ...rest) {
  console.log(param);	// 1
  console.log(rest);	// [2, 3, 4, 5]
}
foo(1, 2, 3, 4, 5);

function bar(param1, param2, ...rest) {
  console.log(param1);	// 1
  console.log(param2);	// 2
  console.log(rest);	// [3, 4, 5]
}
bar(1, 2, 3, 4, 5);
```

- 먼저 선언된 매개변수에 할당된 인수를 제외한 나머지 인수들로 구성된 배열이 할당되므로 Rest 파라미터는 반드시 마지막 파라미터이어야 함

```js
function foo(...rest, param) { }
foo(1, 2, 3, 4, 5);
// SyntaxError: Rest parameter must be last formal parameter
```

- Rest 파라미터는 단 하나만 선언할 수 있음
- 함수 정의 시 선언한 매개변수 개수를 나타내는 함수 객체의 length 프로퍼티에 영향을 주지 않음

```js
function foo(...rest) {}
console.log(foo.length);	// 0

function bar(x, ...rest) {}
console.log(bar.length);	// 1

function baz(x, y, ...rest) {}
console.log(baz.length);	// 2
```

### Rest 파라미터와 `arguments` 객체

- `arguments` 객체는 함수 호출 시 전달된 인수들의 정보를 담고 있는 순회 가능한 유사 배열 객체이며, 함수 내부에서 지역 변수처럼 사용 가능
- 유사 배열 객체이므로 배열 메서드를 사용하려면 `Function.prototype.call`이나 `Function.prototype.apply` 메서드를 사용해 `arguments` 객체를 배열로 변환해야 함

```js
function sum() {
  var array = Array.prototype.slice.call(arguments);
  
  return array.reduce(function (pre, cur) {
    return pre + cur;
  }, 0);
}

console.log(sum(1, 2, 3, 4, 5));	// 15
```

- ES6에서는 Rest 파라미터를 사용하여 가변 인자 함수의 인수 목록을 배열로 직접 전달받을 수 있음

```js
function sum(...args) {
  // Rest 파라미터 args에는 배열 [1, 2, 3, 4, 5]가 할당
  return args.reduce((pre, cur) => pre + cur, 0);
}

console.log(sum(1, 2, 3, 4, 5));	// 15
```

- 함수와 ES6 메서드는 Rest 파라미터와 `arguments` 객체를 모두 사용할 수 있음
- 화살표 함수는 함수 자체의 `arguments` 객체를 가지지 않으므로 화살표 함수로 가변 인자 함수를 구현해야 할 때는 반드시 Rest 파라미터를 사용해야 함

<br>

## 05. 매개변수 기본값

- 인수가 전달되지 않은 매개변수의 값은 `undefined`
  - 방어 코드로 인수가 전달되지 않은 경우 매개변수에 기본값을 할당할 필요가 있음

```js
function sum1(x, y) {
  return x + y;
}

console.log(sum1(1));	// NaN

function sum2(x + y) {
  x = x || 0;
  y = y || 0;
  
  return x + y;
}

console.log(sum2(1));	// 1
```

- ES6에서 도입된 매개변수 기본값을 사용하면 함수 내에서 수행하던 인수 체크 및 초기화를 간소화할 수 있음

```js
function sum(x = 0, y = 0) {
  return x + y;
}

console.log(sum(1));	// 1
console.log(sum(1, 2));	// 3
```

- 매개변수 기본값은 매개변수에 인수를 전달하지 않은 경우와 `undefined`를 전달한 경우에만 유효

```js
function logName(name = 'Lee') {
  console.log(name);
}

logName();	// Lee
logName(undefined);	// Lee
logName(null);	// null
```

- Rest 파라미터에는 기본값을 지정할 수 없음
- 매개변수 기본값은 함수 정의 시 선언한 매개변수 개수를 나타내는 함수 객체의 `length` 프로퍼티와 `arguments` 객체에 아무런 영향을 주지 않음
