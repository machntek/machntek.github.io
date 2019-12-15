---
title: "javascript2(작성중)"
date: 2019-04-18T15:34:30-04:00
categories:
  - javascript
tags:
  - javascript
  - update
---

# 비동기 javascript

## 일반적인 비동기 프로그래밍 개념

자바스크립트는 single thread이다. 그래서 web worker를 사용하면 javascript 처리 중 일부를 별도의 스레드로 보낼 수 있으므로 여러 javascript 구문을 동시에 실행 할 수 있다.
일반적으로 worker를 사용하여 주 스레드에서 비용이 많이 드는 작업을 실행하지 않도록 한다.

* web worker : worker() 생성자를 통해 생성되며 지정된 javascript파일에 포함된 코드를 worker쓰레드에서 실행한다.
          worker는 DOM, window의 기본 메서드와 속성을 다룰 수 없다.
          
### 비동기 코드

웹 워커는 유용하지만 한계가 있다. 1. DOM에 액세스 할 수 없다. 2. worker에서 돌아가는 코드가 block하지는 않지만, 이는 여전히 기본적으로 sync한 방식이다. 어떤 함수가 다른함수에 의존할때(선행작업이 있을때) 문제가 된다.

이러한 문제를 해결하기 위해 브라우저를 통해 특정 작업을 비동기적으로 실행할 수 있다. Promises와 같은 기능을 사용하면 작업 실행을 설정 한 다음, 다른 작업을 실행하기 전에 결과가 반환 될 때까지 기다릴 수 있다.

## 비동기 javascript 소개

### 동기식 javascript

```
const btn = document.querySelector('button');
btn.addEventListener('click', () => {
  alert('You clicked me!');

  let pElem = document.createElement('p');
  pElem.textContent = 'This is a newly-added paragraph.';
  document.body.appendChild(pElem);
});
```

각 작업이 처리되는 동안 렌더링이 일시 중지 된 다른 작업은 수행 할 수 없다.(alert창 확인 눌러야 <p>가 생성)
자바스크립트는 싱글스레드이기 때문이다. 한번에 하나의 메인 스레드에서 한가지 일만 발생할 수 있으며 작업이 완료 될 때까지 다른 모든 항목이 차단된다.

### 비동기 javascript


### 비동기 콜백
비동기 콜백은 백그라운드에서 코드 실행을 시작하는 함수를 호출 할 때 인수로 지정된 함수이다.
백그라운드 코드 실행이 완료되면 콜백 함수를 호출하여 작업이 완료되었음을 알리거나 관심있는 작업이 발생했음을 알려준다.

콜백 함수를 다른 함수의 인수로 전달하면 함수의 참조만 인수로 전달한다. 즉, 콜백 함수는 즉시 실행 되지 않는다. 포함하는 함수의 본문 내부에서 비동기적으로 콜백 하게 된다.
함수는 때가 오면 콜백 함수를 실행해야 한다.

콜백은 다목적이다. 함수가 실행되는 순서와 함수간에 전달되는 데이터를 제어할 수 있을뿐만 아니라 상황에 따라 다른 함수에 데이터를 전달할 수도 있다.

모든 콜백이 비동기적인 것은 아니다. 일부는 동기적으로 실행된다. 예를 들어 Array.prototype.forEach()로 배열에서 항목을 루핑시킬때가 있다.
```
const gods = ['Apollo', 'Artemis', 'Ares', 'Zeus'];

gods.forEach(function (eachName, index){
  console.log(index + '. ' + eachName);
});
```

### Promises

#### 이벤트 큐
promise와 같은 비동기 작업은 이벤트 스레드에 저장된다. 이 대기열은 기본 스레드가 처리를 마친 후에 실행되어 후속 javascript 코드 실행을 차단하지 않는다.
대기중인 작업은 가능한 빨리 완료된 다음 결과를 javascript 환경으로 반환한다.

#### Promises vs callback
프로미스는 async작업을 처리할 수 있도록 특별히 만들어졌고, 콜백에 비해 많은 이점이 있다.
* 프로미스는 anyc작업을 .then()함수로 결과를 다음 인풋으로 넘기면서체이닝할 수 있다. 이는 콜백에서는 어렵다.(콜백 지옥 발생)
* 프로미스 콜백은 항상 이벤트 큐에 배치된 엄격한 순서로 호출된다.
* 오류 처리가 훨씬 낫다. 모든 에러가 단일 .catch() 블록으로 처리된다.
* 프로미스는 3rd파티 라이브러리에 콜백이 전달될 때함수가 실행되는 방식을 완전히 제어하지 못하는 콜백과 달리 제어의 역전을 피할 수 있다.






# Using Promise 
promise는 비동기 작업의 최종 완료 또는 실패를 나타내는 객체이다. 
기본적으로 promise는 함수에 콜백을 전달하는 대신에, 콜백을 첨부하는 방식의 객체이다.

## Guarantess
Promise는 아래와 같은 특징을 보장한다
* 콜백은 자바스크립트 event loop이 현재 실행중인 콜 스택을 완료하기 이전에는 절대 호출되지 않는다.
* 비동기 작업이 성공하거나 실패한 뒤에 then()을 이용하여 추가한 콜백의 경우에도 위와 같다.
* then()을 여러번 사용하여 여러개의 콜백을 추가 할 수 있다. 그리고 각각의 콜백은 주어진 순서대로 하나 하나 실행된다.



# Function 생성자




        
