---
title: "디자인패턴(2) - 전략 패턴(Strategy Pattern)"
date: 2021-03-11T11:35:30-04:00
categories:
  - Design Pattern
tags:
  - OOP
  - Design Pattern
  - Strategy Pattern
---

# DAO의 확장

## 클래스의 분리

변화의 성격이 다르다는건 변화의 이유와 시기, 주기 등이 다르다는 뜻.

슈퍼/서브클래스 구조에서 클래스를 아예 분리할 수 있다. 하지만 단지 서브클래스의 구현부분을 다른 클래스로 분리할 경우 문제가 있다. 사용자 클래스에서 분리클래스를 구체적으로 알아야하고(인스턴스생성을 위해서), 분리클래스에서 사용할 메소드가 어떤것인지도 알아야 한다. 즉, 사용자 클래스가 분리클래스에 대해 알아야 할게 많아지면서 결합도가 다시 높아지고진다. 그리고 분리 클래스의 확장(상속 혹은 다른 메소드 사용)을 하려면 사용자 클래스까지 변경돼야 한다.

## 인터페이스의 도입

이를 해결하기 위해서 **서로 긴밀하게 연결되어 있지 않도록 중간에 추상적인 느슨한 연결고리를 만들어 준다.**

추상화란 어떤것들의 공통적인 성격을 뽑아내어 이를 따로 분리해내는 작업이다. 이를 위한 도구가 인터페이스이다. 인터페이스는 자신을 구현한 클래스에 대한 구체적인 정보는 모두 감춰버린다.

인터페이스는 어떤 일을 하겠다는 기능만 정의해 놓은 것이다. 따라서 인터페이스에는 어떻게 하겠다는 구현 방법은 나타나 있지 않다. 구현 내용은 인터페이스를 구현한 클래스들이 알아서 결정할 일이다.

```java
//고객에게 납품을 할 때는 UserDao클래스와 함께 ConnectionMaker 인터페이스도 전달한다.
public interface ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

```java
//D사에서는 전달받은 ConnectionMaker 인터페이스를 구현하여 UserDao클래스에 쓴다.
public class DConnectionMaker implements ConnectionMaker {
	...
	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		//D사의 독자적인 방법으로 Connection을 생성하는 코드
	}

```

하지만 아래와 같이 어떤 구현체를 사용할지 결정하는 생성자의 코드가 남아있다. 결국 ConnectionMaker인터페이스로 여러 ConnectionMaker구현체를 다루더라도, 구현체가 변경되면 UserDAO의 코드도 변경돼야한다.

```java
public class UserDao {
	private ConnectionMaker connectionMaker;
	public UserDao() {
		connectionMaker = new DConnectionMaker();
	}
	...
```

## 관계설정 책임의 분리

바로 위 코드에서 UserDao의 생성자에는 UserDao와 UserDao가 사용할 ConnectionMaker의 특정 구현 클래스 사이의 관계 설정에 대한 관심사가 있다.

클래스 사이에 관계가 만들어진다는 것은 한 클래스가 인터페이스 없이 다른 클래스를 직접 사용한다는 뜻이다. 따라서 클래스가 아니라 오브젝트와 오브젝트 사이의 관계를 설정해줘야 한다.

오브젝트 사이의 관계는 런타임 시에 한쪽이 다른 오브젝트의 레퍼런스를 갖고있는 방식으로 만들어진다.

클래스 사이에 관계가 만들어지는 것과 오브젝트 사이에 동적으로 관계가 만들어지는 차이를 구분해야 한다.

클래스 사이의 관계는 코드에 다른 클래스 이름이 나타나기 때문에 만들어진다. 하지만 오브젝트 사이의 관계는 그렇지 않다. 코드에서는 특정 클래스를 전혀 알지 못하더라도 해당 클래스가 구현한 인터페이스를 사용했다면, 그 클래스의 오브젝트를 인터페이스 타입으로 받아서 사용할 수 있다. 객체지향의 다형성이라는 특징 덕분이다.

런타임 오브젝트 관계를 갖는 구조를 만들어 주는게 제 3의 오브젝트인 UserDao의 클라이언트가 갖는 책임이다.

클라이언트는 자기가 UserDao를 사용해야 할 입장이기 때문에 UserDao의 세부 전략이라고도 볼 수 있는 ConnectionMaker의 구현 클래스를 선택하고, 선택한 클래스의 오브젝트를 생성해서 UserDao와 연결해 줄 수 있다. 이는 위 코드에서 생성자가 갖는 관심사와 책임이다.

```java
//수정한 생성자
public UserDao(ConnectionMaker connectionMaker) {
	this.connectionMaker = connectionMaker;
}
```

```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao dao = new UserDao(connectionMaker);
		...
	}
}
```

UserDaoTest는 UserDao와 ConnectionMaker 구현 클래스와의 런타임 오브젝트 의존관계를 설정하는 책임을 담당해야 한다. UserDao에 있으면 안되는 다른 관심사항, 책임을 클라이언트로 떠넘기는 작업이 끝났다.

인터페이스를 도입하고 클라이언트의 도움을 얻는 방법은 상속에 비해 훨씬 유연하다.

또한, DAO가 아무리 많아져도 DB접속 방법에 대한 관심은 오직 한군데에 집중되고, DB접속 방법을 변경해야 할 때도 오직 한 곳의 코드만 수정하면 된다.

## 원칙과 패턴

### 개방폐쇄원칙(Open-Closed Principle)

클래스나 모듈은 확장에는 열려있어야 하고 변경에는 닫혀있어야 한다. 즉, 프로그램의 어떤 요소를 쉽게 확장하도록 관심사를 모아서 처리하고, 확장함으로써 해당 관심사 이외의 소스가 변경돼서는 안된다.

인터페이스를 사용해 확장 기능을 정의한 대부분의 API는 바로 이 개방폐쇄 원칙을 따른다고 볼 수 있다.

- 디자인 패턴은 특별한 상황에서 발생하는 문제에 대한 좀 더 구체적인 솔루션이라고 한다면, 객체지향 설계 원칙은 좀 더 일반적인 상황에서 적용 가능한 설계 기준이라고 볼 수 있다.
- 객체지향 설계 원칙(SOLID) : SRP(Single Responsibility Principle) - 단일 책임 원칙, OCP(Open Closed Principle) - 개방폐쇄 원칙, LSP(Liskov Substitution Principle) - 리스코프 치환 원칙, ISP(Interface Segregation Principle) - 인터페이스 분리 원칙, DIP(Dependency Inversion Principle) - 의존관계 역전 원칙

### 높은 응집도와 낮은 결합도

응집도가 높다는건 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻이다.

- 높은 응집도
- 낮은 결합도

    -책임과 관심사가 다른 오브젝트 또는 모듈과는 느슨하게 연결된 형태를 유지하는 것이 바람직하다. 느슨한 연결은 관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공하고, 나머지는 서로 독립적이고 알 필요도 없게 만들어주는 것이다. 

    -결합도가 낮아지면 변화에 대응하는 속도가 높아지고, 확장에도 매우 편리하다.

    -결합도란 **'하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도'** 이다.

### 전략 패턴(Strategy Pattern)

개방폐쇄원칙의 실현에도 가장 잘 들어맞는 패턴이라고 볼 수 있다. 

자신의 기능 맥락(Context)에서, 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴이다. 여기서 말하는 알고리즘은 독립적인 책임으로 분리가 가능한 기능을 뜻한다.

이를 대체 가능한 전략이라고 보기 때문에 전략패턴이라 부른다.

UserDao는 전략패턴의 컨텍스트에 해당한다.

컨텍스트를 사용하는 클라이언트는 컨텍스트가 사용할 전략을 컨텍스트의 생성자 등을 통해 제공해주는게 일반적이다.