# 22장: this

<br>

## 01. `this` 키워드

- 객체는 상태를 나타내는 프로퍼티와 동작을 나타내는 메서드를 하나의 논리적인 단위로 묶은 복합적인 자료구조
- 동작을 나타내는 메서드는 자신이 속한 객체의 상태, 즉 프로퍼티를 참조하고 변경하려면 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 함
- 객체 리터럴 방식으로 생성한 객체의 경우 메서드 내부에서 메서드 자신이 속한 객체를 가리키는 식별자를 재귀적으로 참조할 수 있음

```js
const circle = {
  radius: 5,
  getDiameter() {
    // 메서드는 자신이 속한 객체인 circle을 참조할 수 있어야 함
    return 2 * circle.radius;
  }
};

console.log(circle.getDiameter());	// 10
```

- 생성자 함수 내부에서는 프로퍼티 또는 메서드를 추가하기 위해 자신이 생성할 인스턴스를 참조할 수 있어야 함
  - 생성자 함수에 의한 객체 생성 방식은 먼저 생성자 함수를 정의한 이후 `new` 연산자와 함께 생성자 함수를 호출하는 단계가 추가로 필요함

```js
function Circle(radius) {
  // 생성자 함수를 정의하는 시점에서는 아직 인스턴스를 생성하기 이전이므로
  // 생성자 함수가 생성할 인스턴스를 가리키는 식별자를 알 수 없음
  ????.radius = radius;
}

Circle.prototype.getDiameter = function () {
  return 2 * ????.radius;
}

// 생성자 함수로 인스턴스를 생성하려면 먼저 생성자 함수를 정의해야 함
const circle = new Circle(5);
```

- `this`는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수
  - `this`를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조 가능
  - `this`는 자바스크립트 엔진에 의해 암묵적으로 생성되며, 코드 어디서든 참조 가능
  - `this`가 가리키는 값, 즉 `this` 바인딩은 함수 호출 방식에 의해 동적으로 결정

```js
// 객체 리터럴
const circle = {
  radius: 5,
  getDiameter() {
    // this는 메서드를 호출한 객체를 가리킴
    return 2 * this.radius;
  }
};

// 생성자 함수
function Circle(radius) {
  // this는 생성자 함수가 생성할 인스턴스를 가리킴
  this.radius = radius;
};

Circle.prototype.getDiameter = function () {
  return 2 * this.radius;
}
```

- `this`는 상황에 따라 가리키는 대상이 다름
  - 자바스크립트의 `this`는 함수가 호출되는 방식에 따라 `this`에 바인딩될 값, 즉 `this` 바인딩이 동적으로 결정됨

- `this`는 코드 어디에서는 참조가 가능하므로 전역에서도 함수 내부에서도 참조할 수 있음

```js
// 전역에서 this는 전역 객체 window를 가리킴
console.log(this);	// window

function square(number) {
  // 일반 함수 내부에서 this는 전역 객체를 가리킴
  console.log(this);	// window
  return number * number;
}

const person = {
  name: 'Lee',
  getName() {
    // 메서드 내부에서 this는 메서드를 호출한 객체를 가리킴
    console.log(this);	// {name: "Lee", getName: f}
    return this.name;
  }
};
console.log(person.getName());	// Lee

function Person(name) {
  this.name = name;
  // 생성자 함수 내부에서 this는 생성자 함수가 생성할 인스턴스를 가리킴
  console.log(this);	// Person {name: "Lee"}
}
const me = new Person('Lee');
```

- strict mode가 적용된 일반 함수 내부의 `this`에는 `undefined`가 바인딩
  - `this`는 객체의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수이므로 일반적으로 객체의 메서드 내부 또는 생성자 함수 내부에서만 의미가 있음
  - 일반 함수 내부에서는 `this`를 사용할 필요가 없음

<br>

## 02. 함수 호출 방식과 `this` 바인딩

- `this` 바인딩은 함수 호출 방식, 즉 함수가 어떻게 호출되었는지에 따라 동적으로 결정
  - 함수의 상위 스코프를 결정하는 방식인 렉시컬 스코프는 함수 정의가 평가되어 함수 객체가 생성되는 시점에 상위 스코프를 결정

```js
const foo = function() {
  console.dir(this);
};

// 1. 일반 함수 호출
// foo 함수를 일반적인 방식으로 호출
// foo 함수 내부의 this는 전역 객체 window를 가리킴
foo();	// window

// 2. 메서드 호출
// foo 함수를 프로퍼티 값으로 할당하여 호출
// foo 함수 내부의 this는 메서드를 호출한 객체 obj를 가리킴
const obj = { foo };
obj.foo();	// obj

// 3. 생성자 함수 호출
// foo 함수를 new 연산자와 함께 생성자 함수로 호출
// foo 함수 내부의 this는 생성자 함수가 생성한 인스턴스를 가리킴
new foo();	// foo {}

// 4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출
// foo 함수 내부의 this는 인수에 의해 결정
const bar = { name: 'bar' };

foo.call(bar);		// bar
foo.apply(bar);		// bar
foo.bind(bar)();	// bar
```

### 일반 함수 호출

- 기본적으로 `this`에는 전역 객체가 바인딩

```js
function foo() {
  console.log("foo's this: ", this);	// window
  function bar() {
    console.log("bar's this: ", this);	// window
  }
  bar();
}
foo();
```

- 일반 함수로 호출된 모든 함수(중첩 함수, 콜백 함수 포함) 내부의 `this`에는 전역 객체가 바인딩
  - `this`는 객체의 프로퍼티나 메서드를 참조하기 위한 자기 참조 변수이므로 일반 함수에서 `this`는 의미가 없음 => strict mode가 적용된 일반 함수 내부의 `this`에는 `undefined` 바인딩

```js
function foo() {
  'use strict';
  console.log("foo's this: ", this);	// undefined
  function bar() {
    console.log("bar's this: ", this);	// undefined
  }
  bar();
}
foo();
```

- 메서드 내에서 정의한 중첩 함수도 일반 함수로 호출되면 중첩 함수 내부의 `this`에는 전역 객체가 바인딩

```js
// var 키워드로 선언한 전역 변수 value는 전역 객체의 프로퍼티
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log("foo's this: ", this);	// {value: 100, foo: f}
    console.log(this.value);	// 100
    
    // 메서드 내에서 정의한 중첩 함수
    function bar() {
      console.log("bar's this: ", this);	// window
      console.log(this.value);	// 1
    }
    // 일반 함수로 호출 => this에 전역 객체가 바인딩
    bar();
  }
};

obj.foo();
```

- 콜백 함수가 일반 함수로 호출된다면 콜백 함수 내부의 `this`에도 전역 객체가 바인딩

```js
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log("foo's this: ", this);	// {value: 100, foo: f}
    // 콜백 함수 내부의 this에는 전역 객체가 바인딩
    setTimeout(function() {
      console.log("callback's this: ", this);	// window
      console.log(this.value);	// 1
    }, 100);
  },
  // 메서드 내부의 중첩 함수나 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키기 위한 방법
  bar() {
    // this 바인딩(obj)을 변수 that에 할당
    const that = this;
    setTimeout(function() {
      // 콜백 함수의 내부에서 this 대신 that을 참조
      console.log(that.value);	// 100
    }, 100);
  },
  baz() {
    // 콜백 함수에 명시적으로 this를 바인딩
    setTimeout(function() {
      console.log(this.value);	// 100
    }.bind(this), 100);
  },
  qux() {
    // 화살표 함수 내부의 this는 상위 스코프의 this를 가리킴
    setTimeout(() => console.log(this.value), 100);	// 100
  }
};
obj.foo();
obj.bar();
obj.baz();
```

### 메서드 호출

- 메서드 내부의 `this`에는 메서드를 호출한 객체, 즉 메서드를 호출할 때 메서드 앞의 마침표(`.`) 연산자 앞에 기술한 객체가 바인딩
  - 메서드 내부의 `this`는 메서드를 소유한 객체가 아닌 메서드를 호출한 객체에 바인딩
  - `this`에 바인딩될 객체는 호출 시점에 결정

```js
const person = {
  name: 'Lee',
  // getName 프로퍼티가 가리키는 함수 객체는 person 객체에 포함된 것이 아니라
  // getName 프로퍼티가 함수 객체를 가리키며 독립적으로 존재
  getName() {
    return this.name;
  }
};

// 메서드 getName을 호출한 객체는 person
console.log(person.getName());	// Lee
```

- 메서드는 객체에 포함된 것이 아니라 독립적으로 존재하는 별도의 객체
  - 메서드 내부의 `this`는 프로퍼티로 메서드를 가리키고 있는 객체와는 관계가 없고 메서드를 호출한 객체에 바인딩

- 프로토타입 메서드 내부에서 사용된 `this`도 일반 메서드와 마찬가지로 해당 메서드를 호출한 객체에 바인딩

```js
function Person(name) {
  this.name = name;
}

Person.prototype.getName = function() {
  return this.name;
};

// getName 메서드를 호출한 객체는 me
const me = new Person('Lee');
console.log(me.getName());	// Lee

// getName 메서드를 호출한 객체는 Person.prototype
Person.prototype.name = 'Kim';
console.log(Person.prototype.getName());	// Kim
```

### 생성자 함수 호출

- 생성자 함수 내부의 `this`에는 생성자 함수가 생성할 인스턴스가 바인딩

### `Function.prototype.apply/call/bind` 메서드에 의한 간접 호출

- `apply`, `call`, `bind` 메서드는 `Function.prototype`의 메서드이므로 해당 메서드는 모든 함수가 상속받아 사용할 수 있음
- `Function.prototype.apply`, `Function.prototype.call` 메서드는 `this`로 사용할 객체와 인수 리스트를 인수로 전달받아 함수를 호출

```js
function getThisBinding() {
  console.log(arguments);
  return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

console.log(getThisBinding());	// window

// getThisBinding 함수를 호출하면서 인수로 전달한 개체를 getThisBindnig 함수의 this에 바인딩
console.log(getThisBinding.apply(thisArg));	// {a: 1}
console.log(getThisBinding.call(thisArg));	// {a: 1}

// apply 메서드는 호출할 함수의 인수를 배열로 묶어 전달
console.log(getThisBinding.apply(thisArg, [1, 2, 3]));
// Arguments(3) [1, 2, 3 callee: f, Symbol(Symbol.iterator): f]
// {a: 1}
// call 메서드는 호출할 함수의 인수를 쉼표로 구분한 리스트 형식으로 전달
console.log(getThisBinding.call(thisArg, 1, 2, 3));
// Arguments(3) [1, 2, 3 callee: f, Symbol(Symbol.iterator): f]
// {a: 1}
```

- `apply`와 `call` 메서드의 본질적인 기능은 함수를 호출하는 것
  - 첫 번째 인수로 전달한 특정 객체를 호출한 함수의 `this`에 바인딩
  - 두 함수는 호출할 함수에 인수를 전달하는 방식만 다를 뿐 동일하게 동작
- `arguments` 객체와 같은 유사 배열 객체에 배열 메서드를 사용하는 경우, `arguments` 객체는 배열이 아니기 때문에 `Array.prototype.slice` 같은 메서드를 사용할 수 없지만, `apply`와 `call`을 이용하면 가능

```js
function convertArgsToArray() {
  console.log(arguments);
  
  // arguments 객체를 배열로 반환
  // Array.prototype.slice를 인수 업시 호출하면 배열의 복사본을 생성함
  const arr = Array.prototype.slice.call(arguments);
  // const arr = Array.prototype.slice.apply(arguments);
  console.log(arr);
  
  return arr;
}

convertArgsToArray(1, 2, 3);	// [1, 2, 3]
```

- `Function.prototype.bind` 메서드는 `apply`와 `call` 메서드와 달리 함수를 호출하지 않고 `this`로 사용할 객체만 전달

```js
function getThisBinding() {
  return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

// bind 메서드는 함수에 this로 사용할 객체를 전달하고 함수를 호출하지는 않음
console.log(getThisBinding.bind(thisArg));	// f getThisBinding() { ... }
// 명시적으로 호출을 해야함
console.log(getThisBinding.bind(thisArg)());	// {a: 1}
```

- `bind` 메서드는 메서드의 `this`와 메서드 내부의 중첩 함수 또는 콜백 함수의 `this`가 불일치하는 문제를 해결하기 위해 유용하게 사용

```js
const person = {
  name: 'Lee',
  foo(callback) {
    setTimeout(callback, 100);
  },
  bar(callback) {
    // bind 메서드로 callback 함수 내부의 this 바인딩을 전달
    setTimeout(callback.bind(this), 100);
  }
};

person.foo(function() {
  console.log(`Hi! My name is ${this.name}.`);	// Hi! My name is .
  // 일반 함수로 호출된 콜백 함수의 내부의 this.name은 브라우저 환경에서 window.name과 같음
  // 브라우저 환경에서 window.name은 브라우저 창의 이름을 나타내는 빌트인 프로퍼티이며 기본값은 ''
  // Node.js 환경에서 this.name은 undefined
});

person.bar(function() {
  console.log(`Hi! My name is ${this.name}.`);	// Hi! My name is Lee.
});
```