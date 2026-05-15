# 서블릿 예외 처리 요약

## 서블릿의 예외처리 방법 2가지 (런타임예외, response.sendError)

```@Component
WebServerCustomizer implement WebServerFactoryCustomizer
customize()
```
안에 new ErrorPage만들기 후 factory에 addErrorPages 하고 등록

- ErrorPageController는 필요 없지? 아니지 필요하지 이거는 서버커스터마이저 안에 에러 발생시 경로를 지정했는데 그거에 대한 컨트롤러가 필요한 거임
-> 일단 요청은 @RequestMapping으로 >> get/post 다 받아야 함

## 서블릿이 예외처리하는 방법 (필터편)
- was는 예외에 따른 오류 페이지 정보를 조회


- was -> 필터 -> 서블릿(디스패처) -> 인터셉터 -> 컨트롤러
이거 반대로 온다고 생각하고
컨트롤러에서 에러가 발생하고 역순으로 쭉 오다가 was에서 만나서 다시 에러 페이지를 요청함
그럼 다시 그 에러 페이지 요청이
was -> 필터 -> 서블릿(디스패처) -> 인터셉터 -> 컨트롤러 이렇게 또 감
**근데 웹브라우저는 아예 이런 일들을 모름**


- 로그 필터 설정
setDispatcherTypes(request, error) <- 디폴트가 request만 찍어줌
로그 필터에 doFilter에 한가지 파라미터 추가(request.getDispatcherType())

## 서블릿이 예외처리하는 방법 (인터셉터편)
로그인터셉터를 만든데요 이것도 디스패처타입만 넣어주면 된답니다 어차피 난 없음

## 이제 스프링 부트가 하는 오류 페이지 설정

- ErrorPage를 자동으로 등록해줌 -> /error | BasicErrorController 자동 등록 위에 꺼 자동 매핑

- 뷰 템플릿 등록 규칙이 있음
  - template.error 만들어야 함
  - 4xx.html 400번대 처리 404.html 404오류 처리 (당연히 자세한 것이 우선임)
  - 뷰템플릿 -> 정적리소스 -> 정 없을 때 뷰 이름이 error인 것 >> error.html
  

- 이러면 알아서 BasicErrorController 얘가 알아서 알아서 해준다
즉 오류 페이지만 만들면 스프링이 진짜 알아서 해줌(**에러 로그 필터나 인터셉터는 만들자**)


- BasicErrorController가 제공하는 기본 정보가 있음
>BasicErrorController는 모델에 에러 정보들을 담아서 뷰에 전달해줌 -> 타임리프 활용가능
but, 오류 정보를 고객에게 노출시키는 것은 좋지 않음 그래서 보안기본 설정에 노출 안되게 다 막아놨음 스프링이 / 
> 하지만 application.properties에서 설정 가능 on_param도 있는데 나중에