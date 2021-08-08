# 프로토타입

---

>  *자바스크립트는 명령형(imperative), 함수형(functional), 프로토타입 기반(prototype-based) 객체지향 프로그래밍(OOP-Object Oriented Programming)을 지원하는 멀티 패러다임 프로그래밍 언어다.*



> #### ❗클래스(Class)
>
> ES6에서 클래스가 도입되었다. 하지만 ES6의 클래스가 기존의 프로토타입 기반 객체지향 모델을 폐지하고 새로운 객체지향 모델을 제공하는 것은 아니다. 사실 클래스도 함수이며, 기존 프로토타입 기반 패턴의 문법적 설탕(크...Syntactic Sugar)이라고 볼 수 있다.
>
> 클래스와 생성자 함수는 모두 프로로타입 기반의 인스턴스를 생성하지만 정확히 동일하게 동작하지는 않는다. 클래스는 생성자 함수보다 엄격하며 클래스는 생성자 함수에서는 제공하지 않는 기능도 제공한다.
>
> 따라서 클래스를 프로토타입 기반 객체 생성 패턴의 단순한 문법적 설탕으로 보기보다는 새로운 객체 생성 메커니즘으로 보는 것이 좀 더 합당하다고 할 수 있다.



![prototype](https://user-images.githubusercontent.com/79819941/127773173-3d8da499-8fe1-4291-bed9-720207ad6033.png) 

위 그림은 아래 코드를 추상화한 것이다.

```javascript
let instance = new Constructor();
```

그리고, 아래 그림은 위 코드를 좀 더 구체화한 것이다.

![prototype1](https://user-images.githubusercontent.com/79819941/127773176-713b4423-50e5-46bb-ba85-e6083b4a97af.png) 

윗변(실선)의 왼쪽 꼭짓점에는 Constructor(생성자 함수)를, 오른쪽 꼭짓점에는 Constructor.prototype이라는 프로퍼티를 위치시켰다. 왼쪽 꼭짓점으로부터 아래를 향한 화살표 중간에 new가 있고, 화살표의 종점에는 instance가 있다. 오른쪽 꼭짓점으로부터 대각선 아래로 향하는 화살표의 종점에는 instance.\__proto__이라는 프로퍼티를 위치시켰다. 

* 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출하면
* Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스(Instance)가 생성된다.
* 이때 instance에는 \__proto__ 라는 프로퍼티가 자동으로 부여되는데,
* 이 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조한다.

prototype이라는 프로퍼티와  \_\_proto\_\_ 라는 프로퍼티가 새로 등장했는데, 이 둘의 관계가 프로토타입 개념의 핵심이다. prototype은 객체이다. 이를 참조하는 \_\_proto\_\_ 역시 당연히 객체일 것이다. prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장합니다. 그러면 인스턴스에서도 숨겨진 프로퍼티인 \_\_proto\_\_를 통해 이 메서드들에 접근할 수 있게 된다.

> ❗ ES5.1 명세에는 \_\_proto\_\_가 아니라 [[prototype]]이라는 명칭으로 정의 되어 있다. \_\_proto\_\_라는 프로퍼티는 사실 브라우저들이 [[prototype]]을 구현한 대상에 지나지 않았다. 명세에는 또 instance.\_\_proto\_\_와 같은 방식으로 직접 접근하는 것은 허용하지 않고 오직 Object.getPrototypeOf(instance) / Reflect.getPrototypeOf(instance)를 통해서만 접근할 수 있도록 정의했었다. 그러나 이런 명세에도 불구하고 대부분의 브라우저들이 \_\_proto\_\_에 직접 접근하는 방식을 포기하지 않았고(브라우저 입장에서도 리스크가 있었을 것이다), 결국 ES6에서는 이를 브라우저에서 동작하는 레거시 코드에 대한 호환성 유지 차원에서 정식으로 인정하기에 이르렀다. 다만 어디까지나 브라우저에서의 호환성을 고려한 지원일 뿐 권장되는 방식은 아니며, 브라우저가 아닌 다른 환경에서는 얼마든지 이 방식이 지원되지 않ㅇ르 가능성이 열려있다.
>
> 많은 책에서는 \_\_proto\_\_ 이해와 편의를 위해 \_\_proto\_\_를 계속 사용하고자 하지만, 이를 학습 목적으로만 이해하고, 실무에서는 가급적 \_\_proto\_\_를 사용하지 않기를 권장한다. 대신 Object.getPrototypeOf() / Object.create() 등을 이용하자.



예를 들어, Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정했다고 하면, 

```javascript
let Person = function (name) {
	this._name = name;
}

Person.prototype.getName = function () {
  return this._name;
}
```

이제 Person의 인스턴스는 \_\_proto\_\_프로퍼티를 통해 getName을 호출할 수 있다.

```javascript
let yooyoung = new Person('yooyoung');
yooyoung.__proto__.getName(); // undefined
```



![image](https://user-images.githubusercontent.com/79819941/128179078-3fc32de5-b946-4ddc-af40-9d4c38d2764b.png) 

그림만 봐도 쉽게 알수 있듯이 상속을 통해 기존의 존재했던 기능을 재사용할 수 있도록 하는 것이 프로토타입이 아닐까??

<br>

**여기까지는 이해를 돕기 위해 정재남님의 코어자바스크립트의 내용**

<br>

# 19장 프로토타입

- 자바스크립트는 명령형, 함수형, 프로토타입 기반 객체지향 프로그래밍을 지원하는 멀티 패러다임 프로그래밍 언어

- 자바스크립트는 객체 기반의 프로그래밍 언어이며 자바스크립트를 이루고 있는 거의 모든 것이 객체임
  - 원시 타입의 값을 제외한 나머지 값들은 모두 객체

<br>

## 01. 객체지향 프로그래밍

- 프로그램을 명령어 또는 함수의 목록으로 보는 전통적인 명령형 프로그래밍의 절차지향적인 관점에서 벗어나 여러 개의 독립적인 단위, 즉 객체의 집합으로 프로그램을 표현하려는 프로그래밍 패러다임

- 실체가 가지고 있는 특징이나 성질을 속성이라고 하며, 이를 통해 실체를 인식하거나 구별할 수 있는데, 다양한 속성 중에서 프로그램에 필요한 속성만 간추려 내어 표현하는 것을 **추상화**라고 함

- 객체란 속성을 통해 여러 개의 값을 하나의 단위로 구성한 복합적인 자료구조
  - 객체의 상태를 나타내는 데이터(=프로퍼티)와 상태 데이터를 조작할 수 있는 동작(=메서드)을 하나의 논리적인 단위로 묶음

<br>

## 02. 상속과 프로토타입

- 상속이란 어떤 객체의 프로퍼티 또는 메서드를 다른 객체가 상속받아 그대로 사용할 수 있는 것
- 자바스크립트는 프로토타입을 기반으로 상속을 구현하여 기존의 코드를 적극적으로 재사용함으로써 불필요한 중복을 제거

```js
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
  this.getArea = function() {
    return Math.PI * this.radius ** 2;
  }
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);

// Circle 생성자 함수는 인스턴스를 생성할 때마다
// 동일한 동작을 하는 getArea 메서드를 중복 생성하므로 모든 인스턴스가 중복 소유함
// => getArea 메서드는 하나만 생성하여 모든 인스턴스가 공유해서 사용하는 것이 바람직
console.log(circle1.getArea === circle2.getArea);	// false
```

```js
// 생성자 함수
function Circle(radius) {
  this.radius = radius;
}

// Circle 생성자 함수가 생성한 모든 인스턴스가 getArea 메서드를 공유해서 사용할 수 있도록 프로토타입에 추가
Circle.prototype.getArea = function() {
  return Math.PI * this.radius ** 2;
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);

// Circle 생성자 함수가 생성한 모든 인스턴스는 부모 객체의 역할을 하는
// 프로토타입 Circle.prototype으로부터 getArea 메서드를 상속받음
// 즉, Circle 생성자 함수가 생성하는 모든 인스턴스는 하나의 getArea 메서드를 공유
console.log(circle1.getArea === circle2.getArea);	// true
```

- 생성자 함수가 생성할 모든 인스턴스가 공통적으로 사용할 프로퍼티나 메서드를 프로토타입에 미리 구현해 두면 생성자 함수가 생성할 모든 인스턴스는 별도의 구현없이 상위(부모) 객체인 프로토타입의 자산을 공유하여 사용 가능

<br>

## 03. 프로토타입 객체

- 프로토타입 객체는 객체지향 프로그래밍의 근간을 이루는 객체 간 상속을 구현하기 위해 사용하며, 어떤 객체의 상위(부모) 객체의 역할을 하는 객체로서 다른 객체에 공유 프로퍼티(메서드 포함)를 제공
  - 프로토타입을 상속받은 하위(자식) 객체는 상위 객체의 프로퍼티를 자신의 프로퍼티처럼 자유롭게 사용 가능
- 모든 객체는 ``[[Prototype]]`이라는 내부 슬롯을 가지며, 이 내부 슬롯의 값은 프로토타입의 참조임
  - 객체가 생성될 때 객체 생성 방식에 따라 프로토타입이 결정되고 `[[Prototype]]`에 저장
- 모든 객체는 하나의 프로토타입을 가지며, 모든 프로토타입은 생성자 함수와 연결되어 있음
  - 프로토타입은 자신의 constructor 프로퍼티를 통해 생성자 함수에 접근할 수 있고, 생성자 함수는 자신의 prototype 프로퍼티를 통해 프로토타입에 접근 가능

### `__proto__` 접근자 프로퍼티

- 모든 객체는 `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입, 즉 `[[Prototype]]` 내부 슬롯에 간접적으로 접근 가능
  - 접근자 프로퍼티는 자체적으로 값(`[[Value]]` 프로퍼티 어트리뷰트)을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 사용하는 접근자 함수, 즉 `[[Get]]`, `[[Set]]` 프로퍼티 어트리뷰트로 구성된 프로퍼티
- `[[Prototype]]` 슬롯 내부에는 직접 접근할 수 없지만, `__proto__` 접근자 프로퍼티를 통해 자신의 프로토타입, 즉 자신의 `[[Prototype]]` 내부 슬롯이 가리키는 프로토타입에 간접적으로 접근 가능
  - getter/setter 함수라고 부르는 접근자 함수를 통해 프로토타입을 취득하거나 할당
  - `__proto__` 접근자 프로퍼티를 통해 프로토타입에 접근하면 내부적으로 `__proto__` 접근자 프로퍼티의 getter 함수인 `[[Get]]`이 호출
  - `__proto__` 접근자 프로퍼티를 통해 새로운 프로토타입을 할당하면 `__proto__` 접근자 프로퍼티의 setter 함수인 `[[Set]]`이 호출

```js
const obj = {};
const parent = { x: 1 };

// getter 함수인 get __proto__가 호출되어 obj 객체의 프로토타입을 취득
obj.__proto__;

// setter 함수인 set __proto__가 호출되어 obj 객체의 프로토타입을 교체
obj.__proto__ = parent;

console.log(obj.x);	// 1
```

- 프로토타입에 접근하기 위해 접근자 프로퍼티를 사용하는 이유는 상호 참조에 의해 프로토타입 체인이 생성되는 것을 방지하기 위함
  - 프로퍼티 검색 방향이 한쪽 방향으로만 흘러가야 하기 때문에 프로토타입 체인은 단방향 링크드 리스트로 구성되어야 함
  - 순환 참조하는 프로토타입 체인이 만들어지면 프로토타입 체인 종점이 존재하지 않기 때문에 프로토타입 체인에서 프로퍼티를 검색할 때 무한 루프에 빠짐

```js
const parent = {};
const child = {};

// child의 프로토타입을 parent로 설정
child.__proto__ = parent;
// parent의 프로토타입을 child로 설정
parent.__proto__ = child;	// TypeError: Cyclic __proto__ value
```

- `__proto__` 접근자 프로퍼티를 코드 내에서 직접 사용하는 것은 권장하지 않음
  - `Object.prototype`을 상속받지 않는 객체를 생성할 수도 있으므로 모든 객체가 `__proto__` 접근자 프로퍼티를 사용할 수 있는 것은 아니기 때문
  - `__proto__` 접근자 프로퍼티 대신 프로토타입의 참조를 취득하고 싶은 경우에는 `Object.getPrototypeOf` 메서드를 사용하고, 프로토타입을 교체하고 싶은 경우에는 `Object.setPrototypeOf` 메서드를 사용할 것을 권장

```js
// obj는 프로토타입 체인의 종점이 되므로 따라서 Object._proto__를 상속받을 수 없음
const obj1 = Object.create(null);
console.log(obj1.__proto__);	// undefined

// 따라서 __proto__보다 Object.getPrototypeOf 메서드를 사용하는 편이 좋음
console.log(Object.getPrototypeOf(obj1));	// null

const obj2 = {};
const parent = { x: 1 };

// get Object.prototype.__proto__와 set Object.prototype.__proto__의 처리 내용과 일치
Object.getPrototypeOf(obj2);	// obj2.__proto__;
Object.setPrototypeOf(obj2, parent);	// obj2.__proto__ = parent;

console.log(obj.x);	// 1
```

### 함수 객체의 prototype 프로퍼티

- 함수 객체만이 소유하는 prototype 프로퍼티는 생성자 함수가 생성할 인스턴스의 프로토타입을 가리킴

```js
// 함수 객체는 prototype 프로퍼티를 소유
(function() {}).hasOwnProperty('prototype');	// true

// 일반 객체는 prototype 프로퍼티를 소유하지 않음
({}).hasOwnProperty('prototype');	// false
```

- 생성자 함수로서 호출할 수 없는 함수, 즉 non-constructor인 화살표 함수와 ES6 메서드 축약 표현으로 정의한 메서드는 prototype 프로퍼티를 소유하지 않으며 프로토타입도 생성되지 않음

```js
// 화살표 함수는 non-constructor이므로 prototype 프로퍼티 소유 X
const Person = name => {
  this.name = name;
};
console.log(Person.hasOwnProperty('prototype'));	// false
console.log(Person.prototype);		// undefined

// ES6의 메서드 축약 표현으로 정의한 메서드도 non-constructor
const obj = {
  foo() {}
}
console.log(obj.foo.hasOwnProperty('prototype'));	// false
console.log(ojb.foo.prototype);		// undefined
```

- 생성자 함수로 호출하기 위해 정의하지 않은 일반 함수(함수 선언문, 함수 표현식)도 prototype 프로퍼티를 소유하지만 객체를 생성하지 않는 일반 함수의 prototype 프로퍼티는 아무런 의미가 없음

- 모든 객체가 가지고 있는(엄밀이 말하면 `Object.prototype`으로부터 상속받은) `__proto__` 접근자 프로퍼티와 함수 객체만이 가지고 있는 `prototype` 프로퍼티는 결국 동일한 프로토타입을 가리키나, 프로퍼티를 사용하는 주체가 다름
  - `__proto__` 접근자 프로퍼티는 모든 객체가 사용하며, 객체가 자신의 프로토타입에 접근 혹은 교체하기 위해 사용
  - `prototype` 프로퍼티는 생성자 함수가 사용 가능하며, 생성자 함수가 자신이 생성할 객체(인스턴스)의 프로토타입을 할당하기 위해 사용

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// 결국 둘은 동일한 프로토타입을 가짐
console.log(Person.prototype === me.__proto__);	// true
```

### 프로토타입의 constructor 프로퍼티와 생성자 함수

- 모든 프로토타입은 constructor 프로퍼티를 가지며, 이 constructor 프로퍼티는 prototype 프로퍼티로 자신을 참조하고 있는 생성자 함수를 가리킴
  - 이 연결은 생성자 함수가 생성될 때, 즉 함수 객체가 생성될 때 이뤄짐

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// me 객체의 생성자 함수는 Person이다.
console.log(me.constructor === Person);	// true
```

<br>

## 04. 리터럴 표기법에 의해 생성된 객체의 생성자 함수와 프로토타입

- 생성자 함수에 의해 생성된 인스턴스는 프로토타입의 constructor 프로퍼티에 의해 생성자 함수와 연결
  - constructor 프로퍼티가 가리키는 생성자 함수는 인스턴스를 생성한 생성자 함수

```js
// obj 객체를 생성한 생성자 함수는 Object
const obj = new Object();
console.log(obj.constructor === Object);	// true

// add 함수 객체를 생성한 생성자 함수는 Function
const add = new Function('a', 'b', 'return a + b');
console.log(add.constructor === Function);	// true
```

- 리터럴 표기법에 의해 생성된 객체도 프로토타입이 존재하지만, 프로토타입의 constructor 프로퍼티가 가리키는 생성자 함수가 반드시 객체를 생성한 생성자 함수라고 단정할 수 없음
  - ECMAScript 사양에 의해 Object 생성자 함수에 인수를 전달하지 않거나 `undefined` 또는 `null`을 인수로 전달하면서 호출하면 내부적으로는 추상 연산 `OrdinaryObjectCreate`를 호출하여 `Object.prototype`을 프로토타입으로 갖는 빈 객체를 생성하기 때문
  - 객체 리터럴이 평가될 때는 추상 연산 `OrdinaryObjectCreate`를 호출하여 빈 객체를 생성하고 프로퍼티를 추가하도록 정의되어 있음

```js
// Object 생성자 함수로 생성하지 않고 객체 리터럴로 obj 객체를 생성
const obj = {};

// obj 객체의 생성자 함수는 Object 생성자 함수임
console.log(obj.constructor === Object);	// true

// 그러나 객체 리터럴에 의해 생성된 객체는 Object 생성자 함수가 생성한 객체가 아님
```

```js
// 2. 인수가 전달되지 않았을 때 추상 연산 OrdinaryObjectCreate를 호출하여 빈 객체를 생성
let obj = new Object();
console.log(obj);	// {}

// 1. new.target이 undefined거나 Object가 아닌 경우
// 인스턴스 → Foo.prototype → Object.prototype 순으로 프로토타입 체인이 생성
class Foo extends Objects {}
new Foo();	// Foo {}

// 3. 인수가 전달된 경우에는 인수를 객체로 변환
// Number 객체 생성
obj = new Object(123);
console.log(obj);	// Number {123}

// String 객체 생성
obj = new Object('123');
console.log(obj);	// String{"123"}
```

- 함수의 경우 `Function` 생성자 함수를 호출하여 생성한 함수는 렉시컬 스코프를 만들지 않고 전역 함수인 것처럼 스코프를 생성하며 클로저도 만들지 않음
  - 따라서 함수 선언문과 함수 표현식을 평가하여 함수 객체를 생성한 것은 Function 생성자 함수가 아님
  - 하지만 constructor 프로퍼티를 통해 확인해보면 해당 함수의 생성자 함수는 Function 생성자 함수라고 나옴

- 리터럴 표기법에 의해 생성된 객체도 상속을 위해 가상적인 생성자 함수를 가짐
  - 프로토타입은 생성자 함수와 더불어 생성되며, prototype, constructor 프로퍼티에 의해 연결되어 있기 때문
- 프로토타입과 생성자 함수는 단독으로 존재할 수 없고 언제나 쌍으로 존재함

<br>

## 05. 프로토타입의 생성 시점

- 객체는 리터럴 표기법 또는 생성자 함수에 의해 생성되므로 결국 모든 객체는 생성자 함수와 연결
- 프로토타입은 생성자 함수가 생성되는 시점에 더불어 생성
- 생성자 함수에는 사용자가 직접 정의한 사용자 정의 생성자 함수와 자바스크립트가 기본 제공하는 빌트인 생성자 함수로 구분

### 사용자 정의 생성자 함수와 프로토타입 생성 시점

- 생성자 함수로서 호출할 수 있는 함수, 즉 constructor는 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입도 더불어 생성

```js
console.log(Person.prototype);	// {constructor: f}
// 생성된 프로토타입은 오직 constructor 프로퍼티만을 갖는 객체
// 프로토타입도 객체이고 모든 객체는 프로토타입을 가지므로 프로토타입도 자신의 프로토타입을 가짐
// 생성된 프로토타입의 프로토타입은 Object.prototype

// 런타임 이전에 자바스크립트 엔진에 의해 평가되어 함수 객체가 됨 (함수 호이스팅)
function Person(name) {
  this.name = name;
}
```

- 생성자 함수로서 호출할 수 없는 함수, 즉 non-constructor는 프로토타입이 생성되지 않음

```js
const Person = name => {
  this.name = name;
}

console.log(Person.prototype);	// undefined
```

- 빌트인 생성자 함수가 아닌 사용자 정의 생성자 함수는 자신이 평가되어 함수 객체로 생성되는 시점에 프로토타입도 더불어 생성되며, 생성된 프로토타입의 프로토타입은 언제나 `Object.prototype`이다.

### 빌트인 생성자 함수와 프로토타입 생성 시점

- 빌트인 생성자 함수도 마찬가지로 생성자 함수가 생성되는 시점에 프로토타입이 생성
  - 모든 빌트인 생성자 함수는 전역 객체가 생성되는 시점에 생성되며, 생성된 프로토타입은 빌트인 생성자 함수의 `prototype` 프로퍼티에 바인딩
  - 이후 생성자 함수 또는 리터럴 표기법으로 객체를 생성하면 프로토타입은 생성된 객체의 `[[Prototype]]` 내부 슬롯에 할당

<br>

## 06. 객체 생성 방식과 프로토타입의 결정

- 객체 생성 방법에는 객체 리터럴, Object 생성자 함수, 생성자 함수, `Object.create` 메서드, 클래스(ES6)가 있음
  - 세부적인 객체 생성 방식의 차이는 있으나 추상 연산 `OrdinaryObjectCreate`에 의해 생성된다는 공통점이 있으며, 프로토타입은 추상 연산 `OrdinaryObjectCreate`에 전달되는 인수에 의해 결정됨

### 객체 리터럴에 의해 생성된 객체의 프로토타입

- 자바스크립트 엔진은 객체 리터럴을 평가하여 객체를 생성할 때 추상연산 `OrdinaryObjectCreate`를 호출하며, 추상 연산 `OrdinaryObjectCreate`에 전달되는 프로토타입은 `Object.prototype`
  - 객체 리터럴에 의해서 생성된 객체는 `Object.prototype`을 상속받음

```js
const obj = { x: 1 };

console.log(obj.constructor === Object);	// true
console.log(obj.hasOwnProperty('x'));	// true
```

### `Object` 생성자 함수에 의해 생성된 객체의 프로토타입

- `Object` 생성자 함수를 호출하면 객체 리터럴과 마찬가지로 추상 연산 `OrdinaryObjectCreate`가 호출되며, 추상 연산 `OrdinaryObjectCreate`에 전달되는 프로토타입은 `Object.prototype`
  - `Object` 생성자 함수에 의해서 생성된 객체는 `Object.prototype`을 상속받음

```js
const obj = new Object();
obj.x = 1;

console.log(obj.constructor === Object);	// true
console.log(obj.hasOwnProperty('x'));	// true
```

- 객체 리터럴과 `Object` 생성자 함수에 의한 객체 생성 방식의 차이는 프로퍼티를 추가하는 방식
  - 객체 리터럴 방식은 객체 리터럴 내부에 프로퍼티를 추가하지만 `Object` 생성자 함수 방식은 일단 빈 객체를 생성한 이후 프로퍼티를 추가해야 함

### 생성자 함수에 의해 생성된 객체의 프로토타입

- `new` 연산자와 함께 생성자 함수를 호출하여 인스턴스를 생성하면 다른 객체 생성 방식과 마찬가지로 추상 연산 `OrdinaryObjectCreate`가 호출되며, 추상 연산 `OrdinaryObjectCreate`에 전달되는 프로토타입은 생성자 함수의 prototype 프로퍼티에 바인딩되어 있는 객체

- 프로토타입도 객체이므로 일반 객체와 같이 프로토타입에도 프로퍼티를 추가/삭제할 수 있으며, 이렇게 추가/삭제된 프로퍼티는 프로토타입 체인에 즉각 반영됨

```js
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
// Person 생성자 함수를 통해 생성된 모든 객체는 아래 메서드를 상속받아 자신의 메서드처럼 사용 가능
Person.prototype.sayHello = function() {
  console.log(`Hi! My name is ${this.name}.`);
}

const me = new Person('Kwak');
const you = new Person('Lee');

me.sayHello();	// Hi! My name is Kwak.
you.sayHello();	// Hi! My name is Lee.
```

<br>

## 07. 프로토타입 체인

- 생성자 함수에 의해 생성된 객체는 `Object.prototype`의 메서드인 `hasOwnProperty`를 호출할 수 있음
  - 객체가 생성자 함수의 프로토타입뿐만 아니라 `Object.prototype`도 상속받았다는 것을 의미

- 자바스크립트는 객체의 프로퍼티에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티가 없다면 `[[Prototype]]` 내부 슬롯의 참조를 따라 자신의 부모 역할을 하는 프로토타입의 프로퍼티를 순차적으로 검색 => 프로토타입 체인
  - 프로토타입 체인은 자바스크립트 객체가 객체지향 프로그래밍의 상속을 구현하는 메커니즘
- 하위 객체가 상위 객체의 메서드를 호출하면 자바스크립트 엔진은 먼저 메서드를 호출한 객체에서 메서드를 검색. 해당 객체에 메서드가 없으면 프로토타입 체인을 따라, 다시 말해 `[[Prototype]]` 내부 슬롯에 바인딩되어 있는 프로토타입으로 이동하여 메서드 검색
- 프로토타입 체인의 최상위에 위치하는 객체는 언제나 `Object.prototype`이며, 모든 객체는 `Object.prototype`을 상속받음
  - `Object.prototype`을 프로토타입 체인의 종점이라고 하며, `Object.prototype`의 `[[Prototype]]` 내부 슬롯의 값은 `null`
  - 프로토타입 체인의 종점에서도 프로퍼티를 검색할 수 없는 경우 `undefined`를 반환하며, 이때 에러가 발생하지 않음
- 프로토타입 체인은 상속과 프로퍼티 검색을 위한 메커니즘이고, 스코프 체인은 식별자 검색을 위한 메커니즘
  - 스코프 체인과 프로토타입 체인은 서로 연관없이 별도로 동작하는 것이 아니라 서로 협력하여 식별자와 프로퍼티를 검색하는 데 사용

<br>

## 08. 오버라이딩과 프로퍼티 섀도잉

```js
const Person = (function() {
  // 생성자 함수
  function Person(name) {
    this.name = name;
  }
  // 프로토타입 메서드
  Person.prototype.sayHello = function() {
    console.log(`Hi! My name is ${this.name}.`);
  }
  // 생성자 함수 반환
  return Person;
}());

const me = new Person('Lee');

// 인스턴스 메서드
me.sayHello = function() {
  console.log(`Hey! My name is ${this.name}.`);
}

// 인스턴스 메서드가 호출됨
// 프로토타입 메서드는 인스턴스 메서드에 의해 가려짐
me.sayHello();	// Hey! My name is Lee.
```

- 프로토타입 프로퍼티와 같은 이름의 프로퍼티를 인스턴스에 추가하면 프로토타입 체인을 따라 프로토타입의 프로퍼티를 검색하여 프로토타입 프로퍼티를 덮어쓰는 것이 아니라 인스턴스 프로퍼티로 추가
- 상속 관계에 의해 프로퍼티가 가려지는 현상을 프로퍼티 섀도잉이라고 함

```js
// 인스턴스 메서드 삭제
delete me.sayHello;
// 인스턴스에는 sayHello 메서드가 없으므로 프로토타입 메서드가 호출
me.sayHello();	// Hi! My name is Lee.

// 프로토타입 체인을 통해서 프로토타입 메서드는 삭제되지 않음
delete me.sayHello;
// 프로토타입 메서드 호출
me.sayHello();	// Hi! My name is Lee.
```

- 하위 객체를 통해 프로토타입의 프로퍼티를 변경 또는 삭제하는 것은 불가능
  - 하위 객체를 통해 프로토타입에 get 액세스는 허용되나 set 액세스는 허용되지 않기 때문
- 프로토타입 프로퍼티를 변경 또는 삭제하려면 프로토타입에 직접 접근해야 함

```js
Person.prototype.sayHello = function() {
  console.log(`Hey! My name is ${this.name}.`);
}
```

<br>

## 09. 프로토타입의 교체

- 프로토타입은 임의로 다른 객체로 변경 가능
  - 부모 객체인 프로토타입을 동적으로 변경할 수 있으며, 따라서 객체 간의 상속 관계를 동적으로 변경 가능

### 생성자 함수에 의한 프로토타입의 교체

```js
const Person = (function() {
  function Person(name) {
    this.name = name;
  }
  // 생성자 함수의 prototype 프로퍼티를 통해 프로토타입을 객체 리터럴로 교체
  Person.prototype = {
    sayHello() {
      console.log(`Hi! My name is ${this.name}.`);
    }
  };
  return Person;
}());

const me = new Person('Lee');

// 프로토타입으로 교체한 객체 리터럴에는 constructor 프로퍼티가 없음
// me 객체의 생성자 함수를 검색하면 프로토타입 체인을 따라 Person이 아닌 Object가 나옴
console.log(me.constructor === Person);	// false
console.log(me.constructor === Object);	// true
```

- 프로토타입으로 교체한 객체 리터럴에 `constructor` 프로퍼티를 추가해줌으로써 프로토타입의 `constructor` 프로퍼티를 되살릴 수 있음

```js
const Person = (function() {
  function Person(name) {
    this.name = name;
  }
  Person.prototype = {
    // constructor 프로퍼치와 생성자 함수 간의 연결을 설정
    constructor: Person,
    sayHello() {
      console.log(`Hi! My name is ${this.name}.`);
    }
  };
  return Person;
}());
```

### 인스턴스에 의한 프로토타입의 교체

- 프로토타입은 생성자 함수의 `prototype` 프로퍼티뿐만 아니라 인스턴스의 `__proto__` 접근자 프로퍼티(또는 `Object.getPrototypeOf` 메서드)를 통해 접근 가능하며, 마찬가지로 인스턴스의 `__proto__` 접근자 프로퍼티(또는 `Object.setPrototypeOf` 메서드)를 통해 프로토타입 교체 가능
- 생성자 함수의 `prototype` 프로퍼티에 다른 임의의 객체를 바인딩하는 것은 미래에 생성할 인스턴스의 프로토타입을 교체하는 것이며, `__proto__` 접근자 프로퍼티를 통해 프로토타입을 교체하는 것은 이미 생성된 객체의 프로토타입을 교체하는 것

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Kwak');

// 프로토타입으로 교체할 객체
const parent = {
  sayHello() {
    console.log(`Hi! My name is ${this.name}.`);
  }
};

// me 객체의 프로토타입을 parent 객체로 교체
Object.setPrototypeOf(me, parent);
// me.__proto__ = parent; // 와 동일
```

- 생성자 함수에 의한 프로토타입 교체는 생성자 함수의 `prototype` 프로퍼티가 교체된 프로토타입을 가리키지만, 인스턴스에 의한 프로토타입 교체는 생성자 함수의 `prototype` 프로퍼티가 교체된 프로토타입을 가리키지 않음
  - 따라서 프로토타입으로 교체한 객체 리터럴에 `constructor` 프로퍼티를 추가하고 생성자 함수의 prototype 프로퍼티를 재설정하여 파괴된 생성자 함수와 프로토타입 간의 연결을 되살릴 수 있음

``` js
const parent = {
  // constructor 프로퍼티와 생성자 함수 간의 연결을 설정
  constructor: Person,
  sayHello() {
    console.log(`Hi! My name is ${this.name}.`);
  }
};
// 생성자 함수의 prototype 프로퍼티와 프로토타입 간의 연결을 설정
Person.prototype = parent;

console.log(me.constructor === Person);	// true
console.log(me.constructor === Object);	// false

console.log(Person.prototype === Object.getPrototypeOf(me));	// true
```

- 프로토타입 교체를 통해 객체 간의 상속 관계를 동적으로 변경하지 않는 것이 좋으며, 상속 관계를 인위적으로 설정하려면 직접 상속이 더 편리하고 안전

<br>

## 10. `instanceof` 연산자

- 이항 연산자로서 좌변에 객체를 가리키는 식별자, 우변에 생성자 함수를 가리키는 식별자를 피연산자로 받으며, 우변의 생성자 함수의 `prototype`에 바인딩된 객체가 좌변의 객체의 프로토타입 체인 상에 존재하면 `true`로 평가되고 아니면 `false`로 평가
  - 우변의 피연산자가 함수가 아닌 경우 TypeError 발생

```js
function Person(name) {
  this.name = name;
}

const me = new Person('Lee');

// Person과 Object 둘 다 me 객체의 프로토타입 체인 상에 존재하므로 true 리턴
console.log(me instanceof Person);	// true
console.log(me instanceof Object);	// true
```

- `instanceof` 연산자는 프로토타입의 `constructor` 프로퍼티가 기리키는 생성자 함수를 찾는 것이 아니라 생성자 함수의 `prototype`에 바인딩된 객체가 프로토타입 체인 상에서 존재하는지 확인

- 생성자 함수에 의해 프로토타입이 교체되어 `constructor` 프로퍼티와 생성자 함수 간의 연결이 파괴되어도 생성자 함수의 `prototype` 프로퍼티와 프로토타입 간의 연결은 파괴되지 않으므로 `instanceof`는 아무런 영향을 받지 않음

<br>

## 11. 직접 상속

### `Object.create`에 의한 직접 상속

- 명시적으로 프로토타입을 지정하여 새로운 객체를 생성
- 첫 번째 매개변수에는 생성할 객체의 프로토타입으로 지정한 객체를 전달하고, 두 번째 매개변수는 옵션으로 생성할 객체의 프로퍼티 키와 프로퍼티 디스크립터 객체로 이뤄진 객체를 전달

```js 
// 프로토타입이 null인 객체를 생성. 생성된 객체는 프로토타입 체인의 종점에 위치
// obj → null
let obj = Object.create(null);
console.log(Object.getPrototypeOf(obj) === null);	// true
// Object.prototype을 상속받지 못함
console.log(obj.toString());	// TypeError: obj.toString is not a function

// obj → Object.prototype → null
// obj = {};와 동일
obj = Object.create(Object.prototype);
console.log(Object.getPrototypeOf(obj) === Object.prototype);	// true

// obj → Object.prototype → null
// obj = { x: 1 };와 동일
obj = Object.create(Object.prototype, {
  x: { value: 1, writable: true, enumerable: true, configuragle: true }
});
console.log(obj.x);	// 1
console.log(Object.getPrototypeOf(obj) === Object.prototype);	// true

const myProto = { x: 10 };
// 임의의 객체를 직접 상속받음
// obj → myProto → Object.prototype → null
obj = Object.create(myProto);
console.log(obj.x);	// 10
console.log(Object.getPrototypeOf(obj) === myProto);	// true

// 생성자 함수
function Person(name) {
  this.name = name;
}

// obj → Person.prototype → Object.prototype → null
obj = Object.create(Person.prototype);
obj.name = 'Lee';
console.log(obj.name);	// Lee
console.log(Object.getPrototypeOf(obj) === Person.prototype);	// true
```

- `Object.create` 메서드의 장점
  - `new` 연산자가 없이도 객체를 생성할 수 있음
  - 프로토타입을 지정하면서 객체를 생성할 수 있음
  - 객체 리터럴에 의해 생성된 객체도 상속받을 수 있음
- `Object.prototype`의 빌트인 메서드인 `Object.prototype.hasOwnProperty`, `Object.prototype.isPrototypeOf`, `Object.prototype.prototypeIsEnumerable` 등은 모든 객체의 프로토타입 체인의 종점, 즉 `Object.prototype`의 메서드이므로 모든 객체가 상속받아 호출 가능
- `Object.prototype`의 빌트인 메서드를 객체가 직접 호출하는 것은 권장하지 않음 
  - `Object.create` 메서드를 통해 프로토타입 체인의 종점에 위치하는 객체를 임의로 생성할 수 있으며, 프로토타입 체인의 종점에 위치하는 객체는 `Object.prototype`의 빌트인 메서드를 사용할 수 없음

```js
const obj = Object.create(null);
obj.a = 1;
console.log(Object.getPrototypeOf(obj) === null);	// true

console.log(obj.hasOwnProperty('a'));
// TypeError: obj.hasOwnProperty is not a function

// Object.prototype의 빌트인 메서드는 객체로 직접 호출하지 않는 것이 좋음
console.log(Object.prototype.hasOwnProperty.call(obj, 'a'));	// true
```

### 객체 리터럴 내부에서 `__proto__`에 의한 직접 상속

- `Object.create` 메서드의 의한 직접 상속은 여러 장점이 있으나, 두 번째 인자로 프로퍼티를 정의하거나 객체를 생성한 이후 프로퍼티를 추가하는 방식은 번거롭고 깔끔하지 않음

```js
const myProto = { x: 10 };

// 객체 리터럴에 의해 객체를 생성하면서 프로토타입을 지정하여 직접 상속 가능
const obj = {
  y: 20,
  // 객체를 직접 상속
  // obj → myProto → Object.prototype → null
  __proto__: myProto
};
// 위 코드는 아래와 동일
/*
const obj = Object.create(myProto, {
	y: { value: 20, writable: true, enumerable: true, configurable: true }
});
*/

console.log(obj.x, obj.y);	// 10 20
console.log(Object.getPrototypeOf(obj) === myProto);	// true
```

<br>

## 12. 정적 프로퍼티/메서드

- 생성자 함수로 인스턴스를 생성하지 않아도 참조/호출할 수 있는 프로퍼티/메서드

```js 
function Person(name) {
  this.name = name;
}

// 프로토타입 메서드
Person.prototype.sayHello = function() {
  console.log(`Hi! My name is ${this.name}`);
};

// 정적 프로퍼티
Person.staticProp = 'static prop';
// 정적 메서드
Person.staticMethod = function() {
  console.log('staticMethod');
};

const me = new Person('Lee');

// 생성자 함수에 추가한 정적 프로퍼티/메서드는 생성자 함수로 참조/호출
Person.staticMethod();	// staticMethod
// 정적 프로퍼티/메서드는 생성자 함수가 생성한 인스턴스로 참조/호출할 수 없음
// 인스턴스로 참조/호출할 수 있는 프로퍼티/메서드는 프로토타입 체인 상에 존재해야 함
me.staticMethod();	// TypeError: me.staticMethod is not a function
```

- 생성자 함수가 생성한 인스턴스는 자신의 프로토타입 체인에 속한 객체의 프로퍼티/메서드에 접근할 수 있는데, 정적 프로퍼티/메서드는 인스턴스의 프로토타입 체인에 속한 객체의 프로퍼티/메서드가 아니므로 인스턴스로 접근 불가능
- 인스턴스/프로토타입 메서드에서 `this`를 사용하지 않는다면 그 메서드는 정적 메서드로 변경 가능
  - 인스턴스가 호출한 인스턴스/프로토타입 메서드 내에서 `this`는 인스턴스를 가리킴
  - 프로토타입 메서드를 호출하려면 인스턴스를 생성해야 하지만 정적 메서드는 인스턴스를 생성하지 않아도 호출 가능

<br>

## 13. 프로퍼티 존재 확인

### `in` 연산자

- 객체 내에 특정 프로퍼티가 존재하는지 여부를 확인
- `key in object` 식으로 사용

```js
const person = {
  name: 'Lee',
  address: 'Seoul'
};

console.log('name' in person);	// true
console.log('address' in person);	// true
console.log('age' in person);	// false
```

- `in` 연산자는 확인 대상 객체의 프로퍼티뿐만 아니라 확인 대상 객체가 상속받은 모든 프로토타입의 프로퍼티를 확인하므로 주의가 필요

```Js
console.log('toString' in person);	// true
// person 객체에는 toString이라는 프로퍼티가 없지만
// Object.prototype의 메서드이기 때문에 true를 반환
```

- `in` 연산자 대신 ES6에서 도입된 `Reflect.has`도 사용 가능하며, `in` 연산자와 동일

### `Object.prototype.hasOwnProperty` 메서드

- 인수로 전달받은 프로퍼티 키가 객체 고유의 프로퍼티 키인 경우에만 `true`를 반환하고 상속받은 프로토타입 프로퍼티 키인 경우 `false`를 반환

```js
console.log(person.hasOwnProperty('name'));	// true
console.log(person.hasOwnProperty('age'));	// false
console.log(person.hasOwnProperty('toString'));	// false
```

<br>

## 14. 프로퍼티 열거

### `for ... in` 문

- 객체의 모든 프로퍼티를 순회하며 열거

```js
for (변수선언문 in 객체) { ... }
```

- 객체의 프로토타입 체인 상에 존재하는 모든 프로토타입의 프로퍼티 중에서 프로퍼티 어트리뷰트 `[[Enumerable]]`의 값이 `true`인 프로퍼티를 순회하며 열거하며, 프로퍼티 키가 심벌인 프로퍼티는 열거하지 않음

```js
const person = {
  name: 'Lee',
  address: 'Seoul',
  __proto__: { age: 20 }
};

for (const key in person) {
  console.log(key + ': ' + person[key]);
}

// name: Lee
// address: Seoul
// age: 20
```

- 상속받은 프로퍼티는 제외하고 객체 자신의 프로퍼티만 열거하려면 `Object.prototype.hasOwnProperty` 메서드를 사용하여 객체 자신의 프로퍼티인지 확인해야 함

```js
for (const key in person) {
  if (!person.hasOwnProperty(key)) continue;
  console.log(key + ': ' + person[key]);
}
// name: Lee
// address: Seoul
```

- `for ... in` 문은 프로퍼티를 열거할 때 순서를 보장하지 않음
  - 대부분의 모던 브라우저는 순서를 보장하고 숫자인 프로퍼티 키에 대해서는 정렬을 실시
- 배열에는 `for ... in` 문을 사용하지 말고 일반적인 `for` 문이나 `for ... of` 문 또는 `Array.prototype.forEach` 메서드를 사용하는 것을 권장
  - 배열도 객체이므로 프로퍼티와 상속받은 프로퍼티가 포함될 수 있음

```js
const arr = [ 1, 2, 3 ];
arr.x = 10;	// 배열도 객체이므로 프로퍼티를 가질 수 있음

for (const i in arr) {
  // 프로퍼티 x도 출력됨
  console.log(arr[i]);	// 1 2 3 10
}

// arr.lengt는 3
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);	// 1 2 3
}

// forEach 메서드는 요소가 아닌 프로퍼티는 제외함
arr.forEach(v => console.log(v));	// 1 2 3

// for ... of는 변수 선언문에서 선언한 변수에 키가 아닌 값을 할당
for (const value of arr) {
  console.log(value); // 1 2 3
}
```

### `Object.keys`/`values`/`entries` 메서드

- 객체 자신의 고유의 열거 가능한 프로퍼티만 열거

```js
const person = {
  name: 'Lee',
  address: 'Seoul',
  __proto__: { age: 20 }
};

console.log(Object.keys(person));	// ["name", "address"]
console.log(Object.values(person));	// ["Lee", "Seoul"]
console.log(Obejct.entires(person));	// [["name", "Lee"], ["address", "Seoul"]]

Object.entries(person).forEach(([key, value]) => console.log(key, value));
/*
name Lee
address Seoul
*/
```

# 20장: strict mode

<br>

## 01. strict mode란?

```javascript
function foo() {
  x = 10; // 선언하지 않은 x 변수에 값 10 할당
  // 자바스크립트 엔진은 먼저 foo 함수의 스코프에서 x 변수의 선언을 검색하고,
  // foo 함수의 스코프에는 x 변수의 선언이 없으므로 상위 스코프에서 다시 검색을 함
  // 전역 스코프에도 x가 존재하지 않지만 ReferenceError를 발생되지 않고
  // 자바스크립트 엔진이 암묵적으로 전역 객체에 x 프로퍼티를 동적 생성
}
foo();

console.log(x);	// ?
```

- **암묵적 전역?** 전역 스코프에도 변수의 선언이 존재하지 않을 때, ReferenceError를 발생시키는 것이 아니라 암묵적으로 전역 객체에 해당 프로퍼티를 등록하는 것
  - 반드시 `var`, `let`, `const` 키워드를 사용해서 변수를 선언한 다음 사용해야 함

- ES5부터 자바스크립트 언어의 문법을 좀 더 엄격히 적용하여 오류를 발생시킬 가능성이 높거나 자바스크립트 엔진의 최적화 작업에 문제를 일으킬 수 있는 코드에 대해 명시적인 에러를 발생시키는 strict mode가 추가

- ES6에서 도입된 클래스와 모듈은 기본적으로 strict mode가 적용

<br>

## 02. strict mode의 적용

- 전역의 선두 또는 함수 몸체의 선두에 `'use strict';`를 추가
  - 전역의 선두에 추가하면 스크립트 전체에 strict mode가 적용
  - 함수 몸체의 선두에 추가하면 해당 함수와 중첩 함수에 strict mode가 적용

```js
'use strict';

function foo() {
  x = 10;	// ReferenceError: x is not defined
}
foo();
```

```js
function foo() {
  'use strict';
  x = 10;	// ReferenceError: x is not defined
}
foo();
```

- 코드의 선두에 `'use strict';`를 위치시키지 않으면 strict mode가 제대로 동작하지 않음

<br>

## 03. 전역에 strict mode를 적용하는 것은 피하자

- 전역에 적용한 strict mode는 스크립트 단위로 적용

```html
<!DOCTYPE html>
<html>
  <body>
    <script>
      'use strict';
    </script>
    <script>
      x = 1;	// 에러가 발생하지 않음
      console.log(x);	// 1
    </script>
    <script>
      'use strict';
      y = 1;	// ReferenceError: y is not defined
    </script>
  </body>
</html>
```

- strict mode와 non-strict mode를 혼용하는 것은 오류를 발생시킬 수 있음
  - 특히 외부 라이브러리를 사용하는 경우 라이브러리가 non-strict mode인 경우도 있음
- 즉시 실행 함수로 스크립트 전체를 감싸서 스코프를 구분하고 즉시 실행 함수의 선두에 strict mode를 적용

```js
(function() {
  'use strict';
  
  // Do something...
}());
```

<br>

## 04. 함수 단위로 strict mode를 적용하는 것도 피하자

- 함수 단위로 strict mode를 적용할 수 있음
  - 그러나 어떤 함수는 strict mode를 적용하고 어떤 함수는 strict mode를 적용하지 않는 것은 바람직하지 않음
  - 모든 함수에 일일이 strict mode를 적용하는 것은 번거로움
  - strict mode가 적용된 함수가 참조할 함수 외부의 컨텍스트에 strict mode를 적용하지 않는다면 문제 발생

```js
(function() {
  // non-strict mode
  var let = 10;	// 에러가 발생하지 않음
  function foo() {
    'use strict';
    let = 20;	// SyntaxError: Unexpected strict mode reserved word
  }
  foo();
}());
```

- strict mode는 즉시 실행 함수로 감싼 스크립트 단위로 적용하는 것이 바람직

<br>

## 05. strict mode가 발생시키는 에러

### 암묵적 전역

- 선언하지 않은 변수를 참조하면 ReferenceError 발생

```js
(function() {
  'use strict';
  
  x = 1;
  console.log(x);	// ReferenceError: x is not defined
}());
```

### 변수, 함수, 매개변수의 삭제

- `delete` 연산자로 변수, 함수, 매개변수를 삭제하면 SyntaxError 발생

```js
(function() {
  'use strict';
  
  var x = 1;
  delete x;	// SyntaxError: Delete of an unqualified identifier in strict mode.
  
  function foo(a) {
    delete a;	// SyntaxError: Delete of an unqualified identifier in strict mode.
  }
  delete foo;	// SyntaxError: Delete of an unqualified identifier in strict mode.
}());
```

### 매개변수 이름의 중복

- 중복된 매개변수 이름을 사용하면 SyntaxError 발생

```js
(function() {
  'use strict';
  
  // SyntaxError: Duplicate parameter name not allowed in this context
  function foo(x, x) {
    return x + x;
  }
  console.log(foo(1, 2));
}());
```

### `with` 문의 사용

- `with` 문을 사용하면 SyntaxError 발생
  - `with` 문은 전달된 객체를 스코프 체인에 추가
  - 동일한 객체의 프로퍼티를 반복에서 사용할 때 객체 이름을 생략할 수 있어서 코드가 간단해지는 효과가 있지만, 성능과 가독성이 나빠지므로 사용하지 않는 것이 좋음

```js
(function() {
  'use strict';
  
  // SyntaxError: Strict mode code may not include a with statement
  with({ x: 1 }) {
    console.log(x);
  }
}());
```

<br>

## 06. strict mode 적용에 의한 변화

### 일반 함수의 `this`

- strict mode에서 함수를 일반 함수로서 호출하면 `this`에 `undefined`가 바인딩
  - 생성자 함수가 아닌 일반 함수 내부에서는 `this`를 사용할 필요가 없기 때문

```Js
(function() {
  'use strict';
  
  function foo() {
    console.log(this);	// undefined
  }
  foo();
  
  function Foo() {
    console.log(this);	// Foo
  }
  new Foo();
}());
```

### `arguments` 객체

- strict mode에서는 매개변수에 전달된 인수를 재할당하여 변경해도 `arguments` 객체에 반영되지 않음

```js
(function (a) {
  'use strict';
  // 매개변수에 전달된 인수를 재할당하여 변경
  a = 2;
  
  // 변경된 인수가 arguments 객체에 반영되지 않음
  console.log(arguments);	// { 0: 1, length: 1 }
}(1));
```

# 21장: 빌트인 객체

<br>

## 01. 자바스크립트 객체의 분류

- 자바스크립트 객체는 다음과 같이 크게 3가지로 분류 가능
  - 표준 빌트인 객체
    - ECMAScript 사양에 정의된 객체를 말하며, 애플리케이션 전역의 공통 기능을 제공
    - 자바스크립트 실행 환경(브라우저 또는 Node.js 환경)과 관계없이 사용 가능
    - 전역 객체의 프로퍼티로서 제공되므로 별도의 선언 없이 전역 변수처럼 언제나 참조 가능
  - 호스트 객체
    - ECMAScript 사양에 정의되어 있지 않지만 자바스크립트 실행환경에서 추가로 제공하는 객체
    - 브라우저 환경에서는 DOM, BOM, Canvas, `XMLHttpRequest`, `fetch`, `requestAnimationFrame`, SVG, Web Storage, Web Component, Web Worker와 같은 클라이언트 사이드 Web API를 호스트 객체로 제공하고, Node.js 환경에서는 Node.js 고유의 API를 호스트 객체로 제공
  - 사용자 정의 객체
    - 표준 빌트인 객체와 호스트 객체처럼 기본 제공되는 객체가 아닌 사용자가 직접 정의한 객체

<br>

## 02. 표준 빌트인 객체

- 자바스크립트는 `Object`, `String`, `Number`, `Boolean`, `Symbol`, `Date`, `Math` 등 40여 개의 표준 빌트인 객체를 제공
- `Math`, `Reflect`, `JSON`을 제외한 표준 빌트인 객체는 모두 인스턴스를 생성할 수 있는 생성자 함수 객체
- 생성자 함수 객체인 표준 빌트인 객체는 프로토타입 메서드와 정적 메서드를 제공하고 생성자 함수가 아닌 표준 빌트인 객체는 정적 메서드만 제공
- 생성자 함수의 표준 빌트인 객체가 생성한 인스턴스의 프로토타입은 표준 빌트인 객체의 prototype 프로퍼티에 바인딩된 객체

```js 
// String 생성자 함수에 의한 String 객체 생성
const strObj = new String('Lee');	// String {"Lee"}

// String 생성자 함수를 통해 생성한 strObj 객체의 프로토타입은 String.prototype
console.log(Object.getPrototypeOf(strObj) === String.prototype);	// true
```

- 표준 빌트인 객체의 prototype 프로퍼티에 바인딩된 객체는 다양한 기능의 빌트인 프로토타입 메서드를 제공
- 표준 빌트인 객체는 인스턴스 없이도 호출 가능한 빌트인 정적 메서드를 제공

<br>

## 03. 원시값과 래퍼 객체

- 원시값인 문자열, 숫자, 불리언 값의 경우 이들 원시값에 대해 마치 객체처럼 마침표 표기법(또는 대괄호 표기법)으로 접근하면 자바스크립트 엔진이 일시적으로 원시값을 연관된 객체로 변환
- 원시값을 객체처럼 사용하면 자바스크립트 엔진은 암묵적으로 연관된 객체를 생성하여 생성된 객체로 프로퍼치에 접근하거나 메서드를 호출하고 다시 원시값으로 되돌림
- 문자열, 숫자, 불리언 값에 대해 객체처럼 접근하면 생성되는 임시 객체를 래퍼 객체라고 함

```js
const str = 'hello';

// 원시 타입인 문자열이 프로퍼티와 메서드를 갖고 있는 객체처럼 동작
console.log(str.length);	// 5
console.log(str.toUpperCase());	// HELLO

// 래퍼 객체로 프로퍼티에 접근하거나 메서드를 호출한 후, 다시 원시값으로 되돌림
console.log(typeof str);	// string
```

```js
// 1) 식별자 str은 문자열을 값으로 가지고 있음
const str = 'hello';

// 2) 식별자 str은 암묵적으로 생성된 래퍼 객체를 가리킴
// 식별자 str의 값 'hello'는 래퍼 객체의 [[StringData]] 내부 슬롯에 할당
// 래퍼 객체에 name 프로퍼티가 동적 추가
str.name = 'Lee';

// 3) 식별자 str은 다시 원래의 문자열, 즉 래퍼 객체의 [[StringData]] 내부 슬롯에 할당된 원시값을 가짐
// 이때 2)에서 생성된 래퍼 객체는 아무도 참조하지 않는 상태이므로 가비지 컬렉션의 대상이 됨

// 4) 식별자 str은 새롭게 암묵적으로 생성된(2)에서 생성된 래퍼 객체와는 다른) 래퍼 객체를 가리킴
// 새롭게 생성된 래퍼 객체에는 name 프로퍼티가 존재하지 않음
console.log(str.name);	// undefined

// 5) 식별자 str은 다시 원래의 문자열, 즉 래퍼 객체의 [[StringData]] 내부 슬롯에 할당된 원시값을 가짐
// 이때 4)에서 생성된 래퍼 객체는 아무도 참조하지 않는 상태이므로 가비지 컬렉션의 대상이 됨
console.log(typeof str, str);	// string, hello
```

- 문자열, 숫자, 불리언, 심벌은 암묵적으로 생성되는 래퍼 객체에 의해 마치 객체처럼 사용할 수 있으며, 표준 빌트인 객체인 `String`, `Number`, `Boolean`, `Symbol`의 프로토타입 메서드 또는 프로퍼티를 참조할 수 있음
  - 따라서 `String`, `Number`, `Boolean` 생성자 함수를 `new` 연산자와 함께 호출하여 문자열, 숫자, 불리언 인스턴스를 생상할 필요가 없으며 권장하지 않음 (`Symbol`은 생성자 함수가 아님)
  - 이외의 원시값, 즉 `null`과 `undefined`는 래퍼 객체를 생성하지 않기 때문에 객체처럼 사용하면 에러가 발생

<br>

## 04. 전역 객체

- 코드가 실행되기 이전 단계에 자바스크립트 엔진에 의해 어떤 객체보다도 먼저 생성되는 특수한 객체이며, 어떤 객체에도 속하지 않는 모든 빌트인 객체(표준 빌트인 객체와 호스트 객체)의 최상위 객체
- 자바스크립트 환경에 따라 이름이 제각각이며, 브라우저 환경에서는 `window`(또는 `self`, `this`, `frames`)가 전역 객체를 가리키지만 Node.js 환경에서는 `global`이 전역 객체를 가리킴

- 전역 객체는 표준 빌트인 객체(`Object`, `String`, `Number`, `Function`, `Array` 등)와 환경에 따른 호스트 객체(클라이언트 Web API 또는 Node.js의 호스트 API), 그리고 `var` 키워드로 선언한 전역 변수와 전역 함수를 프로퍼티로 가짐

### 전역 객체의 특징

- 전역 객체는 전역 객체를 생성할 수 있는 생성자 함수가 제공되지 않기 때문에 개발자가 의도적으로 생성할 수 없음
- 전역 객체의 프로퍼티를 참조할 때 `window`(또는 `global`)를 생략할 수 있음

```js
// 문자열 'F'를 16진수로 해석하여 10진수로 변환
// window.parseInt는 그냥 parseInt로도 호출 가능
window.parseInt('F', 16);	// 15
parseInt('F', 16);		// 15

window.parseInt === parseInt;	// true
```

- 전역 객체는 모든 표준 빌트인 객체를 프로퍼티로 가지고 있음
- 자바스크립트 실행 환경에 따라 추가적으로 프로퍼티와 메서드를 가짐
- `var` 키워드로 선언한 전역 변수와 선언하지 않는 변수에 값을 할당한 암묵적 전역, 그리고 전역 함수는 전역 객체의 프로퍼티가 됨

```js
var foo = 1;
console.log(window.foo);	// 1

bar = 2;	// window.bar = 2
console.log(window.bar);	// 2

function baz() { return 3; }
console.log(window.baz());	// 3
```

- `let`이나 `const` 키워드로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니며, `let`이나 `const` 키워드로 선언한 전역 변수는 보이지 않는 개념적인 블록(전역 렉시컬 환경의 선언적 환경 레코드) 내에 존재

```js
let foo = 123;
console.log(window.foo);	// undefined
```

- 브라우저 환경의 모든 자바스크립트 코드는 하나의 전역 객체 `window`를 공유하며, 여러 개의 `script` 태그를 통해 자바스크립트 코드를 분리해도 하나의 전역 객체 `window`를 공유하는 것은 변함이 없음

### 빌트인 전역 프로퍼티

- 전역 객체의 프로퍼티로서, 애플리케이선 전역에서 사용하는 값을 제공

#### `Infinity`

- 무한대를 나타내는 숫자값

```js
console.log(window.Infinity === Infinity);	// true
console.log(typeof Infinity);	// number
```

#### `NaN`

- 숫자가 아님을 나타내는 숫자값이며, `Number.NaN` 프로퍼티와 같음

```js
console.log(window.NaN);	// NaN
console.log(typeof NaN);	// number
```

#### `undefined`

- 원시타입 `undefined`를 값으로 가짐

```js
console.log(window.undefined);	// undefined

var foo;
console.log(foo);	// undefined
console.log(typeof undefined);	// undefined
```

### 빌트인 전역 함수

- 애플리케이션 전역에서 호출할 수 있는 빌트인 함수로서 전역 객체의 메서드

#### `eval`

- 자바스크립트 코드를 나타내는 문자열을 인수로 전달받음
  - 전달받은 문자열 코드가 표현식이라면 `eval` 함수는 문자열 코드를 런타임에 평가하여 값을 생성
  - 전달받은 인수가 표현식이 아닌 문이라면 문자열 코드를 런타임에 실행
  - 문자열 코드가 여러 개의 문으로 이루어져 있다면 모든 문을 실행

```js
// 표현식인 문
eval('1 + 2');		// 3
// 표현식이 아닌 문
eval('var x = 5;');	// undefined
// eval 함수에 의해 런타임에 변수 선언문이 실행
console.log(x);		// 5

// 객체 리터럴, 함수 리터럴은 반드시 괄호로 둘러쌈
const o = eval('({a: 1})');
console.log(o);	// {a: 1}
const f = eval('(function() { return 1; })');
console.log(f());	// 1
```

- 자신이 호출된 위치에 해당하는 기존의 스코프를 런타임에 동적으로 수정

```js
const x = 1;

function foo() {
  // foo 함수 스코프에 선언된 x 변수를 동적으로 추가
  // strict mode에서 eval 함수는 기존의 스코프를 수정하지 않고 eval 함수 자신만의 자체적인 스코프를 생성함
  eval('var x = 2;');
  console.log(x);	// 2
}

foo();
console.log(x);	// 1
```

- `eval` 함수의 사용은 금지해야 함
  - `eval` 함수로 사용자로부터 입력받은 콘텐츠를 실행하는 것은 보안에 매우 취약
  -  `eval` 함수를 통해 실행되는 코드는 자바스크립트 엔진에 의해 최적화가 수행되지 않으므로 일반적인 코드 실행에 비해 처리 속도가 느림

#### `isFinite`

- 전달받은 인수가 정상적인 유한수인지 검사하여 유한수이면 `true`를 반환하고 무한수이면 `false`를 반환
- 전달받은 인수의 타입이 숫자가 아닌 경우, 숫자로 타입을 변환한 후 검사를 수행
  - 인수가 `NaN`로 평가되는 값이라면 `false` 반환
  - `null`은 숫자로 변환하면 0이기 때문에 `isFinite(null)`은 `true`를 반환

#### `isNaN`

- 전달받은 인수가 `NaN`인지 검사하여 그 결과를 불리언 타입으로 반환
- 전달받은 인수의 타입이 숫자가 아닌 경우 숫자로 변환한 후 검사를 수행

#### `parseFloat`

- 전달받은 문자열 인수를 부동 소수점 숫자, 즉 실수로 해석하여 반환
- 공백으로 구분된 문자열은 첫 번째 문자열만 변환하며, 첫 번째 문자열을 숫자로 변환할 수 없다면 `NaN`를 만환
- 전달받은 문자열 인수의 앞 뒤 공백은 무시
- 두번째 인수로 진법을 나타내는 기수(2~36)를 전달 가능
  - 기수를 지정하면 첫 번째 인수로 전달된 문자열을 해당 기수의 숫자로 해석하여 반환
- 두 번째 인수로 진법을 나타내는 기수를 지정하지 않더라도 첫 번째 인수로 전달된 문자열이 `"0x"` 또는 `"0X"`로 시작하는 16진수 리터럴이라면 16진수로 해석하여 10진수 정수로 반환
  - 2진수 리터럴과 8진수 리터럴은 제대로 해석하지 못함

#### `encodeURI` / `decodeURI`

- `encodeURI` 함수는 완전한 URI를 문자열로 전달받아 이스케이프 처리를 위해 인코딩
  - 이스케이프 처리? 네트워크를 통해 정보를 공유할 때 어떤 시스템에서도 읽을 수 있는 아스키 문자 셋으로 변환하는 것
- `decodeURI` 함수는 인코딩된 URI를 인수로 전달받아 이스케이프 처리 이전으로 디코딩

#### `encodeURIComponent` / `decodeURIComponent`

- `encodeURIComponent` 함수는 URI 구성 요소를 인수로 전달받아 인코딩
  - 인수로 전달된 문자열 URI의 구성요소인 쿼리 스트링의 일부로 간주하므로 구분자로 사용되는 =, ?, &까지 인코딩
- `decodeURIComponent` 함수는 매개변수로 전달된 URI 구성 요소를 디코딩

### 암묵적 전역

- 변수가 전역 객체의 프로퍼티가 되어 마치 전역 변수처럼 동작하는 것

```js
// 전역 변수 x는 호이스팅이 발생
console.log(x);	// undefined
// 전역 변수가 아니라 단지 젼역 객체의 프로퍼티인 y는 호이스팅이 발생하지 않음
console.log(y);

var x = 10;	// 전역 변수

function foo () {
  // 선언하지 않은 식별자에 값을 할당
  y = 20;	// window.y = 20;
}
foo();

// 선언하지 않은 식별자 y를 전역에서 참조할 수 있음
console.log(x + y);	// 30

delete x;	// 전역 변수는 삭제되지 않음
delete y;	// 프로퍼티는 삭제 가능

console.log(window.x);	// 10
console.log(window.y);	// undefined
```