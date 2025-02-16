```java
package hello.itemservice.web.basic;  
  
@Controller  
@RequestMapping("/basic/items")  
@RequiredArgsConstructor  
public class BasicItemController {  
  
    //...
  
    @PostMapping("/add")  
    public String addItemV6(Item item, RedirectAttributes redirectAttributes) {  
       Item savedItem = itemRepository.save(item);  
  
       redirectAttributes.addAttribute("itemId", savedItem.getId());  
       redirectAttributes.addAttribute("status", true);  
  
       return "redirect:/basic/items/{itemId}";  
    }  
  
    //...
}
```
- URL 인코딩 처리
- `pathVariable`, `queryParm` 처리
	- pathVariable 바인딩 : `{itemId}`
	- 나머지는 쿼리 파라미터로 처리 : `?status=true`