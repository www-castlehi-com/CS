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
## Mapper
```java
@Mapper  
public interface ItemMapper {  
  
    void save(Item item);  
  
    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto updateParam);  
  
    Optional<Item> findById(Long id);  
  
    List<Item> findAll(ItemSearchCond itemSearch);  
}
```
### 원리
![](https://i.imgur.com/HGYPYXg.png)
1. 애플리케이션 로딩 시점에 MyBatis 스프링 연동 모듈은 `@Mapper`가 붙어있는 인터페이스를 조사
2. 해당 인터페이스가 발견되면 동적 프록시 기술을 사용해서 `ItemMapper` 인터페이스 구현체를 생성
	- `itemMapper class=class jdk.proxy3.$Proxy70`
3. 생성된 구현체를 스프링 빈으로 등록
	- 예외 변환 처리 (`DataAccessException`) 변환
## 문법
- `#{}` : 파라미터
	- `PreparedStatement` 사용
- `useGeneratedKeys` : 데이터베이스가 키를 생성해주는 IDENTITY 전략일 때 사용
	- `keyProperty` : 생성되는 키의 속성 이름
- `resultType` : 반환 타입
- `<where>`, `<if>` : 동적 쿼리
### XML 특수문자
- < : `&lt;`
- > : `&gt;`
- & : `&amp`;
### CDATA 사용
```xml
<![CDATA[
and price <= #{maxPrice}
]]
```
# 기능
## 동적 쿼리
### 동적 SQL
#### if
```xml
<select id="findActiveBlogWithTitleLike" resultType="Blog">
	SELECT * FROM BLOG
	WHERE state = 'ACTIVE'
	<if test="title != null">
		AND title like #{title}
	</if>
</select>
```
- 해당 조건에 따라 값을 추가할지 말지 판단
#### choose, when, otherwise
```xml
<select id="findActiveBlogLike" resultType="Blog">
	SELECT * FROM BLOG WHERE state = 'ACTIVE'
	<choose>
		<when test="title != null">
			AND title like #{title}
		</when>
		<when test="author != null and author.name != null">
			AND author_name like #{author.name}
		</when>
		<otherwise>
			AND featured = 1
		</otherwise>
	</chosse>
</select>
```
- 자바의 switch 구문과 유사
#### trim, where, set
```xml
<select id="findActiveBlogLike" resultType="Blog">
	SELECT * FROM BLOG
	<where>
		<if test="state != null">
			state = #{state}
		</if>
		<if test="title != null">
			AND title like #{title}
		</if>
		<if test="author != null and author.name != null">
			AND author_name like #{author.name}
		</if>
	</where>
</select>
```
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
	...
</trim>
```
#### foreach
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
	SELECT * FROM POST P
	<where>
		<foreach item="item" index="index" collection="list" open="ID in (" separator="," close=")" nullable="true">
			#{item}
		</foreach>
	</where>
</select>
```
- 컬렉션 반복 처리 시 사용
- 파라미터로 `List` 전달
## 애노테이션
```java
@Select("select id, item_name, price, quantity from item where id=#{id}")
Optional<Item> findById(Long id);
```
- `@Insert`, `@Update`, `@Delete`, `@Select` 기능 제공
- XML은 제거해야 하므로 잘 사용하지 않음
## 문자열 대체
```java
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```
- 문자 그대로 처리하고 싶을 경우 `${}` 사용
- SQL 인젝션 공격을 당할 수 있음
## 재사용 가능한 SQL 조각
```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password</sql>
```
```xml
<select id="selectUsers" resultType="map">
	select
		<include refid="userColumns"><property name="alias" value="t1"/></include>
		<include refid="userColumns"><property name="alias" value="t2"/></include>
	from some_table t1
		cross join some_table t2
</select>
```
## Result Maps
```xml
<select id="selectUsers" resultType="User">
	select
		user_id as "id",
		user_name as "userName",
		hashed_password as "hashedPassword"
	from some_table
	where id=#{id}
</select>
```
- AS-IS : 컬럼명과 객체의 프로퍼티 명이 다를 경우 별칭 (`as`) 사용
```xml
<resultMap id="userResultMap" type="User">
	<id property="id" column="user_id">
	<result property="username" column="user_name" />
	<result property="password" column="hashed_password"/>
</resultMap>

<select id="selectUsers" resultMap="userResultMap">
	select user_id, user_name, hashed_password
	from some_table
	where id=#{id}
</select>
```
- TO-BE : `resultMap` 선언
- 복잡한 결과에 객체 연관관계를 고려해서 데이터 조회 시 `<association>`, `<collection`> 등을 사용
	- MyBatis에서는 성능과 실효성 측면에서 공수도 많이 들고, 성능을 최적화하기 어려움