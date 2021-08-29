# core
**2021.08**

**인프런 강의: 스프링 핵심 원리**


**스프링부트 2.4, spring, java, gradle, intellij, JDK 11(java11)**


## 비즈니스 요구사항과 설계
* 회원
  * 회원을 가입하고 조회할 수 있다.
  * 회원은 일반과 VIP 두 가지 등급이 있다.
  * 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)
* 주문과 할인 정책
  * 회원은 상품을 주문할 수 있다.
  * 회원 등급에 따라 할인 정책을 적용할 수 있다.
  *할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
  * 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)
  
##스프링과 객체지향 설계

###스프링의 진짜 핵심
* 스프링은 좋은 객체 지향 애플리케이션을 개발할 수 있게 도와주는 프레임워크

###역할과 구현을 분리 - 자바 언어의 다형성
* 역할 = 인터페이스 / 구현 = 인터페이스를 구현한 클래스, 구현 객체
* 객체 설계시 역할(인터페이스)을 먼저 부여하고, 그 역할을 수행하는 구현 객체 만들기 / 역할과 구현 명확하게 분리
* 클라이언트: 요청, 서버: 응답
* 다형성(IoC, DI...) : 협력 / 클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다.(ex 레고 블럭 조립, 공연 무대의 배우를 선택)

###좋은 객체 지향 설계 5가지 원칙(SOLID)
* SRP: 단일 책임 원칙(single responsibility principle) : 한 클래스는 하나의 책임, 변경이 있을 때 파급효과가 적어야 한다.
* OCP: 개방-폐쇄 원칙 (Open/closed principle): 확장에는 열려 있으나 변경에는 닫힘, 인터페이스, 역할과 구현의 분리
* LSP: 리스코프 치환 원칙 (Liskov substitution principle): 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 교체 가능(ex 자동차 인터페이스의 엑셀은 앞으로 가라는 기능, 뒤로 가게 구현하면 LSP 위반, 느리
더라도 앞으로 가야함)
* ISP: 인터페이스 분리 원칙 (Interface segregation principle) : 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다(ex 자동차 인터페이스 -> 운전 인터페이스, 정비 인터페이스로 분리
, 사용자 클라이언트 -> 운전자 클라이언트, 정비사 클라이언트로 분리)
* DIP: 의존관계 역전 원칙 (Dependency inversion principle) : 추상화에 의존해야지, 구체화에 의존하면 안된다, 역할(Role)에 의존하게 해야 한다는 것과 같다

but, 다형성 만으로는 OCP, DIP를 지킬 수 없다.

스프링은 다음 기술로 다형성 + OCP, DIP를 가능하게 지원
• DI(Dependency Injection): 의존관계, 의존성 주입
• DI 컨테이너 제공
• 클라이언트 코드의 변경 없이 기능 확장
• 쉽게 부품을 교체하듯이 개발

##AppConfig - 관심사의 분리
애플리케이션을 하나의 드라마라고 가정하자. 각각의 인터페이스를 배역(배우 역할)이라 생각하자. 그런데! 실제 배역 맞는 배우를 선택하는 것은 누가 하는가?
드라마 감독(기획자)가 한다.
배우와 기획자의 역할(책임)을 확실히 분리하자


* Controller

```
@Controller
public class HelloController {
 @GetMapping("hello-mvc")
 public String helloMvc(@RequestParam("name") String name, Model model) {
 model.addAttribute("name", name);
 return "hello-template";
 }
}
```

* View

```
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```

### API
* @ResponseBody 를 사용하면 뷰 리졸버( viewResolver )를 사용하지 않음
* 대신에 HTTP의 BODY에 문자 내용을 직접 반환(HTML BODY TAG를 말하는 것이 아님)
* viewResolver 대신에 HttpMessageConverter 가 동작
* 기본 문자처리: StringHttpMessageConverter
* 기본 객체처리: MappingJackson2HttpMessageConverter
* byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음

```
@Controller
public class HelloController {
 @GetMapping("hello-string")
 @ResponseBody
 public String helloString(@RequestParam("name") String name) {
 return "hello " + name;
 }
}
```

### 컴포넌트 스캔 원리
* @Component 애노테이션이 있으면 스프링 빈으로 자동 등록된다.
* @Controller 컨트롤러가 스프링 빈으로 자동 등록된 이유도 컴포넌트 스캔 때문이다.
* @Component 를 포함하는 다음 애노테이션도 스프링 빈으로 자동 등록된다.
 * @Controller
 * @Service
 * @Repository

### @Transactional
@Transactional : 테스트 케이스에 이 애노테이션이 있으면, 테스트 시작 전에 트랜잭션을 시작하고, 테스트 완료 후에 항상 롤백한다. 이렇게 하면 DB에 데이터가 남지 않으므로 다음 테스트에 영향을 주지 않는다.

