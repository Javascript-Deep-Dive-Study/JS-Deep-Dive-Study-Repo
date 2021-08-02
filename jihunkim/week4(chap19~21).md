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