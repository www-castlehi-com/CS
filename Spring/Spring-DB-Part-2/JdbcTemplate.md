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
# 이름 지정 바인딩
## 배경 
- 파라미터를 순서대로 바인딩
- SQL의 코드의 순서를 변경할 경우 문제 발생
## 사용법
- `Map`처럼 `key`, `value` 데이터 구조를 만들어 전달
	- `key` : `:파라미터이름`으로 지정한 파라미터 이름
	- `value` : 해당 파라미터의 값
- `template.update(sql, param, keyHolder);`로 `param` 전달
## 파라미터 종류
### 1️⃣ Map
- `Map` 사용
```java
Map<String, Object> param = Map.of("id", id);
Item item = template.queryForObject(sql, param, itemRowMapper());
```
### 2️⃣ SqlParameterSource
#### 1. MapSqlParameterSource
- `Map`과 유사하지만, SQL 타입을 지정할 수 있는 등 SQL에 좀 더 특화된 기능 제공
- 메서드 체인을 통한 사용 제공
```java
SqlParameterSource param = new MapSqlParameterSource()  
       .addValue("itemName", updateParam.getItemName())  
       .addValue("price", updateParam.getPrice())  
       .addValue("quantity", updateParam.getQuantity())  
       .addValue("id", itemId);
template.update(sql, param);
```
#### 2. BeanPropertySqlParameterSource
- 자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체 생성
- `getXxx() -> xxx`
```java
SqlParameterSource param = new BeanPropertySqlParameterSource(cond);
KeyHolder keyHolder = new GeneratedKeyHolder();
template.update(sql, param, keyHolder);
```
## 결과 매핑
### BeanPropertyRowMapper
- `ResultSet`의 결과를 받아서 자바빈 규약에 맞추어 데이터 변환
- 리플렉션 기능 사용
#### 별칭
- `as`를 사용해 SQL 조회 결과의 이름을 변경
- 데이터베이스 컬럼 이름과 객체 이름이 완전히 다를 때 문제 해결
#### 관례의 불일치
- 자바 객체는 카멜 (`camelCase`) 표기법 사용
- 관계형 데이터베이스에서는 스네이크 (`snake_case`) 표기법 사용
- 스네이크 표기법을 카멜 표기법으로 자동 변환
# SimpleJdbcInsert
- INSERT SQL을 직접 작성하지 않도록 기능 제공
- 생성 시점에 데이터베이스 테이블의 메타 데이터 조회
## 생성
```java
this.jdbcInsert = new SimpleJdbcInsert(dataSource)  
       .withTableName("item")  
       .usingGeneratedKeyColumns("id");  
       // .usingColumns("item_name", "price", "quantity"); // 생략 가능
```
- `withTableName` : 데이터를 저장할 테이블 명 지정
- `usingGeneratedKeyColumns` : `key`를 생성하는 PK 컬럼 명 지정
- `usingColumns` : INSERT SQL에 사용할 컬럼 지정하며 특정 값만 저장하고 싶을 때 사용. 생략할 경우 전체 컬럼 사용
## 사용
```java
@Override  
public Item save(Item item) {  
    SqlParameterSource param = new BeanPropertySqlParameterSource(item);  
    Number key = jdbcInsert.executeAndReturnKey(param);  
    item.setId(key.longValue());  
  
    return item;  
}
```