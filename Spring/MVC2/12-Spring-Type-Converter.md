# 스프링 타입 컨버터

## 타입 전환

- 스프링은 원래 자동 타입 변환을 해준다 (래퍼클래스, 기본타입 등)
- 그리고 브라우저에서 들어오는 모든 것은 String 으로 들어온다
- 하지만 @ModelAttribute, @RequestParam 등 타입만 지정해주면 스프링이 자동으로 변환해준다
- 그리고 사용자 지정 변환기(컨버터)도 만들 수 있다
- org.springframework.core.convert.converter.Converter 이것을 implements
- convert 메서드만 오버라이딩 하면 됨

## ConversionService 등장

- 여러 컨버터를 만들어도 그걸 하나하나 불러와서 사용한다면 아주 귀찮을 것이다 그럼 그냥 스프링이 해주는게 아니지;
- 그래서 ConversionService라는 놈이 있음 여기다가 addConverter만 해주면 컨버전 서비스 객체 하나로 다 가능
```
DefaultConversionService conversionService = new DefaultConversionService();
        conversionService.addConverter(new StringToIntegerConverter());
        conversionService.addConverter(new IntegerToStringConverter());
        conversionService.addConverter(new StringToIpPortConverter());
        conversionService.addConverter(new IpPortToStringConverter());

        //사용
        Integer result = conversionService.convert("10", Integer.class);
        System.out.println("result = " + result);****
```
- 저 서비스가 알아서 컨버터 안에 리턴타입과 파라미터 타입을 보고 추론해서 맞는 컨버터를 사용!
- 물론 저건 테스트코드에서 한 거라 일일이 선언해서 다 등록했지만 실제 비즈니스 로직에서는 빈으로 등록하고
주입 받아서 쓰면 된당 알잘딱깔센 ㅇㅋ?

## ISP(Interface Segregation Principal) - 인터페이스 분리 원칙

- 뭐.. 일단 정의는 클라이언트가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다. 이거임
- DefaultConversionService는 인터페이스를 두개 extends하는데
  1. ConversionService: 컨버터 사용에 초점 (canConvert(), convert())
  2. ConverterRegistry: 컨버터 등록에 초점 (addConverter())
<br> 이런 식임
- 근데 여기서 궁금한 점 -> 그러면 실제 사용할 때 컨버터에 등록을 필요 없고 convert()만 쓰고 싶다 그러면
주입 받을 때 ConversionService converter 이런식으로 주입 받나? -> ***맞음***
- 주입을 받아서 사용할 때 실제로 들어오는 객체는 DefaultConversionService이거지만 타입선언을 ConversionService로
했기 때문에 사용 가능한 메서드는 ConversionService에 있는 메서드만 사용 가능한 거임!(다형성!)
