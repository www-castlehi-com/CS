POST / REDIRECT / GET
# 문제
![](https://i.imgur.com/5D1xQbz.png)
- 새로고침 시 마지막에 서버에 전송한 데이터를 다시 전송
- 상품 등록 폼에서 데이터를 입력하고 저장할 경우 `POST /add + 상품 데이터`를 서버로 전송
- 새로고침을 할 경우 마지막에 전송한 `POST + /add + 상품 데이터`를 서버로 다시 전송하게 되어 **중복 데이터** 발생
# 해결 방법
![](https://i.imgur.com/23SlqdU.png)
- 상품 저장 후 뷰 템플릿 이동이 아닌, 상품 상세 화면으로 리다이렉트 호출
- 리다이렉트의 영향으로 실제 상품 화면으로 이동할 경우 마지막 호출 URL이 `GET /items/{id}`가 됨
- 새로고침을 할 경우 `GET /items/{id}`가 되므로 중복 데이터가 누적되지 않음
## 예시
```java
package hello.itemservice.web.basic;  
  
@Controller  
@RequestMapping("/basic/items")  
@RequiredArgsConstructor  
public class BasicItemController {  
  
    //...  
  
    @PostMapping("/add")  
    public String addItemV5(Item item) {  
       itemRepository.save(item);  
  
       return "redirect:/basic/items/" + item.getId();  
    }  
  
    //... 
}
```
- ` + item.getId()`처럼 URL에 변수를 더해서 사용할 경우 URL 인코딩이 되지 않음
- [[RedirectAttribute]]로 인코딩 처리