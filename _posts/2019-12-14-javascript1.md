---
title: "javascript(작성중)"
date: 2019-04-18T15:34:30-04:00
categories:
  - javascript
tags:
  - javascript
  - update
---

# 객체

자바스크립트에서의 객체는 java, c++과는다르다

객체를 생성하는 방법은 변수에 바로 할당하는방법(object literal)
-> var object1 = {}

생성자 함수는 클래스의 자바스크립트 버전이다. 생성자 함수는 단순히 프로퍼티와 메소드를 정의한다.
생성자함수를 사용하는 방법 (클래스를 생성하는게 아니라 함수에 new 연산자를 붙히면 바로 빈 객체(Object객체)가 생성된다)
function object() {};
var object2 = new object();

객체를 생성한 후 멤버(프로퍼티 혹은 메소드)를 추가할 수 있어서 유연한 프로그래밍 가능
object1.param1 = "테스트";

this는 무엇인가?
this는 지금 동작하고 있는 코드를 가지고 있는 객체를 가리킨다( 주로 메소드에서 사용. this는 메소드를 갖고 있는 객체를 지칭)
this는 객체 멤버의 컨텍스트가 바뀜에 따라 유연하게 적용된다.
동적으로 객체를 생성하는 경우(생성자를 사용하는 경우 등)에 유용

# 생성자

생성자로부터 새로운 객체 인스턴스가 생성되면, 객체의 핵심 기능이 프로토타입 체인에 의해 연결된다.

함수 정의(생성자함수)  - 함수도 객체다.  함수명 첫글자 대문자로.
```
function Person(name) {
  this.name = name;
  this.greeting = function() {
    alert('Hi! I\'m ' + this.name + '.');
  };
}
```
아무것도 리턴하지 않고 객체를 만들지도 않는다. 그저 함수의 정의일뿐. 하지만 생성자로서 역할을 한다.
생성자 함수명은 대문자로 시작 권장. 이 규칙은 생성자 함수가 코드안에서 잘 구별되도록 해줌.

객체 생성
```
var person1 = new Person('Bob');
var person2 = new Person('Sarah');
```


### 객체 인스턴스를 생성하는 다른 방법

Object()생성자
1. 
```
var person1 = new Object();
person1.name = 'Chris';
person1['age'] = 38;
person1.greeting = function() {
  alert('Hi! I\'m ' + this.name + '.');
};
```
2.
```
var person1 = new Object({
  name: 'Chris',
  age: 38,
  greeting: function() {
    alert('Hi! I\'m ' + this.name + '.');
  }
});
```

create()함수 사용
```
var person2 = Object.create(person1);
person2.name
person2.greeting()
```
person2가 person1을 기반으로 만들어짐. 새 객체는 원 객체와 같은 프로퍼티와 메소드들을 가짐.
(IE8에서 지원하지 않음)

# 프로토타입

자바스크립트는 prototype-based language이다. 모든 객체들이 메소드와 속성들을 상속받기 위한 템플릿으로써 프로토타입객체를 가진다.
상속되는 속성과 메소드들은 각 개체가 아니라 객체의 생성자의 prototype이라는 속성에 정의됨
(Person생성자의 객체 person1은 Person.prototype을 상속받는다.)

객체의 prototype( Object.getPrototypeOf(obj) 함수 또는 deprecated된 __proto__속성으로 접근 가능한 ) 과
생성자의 prototype 속성의 차이를 인지하는 것은 중요.
전자는 개별 객체의 속성이며 후자는 생성자의 속성입니다. 이 말은 Object.getPrototypeOf(new Foobar())의 반환값이 Foobar.prototype과 동일한 객체라는 의미입니다.

생성자 함수를 정의하면 그 함수의 prototype이 생성된다
```
function test() {}
var aa = new test();
// test.prototype에 constructor와 __proto__존재
// test생성자로 만들어진 객체는 test.prototype을 상속받음
// aa.__proto__와 test.prototype이 동일
```
객체.__proto__는 생성자함수.prototype에 대한 참조변수인듯.
-> 프로토타입체인 가능

### 프로토타입 속성:상속 받은 멤버들이 정의된 곳
Object.prototype.watch() -> Object를 상속받는 생성자들이 사용 가능
Object.keys() -> 상속되지 않음. Object()생성자에서만 사용 가능

///
프로토타입도 하나의 속성임.
Object.prototype에 정의된것들을 상속
Object.로 시작하는건 상속X
///

# 생성자 속성

모든 생성자 함수는 constructor 속성을 지닌 객체를 프로토타입 객체로 가지고 있다. 이 constructor 속성은 원본 생성자 함수 자신을 가리키고 있다.

# 프로토타입 수정하기

Person.prototype.vv = "123";
prototype에 속성을 정의하는것은 좋은 방법이 아니다.
프로토타입에 상수(한번 할당하면 변하지 않는 값)을 정의하는것은 가능하지만 일반적으로 생성자에서 정의하는게 낫다.

일반적인 방식으로는 속성은 생성자에서, 메소드는 프로토타입에서 정의한다. 


//
# Inheritance in javascript
## 프로토타입 상속

### Teacher()생성자 함수 정의
```
function Teacher(first, last, age, gender, interests, subject) {
  Person.call(this, first, last, age, gender, interests);

  this.subject = subject;
}
```
call()함수의 첫번째 매개변수는 다른 곳에서 정의된 함수를 현재 컨텍스트에서 실행할 수 있도록 한다. 실행하고자 하는 함수의 첫번째 매개변수로 this를 전달하고 나머지는 실제 함수 실행에 필요한 인자들을 전달하면 된다.




