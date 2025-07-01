# 특징
## 장점
1. **설정의 편리함**
	- `spring-jdbc` 라이브러리에 포함
		- 스프링으로 JDBC를 사용할 때 기본으로 사용되는 라이브러리
	- 별도의 복잡한 설정 없이 바로 사용 가능
2. **반복 문제 해결**
	- 템플릿 콜백 패턴 사용하여, 반복 작업 대신 처리
		- 커넥션 획득
		- `statement` 준비, 실행
		- 결과 반복하도록 루프 실행
		- 커넥션 종료, `statement`, `resultset` 종료
		- 커넥션 동기화
		- 예외 발생시 스프링 예외 변환기 실행
## 단점
- 동적 SQL 해결 어려움
# 사용
## 생성자
```java
private final JdbcTemplate template;  
  
public JdbcTemplateItemRepositoryV1(DataSource dataSource) {  
    this.template = new JdbcTemplate(dataSource);  
}
```
- 데이터소스(dataSource)를 주입 받아 사용
- 스프링 빈으로 직접 등록하고 주입받아도 됨
## 저장
```java
template.update(connection -> {  
    // 자동 증가 키  
    PreparedStatement ps = connection.prepareStatement(sql, new String[] {"id"});  
    ps.setString(1, item.getItemName());  
    ps.setInt(2, item.getPrice());  
    ps.setInt(3, item.getQuantity());  
  
    return ps;  
}, keyHolder);  
  
long key = keyHolder.getKey().longValue();  
item.setId(key);
```
- 데이터 변경 시 `update()` 사용
- `KeyHolder`, `prepareStatement`를 사용해서 INSERT 쿼리 실행 이후에 데이터베이스에서 생성된 ID 값을 조회
	- PK 생성에 `identity` 방식을 사용하기 때문에 데이터베이스가 ID를 대신 생성하여 INSERT가 완료되어야 PK를 확인할 수 있음
	- 순수 JDBC 로도 가능
	- 보통 `SimpleJdbcInsert` 사용
## 조회
```java
private RowMapper<Item> itemRowMapper() {  
    return ((rs, rowNum) -> {  
       Item item = new Item();  
       item.setId(rs.getLong("id"));  
       item.setItemName(rs.getString("item_name"));  
       item.setPrice(rs.getInt("price"));  
       item.setQuantity(rs.getInt("quantity"));  
       return item;  
    });  
}
```
- `template.queryForObject()`
	- `<T> T queryForObject(String sql, RowMapper<T> rowMapper, Object... args) throws DataAccessException;`
	- 결과 로우가 하나일 때 사용
	- `RowMapper`는 데이터베이스의 반환 결과인 `ResultSet`을 객체로 변환
		- 결과가 없으면 `EmptyResultDataAccessException` 예외 발생
		- 결과가 둘 이상이면 `IncorrectResultSizeDataAccessException` 예외 발생
- `template.query()`
	- `<T> List<T> query(String sql, RowMapper<T> rowMapper, Object... args) throws DataAccessException;`
	- 결과가 하나 이상일 때 사용
	- `RowMapper`는 데이터베이스의 반환 결과인 `ResultSet`을 객체로 변환
	- 결과가 없으면 빈 컬렉션 반환
# 문제
## 동적 쿼리 문제
- 각 상황에 맞추어 조건문을 써야하고 파라미터도 생성해야 함

> vs **MyBatis** : SQL을 직접 사용할 때 동적 쿼리를 쉽게 작성
