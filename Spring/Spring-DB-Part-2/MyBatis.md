# 소개
- [[JdbcTemplate]] 보다 더 많은 기능을 제공하는 SQL Mapper
- JdbcTemplate이 제공하는 대부분의 기능 제공
- SQL을 XML에 편리하게 작성
- 동적 쿼리를 편리하게 작성
## 장점
### SQL 여러줄일 때
**JdbcTemplate**
```java
String sql = "update item " +
	"set item_name=:itemName, price=:price, quantity=:quantity " +
	"where id=:id";
```
**MyBatis**
```xml
<update id="update">
	update item
	set item_name=#{itemName},
		price=#{price},
		quantity=#{quantity}
	where id = #{id}
</update>
```
- xml에 작성하여 라인이 길어져도 문자 더하기에 대한 불편함이 없음
### 동적 쿼리
**JdbcTemplate**
```java
String sql = "select id, item_name, price, quantity from item";
//동적 쿼리
if (StringUtils.hasText(itemName) || maxPrice != null) {
	sql += " where";
}

boolean andFlag = false;
if (StringUtils.hasText(itemName)) {
	sql += " item_name like concat('%',:itemName,'%')";
	andFlag = true;
}

if (maxPrice != null) {
	if (andFlag) {
		sql += " and";
	}
	sql += " price <= :maxPrice";
}

log.info("sql={}", sql);
return template.query(sql, param, itemRowMapper());
```
**MyBatis**
```xml
<select id="findAll" resultType="Item">
	select id, item_name, price, quantity
	from item
	<where>
		<if test="itemName != null and itemName != ''">
			and item_name like concat('%',#{itemName},'%')
		</if>
		<if test="maxPrice != null">
			and price &lt;= #{maxPrice}
		</if>
	</where>
</select>
```
## 단점
- 설정 필요
# 사용
## 라이브러리
```properties
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'
```
![](https://i.imgur.com/VHzyKRy.png)
- `mybatis-spring-boot-starter` : MyBatis를 스프링 부트에서 편리하게 사용할 수 있게 시작하는 라이브러리
- `mybatis-spring-boot-autoconfigure` : MyBatis와 스프링 부트 설정 라이브러리
- `mybatis-spring` : MyBatis와 스프링을 연동하는 라이브러리
- `mybatis` : MyBatis 라이브러리
## 설정
```properties
#MyBatis  
mybatis.type-aliases-package=hello.itemservice.domain  
mybatis.configuration.map-underscore-to-camel-case=true  
logging.level.hello.itemservice.repository.mybatis=trace
```
- `mybatis.type-aliases-package`
	- MyBatis에서 타입 정보를 사용할 때는 패키지 이름을 적어주어야 하는데, 여기에 명시할 경우 패키지 이름을 생략 가능
	- 지정한 패키지와 그 하위 패키지 자동 인식
	- 여러 위치 지정 시 `,`, `;`로 구분
- `mybatis.configuration.map-underscore-to-camel-case`
	- 언더바를 카멜로 자동 변경해주는 기능을 활성화 (like JdbcTemplate의 `BeanPropertyRowMapper`)