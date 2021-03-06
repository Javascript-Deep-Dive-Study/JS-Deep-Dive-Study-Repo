## 13장 스코프

> 스코프란?
>
> 스코프, 즉 유효범위는 자바스크립트를 포함한 모든 프로그래밍 언어의 기본적이며 중요한 개념이다. 자바스크립트의 스코프는 다른 언어의 스코프와 구별되는 특징이 있으므로 주의가 필요하다. 그리고 var 키워드로 선언한 변수와 let 또는 const 키워드로 선언한 변수의 스코프도 다르게 동작한다. 스코프는 변수 그리고 함수와 깊은 관련이 있다.



```javascript
function add(x, y) {
	// 매개변수는 함수 몸체 내부에서만 참조할 수 있다.
  // 즉, 매개변수의 스코프(유효범위)는 함수 몸체 내부다.
  console.log(x, y); // 2 5
  return x + y;
}
```



```javascript
var var1 = 1; // 코드의 가장 바깥 영역에서 선언한 변수

if (true) {
	var var2 = 2; // 코드 블록 내에서 선언한 변수
  if (true) {
    var var3 = 3; // 중첩된 코드 블록 내에서 선언한 변수
  }
}

function foo() {
  var var4 = 4; // 함수 내에서 선언한 변수
  
  function bar() {
    var var5 = 5; // 중첩된 함수 내에서 선언한 변수
  }
}

console.log(var1); // 1
console.log(var2); // 2
console.log(var3); // 3
console.log(var4); // ReferenceError: var4 is not defined
console.log(var5); // ReferenceError: var5 is not defined
```

> 자자..변수는 자신이 선언된 위치에 의해 자신이 유효한 범위, 즉 다른 코드가 변수 자신을 참조할 수 있는 범위가 결정된다. 변수뿐만 아니라 모든 식별자가 그렇다. 다시 말해, **모든 식별자(변수 이름, 함수 이름, 클래스 이름 등)는 자신이 선언된 위치에 의해 다른 코드가 식별자 자신을 참조할 수 있는 유효 범위가 결정된다. 이를 스코프라 한다. 즉, 스코프는 식별자가 유효한 범위를 말한다.**

```javascript
var x = 'global';

function foo() {
  var x = 'local';
  console.log(x); // ???
}

foo();

console.log(x); // ???
```

> ❗"코드가 어디서 실행되며 주변에 어떤 코드가 있는지"를 **렉시컬 환경(lexical environment)**이라고 부른다. 즉, 코드의 문맥(context)은 렉시컬 환경으로 이뤄진다. 이를 구현한 것이 "**실행 컨텍스트(execution context)**"이며, 모든 코드는 실행 컨텍스트에서 평가되고 실행된다. 스코프는 실행 컨텍스트와 깊은 관련이 있다.
>
> **만약, 스코프라는 개념이 없다면 같은 이름을 갖는 변수는 충돌을 일으키므로 프로그램 전체에서 하나밖에 사용할 수 없다.**



> ❗"위험한 var", var 키워드로 선언한 변수의 중복 선언
>
> var 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언이 허용된다. 이는 의도치 않게 변수값이 재할당되어 변경되는 부작용을 발생시킨다.
>
> ```javascript
> function foo() {
> 	var x = 1;
> // var 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언을 허용한다.
> // 아래 변수 선언문은 자바스크립트 엔진에 의해 var 키워드가 없는 것처럼 동작한다.
> var x =2;
> console.log(x); // 2
> }
> foo();
> ```



* #### 스코프의 종류

  코드는 전역(global)과 지역(local)으로 구분할 수 있다.

  

  | 구분 | 설명                  | 스코프      | 변수      |
  | ---- | --------------------- | ----------- | --------- |
  | 전역 | 코드의 가장 바깥 영역 | 전역 스코프 | 전역 변수 |
  | 지역 | 함수 몸체 내부        | 지역 스코프 | 지역 변수 |

  ![스코프1](https://user-images.githubusercontent.com/79819941/127670952-d855ae6d-5245-4f75-a4b2-a6c03947a838.png)

* #### **전역이란** be discharged from military service가 아니라 **코드의 가장 바깥 영역**을 말하며, 전역 스코프(global scope)를 만든다. 전역에 변수를 선언하면 전역 스코프를 갖는 전역 변수(global variable)가 된다. 그렇기 때문에 **전역 변수는 어디서든지 참조할 수 있다.**

* #### **지역이란** **함수 몸체 내부**를 말한다. 지역은 지역 스코프(local scope)를 만든다. 지역에 변수를 선언하면 지역 스코프를 갖는 지역 변수(local variable)가 된다. 지역 변수는 자신이 선언된 지역과 하위 지역(중첩 함수)에서만 참조할 수 있다. 즉, **지역 변수는 자신의지역 스코프와 하위 지역 스코프에서 유효하다.** 

* #### 스코프 체인(scope chain)

  알다시피, 함수는 함수안의 함수, 즉 중첩될 수 있으므로 함수의 지역 스코프도 중첩될 수 있다. 이는 **스코프가 함수의 중첩에 의해 계층적 구조를 갖는다는 것을 의마**한다. 

  모든 스코프는 하나의 계층적 구조로 연결되며, 모든 지역 스코프의 최상위 스코프는 전역 스코프이다. **스코프가 계층적으로 연결된 것**을 바로바로바로~~~ 스코프 체인(scope chain)이라고 한다.

![Screen Shot 2021-07-30 at 11 35 46 PM](https://user-images.githubusercontent.com/79819941/127670924-fb3144d1-30c4-49c3-9479-97be26861ad4.png)

​	위 그림처럼 스코프 체인은 최상위 스코프인 전역 스코프, 전역에서 선언된 outer 함수의 지역 스코프,outer 내부에서 선언된 inner 	함수의 지역 스코프로 이뤄진다.

​	변수를 참조할 때 자바스크립트 엔진은 스코프 체인을 통해 변수를 참조하는 코드의 스코프에서 시작하여 상위 스코프 방향으로 이동하면서 선언된 변수를 검색(Identifier resolution)한다. 이 행동으로 인해 상위 스코프에서 선언한 변수를 하위 스코프에서도 참조가 가능하다.

> ❗렉시컬 환경(Lexical Environment)
>
> "코드가 어디서 실행되며 주변에 어떤 코드가 있는지"를 렉시컬 환경(Lexical environment)이라고 부른다. 즉, 코드의 문맥(context)은 렉시컬 환경으로 이뤄진다. 이를 구현한 것이 "실행 컨텍스트(excution context)"이며, 모든 코드는 실행 컨텍스트에서 평가되고 실행된다.
>
> 스코프 체인은 실행 컨텍스트의 렉시컬 환경을 단방향으로 연결(chaining)한 것이다. 전역 렉시컬 환경은 코드가 로드되면 곧바로 생성되고 함수의 렉시컬 환경은 함수가 호출되면 곧바로 생성된다.

* #### 정리 

  **상위 스코프에서 유효한 변수는 하위 스코프에서 자유롭게 참조할 수 있지만 하위 스코프에서 유요한 변수를 상위 스코프에서 참조할 수 없다.**



* #### 스코프 체인에 의한 함수 검색

```javascript
// 전역 변수
function foo() {
  console.log('global function foo');
}

function bar() {
  // 중첩 함수
  function foo() {
    console.log('local function foo')
  }
  foo(); // (1)
}
bar();

# 함수 선언문으로 함수를 정의하면 런타임 이전에 함수 객체가 먼저 생성된다. 그리고 자바스크립트 엔진은 함수 이름과 동일한 이름의 식별자를 암묵적으로 선언하고 생성된 함수 객체를 할당한다. 따라서 위 예제의 모든 함수는 함수 이름과 동일한 이름의 식별자에 할당된다. (1)에서 foo 함수를 호출하면 자바스크립트 엔진은 함수를 호출하기 위해 먼저 함수를 가리키는 식별자 foo를 검색한다.
```



* #### 함수 레벨 스코프 

  > **var 키워드로 선언한 변수**는 **오로지 함수의 코드 블록(함수 몸체)만을 지역 스코프로 인정**하며, 이러한 특성을 함수 레벨 스코프(function level scope)라 한다.

```javascript
var x = 1;

if (true) {
  // var 키워드로 선언된 변수는 함수의 코드 블록(함수 몸체)만을 지역 스코프로 인정한다.
  // 함수 밖에서 var 키워드로 선언된 변수는 코드 블록 내에서 선언되었다 할지라도 모두 전역 변수이다.
  // 따라서 x는 전역 변수다. 이미 선어된 전역 변수 x가 있으므로 x 변수는 중복 선언된다.
  // 이는 의도치 않게 변수 값이 변경되는 부작용을 발생시킨다.
  var x = 10;
}

console.log(x); // 10
```

```javascript
# 블록 레벨 스코프를 지원하는 프로그래밍 언어에서는 for 문에서 반복을 위해 선언된 i 변수가 for 문의 코드 블록 내에서만 유효한 지역 변수이다. 이 변수를 for 문 외부에서 사용할 일은 없기 때문이다. 하지만 var 키워드로 선언된 변수는 블록 레벨 스코프를 인정하지 않기 때문에 i 변수는 전역 변수가 된다. 따라서 전역 변수 i는 중복 선언되고 그 결과 의도치 않은 전역 변수의 값이 재할당된다.

var i = 10;
// for 문에서 선언한 i는 전역 변수다. 이미 선언된 전역 변수 i가 있으므로 중복 선언된다.
for (var i = 0; i < 5; i++) {
  console.log(i); // 0 1 2 3 4
}
// 의도치 않게 변수의 값이 변경되었다.
console.log(i); // 5
```



* #### 렉시컬 스코프

  다음 예제의 실행 결과를 예측해보자.

  ```javascript
  var x = 1;
  
  function foo() {
    var x = 10;
    bar();
  }
  
  function bar() {
    console.log(x);
  }
  
  foo(); // ?
  bar(); // ?
  ```

  

## 14장 전역 변수의 문제점

* #### 지역 변수의 생명 주기

  변수는 선언에 의해 생성되고 할당을 통해 값을 갖는다. 그리고 언젠가 소멸한다. 즉, 변수는 생물과 유사하게 생성되고 소멸되는 생명 주기(life cycle)가 있다. 변수에 생명 주기가 없다면 한번 선언된 변수는 프로그램을 종료하지 않는 한 영원히 메모리 공간을 점유하게 된다.

  변수는 자신이 선언된 위치에서 생성되고 소멸한다. **전역 변수의 생명 주기는 애플리케이션의 생명 주기와 같다.** 하지만 **함수 내부에서 선언된 지역 변수**는 **함수가 호출되면 생성되고 함수가 종료하면 소멸**한다. 

  그리고, **지역 변수**의 **생명 주기는 함수의 생명 주기와 일치**한다.

  ```javascript
  function foo() {
    // 변수 x의 생명 주기
    var x = 'local';
    console.log(x); // local
    return x;
  	// 변수 x 소멸
  }
  
  foo();
  console.log(x); // ReferenceError: x is not defined
  # 지역 변수 x는 foo 함수가 호출되기 이전까지는 생성되지 않는다. foo 함수를 호출하지 않으면 함수 내부의 변수 선언문이 실행되지 않기 때문이다.
  ```

  

  다음 예제의 (1)에서 출력되는 값은 무엇인가?

  ```javascript
  var x = 'global';
  
  function foo() {
    console.log(x); // (1)
    var x = 'local';
  }
  
  foo();
  console.log(x); // global
  
  # 호이스팅은 위 예제처럼 스코프를 단위로 동작한다. 즉 호이스팅은 변수 선언이 스코프의 선두로 끌어 올려진 것처럼 동작하는 자바스크립트 고유의 특징을 말한다.
  ```

* #### 전역 변수의 생명 주기

  함수와는 달리 전역 코드는 명시적인 호출 없이 실행된다. 다시 말해, 전역 코드는 함수 호출과 같이 전역 코드를 실행하는 특별한 진입점이 없고 코드가 로드되자마자 곧바로 해석되고 실행된다. 함수는 함수 몸체의 마지막 문 또는 반환문이 실행되면 종료한다. 하지만 전역 코드에는 반환문을 사용할 수 없으므로 마지막 문의 실행되어 더 이상 실행할 문이 없을 때 종료한다.

> ❗전역 객체(global object)
>
> 전역 객체는 코드가 실행되기 이전 단계에 자바스크립트 엔진에 의해 어떤 객체보다도 먼저 생성되는 특수한 객체다. 전역 객체는 클라이언트 사이드 환경(브라우저)에서는 window, 서버 사이드 환경(Node.js)에서는 global 객체를 의미한다. 환경에 따라 전역 객체를 가리키는 다양한 식별자(window, self, this, frames, global)가 존재했으나 ES11(ECMAScript 11)에서 globalTHis로 통일되었다.
>
> 전역 객체는 표준 빌트인 객체(Object, String, Number, Function, Array...)와 환경에 따른 호스트 객체(클라이언트 W eb API 또는 Node.js의 호스트 API), 그리고 var 키워드로 선언한 전역 변수와 전역 함수를 프로퍼티로 갖는다.

브라우저 환경에서 전역 객체는 window이므로 브라우저 환경에서 var 키워드로 선언한 전역 변수는 전역 객체 window의 프로퍼티이다. 전역 객체 window는 웹페이지를 닫을 때까지 유효하다. 즉, **var 키워드로 선언한 전역 변수의 생명 주기는 전역 객체의 생명 주기와 일치한다.**

```javascript
// 전역 변수 x의 생명 주기
var x = 'global';   // 전역 변수 x 생성 및 값 할당

// 지역 변수 x의 생명 주기
function foo() { 
  var x = 'local'; // 지역 변수 x 생성 및 값 할당
  console.log(x);
  return x; // 지역 변수 x 소멸
}
// 지역 변수 x의 생명 주기

foo();
console.log(x);
// 전역 변수 x의 생명 주기
```

* #### 전역 변수의 문제점

  - **암묵적 결합**

    전역 변수를 선언한 의도는 전역, 즉 코드 어디서든 참조하고 할당할 수 있는 변수를 사용하겠다는 것이다.

    이는 모든 코드가 전역 변수를 참조하고 변경할 수 있는 암묵적 결합(Implicit coupling)을 허용하는 것이다.

    ***변수의 유효 범위가 크면 클수록 코드의 가독성은 나빠지고 의도치 않게 상태가 변경될 수 있는 위험성도 높아진다.***

  - #### 긴 생명 주기

    전역 변수는 생명 주기가 길다. 그렇기 때문에 메모리 리소스도 오랜 기간 소비한다. 그리고 전역 변수의 상태를 변경할 수 있는 시간도 길도 기회도 많다.

  - #### 스코프 체인 상에서 종점에 존재

  - #### 네임스페이스 오염

* #### 전역 변수의 사용을 억제하는 방법

  전역 변수의 무분별한 사용은 위험하다. **전역 변수를 반드시 사용해야 할 이유를 찾지 못한다면 지역 변수를 사용해야 한다. 변수의 스코프는 좁을수록 좋다.** 

  ```javascript
  # 즉시 실행 함수 
  // 함수 정의와 동시에 호출되는 즉시 실행 함수는 단 한 번만 호출된다. 모든 코드를 즉시 실행 함수로 감싸면 모든 변수는 즉시 실행 함수의 지역 변수가 된다.
  
  (function () {
    var foo = 10; // 즉시 실행 함수의 지역 변수
    // ...
  }());
  
  console.log(foo); // ReferenceError: foo is not defined
  ```

  ```javascript
  # 네임스페이스 객체
  // 1. 전역에 네임스페이스 역할을 담당할 객체를 생성하고 전역 변수처럼 사용하고 싶은 변수를 프로퍼트로 추가하는 방법
  var MYAPP = {}; // 전역 네임스페이스 객체
  MYAPP.name = 'Lee';
  
  console.log(MYAPP.name); // Lee
  
  // 2. 네임스페이스 객체에 또 다른 네임스페이스 객체를 프로퍼티로 추가해서 네임스페이스를 계층적으로 구성하는 방법
  var MYAPP = {}; // 전역 네임스페이스 객체
  
  MYAPP.person = {
    name: 'Lee',
    address: 'Seoul',
  };
  
  console.log(MYAPP.person.name); // Lee
  # 네임스페이스를 분리해서 식별자 충돌을 방지하는 효과는 있으나 네임스페이스 객체 자체가 전역 변수에 할당되므로 그다지 유용해 보이지는 않는다.
  ```

  ```javascript
  # 모듈 패턴
  // 모듈 패턴은 전역 네임스페이스의 오염을 막는 기능은 물론 한정적이기는 하지만 정보 은닉을 구현하기 위해 사용한다.
  var Counter = (function () {
    // private 변수
    var num = 0;
    
    // 외부로 공개할 데이터나 메서드를 프로퍼티로 추가한 객체를 반환한다.
    return {
      increase() {
        return ++num;
      },
      decrease() {
        return --num;
      }
    };
  }());
  
  // private 변수는 외부로 노출되지 않는다.
  console.log(Counter.num); // undefined
  
  console.log(Counter.increase()); // 1
  console.log(Counter.increase()); // 2
  console.log(Counter.decrease()); // 1
  console.log(Counter.decrease()); // 0
  
  # 위 예제의 즉시 실행 함수는 객체를 반환한다. 이 객체에는 외부에 노출하고 싶은 변수나 함수를 담아 반환한다. 이때 반환되는 객체의 프로퍼티는 외부에 노출되는 퍼블릭 멤버(public member)이다. 외부로 노출하고 싶지 않은 변수나 함수는 반환하는 객체에 추가하지 않으면 외부에서 접근할 수 없는 프라이빌 멤버(private member)가 된다.
  ```

  ```javascript
  # ES6 모듈
  // ES6 모듈을 사용하면 더는 전역 변수를 사용할 수 없다. ES6 모듈은 파일 자체의 독자적인 모듈 스코프를 제공한다. 그렇기 때문에 모듈 내에서 var 키워드로 선언한 변수는 더는 전역 변수가 아니며 window 객체의 프로퍼티도 아니다.
  <script type="module" src="lib.mjs"></script>
  <script type="module" src="app.mjs"></script>
  ```

  

## 15. let, const 키워드와 블록 레벨 스코프

####  **var 키워드로 선언한 변수의 문제점**

 ```javascript
# 변수 중복 선언 허용
var x = 1;
var y = 1;
// var 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언을 허용한다.
// 초기화문의 있는 변수 선어문은 자바스크립트 엔진에 의해 var 키워드가 없는 것처럼 동작한다.
var x = 100;
// 초기화문이 없는 변수 선언문은 무시된다.
var y;

console.log(x); // 100
console.log(y); // 1

// 위 예제의 var 키워드로 선언한 x 변수와 y 변수는 중복 선언되었다. 이처럼 var 키워드로 선언한 변수를 중복 선언하면 초기화문(변수 선언과 동시에 초기값을 할당하는 문) 유무에 따라 다르게 동작한다. 초기화문이 있는 변수 선언문은 자바스크립트 엔진에 의해 var 키워드가 없는 것처럼 동작하고 초기화문이 없는 벼수 선언문은 무시된다. 그리고....에러는 발생하지 않는다.
 ```

```javascript
# 함수 레벨 스코프
// var 키워드로 선언한 변수는 오직 함수의 코드 블록만을 지역 스코프로 인정한다. 따라서 함수 외부에서 var 키워드로 선언한 변수는 코드 블록 내에서 서넌해도 모두 전역 변수가 된다.
var x = 1;

if (true) {
  // x는 전역 변수다. 이미 선언된 전역 변수 x가 있으므로 x 변수는 중복 선언된다.
  // 이는 의도치 않게 변수값이 변경되는 부작용을 발생시킨다. 
  var x = 10;
}

console.log(x); // 10

// for 문의 변수 선언문에서 var 키워드로 선언한 변수도 전역 변수가 된다.
var i = 10;
// for 문에서 선언한 i는 전역 변수다. 이미 선언된 전역 변수 i가 있으므로 중복 선언된다.
for (var i = 0; i < 5; i++) {
  console.log(i); // 0 1 2 3 4
}

// 의도치 않게 i 변수의 값이 변경되었다.
console.log(i); // 5
```

```javascript
# 변수 호이스팅
// var 키워드로 변수를 선언하면 변수 호이스팅에 의해 변수 선언문이 스코프의 선두로 끌어 올려진 것처럼 동작한다. 즉, 변수 호이스팅에 의해 var 키워드로 선언한 변수는 변수 선언문 이전에 참조할 수 있다. 단, 할당문 이전에 변수를 참조하면 언제나 undefined를 반환한다.

// 이 시점에는 변수 호이스팅에 의해 이미 foo 변수가 선언되었다(1. 선언 단계)
// 변수 foo는 undefined로 초기화된다(2. 초기화 단계)
console.log(foo); // undefined

// 변수에 값을 할당(3. 할당 단계)
foo = 123;

console.log(foo); // 123

// 변수 선언은 런타임 이전에 자바스크립트 엔진에 의해 암묵적으로 실행된다.
var foo;
```

#### let 키워드

Var 키워드의 단점을 보완하기 위해 ES6에서 새로운 변수 선언 키워드인 let과 const를 도입했다.

```javascript
# 변수 중복 선언 금지
var foo = 123;
// var 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언을 허용한다.
// 아래 변수 선언문은 자바스크립트 엔진에 의해 var 키워드가 없는 것처럼 동작한다.
var foo = 456;

let bar = 123;
// let이나 const 키워드로 선언된 변수는 같은 스코프 내에서 중복 선언을 허용하지 않는다.
let bar = 456; // SyntaxError: Identifier 'bar' has already been declared
```

```javascript
# 블록 레벨 스코프
// var 키워드로 선언한 변수는 오로지 함수의 코드 블록만을 지역 스코프로 인정하는 함수 레벨 스코프를 따른다. 하지만 let 키워드로 선언한 변수는 모든 코드 블록(함수, if 문, for 문, while 문, try/catch 문 등)을 지역 스코프로 인정하는 블록 레벨 스코프(block-level-scope)를 따른다.

let foo = 1; // 전역 변수

{
  let foo = 2; // 지역 변수
  let bar = 3; // 지역 변수
}

console.log(foo); // 1
console.log(bar); // ReferenceError: bar is not defined
```

```javascript
let i = 10; 										   // 전역 스코프

function foo() {                   // 함수 레벨
  let i = 100; 										 // 스코프
  
  for (let i = 1; i < 3; i++) {    //  블록
    console.log(i); // 1 2         //  레벨
  }   														 //  스코프
  
  console.log(i); // 100					 // 함수 레벨
}																	 // 스코프
foo();														 // 전역 스코프

console.log(i); // 10							 // 전역 스코프
```

```javascript
# 변수 호이스팅 Var
console.log(foo); // ReferenceError: foo is not defined
let foo;

// var 키워드로 선언한 변수는 런타임 이전에 선언 단계와 초기화 단계가 실행된다.
// 따라서 변수 선언문 이전에 변수를 참조할 수 있다.
console.log(foo); // undefined

var foo;
console.log(foo); // undefined

foo = 1; // 할당문에서 할당 단계가 실행된다.
console.log(foo); // 1

# 변수 호이스팅 Let
// 런타임 이전에 선언 단계가 실행된다. 아직 변수가 초기화되지 않았다.
// 초기화 이전의 일시적 사각지대에서는 변수를 참조할 수 없다.
console.log(foo); // ReferenceError: foo is not defined

let foo; // 변수 선언문에서 초기화 단계가 실행된다.
console.log(foo); // undefined

foo = 1; // 할당문에서 할당 단계가 실행된다.
console.log(foo); // 1
```

> #### ❗결론
>
> 자바스크립트는 ES6에서 도입된 let, const를 포함해서 모든 선언(var, let, const, function, function*, class 등)을 호이스팅한다. 단, ES6에서 도입된 let, const, class를 사용한 선언문은 호이스팅이 발생하지 않는 것처럼 동작한다.

```javascript
# const 키워드
// const 키워드로 선언한 변수는 반드시 선언과 동시에 초기화해야 한다.
const foo = 1;

const foo; // SyntaxError: Missing initializer in const declaration

{
  // 변수 호이스팅이 발생하지 않는 것처럼 동작한다.
  console.log(foo); // ReferenceError: Cannot access 'foo' before initialization
  const foo = 1;
  console.log(foo); // 1
}

// 블록 레벨 스코프를 갖는다.
console.log(foo); // ReferenceError: foo is not defined

// 재할당 금지
const foo = 1;
foo = 2; // TypeError: Assignment to constant variable.

// const 키워드로 선언한 변수에 원시 값을 할당한 경우 변수 값을 변경할 수 없다. 원시 값은 변경 불가능한 값(immutable value)이므로 재할당 없이 값을 변경할 수 있는 방법이 없기 때문이다. 
```

```javascript
# const 키워드와 객체
// const 키워드로 선언된 변수에 객체를 할당한 경우 값을 변경할 수 있다. 변경 불가능한 값인 원시 값은 재할당 없이 변경(교체)할 수 있는 방법이 없지만 변경 가능한 값인 객체는 재할당 없이도 직접 변경이 가능하기 때문이다.

const person = {
  name: 'Lee',
};

// 객체는 변경 가능한 값이다. 따라서 재할다 없이 변경이 가능하다.

person.name = 'Kim';

console.log(person); // {name: "Kim"}


❗const 키워드는 재할당을 금지할 뿐 "불변"을 의미하지 않는다. 다시 말해, 새로운 값을 재할당하는 것은 불가능하지만 프로퍼티 동적 생성, 삭제, 프로퍼티 값의 변경을 통해 객체를 변경하는 것은 가능하다. 이때 객체가 변경되더라도 변수에 할당된 참조 값은 변겨되지 않는다.
```

> #### 정리
>
> **var vs. let vs. const**
>
> 변수 선언에는 기본적으로 const를 사용하고 let은 재할당이 필요한 경우에 한정해 사용하는 것이 좋다. const 키워드를 사용하면 의도치 않은 재할당을 방지하기 때문에 좀 더 안전하다.
>
> Var와 let, const 키워드는 다음과 같이 사용하는 것을 권장한다.
>
> * ES6를 사용한다면 var 키워드는 사용하지 않는다.
> * 재할당이 필요한 경우에 한정해 let 키워드를 사용한다. 이때 변수의 스코프는 최대한 좁게 만든다.
> * 변경이 발생하지 않고 읽기 전용으로 사용하는(재할당이 필요 없는 상수) 원시 값과 객체에는 const 키워드를 사용한다. const 키워드는 재할당을 금지하므로 var, let 키워드보다 안전하다.
>
> ❗ 변수를 선언하는 시점ㅇ는 재할당이 필요할지 잘 모르는 경우가 많다. 그리고 객체는 의외로 재할당하는 경우가 드물다. 
>
> 따라서 변수를 선어할 때는 일단 const 키워드를 사용하자. 반드시 재할당이 필요하다면(반드시 재할당이 필요한지 한번 생각해 볼 일이다) 그때 const 키워드를 let 키워드로 변경해도 결코 늦지 않다.

​		

## 16장 프로퍼티 어트리뷰트

이 장의 주제인 프로퍼티 어트리뷰트를 이해하기 위해 먼저 내부 슬롯(internal slot)과 내부 메서드(internal method)의 개념에 대해 알아본다.

내부 슬롯과 내부 메서드는 자바스크립트 엔진의 구현 알고리즘을 설명하기 위해 ECMAScript사양에서 사용하는 의사 프로퍼티(pseudo property)와 의사 메서드(pseudo method)이다. 바로 ECMAScript 사양에 등장하는 이중 대괄호([[...]])로 감싼 이름들이 내부 슬롯과 내부 메서드이다.

내부 슬롯과 내부 메서드는  ECMAScript 사양에 저의된 대로 구현되어 자바스크립트 엔진에서 실제로 동작 하지만 개발자가 직접 접근할 수 있도록 외부로 공개된 객체의 포로퍼티는 아니다. 즉, 내부 슬롯과 내부 메서드는 자바스크립트 엔진의 내부 로직이므로 원칙적으로 자바스크립트는 내부 슬롯과 내부 메서드에 직접적으로 접근하거나 호출할 수 있는 방법을 제공하지 않는다. 단, 일부 내부 슬롯과 내부 메서드에 한하여 간적적으로 접근할 수 있는 수단을 제공하기는 한다.

예를 들어, 모든 객체는 [[Prototype]]이라는 내부 슬롯을 갖는다. 내부 슬롯은 자바스크립트 엔진의 내부로직이므로 원칙적으로 직접 접근할 수 없지만 [[Prototype]] 내부 슬롯의 경우, \__proto__를 통해 간접적으로 접근할 수 있다.

```javascript
const o = {};
// 내부 슬롯은 자바스크립트 엔진의 내부 로직이므로 직접 접근할 수 없다.
o.[[Prototype]] // -> Uncaught SyntaxError: Unexpected token '['
// 단, 일부 내부 슬롯과 내부 메서드에 한하여 간접적으로 접근할 수 있는 수단을 제공하기는 한다.
o.__proto__ // -> Object.prototype
```

#### 프로퍼티 어트리뷰트와 프로퍼티 디스크립터 객체

property attribute 확인

- Object.getOwnPropertyDescriptor
- Object.getOwnPropertyDescriptors

```javascript
const person = {
    name: 'Lee'
};

// 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체를 반환한다
console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// { value: "Lee", writable: true, enumerable: true, configurable: true }
```

```javascript
const person = {
    name: 'Lee'
};

// 프로퍼티 동적 생성
person.age = 20;

// 모든 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체들을 반환한다
console.log(Object.getOwnPropertyDescriptor(person));
/*
{
    name: { value: "Lee", writable: true, enumerable: true, configurable: true },
    age: { value: 20, writable: true, enumerable: true, configurable: true },
}
*/
```

#### 접근자 프로퍼티

객체의 getter, setter 메소드
get, set, enumerable, configurable을 가짐

```javascript
const person = {
    // 데이터 프로퍼티
    firstName: 'Jongfeel',
    lastName: 'Kim',

    // fullName은 접근자 함수로 구성된 접근자 프로터티다.
    // getter 함수
    get fullName() {
        return `${this.firstName} ${this.lastName}`
    },
    // setter 함수
    set fullName(name) {
        // 배열 디스트럭처링 할당: "31.1 배열 디스트럭처링 할당" 참고
        [this.firstName, this.lastName] = name.split(' ');
    }
};

// 데이터 프로퍼티를 통한 프로터티 값의 참조
console.log(person.firstName + ' ' + person.lastName); // Jongfeel Kim

// 접근자 프로퍼티를 통한 프로퍼티 값의 저장
// 접근자 프로퍼티 fullName에 값을 저장하면 setter 함수가 호출된다.
person.fullName = 'Heegun Lee';
console.log(person); // { firstName: "Heegun", lastName: "Lee" }

// 접근자 프로터티를 통한 프로터티 값의 참조
// 접근자 프로터티 fullName에 접근하면 getter 함수가 호출된다.
console.log(person.fullName); // Heegun Lee

// firstName은 데이터 프로퍼티다.
// 데이터 프로퍼티는 [[Value]], [[Writable]], [[Enumerable]], [[Configurable]]
// 프로터티 어트리뷰트를 갖는다.
let descriptor = Object.getOwnPropertyDescriptor(person, 'firstName');
console.log(descriptor);
// { value: "Heegun", writable: true, enumerable: true, configurable: true }

// fullName은 접근자 프로퍼티다.
// 접근자 프로터티는 [[Get]], [[Set]], [[Enumerable]], [[Configurable]]
// 프로터티 어트리뷰트를 갖는다.
descriptor = Object.getOwnPropertyDescriptor(person, 'fullName');
console.log(descriptor);
// { get: f, set: F, enumerable: true, configurable: true }
```

#### 프로퍼티 정의

Object.defineProperty

```javascript
const person = {};

// 데이터 프로퍼티 정의
Object.defineProperty(person, 'firstName', {
    value: 'Ungmo',
    writable: true,
    enumerable: true,
    configurable: true
});

Object.defineProperty(person, 'lastName', {
    value: 'Lee' 
});

let descriptor = Object.getOwnPropertyDescriptor(person, 'firstName');
console.log('firstName', descriptor);
// firstName { value: "Ungmo", writable: true, enumerable: true, configurable: true }

// 디스크립터 객체의 프로퍼티를 누락시키면 undefined, false가 기본값이다.
descriptor = Object.getOwnPropertyDescriptor(person, 'lastName');
console.log('lastName', descriptor);
// lastName { value: "Lee", writable: false, enumerable: false, configurable: false }

// [[Enumerable]]의 값이 false인 경우
// 해당 프로퍼티는 for...in 문이나 Object.keys 등으로 열거할 수 없다.
// lastName 프로퍼티는 [[Enumerable]]의 값이 false이므로 열거되지 않는다.
console.log(Object.keys(person));   // ["firstName"]

// [[Writable]]의 값이 false인 경우 해당 프로퍼티의 [[Value]]의 값을 변경할 수 없다.
// lastName 프로퍼티는 [[Writable]]의 값이 false이므로 값을 변경할 수 없다.
// 이때 값을 변경하면 에러는 발생하지 않고 무시된다.
person.lastName = "Kim";

// [[Configurable]]의 값이 false인 경우 해당 프로퍼티를 삭제할 수 없다.
// lastName 프로퍼티는 [[Configurable]]의 값이 false이므로 삭제할 수 없다.
// 이때 프로퍼티를 삭제하면 에러는 발생하지 않고 무시된다.
delete person.lastName;

// [[Configurable]]의 값이 false인 경우 해당 프로퍼티를 재정의할 수 없다.
// Object.defineProperty(person, 'lastName', { enumerable: true });
// Uncaught TypeError: Cannot redefine property: lastName

descriptor = Object.getOwnPropertyDescriptor(person, 'lastName');
console.log('lastName', descriptor);
// lastName { value: "Lee", writable: false, enumerable: false, configurable: false }

// 접근자 프로퍼티 정의
Object.definePropety(person, 'fullName', {
    // getter 함수
    get() {
        return `${this.firstName} ${this.lastName}`;
    },
    // setter 함수
    set(name) {
        [this.firstName, this.lastName] = name.split(' ');
    },
    enumerable: true,
    configurable: true
});

descriptor = Object.getOwnPropertyDescriptor(person, 'fullName');
console.log('fullName', descriptor);
// fullName { get: f, set: f, enumerable: true, configurable: true }

person.fullName = "Heegun Lee";
console.log(person); // { firstName: "Heegun", lastName: "Lee" }
```

#### 객체 확장 금지

Object.preventExtensions 메서드는 객체의 확장을 금지한다.

```javascript
const person = { name: 'Lee' };

// person 객체는 확장이 금지된 객체가 아니다.
console.log(Object.isExtensible(person)); // true

// person 객체의 확장을 금지하여 프로퍼티 추가를 금지한다.
Object.preventExtensions(person);

// person 객체는 확장이 금지된 객체다.
console.log(Object.isExtensible(person)); // false

// 프로퍼티 추가가 금지된다.
person.age = 20; // 무시. strict mode에서는 에러
console.log(person); // { name: "Lee" }

// 프로퍼티 추가는 금지되지만 삭제는 가능하다.
delete person.name;
console.log(person); // {}

// 프로퍼티 정의에 의한 프로퍼티 추가도 금지된다.
Object.defineProperty(person, 'age', { value: 20 });
// TypeError: Cannot define property age, object is not extensible
```

#### 객체 밀봉

Object.seal 메서드는 객체를 밀봉한다. 밀봉된 객체는 읽기와 쓰기만 가능하다.

[예제 16-11]

```javascript
const person = { name: 'Lee' };

// person 객체는 밀봉(seal)된 객체가 아니다.
console.log(Object.isSealed(person)); // false

// person 객체를 밀봉(seal)하여 프로퍼티 추가, 삭제, 재정의를  금지한다.
Object.seal(person);

// person 객체는 밀봉(seal)된 객체다.
console.log(Object.isSealed(person)); // true

// 밀봉(seal)된 객체는 configurable이 false다.
console.log(Object.getOwnPropertyDescriptors(person));
/*
{
    name: { value: "Lee", writable: true, enumerable: true, configurable: false }
}
*/

// 프로퍼티 추가가 금지된다.
person.age = 20; // 무시. strict mode에서는 에러
console.log(person); // { name: "Lee" }

// 프로퍼티 삭제가 금지된다.
delete person.name; // 무시. strict mode에서는 에러
console.log(person); // { name: "Lee" }

// 프로퍼티 값 갱신은 가능하다.
person.name = 'Kim';
console.log(person); // { name: "Kim" }

// 프로퍼티 어트리뷰트 재정의가 금지된다.
Object.defineProperty(person, 'name', { configurable: true });
// TypeError: Cannot redefine property: name
```

#### 객체 동결

Object.freeze 메서드는 객체를 동결한다. 동결된 객체는 읽기만 가능하다.

```javascript
const person = { name: 'Lee' };

// person 객체는 동결(freeze)된 객체가 아니다.
console.log(Object.isFrozen(person)); // false

// person 객체를 동결(freeze)하여 프로퍼티 추가, 삭제, 재정의, 쓰기를  금지한다.
Object.freeze(person);

// person 객체는 동결(freeze))된 객체다.
console.log(Object.isFrozen(person)); // true

// 동결(freeze)된 객체는 writable과 configurable이 false다.
console.log(Object.getOwnPropertyDescriptors(person));
/*
{
    name: { value: "Lee", writable: false, enumerable: true, configurable: false }
}
*/

// 프로퍼티 추가가 금지된다.
person.age = 20; // 무시. strict mode에서는 에러
console.log(person); // { name: "Lee" }

// 프로퍼티 삭제가 금지된다.
delete person.name; // 무시. strict mode에서는 에러
console.log(person); // { name: "Lee" }

// 프로퍼티 값 갱신이 금지된다.
person.name = 'Kim'; // 무시. strict mode에서는 에러
console.log(person); // { name: "Lee" }

// 프로퍼티 어트리뷰트 재정의가 금지된다.
Object.defineProperty(person, 'name', { configurable: true });
// TypeError: Cannot redefine property: name
```

## 17장 생성자 함수에 의한 객체 생성

#### **생성자 함수**

#### constructor와 non-constructor의 구분

- constructor: 함수 선언문, 함수 표현식, 클래스(클래스도 함수다)
- non-constructor: 메서드(ES6 메소드 축약 표현), 화살표 함수

```javascript
// 일반 함수 정의: 함수 선언문, 함수 표현식
function foo() {}
const bar = function () {};
// 프로퍼티 x의 값으로 할당된 것은 일반 함수로 정의된 함수다. 이는 메서드로 인정하지 않는다.
const baz = {
    x: function () {}
};

// 일반 함수로 정의된 함수만이 constructor다.
new foo(); // -> foo {}
new bar(); // -> bar {}
new baz.x(); // -> x {}

// 화살표 함수 정의
const arrow = () => {};

new arrow(); // TypeError: arrow is not a constructor

// 메서드 정의: ES6의 메서드 축약 표현만 메서드로 인정한다.
const obj = {
    x() {}
};

new obj.x(); // TypeError: obj.x is not a constructor
```

### new.target

함수 내부에서 new.target을 사용하여 new 연산자와 생성자 함수로서 호출했는지 확인

```javascript
// 생성자 함수
function Circle(radius) {
    // 이 함수가 new 연산자와 함께 호출되지 않았다면 new.target은 undefined다.
    if (!new.target) {
        // new 연산자와 함께 생성자 함수를 재귀 호출하여 생성된 인스턴스를 반환한다.
        return new Circle(radius);
    }

    this.radius = radius;
    this.getDiameter = function () {
        return 2 * this.radius;
    };
}

// new 연산자 없이 생성자 함수를 호출하여도 new.target을 통해 생성자 함수로서 호출된다.
const circle = Circle(5);
console.log(circle.getDiameter());
```

## 18장 함수와 일급 객체

#### 일급 객체

다음과 같은 조건을 만족하는 객체를 일급 객체라 한다.

1. 무명의 리터럴로 생성할 수있다. 즉, 런타임에 생성이 가능하다.
2. 변수나 자료구조(객체, 배열 등)에 저장할 수 있다.
3. 함수의 매개변수에 전달할 수 있다.
4. 함수의 반환값으로 사용할 수 있다.

#### 함수 객체의 프로퍼티

#### arguments 프로퍼티

arguments 객체는 **함수 호출 시 전달된 인수argument 들의 정보를 담고 있는 순회 가능한iterable 유사 배열 객체**이며, **함수 내부에서 지역 변수처럼 사용**된다.

```javascript
function multiply(x y) {
    console.log(arguments);
    return x * y;
}

console.log(multiply());            // NaN
console.log(multiply(1));           // NaN
console.log(multiply(1, 2));        // 2
console.log(multiply(1, 2, 3));     // 2
```

arguments 객체는 매개변수 개수를 확정할 수 없는 가변 인자 함수를 구현할 때 유용하다.

```javascript
function sum() {
    let res = 0;

    // arguments 객체는 length 프로퍼티가 있는 유사 배열 객체이므로 for 문으로 순회할 수 있다.
    for (let i = 0; i < arguments.length; i++) {
        res += arguments[i];
    }

    return res;
}

console.log(sum());         // 0
console.log(sum(1, 2));     // 3
console.log(sum(1, 2, 3));  // 6
```

ES6의 Rest 파라미터 사용

```javascript
// ES6 Rest parameter
function sum(...args) {
    return args.reduce((pre, cur) => pre + cur, 0);
}

console.log(sum(1, 2));             // 3
console.log(sum(1, 2, 3, 4, 5));    // 15
```

#### length 프로퍼티

함수 객체의 length 프로퍼티는 함수를 정의할 때 선언한 매개변수의 개수를 가리킨다.

```javascript
function foo() {}
console.log(foo.length);    // 0

function bar(x) {
    return x;
}
console.log(bar.length);    // 1

function baz(x, y) {
    return x * y;
}
console.log(baz.length);    // 2
```

#### name 프로퍼티

함수 객체의 name 프로퍼티는 함수 이름을 나타낸다.

```javascript
// 기명 함수 표현식
var namedFunc = function foo() {};
console.log(namedFunc.name); // foo

// 익명 함수 표현식
var anonymousFunc = function() {};
// ES5: name 프로퍼티는 빈 문자열을 값으로 갖는다.
// ES6: name 프로퍼티는 함수 객체를 가리키는 변수 이름을 값으로 갖는다.
console.log(anonymousFunc.name); // anonymousFunc

// 함수 선언문(Function declaration)
function bar() {}
console.log(bar.name); // bar
```

