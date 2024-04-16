# 설명
- 단방향 암호화를 위해 만들어진 해시 함수

# 보완점
- **rainbow table attack**
	미리 해시 값들을 계산해 놓은 테이블을 가지고 해시 함수 반환 값을 역추적해 원래 정보를 찾아내는 해킹 방법

# 원리
## 솔팅
- 실제 정보 이외에 추가적으로 무작위 데이터를 더해서 해시 값을 계산
- salt로 인해 해시값이 달라지기 때문에 rainbow table attack 방지 가능
- 비밀번호마다 salt값이 다르기 때문에 같은 비밀번호라도 해시값이 달라짐

## 키 스트레칭
- 단방향 해시 값을 계산한 후 그 해시 값을 다시 해시하고 반복함
 ![](https://i.imgur.com/0Z0kAA6.png)

## 코드
```java
public static String gensalt(String prefix, int log_rounds, SecureRandom random) throws IllegalArgumentException {  
   StringBuilder rs = new StringBuilder();  
   byte rnd[] = new byte[BCRYPT_SALT_LEN];  
  
   if (!prefix.startsWith("$2")  
         || (prefix.charAt(2) != 'a' && prefix.charAt(2) != 'y' && prefix.charAt(2) != 'b')) {  
      throw new IllegalArgumentException("Invalid prefix");  
   }  
   if (log_rounds < 4 || log_rounds > 31) {  
      throw new IllegalArgumentException("Invalid log_rounds");  
   }  
  
   random.nextBytes(rnd);  
  
   rs.append("$2");  
   rs.append(prefix.charAt(2));  
   rs.append("$");  
   if (log_rounds < 10) {  
      rs.append("0");  
   }  
   rs.append(log_rounds);  
   rs.append("$");  
   encode_base64(rnd, rnd.length, rs);  
   return rs.toString();  
}
```

![](https://i.imgur.com/fxRKaA1.png)

1. **접두사**검증 
	- "$2a", "$2y", "$2b" 등으로 시작해야 함
	- 버전과 호환성을 나타냄
2. **로그 라운드** 검증
	- 키 스트레칭 과정의 강도 결정
3. 랜덤 바이트 생성 : 솔트를 생성하는데 사용
4. 솔트 문자열 생성 : 랜덤 바이트를 Base64 인코딩을 통해 문자열로 변환하고, BCrypt 형식에 맞게 문자열로 조합