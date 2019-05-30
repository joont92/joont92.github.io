---
title: 초난감 DAO(1) / 관심사 분리
date: 2017-11-15 15:00:00
tags:
---

# 간단한 DAO 작성
앞으로 발전시켜나갈 DAO를 작성해보도록 하겠습니다.  

아래는 이번 DAO에서 사용할 테이블의 모습입니다.  
![초난감 DAO 스키마](/temp/초난감DAO스키마.png)

이제 정보를 저장할 자바 빈을 하나 만들도록 하겠습니다.

> **자바빈**  
> 자바빈이란 아래의 두 관례를 따라 만들어진 오브젝트를 말합니다.  
> 1. 디폴트 생성자 : 파라미터가 없는 디폴트 생성자를 가지고 있어야 합니다. 툴이나 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문입니다.  
>   - 리플렉션 : runtime 시에 객체를 통해 클래스의 정보를 분석(추출)해내는 프로그래밍 기법  
> 2. 프로퍼티 : 내부의 속성입니다. 또한 이 속성값은 setter와 getter를 통해 수정, 조회가 되도록 해야 합니다.  

```java
public class User {
    String id;
    String name;
    String password;
    
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
}
```

이제 간단한 DAO를 만들어 보고, main 메서드를 이용해 테스트 해보겠습니다.  
(main 메서드는 자신을 엔트리 포인트로 만들어 직접 실행이 가능하도록 해주는 메서드입니다.)  

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring","spring");

        PreparedStatement ps = conn.prepareStatement("insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        conn.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "spring");

        PreparedStatement ps = conn.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();

        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        conn.close();

        return user;
    }
    
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        UserDao dao = new UserDao();

        User user = new User();
        user.setId("joont");
        user.setName("준티");
        user.setPassword("pwd");

        dao.add(user);
            
        System.out.println(user.getId() + " 등록 성공");
        
        User user2 = dao.get(user.getId());
        System.out.println(user2.getName());
        System.out.println(user2.getPassword());
            
        System.out.println(user2.getId() + " 조회 성공");
    }
}
```

보시다 시피 잘 동작은 하나.. 이 코드는 문제점이 많습니다.  
한번 고쳐봅시다  

# DAO의분리
프로그래밍 기초 개념중 **관심사의 분리** 라는 것이 있습니다.  
이를 객체지향에 적용해보면, 관심이 같은 것끼리는 하나의 객체 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것을 말합니다.  

이와 같은 작업은 처음엔 힘들고 시간이 많이 걸리지만, 나중에 가면 유지보수와 변화에 수월해진다는 큰 장점이 있습니다.  
특히 객체지향 프로그래밍에서는 꼭 필요한 작업입니다.  

UserDao의 관심사를 분리하기 위해 살펴보면, 3가지의 관심사항을 발견할 수 있습니다.  
1. DB 커넥션
2. 파라미터를 받아 쿼리를 실행하는 과정
3. 리소스 반납

지금 가장 문제가 되는것은 첫번째 관심사입니다. 공통된 모듈이라 두개의 메서드에 똑같은 코드가 중복!!되어 있는것을 볼 수 있습니다.  
이는 변경이 일어날 때 중복하여 수정을 해줘야한다는 것을 의미하고, 규모가 커지면 굉장히 고통스러워지기도 합니다.. 실수라도 하면 ㅠ_ㅠ  
DB커넥션을 맺는 코드를 getConnection이라는 메서드로 분리해내겠습니다.  

```java
public class UserDao {
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection conn = getConnection();

        PreparedStatement ps = conn.prepareStatement("insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        conn.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection conn = getConnection();
        PreparedStatement ps = conn.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();

        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        conn.close();

        return user;
    }

    private Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "spring");
        return conn;
    }
}
```
(user 컬럼이 primary key인것은 주의하시고요, 아까와 같이 main 메서드를 추가해서 테스트해보시기 바랍니다.)  

> 이와 같이 기능에는 영향을 주지 않으면서, 코드의 구조를 바꾼 것을 `리팩토링` 이라고 합니다.
> 리팩토링 과정 중에 중복된 코드를 뽑아내는 것을 `메소드 추출` 이라고 합니다.  
> 리팩토링은 자바 개발자라면 반드시 익혀야 하는 기법입니다.  

여기서 만약, 제가 이 UserDao 클래스를 사람들에게 배포하게 되었다고 가정합시다.  
(java 파일이 아닌 class 파일만을 배포했다고 가정하겠습니다)  

근데 받는쪽에서 mysql DB를 사용하지 않는다면 어떡할까요? 아니면 자신만의 독창적인 DB 연결방법을 쓰고 싶다면?  
결론은 java 파일의 getCOnnection을 수정하여 컴파일하는 수밖에 없습니다. 결국 확장성이 떨어지는 상황이 온 것이죠.  

이럴경우 첫번째 방안으로 getConnection 메서드를 추상메서드로 만드는 방법이 있습니다.  
그리고 사용하는 쪽에서는 UserDao를 상속받고 getConnection만 재정의하여 사용하게 하면 됩니다.  

이로써 부모클래스인 UserDao는 getConnection이 어떻게 구현되는지에 대해선 관심을 가질 필요가 없어졌습니다.  
구현은 UserDao를 상속한 자식이 관심을 가져야 하는것이고, UserDao는 그냥 정의된 메서드만 사용할 뿐입니다.  
결국, 깔끔한 방법으로 관심사항이 분리되었습니다.  

> 상위 클래스에 기본적인 로직을 만들어두고, 기능의 일부를 추상메서드나 오버라이딩이 가능한 protected 메서드로 만든 뒤 서브클래스에서 필요에 맞게 구현해서 사용하는 방법을 템플릿 메서드 패턴 이라고 합니다.  
> 여기서 오버라이딩이 가능한 메서드를 훅 메서드 라고 하고, 추상메서드와 훅 메서드를 합쳐 팩토리 메서드 라고 합니다.  
> 이번엔 서브클래스의 입장에서 한번 보죠.  
> 서브클래스는 UserDao를 상속받아 getConnection 메서드를 오버라이딩 할 것입니다.  
> 즉, Connection 객체생성 방법에 대한 선택은 자신이 하게 되는 것이죠. 이렇듯 서브쪽에서 구체적 오브젝트 생성방법을 결정하게 하는것을 팩토리 메서드 패턴 이라고 합니다.
> 팩토리 메서드 패턴은 템플릿 메서드 패턴에 종속적이라고 볼 수 있겠네요 ㅎㅎ  

우리는 방금 전 깔끔한 방법으로 관심사항을 분리했습니다.  
그러나 여기서도 문제 발생 소지가 되는 점이 있습니다. 바로 상속을 사용했다는 점입니다.  
자식 클래스에서 다른 클래스를 상속해야 한다면? 부모 클래스의 내용이 변경된다면?  
규모가 커질수록 훨씬 많은 문제점을 가지고 오게 될 것입니다.  

# DAO의 확장
이번에는 상속의 문제를 벗어나기 위해 getConnection 메서드 부분을 아예 클래스로 분리해 보겠습니다.  
```java
class ConnectionMaker {
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection(
                "jdbc:mysql://localhost/springbook?characterEncoding=UTF-8", "spring", "spring");
        return conn;
    }
}

class UserDao {
    private ConnectionMaker connectionMaker;
    
    public UserDao() {
        this.connectionMaker = new ConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection conn = this.connectionMaker.makeConnection();

        PreparedStatement ps = conn.prepareStatement(
            "insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        conn.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection conn = this.connectionMaker.makeConnection();
        PreparedStatement ps = conn
                .prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        conn.close();

        return user;
    }
}
```
코드를 확실히 분리한 것은 분명 현명한 선택이었습니다. 그러나 아직 확장성이 뛰어나다고는 말할 수 없습니다.  

첫째로, 이 코드가 문제가 됩니다.  
```java
this.connectionMaker = new ConnectionMaker();
```
UserDao를 사용하는 쪽에서는 DB커넥션을 변경하려면 커넥션을 제공하는 클래스가 어떤것인지 정확히 알고 있어야 합니다.  

둘째로, connectionMaker 클래스의 makeConnection 메서드가 문제입니다.  
혹시라도 첫번째 문제에 해당하는 클래스명을 수정한다고 해도, UserDao내에서 makeConnection 메서드를 직접 사용하고 있기 때문에 메서드 변경이 불가능하고, 변경하게 된다면 UserDao내에서 makeConnection 사용 부분을 모두 바꾸어야 할 것입니다.  

즉, UserDao 쪽에서 ConnectionMaker 클래스에 대한 정보를 너무 많이 알고 있다는게 문제가 됩니다.  
결국, 이는 아직까지 확장성이 떨어지는 코드임을 말해주고 있습니다.  

이제 차근차근 해결해나가보겠습니다.  
먼저, 클래스간에 직접적인 관계 사이에 하나의 다리를 두어 연결을 느슨하게 만들어야 합니다.  
여기에 사용되기에 아주 좋은것이 있죠. 바로 **인터페이스(interface)** 입니다.  

ConnectionMaker 클래스를 인터페이스로 변경한다면 사용하는 입장에서는 꼭 ConnectionMaker라는 이름의 클래스를 만들필요가 없이 ConnectionMaker를 구현한 클래스만 사용하면 되므로 훨씬 확장성이 좋아집니다.  

그렇다면, new ConnectionMaker(); 이 부분은 어떻게 해야할까요? ConnectionMaker를 구현한 클래스를 생성해야 할까요?  
그렇다면 결국 다시 원점으로 돌아가버리는 꼴이 됩니다.  
하지만 파라미터를 이용하여 외부에서 가져온다면, 문제점을 해결할 수 있습니다.

```java
public interface ConnectionMaker {
   public Connection makeConnection() throws ClassNotFoundException, SQLException;
}

public class UserDao {
    private ConnectionMaker connectionMaker;
    
    public UserDao(ConnectionMaker simpleConnectionMaker) {
        this.connectionMaker = simpleConnectionMaker;
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        // 추가
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
       // 조회
    }
}
```

이로써 UserDao와 ConnectionMaker에 대한 관심은 좀 더 확연하게 분리되었습니다.  
그리고 아래처럼 외부에서 Connection을 위한 클래스를 결정하고, 원래 UserDao의 역할이었던 테스트를 담당하는 관심사를 가져가게 됩니다.  

```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ConnectionMaker connectionMaker = new ConnectionMaker_구현클래스();
		UserDao dao = new UserDao(connectionMaker);

		User user = new User();
		user.setId("spring");
		user.setName("spring");
		user.setPassword("spring");

		dao.add(user);
			
		System.out.println(user.getId() + " 등록 성공");
		
		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());
			
		System.out.println(user2.getId() + " 조회 성공");
	}
}
```

이처럼 UserDao, UserDaoTest 오브젝트가 있다고 할 때, UserDao는 기능을 제공해주므로 서비스 오브젝트 라 하고, 그 서비스를 사용하는 UserDaoTest를 클라이언트 오브젝트 라고 합니다.  
이로써 DAO의 개선을 일정수준 이상 마쳤습니다. 하지만 이것도 완벽한 코드는 아닐것입니다.(멀고도 험난한 여정이 ㅠㅠ)  

이제 마지막으로, 스프링에서 자주 사용될 용어들에 대해 한번 배워보고 포스팅을 마치겠습니다.  

- 개방 폐쇄 원칙(OCP)
    - '클래스나 모듈은 확장에는 열러있어야 하고, 변경에는 닫혀있어야 한다.' 라는 의미를 지니고 있습니다.
    - 앞서 작성했던 DAO를 가지고 설명하자면, DB Connection 기능을 확장하는 데는 열려있지만, UserDao는 영향을 받지 않기 때문에 변경에는 닫혀있다고 말할 수 있습니다.
- 높은 응집도와 낮은 결합도
    - 높은 응집도 : 하나의 모듈, 클래스가 하나의 관심사에만 집중되어 있다는 것을 의미합니다. connectionMaker가 DB 접속에 관한 것만 다루는 것처럼.
    - 낮은 결합도 : 하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도가 낮은것을 말합니다.
    > 즉, 완벽한 객체지향 프로그램이 되기 위해선 응집도를 높이고 결합도를 낮추어야 합니다!  
- 전략 패턴 
    - 기능 로직에서 변경이 필요한 부분은 인터페이스를 통해 외부로 분리시키고, 이를 구현한 클래스를 필요에 따라 바꿔 사용할 수 있게 하는 디자인 패턴을 말합니다.
    - 예시를 들자면 UserDao가 기능 로직이 되겠고, DB 연결방식이라는 것은 필요에 따라 변경이 필요하므로 인터페이스로 통째로 빼버렸고, 이 인터페이스를 구현한 (ConnectionMaker를 구현한) 클래스는 전략이 되겠죠. ㅎㅎ 전략을 바꿔끼운다고 표현 할 수 있겠네요.

<!-- more -->