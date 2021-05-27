## HikariCp ?
- 매우 가볍고 빠르고 안정적인 JDBC Connection Pool

- Spring boot 2.0 부터 default JDBC Connection Pool
- Database connection pool 관리 해주는 역할
- 미리 정해놓은 만큼의 connection을 pool에 담아 놓고 요청이 들어왔을 때 pool 내 있는 connection 연결해준다.

### Connection Pool 이란?
<img width="714" alt="스크린샷 2021-05-27 오후 11 23 18" src="https://user-images.githubusercontent.com/34999925/119843540-9d231180-bf42-11eb-9372-3c721ddd540d.png">  

#

### HikariCp 내부 동작 과정
(출처 : [우아한형제들 기술블로그](https://woowabros.github.io/experience/2020/02/06/hikaricp-avoid-dead-lock.html))
```java
  Connection connection = null;
  PreparedStatement preparedStatement = null

  try {
    connection = hikariDataSource.getConnection();
    preparedStatement = connection.preparedStatement(sql);
    preparedStatement.executeQuery();
  } catch(Throwable e) {
    throw new RuntimeException(e);
  } finally {
    if (preparedStatement != null) {
      preparedStatement.close();
    }
    if (connection != null) {
      connection.close(); // 여기서 connection pool에 반납
    }
  }
```

- HikariPool에서는 Connection 객체를 한번 wrapping한 `PoolEntry`라는 Type으로 내부적으로 Connection을 관리

- connectionBag : Connection Entry를 담고 있는 객체 / connectionBag을 이용해 connection 을 관리한다.

- `HikariPool.getConnection() -> ConcurrentBag.borrow()`라는 메서드를 통해 사용 가능한(idle) Connection을 리턴

- HikariCP에서 얻은 Connection은 `(ProxyConnection) Connection.close()`를 하게 되면 HikariPool에 반납

    (정상적으로 transaction이 commit 되거나, 에러로 인해 rollback이 호출 되면 connection.close()가 호출되어 Connection을 Pool에 반납)


**connection 얻는 과정**

<img width="600" alt="스크린샷 2021-05-27 오후 11 52 06" src="https://user-images.githubusercontent.com/34999925/119848199-95656c00-bf46-11eb-89fd-4f2ce1ab8f14.png">

**connection 반납 과정**

<img width="600" alt="스크린샷 2021-05-27 오후 11 57 11" src="https://user-images.githubusercontent.com/34999925/119849086-4966f700-bf47-11eb-9f96-c334353c7cb9.png">