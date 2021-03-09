---
title: "디자인패턴(1) - 템플릿 메소드 패턴 & 팩토리 메소드 패턴"
date: 2021-03-10T00:42:30-04:00
categories:
  - Design Pattern
tags:
  - OOP
  - Design Pattern
---

이번 포스팅은 토비의스프링 책에서 설명하고있는 객체지향의 디자인패턴들을 정리한 포스팅이다.

틀린 부분이 있을 수 있다. 그러한 부분은 추후 업데이트 할 것이다.

# 관심사의 분리

모든 변경과 발전은 한 번에 한 가지 관심사항에 집중해서 일어난다. 문제는, 변화는 대체로 집중된 한 가지 관심에 대해 일어나지만 그에 따른 작업은 한곳에 집중되지 않는 경우가 많다는 점이다.

그러므로 관심사의 분리(Separation of Concerns)를 통해 관심사가 같은 것끼리 모으고 다른 것은 분리해줌으로써 같은 관심에 효과적으로 집중해야 한다.

# 초기 구현 코드

오늘 리팩토링 작업을 수행할 초기 구현 코드이다. db커넥션을 가져오는 중복코드를 클래스 내에 메소드로 빼낸 상태부터 시작하자.

```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}
	private Connection getConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
			"jdbc:mysql://localhost/springbook", "spring", "book");
		return c;
	}
}
```

위의 코드는 DB커넥션을 가져오는 방식이나 생성할 DB커넥션의 종류를 바꾸려면 UserDao의 소스가 바뀌어야 한다. 즉, 확장이 용이하지 않다.

# 템플릿 메소드 패턴

상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 가장 대표적인 방법이다. 변하지 않는 기능은 슈퍼클래스에 만들어 두고 자주 변경되며 확장할 기능은 서브클래스에서 만든다.

위의 코드에 상속을 통한 확장을 적용해보자. 이렇게 하면, UserDao를 상속한 클래스에서는 getConnection()을 오버라이드 하면서 DB커넥션에 대한 관심에만 집중할 수 있고, DB에서 read/write하는 관심은 신경안써도 된다. 관심사가 분리됐다. 

즉, 최초 설계할때 변경가능성이 있는 부분만 추상메소드로 남겨놓은채 설계를 진행하여 이 클래스를 제공하고, 실제 구현자는 해당 클래스를 상속하여 변경가능성이 있는 부분만 구현하면 된다. 

단순 변경이 아니라, 확장이 용이해졌다.

```java
public abstract class UserDao {
	//템플릿 메소드
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}
	//템플릿 메소드
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
	}
	public abstract Connection getConnection() throws ClassNotFoundException, 
			SQLException;
}

public class NUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, 
			SQLException {
		//N사 생성코드
	}
}
```

이렇게 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 **템플릿 메소드 패턴**이라고 한다.

### 훅(hook) 메소드

슈퍼클래스에서 디폴트 기능을 정의해두거나 비웠다가 서브클래스에서 선택적으로 오버라이드 할 수 있도록 만들어둔 메소드(추상메소드 아님)

### 템플릿 메소드

기본 알고리즘 골격을 담은 메소드를 템플릿 메소드라고 부른다. 템플릿 메소드는 서브클래스에서 오버라이드하거나 구현할 메소드를 사용한다.

### 예시

```java
public abstract class Super {
	public void templateMethod() {
		//기본 알고리즘 코드
		hookMethod();
		abstractMethod();
		...
	}
	protected void hookMethod() {}
	public abstract void abstractMethod();
}

public class Sub1 extends Super {
	protected void hookMethod() {
		...
	}
	protected void abstractMethod() {
		...
	}
}
```

### 템플릿 메소드 패턴 한 줄 정리

자주 변경되며 확장될 가능성이 있는 관심사를 격리하고, 그에 대한 실제 구현은 추상메서드 혹은 훅메서드를 오버라이드 한 자식클래스에서 하도록 하여, 상속을 통해 확장이 용이해진다.

# 팩토리 메소드 패턴

템플릿 메소드 패턴과 마찬가지로 상속을 통해 기능을 확장하게 하는 패턴이다.

슈퍼클래스 코드에서는 서브클래스에서 구현할 메소드를 호출해서 필요한 타입의 오브젝트를 가져와 사용한다.

이 메소드는 주로 **인터페이스 타입으로 오브젝트를 리턴**(위의 샘플코드에서는 Connection 타입)하므로 서브클래스에서 정확히 어떤 클래스의 오브젝트를 만들어 리턴할지는 슈퍼클래스에서는 알지 못한다. 사실 관심도 없다.

서브클래스는 다양한 방법으로 오브젝트를 생성하는 메소드를 재정의할 수 있다. 이렇게 서브클래스에서 오브젝트 생성 방법과 클래스를 결정할 수 있도록 미리 정의해 둔 메소드를 **팩토리 메소드**라고 하고, 이 방식을 통해 오브젝트 생성 방법을 나머지 로직, 즉 슈퍼클래스의 기본 코드에서 독립시키는 방법을 팩토리 메소드 패턴이라고 한다.

같은 종류의 구현클래스의 오브젝트를 리턴하더라도, 오브젝트 생성 방식이 달라지면 이는 팩토리 메소드 패턴이라고 볼 수 있다.

UserDao의 getConnection() 메소드는 Connection 타입 오브젝트를 생성한다는 기능을 정의해놓은 추상메소드다. 그리고 UserDao의 서브클래스의 getConnection() 메소드는 어떤 Connection 클래스의 오브젝트를 어떻게 생성할 것인지를 결정하는 방법이라고도 볼 수 있다. 이렇게 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 **팩토리 메소드 패턴** 이라고 부른다.

### 예시

위 템플릿 메서드의 코드와 동일하다

```java
public abstract class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		//팩토리 메소드로부터 받은 오브젝트를 사용한다. 인터페이스형을 다루므로 정확히 어떤클래스인지 알 필요 없다.
		Connection c = getConnection();
		...
	}
	public User get(String id) throws ClassNotFoundException, SQLException {
		//팩토리 메소드로부터 받은 오브젝트를 사용한다. 인터페이스형을 다루므로 정확히 어떤클래스인지 알 필요 없다.
		Connection c = getConnection();
		...
	}
	//팩토리 메소드. 일정한 타입의 오브젝트를 생성하여 넘겨준다.
	public abstract Connection getConnection() throws ClassNotFoundException, 
			SQLException;
}

//오브젝트 생성 방식 및 정확히 어떤 클래스를 생성하는지를 구현.
public class NUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, 
			SQLException {
		//N사 생성코드
	}
}
```

### 팩토리 메서드 패턴 한 줄 정리

어떤 클래스의 메소드에서 변경 및 확장이 가능한 오브젝트를 사용할 때, 이 오브젝트를 인터페이스 참조형으로 다루고, 실제 오브젝트의 생성은 추상 메서드를 구현한 자식클래스에 책임을 위임한다.

# 총 정리 및 사견

## 정리

예시로 쓰인 UserDao를 리팩토링 한 결과에는 템플릿 메소드 패턴과 팩토리 메소드 패턴이 모두 들어있다. 템플릿 메소드 패턴은 확장 가능한 관심사를 상속을 통해 서브클래스에 위임하는 것이고, 팩토리 메소드 패턴은 확장 가능한 "오브젝트 생성"이라는 관심을 상속을 통해 서브클래스에 위임하는 것이다. 

같은 코드라도 보는 관점에 따라 템플릿 메소드 패턴으로 볼 수 도 있고 팩토리 메소드 패턴으로 볼 수도 있는 것이다. 아마 실제 프로덕션 코드에도 이처럼 패턴들이 혼재돼있는 경우가 많을듯 싶다.

![design_pattern_example](../assets/images/templateMethod%26factoryMethod.png)

## 사견

**팩토리 메소드 패턴은** 슈퍼클래스의 메소드에서 사용할 어떠한 오브젝트가 상황이나 환경에 따라 바뀔 가능성이 있을때, 이 오브젝트를 생성하는 부분을 추상메소드로 격리하고 서브클래스에서 확장을 통해 오브젝트를 생성하도록 하는 패턴이다.

슈퍼클래스에서는 이 오브젝트를 인터페이스 참조형으로 다루고, 팩토리 메소드에서는 이 인터페이스타입의 구상오브젝트를 리턴해주는 기능을 자식클래스에 위임한다.

즉, 하위클래스의 관심은 특정 인터페이스를 구현한 클래스들 중 어떤 클래스를 생성할지, 그리고 이 클래스를 어떤 방법으로 만들어 낼지(DB커넥션 새로 맺거나 커넥션 풀에서 가져오거나 등) 이다.

그리고 팩토리 메소드의 리턴타입이 인터페이스여야, 팩토리 메소드 패턴을 통해 확장이 가능하다.