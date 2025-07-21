JDBC(Java Database Connectivity)는 Java 기반 애플리케이션의 데이터를 관계형 데이터베이스에 접근하고, SQL을 통해 데이터를 CRUD 할 수 있도록 도와주는 자바 표준 API 이다.
> 1997.02.19 JDK 1.1의 일부로 출시 한 이후 현재까지도 자바 SE의 일부이다.

### **JDBC의 동작 흐름**

![](https://blog.kakaocdn.net/dna/bHk4IH/btrR2EZokg5/AAAAAAAAAAAAAAAAAAAAACwZBBZVWz-udPXvM_Hn9EjugiT9jhZdY647e-CYOLrX/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=Qz6nV96ZTpu8ZFq6rBXYX%2BUMr4U%3D)

JDBC는 Java 애플리케이션 내에서 JDBC API를 사용하여 데이터베이스에 접근하는 단순한 구조이다.

JDBC API를 사용하기 위해서는 JDBC 드라이버를 먼저 로딩한 후 데이터베이스와 연결하게 된다.

#### JDBC 드라이버
> 데이터베이스마다 통신 방식이 다르면, Java 코드를 매번 바꿔야 하지 않을까?  
> → 데이터베이스만 교체해도 그대로 동작할 수 있도록 만들 수는 없을까?  
> 이런 생각에서 JDBC 드라이버가 등장하게 되었다.

JDBC는 인터페이스 기반 명세이며, 각 DB 벤더(회사, 오라클, MS 등...)에서 자신의 DB(MySQL, MariaDB, MSSQL, .etc)에 맞도록 위 JDBC 인터페이스를 구현해서 라이브러리 형태로 제공한다. 이를 JDBC 드라이버라고 한다.
-  JDBC API 호출을 실제 데이터베이스 벤더의 프로토콜에 맞게 변환하여 통신하는 라이브러리(구현체)이다.
- 따라서 JDBC 드라이버의 구현체를 이용해서 특정 벤더의 데이터베이스에 접근할 수 있게 되므로, Java 애플리케이션은 JDBC API만 사용하면서도 다양한 DB에 유연하게 접근할 수 있다.


### **JDBC API 사용 흐름**

JDBC API의 구성 요소들의 동작 흐름은 다음과 같다.

![](https://blog.kakaocdn.net/dna/wTyMC/btrR5yww0DA/AAAAAAAAAAAAAAAAAAAAACXYlb8-uxrfTENOIotfDNIk05y7awoASWIojnESxQww/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=6KQ3ivSpepZ3ZYmt5%2FcLJ1hxhis%3D)

1. JDBC 드라이버 로딩 : 사용하고자 하는 JDBC 드라이버를 로딩한다. JDBC 드라이버는 DriverManager 클래스를 통해 로딩된다.
2. Connection 객체 생성 : JDBC 드라이버가 정상적으로 로딩되면 DriverManager를 통해 데이터베이스와 연결되는 세션(Session)인 Connection 객체를 생성한다.
3. Statement 객체 생성 : Statement 객체는 작성된 SQL 쿼리문을 실행하기 위한 객체로 정적 SQL 쿼리 문자열을 입력으로 가진다.
4. Query 실행 : 생성된 Statement 객체를 이용하여 입력한 SQL 쿼리를 실행한다.
5. ResultSet 객체로부터 데이터 조회 : 실행된 SQL 쿼리문에 대한 결과 데이터 셋이다.
6. ResultSet, Statement, Connection 객체들의 Close : JDBC API를 통해 사용된 객체들은 생성된 객체들을 사용한 순서의 역순으로 Close 한다.

### **커넥션 풀(Connection Pool)**

JDBC API를 사용하여 데이터베이스와 연결하기 위해 Connection 객체를 생성하는 작업은 비용이 많이 드는 작업 중 하나이다.

**커넥션 객체를 생성하는 과정**

1. 애플리케이션에서 DB 드라이버를 통해 커넥션을 조회한다.
2. DB 드라이버는 DB와 TCP/IP 커넥션을 연결한다. (3 way handshake와 같은 네트워크 연결 동작 발생)
3. DB 드라이버는 TCP/IP 커넥션이 연결되면 아이디와 패스워드, 기타 부가 정보를 DB에 전달한다.
4. DB는 아이디, 패스워드를 통해 내부 인증을 거친 후 내부에 DB를 생성한다.
5. DB는 커넥션 생성이 완료되었다는 응답을 보낸다.
6. DB 드라이버는 커넥션 객체를 생성해서 클라이언트에 반환한다.

이처럼 커넥션을 새로 만드는 것은 비용이 많이 들며, 비효율적이다.

이러한 문제를 해결하기 위해 애플리케이션 로딩 시점에 Connection 객체를 미리 생성하고, 애플리케이션에서 데이터베이스에 연결이 필요할 경우 미리 준비된 Connection 객체를 사용하여 애플리케이션의 성능을 향상하는 커넥션 풀(Connection Pool)이 등장하게 된다.

**Connection 객체를 미리 생성하여 보관**하고 애플리케이션이 **필요할 때 꺼내서 사용할 수 있도록 관리**해 주는 것이 **Connection Pool**이다.

**커넥션 풀 동작 구조**

![](https://blog.kakaocdn.net/dna/bJrVSk/btrRZOhiAim/AAAAAAAAAAAAAAAAAAAAAH7B0R2WuK4b6KK6NtUIATtGeIcATKmgvbhsrPB5vjqG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=iHClAvae%2Fzspy%2FaEZO%2BCGdohM5g%3D)

- 애플리케이션을 시작하는 시점에 커넥션 풀은 필요한 만큼 커넥션을 미리 생성하여 보관한다.
- 서비스의 특징과 스펙에 따라 생성되는 Connection 객체의 개수는 다르지만 일반적으로 기본값으로 10개를 생성한다.
- 커넥션 풀에 들어있는 Connection 객체는 TCP/IP로 DB와 연결되어 있는 상태이기 때문에 즉시 SQL을 DB에 전달할 수 있다.
- 즉, DB 드라이버를 통해 새로운 커넥션을 획득하는 것이 아닌 이미 생성되어 있는 커넥션을 참조하여 사용하게 된다.
- 커넥션 풀에 있는 커넥션을 요청하면 커넥션 풀은 자신이 가지고 있는 커넥션 객체 중 하나를 반환한다.

> Thread Pool과 유사하지만, 관리하는 게 다르다.
> Thread Pool은 스레드 관리, Connection Pool은 DB 연결 객체 관리. 당연하게도 Thread Pool이 Connection Pool보다 커야 한다.
> DI 컨테이너와도 유사한 구조이다. 컴파일 타임에 자동으로 커넥션 확인 후 여기서는 미리 연결 객체 생성...
> 보이는 것처럼 당연하게도 싱글톤 패턴이다.

따라서, DB 드라이버를 통해 커넥션을 조회, 연결, 인증, SQL을 실행하는 시간 등 커넥션 객체를 생성하기 위한 과정을 생략할 수 있게 된다.

Spring Boot 2.0 이전 버전에서는 Apache 재단의 오픈 소스인 Apache Commons DBCP를 주로 사용하였지만, 스프링 부트 2.0 이후 HikariCP를 기본 DBCP로 채택하여 사용되고 있다.

### **HikariCP란?**

HikariCP는 가벼운 용량과 빠른 속도를 가지는 우수한 성능의 JDBC Connection Pool Framework이다.
!! **Connection Pool**이다. JDBC 관리를 위한 라이브러리임을 기억하자.

스프링 부트 2.0 이후부터는 커넥션 풀을 관리하기 위해 HikariCP를 사용하고 있다.

![](https://blog.kakaocdn.net/dna/cbcUAA/btrR3Bt4B0C/AAAAAAAAAAAAAAAAAAAAAH2-woOBvebAMVezI8YsPpb3XnEoPnp2NOjvXInGZ494/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1753973999&allow_ip=&allow_referer=&signature=vBj0vqF%2FrrEOwzaBHi76PcZNiX4%3D)

HikariCP는 미리 정해놓은 크기만큼의 Connection을 Connection Pool에 담아 놓는다.

이후 요청이 들어오면 Thread가 Connection을 요청하고, Connection Pool에 있는 Connection을 연결해 준다.


# JDBC 단점?

1. SQL 의존적이다.
   개발자가 직접 쿼리를 작성해야 하며, SQL을 소스코드 내에서 작성해야 한다.
2. connection 관리를 해야 한다.
   connection 객체를 개발자가 관리해야 한다.
3. preparestatement가 sql을 전달하며, resultset을 통해 결과값을 전달한다.

버퍼를 통해서 SQL, 결과값을 날린다.


# JPA
DB - jdbc api - jpa api - java application
형태로, jdbc api와 java app 사이에 위치해 있다.

sql 데이터를 쉽게 접근하도록 한다.
장점
1. Java 코드 내에서 SQL 쿼리를 직접 작성할 필요가 줄어든다. 대신 엔티티 기반의 메서드 호출만으로 대부분의 데이터 조작이 가능하다.
2. sql 구조를 java app 내에서 적용하지 않아도 된다.

@ManyToOne 등의 어노테이션을 지원해서 Join 쿼리 같은 걸 미리 해준다.
이렇게 해주면 findbyId 등을 사용하지 않고 그냥 멤버 객체로도 조회가 가능해진다...

JPA도 JDBC와 마찬가지로 인터페이스이기 때문에 구현체가 필요하고, 그 구현체중 하나가 Hibernate(가장 널리 사용됨)이다.

왜 jpa 하이버네이트는 끔찍한가?
I followed Hibernate ORM to hell and came back alive to tell about it
-> 현재는 글이 삭제되었다.
하이버네이트가 왜 내 커리어를 망쳤는가... 등 직접 경험한 글들을 구글링 가능하다고 함.

이건 테코톡에서 직접 찾아보라고 한다.

# Spring Data JPA
Spring Data JPA는 JPA를 Repository 기반으로 간편하고 효율적으로 사용할 수 있는 모듈이라고 설명되어 있다.

Spring Data JPA는 Repository의 메소드를 통해 쿼리를 날릴 수 있다.

쿼리는 Repository의 메소드 명을 통하여 생성되는데, 컨벤션이 존재한다.
또한 쿼리를 직접적으로 만들고 싶다면 @Query 어노테이션을 사용한다.
이 때 `JPQL` 혹은 일반 쿼리를 사용할 수 있고 nativeQuery의 값에 따라 선택 가능하다.

`JPQL`: JPA에서 지원하는 SQL 추상화 객체 지향 쿼리 언어
SQL과 유사한 문법을 가지지만, 데이터베이스 테이블이 아닌 엔티티 객체를 대상으로 쿼리를 수행하는 객체지향 쿼리 언어이다.

**JPQL 사용 예시**

```java
String jpql = "select m From Member m where m.name like ‘%hello%'";
List<Member> result = em.createQuery(jpql, Member.class).getResultList();

for (Member member : result) {
    System.out.println("member = " + member);
}
```

> JPA는 JPQL 뿐만 아니라 다양한 검색 방법을 제공한다. 다음은 JPA가 공식 지원하는 기능이다.

- JPQL
- Criteria Query : JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음.
- Native SQL : JPA에서 JPQL 대신 직접 SQL을 사용하는 기능

다음은 공식 지원하는 기능은 아니나 알아둘 가치가 있다.

> Criteria 쿼리, QueryDSL, 네이티브 SQL 등...
> 또는 JDBC를 직접 사용하거나, MyBatis 같은 SQL 매퍼 프레임워크를 사용할 수도 있다.

# 차이점 요약

## 개요

Spring 프레임워크는 어플리케이션을 개발할 때 필요한 수많은 강력하고 편리한 기능을 제공해준다. 하지만 많은 기술이 존재하는 만큼 Spring 프레임워크를 처음 사용하는 사람이 Spring 프레임워크에 대한 정확한 이해를 하기는 매우 어렵다.

## JPA는 기술 명세이다

JPA는 Java Persistence API의 약자로, **자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스**이다. 여기서 중요하게 여겨야 할 부분은, JPA는 말 그대로 **인터페이스**라는 점이다. JPA는 특정 기능을 하는 **라이브러리가 아니다**. 마치 일반적인 백엔드 API가 클라이언트가 어떻게 서버를 사용해야 하는지를 정의한 것처럼, JPA 역시 자바 어플리케이션에서 관계형 데이터베이스를 어떻게 사용해야 하는지를 정의하는 한 방법일 뿐이다.  
  

JPA는 단순히 명세이기 때문에 구현이 없다. JPA를 정의한 `javax.persistence` 패키지의 대부분은 `interface`, `enum`, `Exception`, 그리고 각종 `Annotation`으로 이루어져 있다. 예를 들어, JPA의 핵심이 되는 `EntityManager`는 아래와 같이 `javax.persistence.EntityManager` 라는 파일에 `interface`로 정의되어 있다.

```java
package javax.persistence;
import ...

public interface EntityManager {
	public void persist(Object entity);
	public <T> T merge(T entity);
	public void remove(Object entity);
	public <T> T find(Class<T> entityClass, Object primaryKey);    // More interface methods...
	}
```
## Hibernate는 JPA의 구현체이다
![JPA와 Hibernate의 상속 및 구현 관계](https://suhwan.dev/images/jpa_hibernate_repository/jpa_hibernate_relationship.png)

위 사진은 JPA와 Hibernate의 상속 및 구현 관계를 나타낸 것이다. JPA의 핵심인 `EntityManagerFactory`, `EntityManager`, `EntityTransaction`을 Hibernate에서는 각각 `SessionFactory`, `Session`, `Transaction`으로 상속받고 각각 `Impl`로 구현하고 있음을 확인할 수 있다.  

“Hibernate는 JPA의 구현체이다”로부터 도출되는 중요한 결론 중 하나는 **JPA를 사용하기 위해서 반드시 Hibernate를 사용할 필요가 없다**는 것이다. Hibernate의 작동 방식이 마음에 들지 않는다면 언제든지 DataNucleus, EclipseLink 등 다른 JPA 구현체를 사용해도 되고, 심지어 본인이 직접 JPA를 구현해서 사용할 수도 있다. 다만 그렇게 하지 않는 이유는 단지 Hibernate가 굉장히 성숙한 라이브러리이기 때문일 뿐이다.

## Spring Data JPA는 JPA를 쓰기 편하게 만들어놓은 모듈이다

Spring Data JPA는 Spring에서 제공하는 모듈 중 하나로, 개발자가 JPA를 더 쉽고 편하게 사용할 수 있도록 도와준다. 이는 **JPA를 한 단계 추상화시킨 `Repository`라는 인터페이스를 제공함으로써 이루어진다**. 사용자가 `Repository` 인터페이스에 정해진 규칙대로 메소드를 입력하면, Spring이 알아서 해당 메소드 이름에 적합한 쿼리를 날리는 구현체를 만들어서 Bean으로 등록해준다.  
  

Spring Data JPA가 JPA를 추상화했다는 말은, **Spring Data JPA의 `Repository`의 구현에서 JPA를 사용하고 있다**는 것이다. 예를 들어, `Repository` 인터페이스의 기본 구현체인 `SimpleJpaRepository`의 코드를 보면 아래와 같이 내부적으로 `EntityManager`을 사용하고 있는 것을 볼 수 있다.

```java
package org.springframework.data.jpa.repository.support;
import ...

public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
	private final EntityManager em;    
	public Optional<T> findById(ID id) {        
		Assert.notNull(id, ID_MUST_NOT_BE_NULL);        
		Class<T> domainType = getDomainClass();  
		      
		if (metadata == null) {            
			return Optional.ofNullable(em.find(domainType, id));
		}        
		
		LockModeType type = metadata.getLockModeType();
		Map<String, Object> hints = getQueryHints().withFetchGraphs(em).asMap();
		
		return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));    
	}
	// Other methods...
}
```

## 요약 - 셋을 혼동하지 말고 사용하자

아래 사진은 위의 내용을 요약하여 JPA, Hibernate, 그리고 Spring Data JPA의 전반적인 개념을 그림으로 표현한 것이다.

![JPA, Hibernate, Spring Data JPA의 전반적인 그림](https://suhwan.dev/images/jpa_hibernate_repository/overall_design.png)

특히 JPA와 Spring Data JPA는 똑같이 JPA가 들어가서 처음 접하는 사람은 상당히 헷갈릴 수 있다. 이 세 개념의 차이점을 정확히 인지하고 숙지하고 있으면 개발이 한층 편해질 것이다.






Referenced
https://dkswnkk.tistory.com/685
https://www.youtube.com/watch?v=Ppqc3qN75EE
https://velog.io/@pppp0722/JDBC-JPA-Hibernate-Spring-Data-JPA-%EC%B0%A8%EC%9D%B4
https://ittrue.tistory.com/270
https://suhwan.dev/2019/02/24/jpa-vs-hibernate-vs-spring-data-jpa/
https://code-lab1.tistory.com/272