# API 예외 처리 요약

## HTML이랑 다른 점
- 일단 html은 BasicErrorController를 사용하고 api도 쓸 수 있음. 하지만,
api의 경우 사용하는 주체?주제? 이런게 달라서 응답 형식이 매번 달라질 거임
예를 들면 상품 관련 api 회원 관련 api 등 ㅇㅋ?
- 그래서 BasicErrorController을 안쓰고 ExceptionHandler라는 걸 쓸거임

## ExceptionHandler

- 기능
  - 예외 상태코드 변환해서(response.sendError) 서블릿이 상태코드에 따른 처리 도와줌
  - ModelAndView 생성자 파라미터에 값 넣어서 다른 오류화면 렌더링도 가능
  - HTTP 응답 body 안에 직접 데이터 넣어서 응답 가능 
-> JSON 형식으로 쓰면 JSON도 가능  
```response.getWriter().println("hello")```


- 전에 배운 BasicErrorController는 html화면을 에러에 맞게 대응해주는 놈이라면 얘는 api나 json을 반환할 때
도와주는 놈임 내 입맛대로 | ***중요한 내용인데 백엔드로 내가 취업하면 화면을 반환하는 일은 거의 없을 거고 DTO객체나 JSON을 넘겨줄텐데
그러면 이걸 거의 맨날 쓴다고 한다 ㅇㅇ***

- try-catch로 감싸고 try부분에 if문 달아서 예외별로 처리가 가능함 그리고 상태코드를 변경하고
끝낼 때는 빈 ModelAndView 객체를 반환함
-> **이게 빈 객체를 반환해서 흐름 자체는 정상흐름으로 이어지게 만드는 거임 ㅇㅋ?**


## 활용하기!

- 일단 중요한 점 하나 BasicErrorController는 에러를 서블릿까지는 가지고 가고 그 다음에 다시 에러페이지 호출이 이루어짐
-> 2번 왔다갔다 함 너무 많음
- 하지만 ExceptionHandler를 활용하면 스프링MVC 안에서 처리가 쌉가능
Why? -> ModelAndView 반환하고 우리가 안에서 처리해서 WAS는 에러가 난 줄 모름

- 근데 이걸 모두 다 구현하기는 너무나도 귀찮음 역시 스프링이 제공해주는 것들이 있음

## 스프링이 제공하는 ExceptionResolver

- 종류(우선순위 순임)
1. ExceptionHandlerExceptionResolver
2. ResponseStatusExceptionResolver
3. DefaultHandlerExceptionResolver 

- ResponseStatusExceptionResolver
  - @ResponseStatus 어노테이션을 사용해서 HTTP 상태코드를 설정할 수 있음
  - 커스텀 예외에 어노테이션으로 등록이 가능함(원래 있는 수정 불가능한 라이브러리에 있는 에러는 사용 불가능)
  - reason 파라미터로 메시지도 등록 가능함
  - 얘는 단순히 sendError() 메서드를 호출하는 거라서 WAS를 한 번 다녀온다.
  - 그래서 커스텀이 아닌 기존 예외는```ResponseStatusException(HttpStatus.NOT_FOUND, 
"error.bad", new IllegalArgumentException())``` 이따구로 씀


- DefaultHandlerExceptionResolver
  - 얘는 스프링 내부에서 발생하는 스프링 예외를 처리해줌 (ex=TypeMismatchException)
  - 예를 들어 파라미터 바인딩은 보통 클라이언트의 잘못이 대부분임 HTTP에서는 이걸 400에러로 정의하고 있는데
  WAS는 모르겠고 런타임에러니까 500으로 처리하려고 함, **하지만 DefaultHandlerExceptionResolver는 400으로 처리해줌**


- **대망의 ExceptionHandlerExceptionResolver**
  - 이 친구는 API 응답을 아주 유동적으로 가능하게 해준다.
  - 컨트롤러 안에 메서드 형태로 정의하면 그 컨트롤러 안에서 발생한 설정한 예외처리를 해줌
  - ```
    @ResponseStatus(code = HttpStatus.BAD_REQUEST) // 응답 코드를 바꿔주고 싶으면 이거 호출하고 바꿔주자!
    @ExceptionHandler(IllegalArgumentException.class) // 컨트롤러 안에 정의하면 이 컨트롤러에서 발생한 이 예외처리 해줌
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        log.error("[exceptionHandler] ex", e);
        return new ErrorResult("BAD", e.getMessage()); //ErrorResult는 필드객체(JSON 형식 정해놓은 객체임)
        // 여기서는 알아서 HttpMessageConverter + ObjectMapper 호출해서 응답바디까지 싹 바꿔서 올려주고
        // 마지막에 return new ModelAndView 해줘서 /**정상 흐름으로 바꿔줌**/
        // 결국 WAS까지 안올라가서 2번 반복 안함
        // 하지만 정상이라서 statusCode가 200으로 나감 -> @ResponseStatus 사용
    }
    ```
  - ErrorResult와 같이 JSON 형식의 응답 객체를 사용하여 API 응답을 유동적으로 처리할 수 있다.
  - 주석처럼 스프링에서 알아서 ModelAndView를 리턴해줘서 정상 흐름으로 바꿔줌 -> 에러코드를 설정하고 싶으면 @ResponseStatus 사용
  - @ExceptionHandler에 파라미터 생략 가능 
  - ```
    @ExceptionHandler // 어노테이션 부분 생략(메서드 파라미터에 적어준 예외로 동작)
    public ResponseEntity<ErrorResult> userExHandler(UserException e)
    ```
  - **중요함! 위 코드처럼 반환값을 객체가 아닌 ResponseEntity로 사용하게 되면 API 객체와 상태코드를 유동적으로 바꿀 수 있음**
-> if문 등을 사용해서 에러 메세지나 코드를 보고 다른 예외코드와 메시지를 던져줄 수 있음
  - 그리고 @ExceptionHandler 파라미터 code에(에러코드 부분) 2개 이상 넣을 수도 있음
  - 메서드 파라미터에 Exception 처럼 상위 예외까지 등록하면 자세한 에러(IllegalArgumentException 등)에 해당하지 않는 나머지를 
싹 다 처리 가능함 ㅇㅋ?
  - 아 진짜 마지막임 그 @ResponseBody나 @RestController가 아닌 메서드에는 return new ModelAndView("/error") 처럼 에러페이지 반환 가능!