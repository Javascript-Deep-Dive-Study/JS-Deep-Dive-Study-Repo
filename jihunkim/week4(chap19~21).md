## 19장 프로토타입

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