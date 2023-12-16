# 트랜잭션

## 트랜잭션이란?

소프트웨어 개발 영역에서 트랜잭션은 데이터베이스의 상태를 변화시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위이다.

여기서 데이터베이스의 상태를 변환시킨다는 것은 무슨 의미인가?

질의어(SQL)를 이용해 데이터베이스에 접근 하는 것을 말한다.

- SELECT
- INSERT
- DELETE
- UPDATE

그리고 작업 단위는 사람이 정하는 기준에 따라서 정하는 것을 의미 한다.

예를 들면 A 사용자가 계좌에서 5만원을 B사용자에게 송금한다고 했을때

<img width="1269" alt="스크린샷 2023-08-20 오후 5 34 45" src="https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/4f282284-06b7-4887-b7ae-a96731a9dc1a">

이렇게 A작업,B작업 2개로 작업 단위를 나누게 된다.

만약 작업중에 A작업,B작업 사이에 오류가 발생하면,

이미 A작업은 처리된 시점이라 문제가 발생한다.

rollback을 했을때 A 작업 끝난 시점으로 돌아간다. 그러면 5만원 차감만 되는 문제가 발생한다.

이런 문제를 작업 단위로 직접 지정해서 해결한다.

<img width="1269" alt="스크린샷 2023-08-20 오후 5 51 22" src="https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/fe0da11e-bf45-46bd-a6b2-611d76543fa6">

auto-commit을 false로 해두었기 때문에 A 작업을 진행하던 도중에 어디에서든 문제가 발생해도 맨 처음 “A계좌 조회” 전의 시작단계로 rollback하게 된다.

이런 은행 업무 예를 보면 하나의 작업 단위, 트랜잭션으로 볼수 있고, 트랜잭션으로 데이터 베이스 안정성을 확보 할수 있게 되는것이다.

- **Commit** 연산: 하나의 트랜잭션이 성공적으로 끝나서 데이터베이스가 일관성 있는 상태에 있음을 의미한다.
- **Rollback** 연산: 하나의 트랜잭션 처리가 비정상적으로 종료되어 트랜잭션의 원자성이 깨진 상태를 의미한다.

### Transactional의 속성 ACID

트랜잭션은 특히 데이터베이스 관련 작업에서 무결성과 일관성을 유지하는게 중요한데, “ACID” 속성이 트랜잭션을 안정적으로 실행되도록 해준다.

- **Atomicity(원자성)**
    - 트랜잭션 내의 모든 명령은 반드시 완벽히 수행되어야 한다.(부분적으로 실행되거나 중단되지 않는 것을 보장함)
    - 어느 하나라도 오류가 발생하면 트랜잭션 전부가 취소되어야 한다.
    - 트랜잭션의 연산은 모두 반영되도록 commit 되거나 전혀 반영되지 않도록 rollback 되어야 한다.
- **Consistency(일관성)**
    - 트랜잭션의 작업 처리 결과는 항상 일관성이 있어야 한다.
- **Isolation(독립성, 격리성)**
    - 트랜잭션 수행 시 다른 트랜잭션 연산에 끼어들지 못하도록 보장한다.
    - 수행 중인 트랜잭션은 완전히 완료될 때까지 다른 트랜잭션에서의 수행 결과를 참조할 수 없다.
- **Durability(영속성, 지속성)**
    - 성공적으로 완료된 트랜잭션의 결과는 시스템이 고장 나더라도 영구적으로 반영되어야 한다.

### 트랜잭션 상태

<img width="1130" alt="스크린샷 2023-08-20 오후 6 09 19" src="https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/bd6ddf4c-ba4f-4172-af66-dc4357209b81">

- Active : 트랜잭션이 실행 중이 상태 (활동)
- Failed : 트랜잭션이 실행에 오류가 발생하여 중단된 상태 (실패)
- Aborted : 트랜잭션이 비정상적으로 종료 되어 rollback 연산을 수행한 상태(철회)
- Patially Committed : 트랜잭션의 마지막 연산까지 실행 했지만, Commit 연산이 실행되기 직전의 상태 (부분완료)
- Committed : 트랜잭션이 성공적으로 종료 되어 commit 연산을 실행한 후의 상태 (완료)

## @Transactional 이란?

비즈니스 로직이 트랜잭션 처리를 필요 할 때 트랜잭션 처리 코드가 비즈니스 로직과 공존한다면 코드 중복이 발생하고 비즈니스 로직에 집중 또한 힘들어 질수 있다.

`@Transational`  이 어노테이션은 스프링에서 제공해주는 선언적 트랜잭션 방식으로 `getConnetction()`, `setAutoCommit(false)`, 예외 발생시 rollback, 정상 종료 시에 commit 등의 필요한 코드를 삽입해준다.

즉, 내부적으로 AOP를 통해 트랜잭션 처리 코드가 전후로 수행되는 것이다.

### 동작원리

아래 코드를 보면 Spring에서 자주 사용되는 Transaction management 방식이다.

```java
@Service
public class UserService {
    @TranSactional
    public Long registerUser(User user) {
        // 예를 들어 일부 SQL을 실행한다.
        // 사용자를 db에 삽입하고 자동 생성된 ID를 검색한다.
        // userRepository.save(user);
        return id;
    }
}
```

`@Transactional`을 붙이면,

Spring Configuration에 `@EnableTransactionManagement` 어노테이션을 붙인다.(스프링 부트에서는 자동으로 해줌)

Spring Configuration에서 트랜잭션 매니저를 지정한다.

이렇게만 해주면 스프링은 트랜잭션을 처리해준다. 즉, `@Transactional` 어노테이션이 달린 public 메서드에 대해서 내부적으로 데이터베이스 트랜잭션 처리를 해준다.

따라서 `@Transactional`이 적용되면 코드는 다음과 같아진다.

```java
@Configuration
@EnableTransactionManagement
public class MySpringConfig {
    @Bean
    public PlatformTransactionManager txManager() {
        return yourTxManaher;
    }
}
```

따라서 `@Transactional`이 쓰인 UserService 코드를 간단히 변환하면 아래 코드와 같다.

```java
@Service
public class UserService {
    public Long registerUser(User user) {
        Connection connection = dataSource.getConnection(); // (1)
        try (connection) {
            connection.setAutoCommit(false); // (1)
            // 예를 들어 일부 SQL을 실행한다.
            // 사용자를 db에 삽입하고 자동 생성된 ID를 검색한다.
            // userRepository.save(user); (2)
            connection.commit(); // (1)    
        } catch (SQLException e) {
            connection.rollback(); // (1)
        }
    }
}
```

(1) `@Transactional`  추가하면 JDBC에서 필요한 코드를 알아서 자동 삽입해준다.

Connection도 가져오고, setAutoCommit(false) 추가해주고, 해당 메소드가 끝나면 commit, 예외가 발생하면 rollback까지 해준다.

(2) userRepository에 user를 저장하는 코드 부분

### @**Transactional - CGLIB와 JDK Proxies**

스프링이 실제로 내가 작성한 자바 코드에 추가로 작성을 할수 없다.

위에서 나온 registerUser() 메소드를 호출하면 `userRepository.save(user)`를 호출하는 것은 바뀌지 않는다.

그 대신에 Spring은 IoC 컨테이너를 **활용**하는데, 우리가 `@Transactional`을 사용하면 UserSevice를 초기화 할 뿐만 아니라, UserServiceProxy 또한 초기화 하는 것이다.

CGLIB 라이브러리의 도움을 받아 만든 proxy를 사용하면 마치 실제 UserService 코드에서 보여준 Transaction 코드를 추가하여 사용하는 것처럼 동작하게 된다.

<img width="1292" alt="스크린샷 2023-08-20 오후 7 33 15" src="https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/50fa5492-0e56-45ee-b5f8-42decf41c030">

그림을 보면 Proxy는 일 한가지만 한다.

데이터베이스의 connections/transactions를 열거나 닫고, 실제 UserService에게 나머지 역할을 위임한다.

그러면 이를 사용하는 UserRestController 에서는 이게 Proxy인지 진짜인지 알수 없다.

### **Transaction Manager 필요한 이유**

예시에서 보여준 코드엔 proxy도 적용되었고, 심지어 proxy가 트랜잭션을 관리하고 있다.

하지만 트랜잭션의 상태(open,commit,close)를 proxy가 스스로 알아서 결정할 수는 없다.

따라서 Proxy는 이에 대한 결정을 Transaction Manager에게 위임한다.

Spring은 기본적으로 몇 가지 편리한 구현체와 함께 제공되는 **PlatformTransactionManager / TransactionManager 인터페이스**를 제공하는데, 그 중 하나가 **DatasourceTransactionManager**이다.

Spring의 Configuration부터 보면

```java
@Bean
    public DataSource dataSource() {
        return new MysqlDataSource();// (1)
    }

    @Bean
    public PlatformTransactionManager txManager() {
        return new DataSourceTransactionManager(dataSource());// (2)
    }
```

(1) 특정 데이터베이스에 종속적인 datasource를 생성하고 있다.

(2) TransactionManager를 생성하고 있는데, 트랜잭션을 관리하기 위해 dataSource를 필요로 한다.

모든 TransactionManager들은 doBegin 이나 doCommit과 같은 메소드를 가지고 있다.

DataSourceTranscationManager 코드보면

```java
public class DataSourceTransactionManager implements PlatformTransactionManager{
    @Override
    protected void doBegin(Object transaction, TransactionDefinition definition) {
        Connection newCon = obtainDataSource().getConnection();
        // ...
        newCon.setAutoCommit(false);
        // yes, that's it!
    }

    @Override
    protected void doCommit(DefaultTransactionStatus status) {
        // ...
        Connection connection = status.getTransaction().getConnectionHolder().getConnection();
        try {
            con.commit();
        } catch (SQLException e) {
            throw new TransactionSystemException("Cloud not commit JDBC transaction", ex);
        }
    }
}
```

DataSourceTransactionManager를 보면 JDBC가 transaction을 관리하는 방식과 정확하게 동일한 것을 확인 할 수 있다.

<img width="1292" alt="스크린샷 2023-08-20 오후 8 04 19" src="https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/f3379e8b-60b5-4c37-98f7-f7da64291601">

정리하면

1. 만약 Spring이 어떠한 Bean에 붙어 있는 `@Transactional`을 발견한다면, 해당 빈의 동적 proxy를 만든다.
2. proxy는 TransactionManager에 접근할 수 있으며, transactions나 connections를 열고 닫고록 요청한다.
3. TransactionManager는 JDBC 방식으로 connection을 관리할 뿐이다.
- **물리적인 Transaction과 논리적 Transaction의 차이**
    - 물리적 트랜잭션: 실제 JDBC 트랜잭션
    - 논리적 트랜잭션: 중첩된 `@Transactional`을 갖는 메서드
    

### **@Transactional Propagation Levels의 용도(전파 동작)**

전파 동작은 다른 메소드 내에서 호출될때 트랜잭션이 동작하는 방식.

@Transactional 에 옵션으로 줄수 있는 Propagation Lebel에 여러개 있다.

```java
@Transactional(propagation = Propagation.REQUIRED)
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

- Required(default): 메소드는 트랜잭션을 필요로 해서 트랜잭션을 새로 하나 열든지, 기존에 있던 거를 쓴다는 말이다. → `getConnection();` `setAutoCommit(false);` `commit();`
- Supports: 트랜잭션을 열든 말든 상관 없다. → JDBC는 아무것도 안함
- Mandatory: 스스로 트랜잭션을 열진 않을 거지만, 아무도 트랜잭션을 열지 않으면 예외를 던진다. → JDBC는 아무것도 안함
- Required_new: 온전히 나의 소유인 트랜잭션이 필요하다. → `getConnection();` `setAutoCommit(fale);` `commit();`
- Not_Supported: 트랜잭션을 사용하지 않겠다는 말이다. → JDBC는 아무것도 안함
- Never: 누군가 트랜잭션을 시작한다면 예외를 던진다. → JDBC는 아무것도 안함
- Nested: 복잡하지만 저장 점을 잡아준다. → `connection.setSavePoint();`

JDBC는 아무것도 안하는게 대부분이다.

Spring과 프로그램을 어떻게 구성하는지, 언제어디서 어떻게 등등 트랜잭션이 적용될지를 예상할 수 있다.

### Isolation Lebels 용도 (격리 수준)

트랜잭션이 서로 상호 작용하는 방식을 정의.

Spring은 READ_COMMITTED, READ_UNCOMMITED, REAPEATABLE_READ 및 SERIALIZABLE과 같은 다양한 격리 수준을 지원한다.

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

위 코드와 같이 isolation 옵션을 지정해 주면 proxy에서 아래와 같이 만들어 준다.

```java
connection.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
```

트랜잭션 중에 격리 수준을 전환할 때, 사용하고 있는 데이터베이스나 JDBC 드라이버에서 기능이 지원되는지 먼저 확인해야 한다. 

각 데이터베이스 회사마다 지원하는 기본 격리 수준이 다르기 때문이다.

### 주의사항

- private 메서드는 트랜잭션의 대상이 될 수 없다.
- Spring에서 트랜잭션은 처음으로 호출하는 메서드나 클래스의 속성을 따라간다.
    - 따라서 동일한 빈 안에서 하위 메서드에서만 트랜잭션이 정의되어 있으면, 전이되지 않는다.
    - 클래스를 분리하거나, 상위 메서드에 트랜잭션을 설정해야 한다.

```java
public void test1() { test2();
}
@Transactional
public void test2() { // 상위 메서드인 test1이 타깃 오브젝트가 되어 트랜잭션 적용 안됨
}
```

- RuntimeException이나 Error의 경우에만 실패시 rollback이 된다. Exception의 경우 rollbackFor 옵션을 주어 처리하는 방법도 있다.

## 정리

- `@Transactional` 어노테이션으로 활성화된 Spring의 트랜잭션 관리는 복잡한 트랜잭션 처리 작업을 단순화한다.
- **Spring에 트랜잭션 관리를 맡기면서 비즈니스로직에 집중할 수 있다.**
- 격리 수준, 전파 동작, 중첩된 트랜잭션 및 기본 프록시 기반 메터니즘을 이해하면 좀 더 안정적이고 좋은 애플리케이션을 구축할 수 있다.

# 참고

- https://mommoo.tistory.com/62
- https://cocoon1787.tistory.com/808
- https://hwannny.tistory.com/98
- https://blogshine.tistory.com/291
- [https://sasca37.tistory.com/267#article-4--@transcational-동작-원리](https://sasca37.tistory.com/267#article-4--@transcational-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)
