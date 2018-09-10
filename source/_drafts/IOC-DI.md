---
title: IOC/DI
tags:
---

# 스프링이란?
스프링이 자바를 택한 이유는 객체지향이기 때문이다.  
엔터프라이즈에서 잃어버렸던 객체지향의 가치를 다시 되찾고 객체지향이 제공해주는 무수한 이점을 최대로 활용하자는게 스프링의 지향점이다!  
객체지향의 가치를 최대화 하기 위해 가장 관심을 두어야 할 부분은 오브젝트이기 때문에, 스프링이 가장 가치를 두는 부분도 오브젝트이다.  
오브젝트가 어떻게 생성되고, 사용되고, 관계를 맺고, 소멸되는지 신중하게 살펴봐야 하고, 이를 살펴보다 보면 자연스럽게 오브젝트의 설계에도 관심을 가지게 된다.  
하지만 효과적인 오브젝트 설계를 하기 위해선 기본적인 객체의 원칙, 디자인 패턴, 리팩토링, 단위 테스트 등 많은 지식이 요구된다.  
스프링은 이런 오브젝트 설계에 대해 명쾌한 방향을 제시해준다.  

# 초난감 DAO
객체지향에서는 모든 것이 변한다.  
개발자는 항상 미래의 변화에 대비할 수 있어야하고, 가장 효율적인 방법은 변화를 최소화 하는 것이다.  
보통 변화는 한가지 관심에 의해 일어난다. 하지만 실제로 한가지 관심의 변화가 한가지 작업의 수정까지로는 이어지지 않는 경우가 대부분이다.  
이는 관심사가 제대로 분리되지 못하고, 여러군데 산재되어 있기 때문에 발생한 일이다.  

아래의 난감한 DAO를 보며 관심사를 분리하는 작업을 진행해보자.  

```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");

		PreparedStatement ps = 
            c.prepareStatement("insert into users(id, name, password) values(?,?,?)");

		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}


	public User get(String id) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");
		
        PreparedStatement ps = 
            c.prepareStatement("select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}
```

간단하게 사용자를 저장하고 가져오는 로직을 가진 DAO 클래스이다.  
하지만 보다시피 각 메서드는 하나의 관심사에만 집중되어 있지 않다.  
관심사를 크게 나누어 보면 아래와 같다.  
1. Connection을 가져오는 과정
2. SQL을 실행하고 결과를 받는 과정
3. 자원을 반환하는 과정

이렇게 관심사가 분리되지 않고, 여러군데 산재되어 있다면 변화에 매우 취약하게 된다.  
만약 DB Connection을 맺는 주소가 바뀐다면 어떡할 것인가?  
지금은 메서드가 2개라서 별로 안 커보이지만, 메서드가 수십, 수백개에 달한다면 어떻게 할 것인가?  
먼저 Connection을 가져오는 관심사부터 분리해보자.  

```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		// insert 작업
	}


	public User get(String id) throws ClassNotFoundException, SQLException {	
		Connection c = getConnection();
		// select 작업
	}

    private Connection getConnection(){
        Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "book");

        return c;
    }
}
```

Connection을 가져오는 과정을 하나의 메서드로 분리해버렸다. 이를 메서드 추출이라고 한다.  
이로 인해 Connection 관련된 변화에 대해선 getConnection 메서드만 수정해주면 된다.  

# DAO 확장
메서드 추출을 통해 깔끔하게 관심을 분리했다. 그런데, 확장의 관점에서는 어떨까?  
가령 위의 UserDao를 사용하는 프로젝트가 여러군데 있고, 각각 다른 DB를 사용한다고 한다면, 어떻게 할 것인가?  
현재로써는 각각 소스를 수정해서 사용하는 방법밖에 없다. 이건 확장성이 없는 것이다.  

```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
        // insert 작업
	}


	public User get(String id) throws ClassNotFoundException, SQLException {	
		Connection c = getConnection();
        // select 작업
	}

    public abstract Connection getConnection();
}

public class NUserDao extends UserDao{
    @Override
    public Connection getConnection(){
        // getConnection from ORACLE
    }
}
public class DUserDao extends UserDao{
    @Override
    public Connection getConnection(){
        // getConnection from MySQL
    }
}
```

추상 메서드를 사용해 getConnection을 자식이 구현하도록 하였다. 이로인해 사용하는 곳에서 직접 Connection을 얻는 코드를 구현하면 된다. 확장에도 유연해졌다!  
> 위와 같이 기능에는 변함없이 코드의 구조를 바꾸는 행위를 리팩토링 이라고 한다.  
현재 템플릿 메서드 패턴, 팩토리 메서드 패턴이라는 디자인 패턴을 적용하여 리팩토링 하였다.  
템플릿 메서드 패턴 링크  
팩토리 메서드 패턴 링크  

뭔가 깔끔하게 해결된 듯 보이지만.. 상속을 사용하여 관심사를 분리하는 과정이 사실 좋은 방법은 아니다.  
1. 상속을 통한 부모자식 클래스의 관계는 생각보다 긴밀하다. 부모가 변하면 자식도 영향을 받기 때문이다.  
2. 자바는 다중 상속이 불가능하다. NUserDao나 DUserDao에 다른 상속이 있었다면 어떡할 것인가?  
3. 이런 방식이라면 Dao가 추가될 때 마다 상속하고 Connection 관심사를 구현해줘야 한다. 이건 관심사가 분리되었다고 볼 수 없다.  