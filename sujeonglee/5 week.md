# Chapter 21


### 빌트인 객체란?

자바스크립트는 Object, String, Number, Date, Math, RegExp, Array, Map, Set, Promise, Reflect, Proxy, JSON, Error 등 40여 개의 표준 빌트인 객체를 제공한다.

Math, Reflect, JSON 을 제외한 표준 빌트인 객체는 모두 인스턴스를 생성할 수 있는 **생성자 함수 객체**다.

> 객체의 종류
1. 표준 빌트인 객체 : ECMAScript 사양에 정의된 객체(Object, String, Data, Promise, JSON 등등)
2. 호스트 객체 : ECMAScript 사양에 정의되어 있지 않지만 자바스크립트 실행 환경에서 추가로 제공하는 객체
(브라우저는 DOM, BOM, Canvas, SVG 등등, Nodejs는 고유 API)
3. 사용자 정의 객체 : 사용자가 직접 정의한 객체

 

생성자 함수인 표준 빌트 객체가 생성한 인스턴스의 프로토타입은 표준 빌트인 객체의 prototype 프로퍼티에 바인딩된 객체다. 예를 들어, 표준 빌트인 객체인 String을 생성자 함수로서 호출하여 생성한 String 인스턴스의 프로토타입은 String.prototype이다.

```
// String 생성자 함수에 의한 String 객체 생성
const strObj = new String('Lee'); // String {"Lee"}

// String 생성자 함수를 통해 생성한 strObj 객체의 프로토타입은 String.prototype이다.
console.log(Object.getPrototypeOf(strObj) === String.prototype); // true
```

prototype 프로퍼티에 바인딩된 객체는 다양한 기능의 빌트인 프로토타입 메서드를 제공한다.

```
const numObj = new Number(1.5); // Number {1.5}

// 소수점 자리를 반올림하여 그 결과를 반환한다.
console.log(numObj.toFixed()); // 2

// 정수인지 검사하여 그 결과를 반환한다.
console.log(Number.isInteger(0.5)); // false
```

### 원시값과 래퍼 객체
원시값인 문자열, 숫자, 불리언 값의 경우 이들 원시값에 대해 마치 객체처럼 마침표 표기법(또는 대괄호 표기법)으로 접근하면 자바스크립트 엔진이 일시적으로 원시값을 연관된 객체로 변환해 준다.

이처럼 문자열, 숫자, 불리언 값에 대해 객체처럼 접근하면 생성되는 임시 객체를 래퍼 객체라 한다.

String.prototype의 메서드를 상속받아 사용할 수 있다.

```
const str = 'hello';

console.log(str.length); // 5
console.log(str.toUpperCase()); // HELLO

console.log(typeof str); // string
```
> 자바스크립트 엔진은 원시값을 객체처럼 사용하면 암묵적으로 연관된 객체를 생성해서, 생성된 객체로 프로퍼티에 접근하거나 메서드를 호출하고 다시 원시 값으로 되돌린다.

문자열, 숫자, 불리언 값에 대해 객체처럼 접근하면 생성되는 임시 객체를 래퍼 객체(wrapper object) 라 한다.

이처럼 래퍼 객체에 의해 문자열, 숫자, 불리언은 마치 객체처럼 사용할 수 있고 표준 빌트인 객체의 프로토타입 메서드 또는 프로퍼티를 참조할 수 있다. 그렇기 때문에 String, Number, Boolean 생성자 함수를 new 연산자와 함께 호출하여 문자열, 숫자, 불리언 인스턴스를 생성할 필요가 없으며 권장하지도 않는다.

 ### 전역객체

전역 객체는 표준 빌트인 객체와 환경에 따른 호스트 객체 그리고 var 키워드로 선언한 전역 변수와 전역 함수를 프로퍼티로 갖는다. 즉, 전역 객체는 모든 빌트인 객체의 최상위 객체다. 전역 객체는 어떤 객체의 프로퍼티도 아니며 객체의 계층적 구조상 표준 빌트인 객체와 호스트 객체를 프로퍼티로 소유한다는 것을 말한다.

 

var 키워드로 선언한 전역 변수와 선언하지 않은 변수에 값을 할당한 암묵적 전역, 그리고 전역 함수는 전역 객체의 프로퍼티가 된다. let, const로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니다.

 

- 빌트인 전역 프로퍼티 : Infinity, NaN, undefined

- 빌트인 전역 함수 : eval, isFinite, isNaN, parseFloat, parseInt, encodeURI, decodeURI

 

- parseint(숫자,진수) 로 n진법 =>10진법 반환

- Number.prototype.toString(진수)으로 10진법 => n진법변환

 

인코딩이란 URI의 문자들을 이스케이프 처리하는 것을 의미한다. 이스케이프 처리는 네트워크를 통해 정보를 공유할 때 어떤 시스템에서도 읽을 수 있는 아스키 문자 셋 으로 변환하는 것이다. URL 문법에 따르면 한글을 포함한 대부분의 외국어나 특수 문자는 URL에 포함될 수 없다. 이 때 이스케이프 처리가 필요하다.

# Chapter 22

### this

바인딩 : 식별자와 값을 연결하는 과정.

객체 리터럴의 메서드 내부에서의 this는 메서드를 호출한 객체를 가리킨다.

생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.

자바나 C++같은 클래스 기반 언어에서 this는 언제나 클래스가 생성하는 인스턴스를 가리킨다. 하지만 자바스크립트의 this는 함수가 호출되는 방식에 따라 this에 바인딩될 값, 즉 this 바인딩이 동적으로 결정된다.

```
// this는 어디서든지 참조 가능하다.
// 전역에서 this는 전역 객체 window를 가리킨다.
console.log(this); // window

function square(number) {
  // 일반 함수 내부에서 this는 전역 객체 window를 가리킨다.
  console.log(this); // window
  return number * number;
}
square(2);

const person = {
  name: 'Lee',
  getName() {
    // 메서드 내부에서 this는 메서드를 호출한 객체를 가리킨다.
    console.log(this); // {name: "Lee", getName: ƒ}
    return this.name;
  }
};
console.log(person.getName()); // Lee

function Person(name) {
  this.name = name;
  // 생성자 함수 내부에서 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
  console.log(this); // Person {name: "Lee"}
}

const me = new Person('Lee');
```
this 바인딩은 함수 호출 방식, 즉 함수에 어떻게 호출되었는지에 따라 동적으로 결정된다.

렉시컬 스코프 : 함수를 어디서 정의했는지에 따라 함수의 상위 스코프를 결정 (자바스크립트 포함 대부분 언어들)

 
> 
*화살표 함수는 자신의 this가 없습니다.  대신 화살표 함수를 둘러싸는 렉시컬 범위(lexical scope)의 this가 사용됩니다; 화살표 함수는 일반 변수 조회 규칙(normal variable lookup rules)을 따릅니다. 때문에 현재 범위에서 존재하지 않는 this를 찾을 때, 화살표 함수는 바로 바깥 범위에서 this를 찾는것으로 검색을 끝내게 됩니다.  

 

함수를 호출하는 방식

- 1. 일반 함수 호출
- 2. 메서드 호출
- 3. 생성자 함수 호출
- 4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출

메서드 내에서 정의한 중첩 함수도 일반 함수로 호출되면 중첩 함수 내부의 this에는 전역 객체가 바인딩된다.

콜백 함수가 일반 함수로 호출된다면 콜백함수 내부의 this에도 전역 객체가 바인딩된다. 어떠한 함수라도 일반 함수로 호출되면 this에 전역 객체가 바인딩된다.

메서드 내부의 중첩 함수나 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키려면

this를 변수에 미리 할당시키거나 apply,call,bind 메서드 사용, 화살표 함수 사용하자.

 

메서드 내부의 this는 프로퍼티로 메서드를 가리키고 있는 객체와는 관계가 없고 메서드를 호출한 객체에 바인딩된다.

즉 메서드는 객체에 포함된 것이 아니라 독립적으로 존재하는 별도의 객체다.

### 메서드 호출
메서드 내부의 this는 메서드를 소유한 객체가 아닌 메서드를 호출한 객체에 바인딩된다.
```
const person = {
  name: 'Lee',
  getName() {
    return this.name;
  }
};

console.log(person.getName()); // Lee
```
메서드는 프로퍼티에 바인딩된 함수다. person 객체의 getName 프로퍼티가 가리키는 함수 객체는 person 객체에 포함된 것이 아니라 독립적으로 존재하는 별도의 객체다. getName 프로퍼티가 함수 객체를 가리키고 있을 뿐이다.

그렇기 때문에 getName 프로퍼티가 가리키는 함수 객체인 getName 메서드는 다른 객체의 프로퍼티에 할당하는 것으로 다른 객체의 메서드가 될 수도 있고, 일반 변수에 할당해 일반 함수로 호출될 수도 있다.

```
const person = {
  name: 'Lee',
  getName() {
    return this.name;
  }
};

const anotherPerson = {
  name: 'Kim'
};
// anotherPerson 객체에 할당
anotherPerson.getName = person.getName;

console.log(anotherPerson.getName()); // Kim

// 변수에 할당
const getName = person.getName;

console.log(getName()); // ''

```
getName 메서드를 anotherPerson 객체의 메서드로 할당하고, anotherPerson 객체의 getName 메서드를 호출하면 결과는 'Lee'가 아닌 'Kim'이 나온다.

또한 일반 함수로 호출된 getName 함수 내부의 this.name은 브라우저 환경에서 window.name과 같다. 브라우저 환경에서 window.name은 브라우저 창의 이름을 나타내고 기본값은 ''이다.

이처럼 메서드 내부의 this는 프로퍼티로 메서드를 가리키고 있는 객체와는 관계가 없고, 메서드를 호출한 객체에 바인딩된다.

### 생성자 함수 호출
생성자 함수 내부의 this에는 생성자 함수가 생성할 인스턴스가 바인딩된다.
```
function Circle(radius) {
  this.radius = radius;
  this.getDiameter = function () {
    return 2 * this.radius;
  };
}

// 생성자 함수로 호출
const circle1 = new Circle(5);
console.log(circle1.getDiameter()); // 10

// 일반 함수로 호출
const circle2 = Circle(10);
console.log(circle2); // undefined

```
new 연산자와 함께 호출하면 해당 함수는 생성자 함수로 동작하고, new 연산자와 함께 생성자 함수를 호출하지 않으면 일반 함수로 동작한다. 일반 함수로 동작한다면 Circle 함수에는 반환문이 없으므로 암묵적으로 undefined를 반환한다.
```
function getThisBinding() {
  return this;
}

// this로 사용할 객체
const thisArg = { a: 1 };

console.log(getThisBinding()); // window

console.log(getThisBinding.apply(thisArg)); // { a: 1 }
console.log(getThisBinding.call(thisArg)); // { a: 1 }

```

### apply/call/bind()를 통한 호출

apply, call, bind 메서드는 Function.prototype 메서드이기 때문에 이들 메서드들은 모든 함수가 상속받아 사용할 수 있다.

apply, call 메서드는 this로 사용할 객체를 받아 함수를 호출한다.

getThisBinding 함수를 호출하면서 인수로 전달한 객체를 getThisBinding 함수의 this에 바인딩한다.

이와 달리 bind 메서드는 함수를 호출하지 않고 this로 사용할 객체만 전달한다.
bind 메서드는 함수에 this로 사용할 객체만을 전달하기 때문에, 함수를 호출하지는 않아, 에 명시적으로 호출해서 사용해야 한다.

### 함수 호출 방식 정리

|함수 호출 방식|this 바인딩|
|:------|:---|
|일반 함수 호출|전역 객체|
|메서드 호출|메서드를 호출한 객체|
|생성자 함수 호출|생성자 함수가 생성할 인스턴스|
|apply/call/bind()에 의한 호출|apply/call/bind() 메서드에 인수로 전달한 객체|
