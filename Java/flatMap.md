# flatMap이란?
- stream에서 컬렉션을 다룰 때 유용하게 사용되는 함수
- 각 요소의 결과를 평평하게 하여 새로운 배열 생성
## vs Map
### Map이란?
- 각 요소의 결과로 구성된 새로운 스트림 반환
- 각 요소가 리스트로 변환되어 스트림의 스트림 구조가 생김 `Stream<Stream<T>>`
### 예시
#### Map
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class MapExample {
    public static void main(String[] args) {
        List<String> sentences = Arrays.asList("Hello World", "Java Programming", "Stream API");

        List<List<Integer>> wordLengthsPerSentence = sentences.stream()
                                                               .map(sentence -> Arrays.stream(sentence.split(" "))
                                                                                      .map(String::length)
                                                                                      .collect(Collectors.toList()))
                                                               .collect(Collectors.toList());

        System.out.println(wordLengthsPerSentence); // [5, 5], [4, 11], [6, 3]]
    }
}
```
`List<List<Integer>>` 구조로 중첩됨
#### flatMap
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class FlatMapExample {
    public static void main(String[] args) {
        List<String> sentences = Arrays.asList("Hello World", "Java Programming", "Stream API");

        List<Integer> allWordLengths = sentences.stream()
                                                .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
                                                .map(String::length)
                                                .collect(Collectors.toList());

        System.out.println(allWordLengths); // [5, 5, 4, 11, 6, 3]
    }
}
```
`List<Integer>`로 단일 리스트로 평탄화됨
