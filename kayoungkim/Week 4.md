# 프로토타입
## 오버라이딩과 프로퍼티 섀도잉
``` javascript
//생성자 함수로 객체(인스턴스) 생성
const Person = (function() {
  function Person(name) {
    this.name = name;
}

//프로토타입 메서드 
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

//생성자 함수 반환
return Person;
}());

const me = new Persnon('Lee');

//인스턴스 메서드 추가
me.sayHello = function() {
  console.log(`Hey! My name is ${this.name}`);
};

// 인스턴스 메서드 호출! (프로토타입 메서드는 인스턴스 메서드에 의해 가려진다 = 프로퍼티 섀도잉)
me.sayHello(); // Hey! My name is Lee
```
- 프로토타입 프로퍼티와 같은 이름의 프로퍼티를 인스턴스에 추가하면 프로토타입 체인을 따라 프로토타입 프로퍼티를 검색하여 프로토타입 프로퍼티를 덮어쓰는 것이 아니라, 인스턴프 프로퍼티로 추가한다.
- 인스턴스 메서드는 프로토타입 메서드를 오버라이딩(상위 클래스 메서드를 하위 클래스가 재정의)하고, 프로토타입 메서드는 가려진다.
- **상속 관계에 의해 프로퍼티가 가려지는 현상을 프로퍼티 셰도잉(property shadowing)** 이라 한다.
### 프로퍼티 변경/삭제
- 프로토타입 프로퍼티를 변경/삭제하려면 프로토타입에 직접 접근해야 한다. (하위 객체 통할 수 없음)
``` javascript
//프로토타입 메서드 
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

// 프로퍼티 메서드 변경
me.sayHello(); // Hey! My name is Lee

// 프로퍼티 메서드 삭제
delete Person.prototype.sayHello;
me.sayHello();
```
## 프로토타입의 교체
### 생성자 함수에 의한 교체
``` javascript
//생성자 함수로 객체(인스턴스) 생성
const Person = (function() {
  function Person(name) {
    this.name = name;
}

//프로토타입 메서드 
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};
```
- 프로토타입으로 교체한 객체 리터럴에는 constructor 프로퍼티가 없다. me 객체의 생성자 함수를 검색하면 Person이 아닌 Object가 나온다. 
``` javascript
console.log(me.constructor === Person); // false
console.log(me.constructor === Object); // true
=> 프로토타입을 교체하면 생성자 함수 간의 연결이 파괴된다.
```
- constructor property - 생성자 함수 간의 연결을 되살리려면, constructor property를 추가한다.
``` javascript
//프로토타입 메서드 
Person.prototype = {
  constructor: Person,
  sayHello() {
  console.log(`Hi! My name is ${this.name}`);
};
```
### 인스턴스에 의한 교체
- 인서턴스의 __proto__ 접근자 또는 Object.setPrototypeOf 메서드를 통해 교체할 수 있다. 
`Object.setPrototypeOf(me, parent); // = me.__proto__ = parent;`
=> 프로토타입 교체를 통해 객체 간의 상속 관계를 동적으로 변경하는 것은 괘나 번거롭기 때문에 직접 교체하지 않는 것이 좋다. (직접 상속이 더 편리하고 안전하다)

## instanceof 연산자
`객체 instanceOf 생성자 함수`
- 생성자 함수 프로토타입에 바인딩된 객체가 좌변의 객체가 프로토타입 체인 상에 존재하는지 확인한다.(true/false)
``` javascript
const Person = (function() {
  function Person(name) {
    this.name = name;
}
console.log(me instanceof Person); //true
console.log(me instanceof Object); //true
```
## 직접 상속
### Object.create
**Object.create** 메서드는 명시적으로 프로토타입을 지정하여 새로운 객체를 생성한다.
- **OrdinaryObjectCreate**를 호출한다.
```Object.create(prototype[, propertiesObject])```
- 1) 생성할 객체의 프로토타입으로 지정할 객체 2) 생성할 객체의 프로퍼티 키와 프로퍼티 디스크립터 객체로 이뤄진 객체 전달
- 장점: new 연산자 없이 객체 생성 가능, 프로토타입을 지정하면서 객체 생성 가능, 객체 리터럴에 의해 생성된 객체도 상속받을 수 있다.
### 객체 리터럴 내부에서 __proto__에 의한 직접 상속
``` javascript
const myProto = {x:10}
const obj = {
 y: 20,
 __proto__:myProto
};
console.log(obj.x, obj.y); // 10 20
console.log(Object.getPrototypof(obj) === myProto); // true
```
## 정적 프로퍼티/메서드
- 정적(static) 프로퍼티/메서드는 생성자 함수로 인스턴스를 생성하지 않아도 참조/호출할 수 있다.
``` javascript
//생성자 함수로 객체(인스턴스) 생성
const Person = (function() {
  function Person(name) {
    this.name = name;
}

//프로토타입 메서드 
Person.prototype.sayHello = function () {
  console.log(`Hi! My name is ${this.name}`);
};

//정적 프로퍼티
Person.staticProp = 'static prop';
//정적 메서드
Person.staticMethod = function() {
console.log('staticMethod')
};

//생성자 함수에 추가한 정적 프로퍼티/메서드는 생성자 함수로 참조/호출한다.
const me = new Person('Lee')
//인스턴스로는 참조/호출할 수 없다.
Person.staticMethod(); //staticMethod
me.staticMethod(); //TypeError
```
## 프로퍼티 존재 확인
### in 연산자
- 객체 내에 특정 프로퍼티가 존재하는지 여부를 확인한다.
```
*key: 프로퍼티 키를 나타내는 문자열
*object: 객체로 평가되는 표현식
key in object
```
```javascript
const person = {
  name: 'Lee',
  address: 'Seoul'
};

console.log('name' in person);
console.log('address' in person);
console.log('age' in person);
```
- 확인 대상 객체가 상속받은 모든 프로토타입의 프로퍼티도 확인한다. ``` console.log('toString' in poerson); // true
- ES6 Reflect.has method도 같은 기능이다. 
### Object.prototype.hasOwnPropety method
- 인수로 전달받은 프로퍼티 키가 객체 고유의 프로퍼티 키인 경우에만 true, 상속받은 프로토타입의 프로퍼티 키인 경우 false 반환
``` 
console.log('toString' hasOwnProperty poerson); //false
console.log('name' hasOwnProperty person); //true
console.log('address' hasOwnProperty person); //true
```
## 프로퍼티 열거
### for..in 문
- 객체의 모든 프로퍼티를 순회하며 열거한다.
``` for (변수선언문 in 객체) {...}``` 
- 프로퍼티 키가 심벌인 프로퍼티는 열거하지 않는다. ([sym])
- for...in 문은 프로퍼티 열거 시 순서를 보장하지 않으므로 배열에서는 일반적인 for문이나 for..of 문, Array.prototype.forEach를 사용하기를 권장한다.
### Object.keys/values/entries method
-Object.keys : 객체 자신의 열거 가능한 프로퍼티 키를 배열로 반환한다.
-Object.values : 객체 자신의 열거 가능한 프로퍼티 값을 배열로 반환한다. (ES8)
-Object.entries : 객체 자신의 열거 가능한 프로퍼티 키와 값의 쌍의 배열을 배열에 담아 반환한다. (ES8)
```javascript
const person = {
  name: 'Lee',
  address: 'Seoul'
  __proto__: {age:20}
};

console.log(Object.keys(person)); // ["name", "address"]
console.log(Object.values(person)); // ["Lee", "Seoul"]
console.log(Object.entries(person)); // [["name", "Lee"], ["address", "Seoul"]]
Object.entries(person).forEach(([key, value]) => console.log(key, value))' // name Lee address Seoul
```


