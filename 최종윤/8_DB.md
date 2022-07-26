스프링  -  DB연동할때 스프링의 특징, 사용하는 기술,
JDBC 프로그래밍의 단점을 보완한다.   - 스프링이 제공하는 JdbcTemplate을 활용하여 DB연동 프로그래밍을 한다.
- JDBC 프로그래밍하지 않고 Template을 사용하면 반복되는 코드를 줄이기 위해 ,
JDBC API를 이용하면 사실상 데이터 처리와는 상관 없는 구조적인 코드가 반복된다.
이를 줄이기 위해 템플릿 메서드 패턴과 전략 패턴을 함께 사용한다.   
-  ? Jdbc API 이용할때 반복되는 코드를 줄이기 위해 사용되는  템플릿 메서드 패턴과 전략 패턴이 무엇인가요 ?
스프링은 이 두 패턴을 엮은 JdbcTemplate을 제공한다.


- 스프링 사용할 떄 장점 (트랜잭션 관리)
스프링의 장점 중 하나는 트랜잭션 관리가 쉽다는 것이다. 커밋과 롤백 처리는 스프링이 알아서 처리해준다.
-  ?커밋과 롤백 처리가 무엇인가요? 
스프링을 사용하면 트랜잭션을 적용하고 싶은 메서드에 @Transactional 애노테이션을 붙이기만 하면 된다.



커넥션 풀이란?   - 스프링에서 DB연동할때 동시접속자가 많을때 부하를 줄이기 위해 사용하는 것 
실제 서비스 운영 환경에서는 서로 다른 장비를 이용해서 자바 프로그램과 DBMS를 실행한다. 
- 커넥션 비용이 커서 커넥션 풀을 이용해 미리 만든다.
자바 프로그램에서 DBMS로 커넥션을 생성하는데 비용이 매우 크다.

최초 연결에 따른 응답 속도 저하와 동시 접속자가 많을 때 발생하는 부하를 줄이기 위해 사용하는 것이 커넥션 풀이다. 커넥션 풀은 일절 개수의 DB 커넥션을 미리 만들어두는 기법이다. DB 커넥션이 필요한 프로그램은 커넥션 풀에서 커넥션을 가져와 사용한 뒤 커넥션을 다시 풀에 반납한다. 커넥션을 미리 생성해두기 때문에 커넥션을 사용하는 시점에서 커넥션을 생성하는 시간을 아낄 수 있다. 또한 동시 접속자가 많더라도 커넥션을 생성하는 부하가 적기 때문에 더 많은 동시 접속자를 처리할 수 있다. 커넥션도 일정 개수로 유지해서 DBMS에 대한 부하를 일정 수준으로 유지할 수 있게 해준다.



DataSource 설정  -  DB연동할 때 connection 구하는 법 
스프링이 제공하는 DB 연동 기능은 DataSource를 사용해서 DB Connection을 구한다.
- datasource  구현법
DB 연동에 사용할 DataSource를 스프링 빈으로 등록하고 DB 연동 기능을 구현한

빈 객체는 DataSource를 주입받아 사용한다.


- 커넥션 풀   구현
커넥션 풀은 커넥션을 생성하고 유지한다.
- 커넥션 풀에 커넥션을 요청하면 해당 커넥션은 활성(active) 상태가 되고,

커넥션을 다시 커넥션 풀에 반환하면 유휴(idle) 상태가 된다.


- ?datasource 빈에서 꺼내야 하는거 아닌가
public class DbQuery {

    private DataSource dataSource;

    public DbQuery(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public int count() {
        Connection conn = null;
        try {
            conn = dataSource.getConnection(); // 풀에서 구함
            try (Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("select count(*) from MEMBER")) {
                rs.next();
                return rs.getInt(1);
            }
        } catch (SQLException e) {
            throw new RuntimeException(e);
        } finally {
            if (conn != null) {
                try {
                    conn.close(); // 풀에 반환
                } catch (SQLException e) {

                }
            }
        }
    }
}
maxActive는 활성 상태가 가능한 최대 커넥션 개수를 지정한다. 활성 상태 커넥션이 40개인데 풀에 다시 커넥션을 요청하면 다른 커넥션이 반환될 때까지 대기한다. 이 대기 시간이 maxWait이다. 대기 시간 내에 풀에 반환된 커넥션이 있으면 해당 커넥션을 구하고, 대기 시간내에 반환된 커넥션이 없으면 익셉션이 발생한다.

커넥션 풀을 초기화할 때 최소 수준의 커넥션을 미리 생성하는 것이 좋다. 이때 생성할 커넥션 개수를 initialSize로 정한다.

커넥션 풀에 생성된 커넥션은 지속적으로 재사용된다. DBMS 설정에 따라 일정 시간 내에 쿼리를 실행하지 않으면 연결을 끊기도 한다. 커넥션 풀에 특정 커넥션이 5분 넘게 유휴 상태로 존재한다고 하자. 이 경우 DBMS는 해당 커넥션의 연결을 끊지만 커넥션은 여전히 풀 속에 남아있다. 이 상태에서 해당 커넥션을 풀에서 가져와 사용하면 연결이 끊어진 커넥션이므로 익셉션이 발생한다.

커넥션 풀의 커넥션이 유효한지 주기적으로 검사해야하는데, 이와 관련된 속성이 minEvictableidleTimeMills, timeBetweenEvictionRunsMillstestWhileIdle이다.


JdbcTemplate을 이용한 쿼리 실행
스프링을 사용하면 DataSource나 Connection, Statement, ResultSet을 직접 사용하지 않고 
JdbcTemplate을 이용해서 편리하게 쿼리를 실행할 수 있다.
DataSource를 컨테이너에서 가져와야 하는거 아닌감 
```
public class MemberDao {
    
    private JdbcTemplate jdbcTemplate;

    public MemberDao(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }
}


@Configuration
public class AppCtx {

    

    @Bean
    public MemberDao memberDao() {
        return new MemberDao(dataSource());
    }
}

```
JdbcTemplate을 이용한 조회 쿼리 실행
JdbcTemplate 클래스는 SELECT 쿼리 실행을 위한 query() 메서드를 제공한다.
query() 메서드는 sql 파라미터로 전달받은 쿼리를 실행하고
RowMapper를 이용해서 ResultSet의 결과를 자바 객체로 변환한다.
```
RowMapper 인터페이스
public interface RowMapper<T> {
    T mapRow(ResultSet rs, int rowNum) throws SQLException;
}
```
RowMapper의 mapRow() 메서드는 SQL 실행결과로 구한 ResultSet에서 한 행의 데이터를 읽어와 
자바 객체로 변환하는 매퍼 기능을 구현한다. RowMapper 인터페이스를 구현한 클래스를 작성할 수도 있지만
임의 클래스나 람다식으로 RowMapper의 객체를 생성해서 query() 메서드에 전달할 때도 많다.


MemberDao
query()와 rowmapper를 사용하여 구현한다. 

public class MemberDao {
    public Member selectByEmail(String email) {
        List<Member> results = jdbcTemplate
            .query("select * from MEMBER where EMAIL = ?", new RowMapper<Member>(){
                @Override
                public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
                    Member member = new Member(
                        rs.getString("EMAIL"),
                        rs.getString("PASSWORD"),
                        rs.getString("NAME"),
                        rs.getTimestamp("REGDATE").toLocalDateTime());
                    return member;
                }
            }, email);
        return results.isEmpty() ? null : results.get(0);
    }
    
    ... 
}
MemberDao

package spring;

public class MemberDao {

    ??? 어 ㅏ아 아앙아아아아아아아아아아아
query로 조회 select query를 실행해서 가져오고 , mapRow()로 각 요소를 가져온다. 
    public List<Member> selectAll() {
        List<Member> results = jdbcTemplate.query("select * from MEMBER", new RowMapper<Member>() {
            @Override
            public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
                Member member = new Member(
                    rs.getString("EMAIL"),
                    rs.getString("PASSWORD"),
                    rs.getString("NAME"),
                    rs.getTimestamp("REGDATE").toLocalDateTime());
                return member;
            }
        });
        return results;
    }
}
    public int count() {
        Integer count = jdbcTemplate.queryForObject("select count(*) from MEMBER", Integer.class);
        return count;
    }

스프링의 익셉션 변환 처리
SQL 문법이 잘못됐을 때 발생한 메세지를 보면 익셉션 클래스가 org.spring.framework.jdbc 패키지에 속한 BadSqlGrammarException 클래스임을 알 수 있다.

JDBC API를 사용하는 과정에서 SQLException이 발생하면 이 익셉션을 알맞은 DataAccessException으로 변환해서 발생한다.

예를 들어 MySQL용 JDBC 드라이버는 SQL 문법이 잘못된 경우 SQLException을 상속받은 MySQLSyntaxErrorException을 발생시키는데 JdbcTemplate은 이 익셉션을 DataAccessException을 상속받은 BadSqlGrammarExcepation으로 변환한다.

그렇다면 스프링은 왜 SQLException을 그대로 전파하지 않고 SQLException을 DataAccessException으로 변환할까?

주된 이유는 연동 기술에 상관없이 동일하게 익셉션을 처리할 수 있도록 하기 위함이다. 스프링은 JDBC뿐만 아니라 JPA, 하이버네이트 등에 대한 연동을 지원하고 MyBatis는 자체적으로 스프링 연동 기능을 제공한다. 그런데 각각의 구현기술마다 익셉션을 다르게 처리해야 한다면 개발자는 기술마다 익셉션 처리 코드를 작성해야 할 것이다.



트랜잭션 처리
두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용하는 것이 트랜잭 션(transaction)이다. 트랜잭션은 여러 쿼리를 논리적으로 하나의 작업으로 묶어준다. 한 트랜잭션으로 묶인 쿼리 중 하나라도 실패하면 전체 쿼리를 실패로 간주하고 실패 이전에 실행한 쿼리를 취소한다.
쿼리 실행 결과를 취소하고 DB를 기존 상태로 되돌리는 것을 롤백(rollback)이라고 부른다. 반면에 트랜잭션으로 묶인 모든 쿼리가 성공해서 쿼리 결과를 DB에 실제로 반영하는 것을 커밋(commit)이라고 한다.

JDBC는 Connection의 setAutoCommit(false)를 이용해서 트랜잭션을 시작하고 commit()과 rollback()을 이용해서 트랜잭션을 반영(커밋)하거나 취소(롤백)한다.

@Transactional을 이용한 트랜잭션 처리
스프링이 제공하는 @Transactional 애노테이션을 사용하면 트랜잭션 범위를 매우 쉽게 지정할 수 있다.

스프링은 @Transactional 애노테이션이 붙은 changePassword() 메서드를 동일한 트랜잭션 범위에서 실행한다. @Transactional 애노테이션이 제대로 동작하려면 다음의 두 가지 내용을 스프링 설정에 추가해야 한다.

플랫폼 트랜잭션 매니저(PlatformTransactionManager) 빈 설정
@Transactional 애노테이션 활성화 설정
