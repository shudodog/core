# core
**2021.08**


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
  
## 스프링과 객체지향 설계

### 역할과 구현을 분리 - 자바 언어의 다형성
* 역할 = 인터페이스 / 구현 = 인터페이스를 구현한 클래스, 구현 객체
* 객체 설계시 역할(인터페이스)을 먼저 부여하고, 그 역할을 수행하는 구현 객체 만들기 / 역할과 구현 명확하게 분리
* 클라이언트: 요청, 서버: 응답
* 다형성(IoC, DI...) : 협력 / 클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다.(ex 레고 블럭 조립, 공연 무대의 배우를 선택)

### 좋은 객체 지향 설계 5가지 원칙(SOLID)
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

## AppConfig - 관심사의 분리
애플리케이션을 하나의 드라마라고 가정하자. 각각의 인터페이스를 배역(배우 역할)이라 생각하자. 그런데! 실제 배역 맞는 배우를 선택하는 것은 누가 하는가?
드라마 감독(기획자)가 한다.
배우와 기획자의 역할(책임)을 확실히 분리하자

### AppConfig
* AppConfig가 드라마 기획자
* 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스

```
@Configuration // AppConfig에 설정을 구성한다는 뜻의 @Configuration 을 붙여준다.
public class AppConfig {
 @Bean // 스프링 컨테이너에 스프링 빈으로 등록
 public MemberService memberService() {
 return new MemberServiceImpl(memberRepository());}
 @Bean
 public OrderService orderService() {
 return new OrderServiceImpl(memberRepository(),discountPolicy()); }
 @Bean
 public MemberRepository memberRepository() {
 return new MemoryMemberRepository() }
 @Bean
 public DiscountPolicy discountPolicy() {
 return new RateDiscountPolicy();}
}

```

* AppConfig는 애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다.
 MemberServiceImpl, MemoryMemberRepository, OrderServiceImpl, FixDiscountPolicy
* AppConfig는 생성한 객체 인스턴스의 참조(레퍼런스)를 생성자를 통해서 주입(연결)해준다.
 MemberServiceImpl MemoryMemberRepository / OrderServiceImpl MemoryMemberRepository , FixDiscountPolicy

![image](https://user-images.githubusercontent.com/76150392/131256347-4007cef0-98fc-4a2a-bb78-615c4811643f.png)
* 객체의 생성과 연결은 AppConfig 가 담당한다.
* DIP 완성: MemberServiceImpl 은 MemberRepository 인 추상에만 의존하면 된다. 이제 구체 클래스를 몰라도 된다.
* 관심사의 분리: 객체를 생성하고 연결하는 역할과 실행하는 역할이 명확히 분리되었다.
* OrderServiceImpl 은 기능을 실행하는 책임만 지면 된다.

### 제어의 역전 IoC(Inversion of Control)
* 프로그램에 대한 제어 흐름에 대한 권한은 모두 AppConfig가 가지고 있는데, 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다.

### 의존관계 주입 DI(Dependency Injection)
* 애플리케이션 실행 시점(런타임)에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것
* 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경 가능
* 정적인 클래스 의존관계(import로 알 수 있음)를 변경하지 않고, 동적인(애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계) 객체 인스턴스 의존 관계를 쉽게 변경

## 스프링 컨테이너
* ApplicationContext 를 스프링 컨테이너라 한다
* 스프링 컨테이너는 @Configuration 이 붙은 AppConfig 를 설정(구성) 정보로 사용한다. 여기서 @Bean이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록
* 스프링 빈은 applicationContext.getBean() 메서드를 사용해서 찾을 수 있다.
* 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.

## 컴포넌트 스캔과 자동 의존관계 주입
```
@Configuration
@ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =Configuration.class))
public class AutoAppConfig {
 
}
```

```
@Component
public class MemberServiceImpl implements MemberService {
 private final MemberRepository memberRepository;
 @Autowired
 public MemberServiceImpl(MemberRepository memberRepository) {
 this.memberRepository = memberRepository;
 }
}
...
```

* @ComponentScan 은 @Component(@Controller, @Repository, @Configuration, @Service) 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
* 생성자에 @Autowired 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.

### 조회 빈 2개 이상
* @Qualifier @Qualifier끼리 매칭
* @Primary 사용 : @Primary 는 우선순위를 정하는 방법이다. @Autowired 시에 여러 빈이 매칭되면 @Primary 가 우선권을 가진다.

## 롬복
* 롬복 라이브러리가 제공하는 @RequiredArgsConstructor 기능을 사용하면 final이 붙은 필드를 모아서 생성자를 자동으로 만들어준다. 
```
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
 private final MemberRepository memberRepository;
 private final DiscountPolicy discountPolicy;
}
```

## 빈 생명주기 콜백
* 스프링 빈의 라이프사이클 : 객체 생성 > 의존관계 주입
* 스프링 빈의 이벤트 라이프사이클: 스프링 컨테이너 생성 > 스프링 빈 생성 > 의존관계 주입 > 초기화 콜백 > 사용 > 소멸전 콜백 > 스프링 종료
 * 초기화 콜백: 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
 * 소멸전 콜백: 빈이 소멸되기 직전에 호출

### @PostConstruct, @PreDestroy 
* @PostConstruct , @PreDestroy 이 두 애노테이션을 사용하면 가장 편리하게 초기화와 종료를 실행할 수 있다.
* 유일한 단점은 외부 라이브러리에는 적용하지 못한다는 것

## 빈 스코프
* 프로토타입: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프이다.
* 웹 관련 스코프 - request: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다
* 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다 그래서 @PreDestroy 같은 종료 메서드가 호출되지 않는다.

### 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider로 문제 해결
