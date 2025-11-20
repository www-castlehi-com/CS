## 1. 기능
서블릿 환경에서 request thread를 오래 점유하지 않고, 응답 스트림을 계속 열어둔 상태로 데이터를 chunk 단위로 흘려보내기 위한 메커니즘이다.
즉, 전체 파일을 메모리에 올리지 않고 OutputStream에 바로 흘려보내는 방식이다.
언뜻 보면 NIO같지만 I/O나 네트워크 전송은 모두 blocking 이다. Servlet 환경만 비동기이다.

대용량 파일 다운로드, 실시간 로그 스트리밍, SSE 등에 적합하다.
## 2. 원리
우선 이 과정을 이해하려면 스프링의 응답 과정을 알아야 한다.
간단하게 정리해보자.
![](https://i.imgur.com/1J0KEJd.png)
DispatcherServlet이 요청 처리를 RequestMappingHandlerAdapter에게 위임한다.
핸들러를 통해 반환된 객체는 적절한 HandlerMethodReturnValueHandler를 선택하여 처리하도록 위임한다.
그리고 적절한 Converter를 통해 반환값을 OutputStream에 write한다.

그럼 이제 컨트롤러가 StreamingResponseBody를 반환했다고 가정하자.
핸들러를 통해 반환된 StreamingResponseBody는 StreamingResponseBodyReturnValueHandler를 통해 처리된다.
StreamingResponseBodyReturnValueHandler에서는 StreamingResponseBody를 StreamingResponseBodyTask로 변환하여 WebAsyncManager에게 callable로 보내며 비동기 처리를 위임한다.

우선 StreamingResponseBodyTask를 확인해보자.
```java
private static class StreamingResponseBodyTask implements Callable<Void> {  
    private final OutputStream outputStream;  
    private final StreamingResponseBody streamingBody;  
  
    public StreamingResponseBodyTask(OutputStream outputStream, StreamingResponseBody streamingBody) {  
        this.outputStream = outputStream;  
        this.streamingBody = streamingBody;  
    }  
  
    public Void call() throws Exception {  
        this.streamingBody.writeTo(this.outputStream);  
        this.outputStream.flush();  
        return null;  
    }  
}
```
Output으로 받은 StreamingResponseBody가 OutputStream과 함께 StreamingResponseBodyTask로 캡슐화된다.
그리고 `call()` 메서드를 호출함과 동시에 outputStream에 write하고 flush까지 호출한다.

다음은 WebAsyncManager의 실제 처리 부분이다.
```java
public void startCallableProcessing(final WebAsyncTask<?> webAsyncTask, Object... processingContext) throws Exception {  
    //...
    this.startAsyncProcessing(processingContext);  
  
    try {  
        Future<?> future = this.taskExecutor.submit(() -> {  
            //...  
        });  
        //... 
    } catch (RejectedExecutionException var10) {  
        //...
    }  
}
```
먼저 내부에서 `startAsyncProcessing()`이라는 함수를 호출하는걸 볼 수 있다.
함수 호출 시 내부적으로 `this.asyncWebRequest.startAsync();`가 호출되는데, Servlet 3.1 비동기 모델에서 해당 함수가 호출되는 순간 서블릿 요청 스레드는 즉시 반환된다.
즉, 이 순간부터 DispatcherServlet으로 요청을 한 스레드는 더 이상 `write()`에 관여하지 않는다.

future는 `taskExecutor.submit()`을 호출하는 순간 새로운 스레드에서 실행된다.
그리고 `StreamingResponseBodyTask.call()`이 실행되어 outputStream에 write 및 flush 된다.
outputStream에 write/flush할 경우 streamingBody 데이터는 os 커널로 넘어가게 된다.