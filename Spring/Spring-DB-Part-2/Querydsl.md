# 소개
![](https://i.imgur.com/ZlBafj5.png)
- SQL을 Java로 type-safe하게 개발할 수 있게 지원하는 프레임워크
- 쿼리에 특화된 프로그래밍 언어
- 주로 JPA 시 사용
- Domain(도메인) + Specific(특화) + Language(언어)
	- **DSL** : 도메인에 초점을 맞춘 제한적인 표현력을 가진 컴퓨터 프로그래밍 언어
- 단순, 간결, 유창
## 예시
```java
JPAQueryFactory query = new JPAQueryFactory(entityManager);
QMember m = QMember.member;

List<Member> list = query
	.select(m)
	.from(m)
	.where(m.age.between(20,40).and(m.name.like("김%")))
	.orderBy(m.age.desc())
	.limit(3)
	.fetch(m);
```
## 장점
- type-safe
- 단순함
- 쉬움
## 단점
- Q코드 생성을 위한 APT 설정 필요
# Type-Safe
![](https://i.imgur.com/LfgNUBC.png)
- **APT** (코드 생성기) : Annotation Processing Tool
	> `@Entity`
