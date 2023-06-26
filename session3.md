# 스프링 핵심 원리 이해2 - 객체 지향 원리 적용
`#스프링/2.스프링 핵심원리/ 기본편#`

## 새로운 할인 정책 개발
`새로운 할인 정책을 확장해보자.`
* **악덕 기획자** : 서비스 오픈 직전에 할인 정책을 지금처럼 고정 금액 할인이 아니라 좀 더 합리적인 주문 금액 당 확인하는 정률 %할인으로 변경하고 싶어요.예를 들어서 기존 정책은 VIP가 10000원을 주문하든 20000원을 주문하든 항상 1000원을 할인했는데, 이번에 새로 나온 정책은 10%로 지정해두면 고객이 10000원 주문시 1000원을 할인해주고, 20000원을 주문시에 2000원을 할인 해주는 거에요!
* **순진 개발자** : 제가 처음부터 고정 금액 할인은 아니라고 했잖아요.
* **악덕 기획자** : 애자일 소프트웨어 개발 선언 몰라요? "계획을 따르기보다 변화에 대응하기를"
* **순진 개발자** : ... (하지만 난 유연한 설계가 가능하도록 객체 지향 설계 원칙을 준수했지 후후)

> 참고 : 애자일 소프트웨어 개발 선언 https://agilemanifesto.org/iso/ko/manifesto.html


순진 개발자가 정말 객체지향 설게 원칙을 잘 준수했는지 확인해보자. 이번에는 주문한 금액의 %를 할인해주는 새로운 정률 할인 정책을 추가하자.


`RateDiscountPolicy 추가`
<img src="https://ifh.cc/g/At6RNo.jpg">


### 할인 정책을 애플리케이션에 적용해보자.

할인 정책을 변경하려면 클라이언트인 `OrderServiceImpl` 코드를 고쳐야만 한다.

```java
import hello.core.discount.RateDiscountPolicy;

public class OrderServiceImpl implements OrderService {
    // private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

`문제점 발견`
* 우리는 역할과 구현을 충실하게 분리했다. → OK
* 다형성도 활용하고, 인터페이스와 구현 객체를 분리했다. → OK
* OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수했다.
  * → 그렇게 보이지만 사실은 아니다.
* DIP : 주문 서비스 클라이언트 `OrderServiceImpl`는 `DiscountPolicy` 인터페이스에 의존하면서 DIP를 지킨 것 같은데?
  * → 클래스 의존관계를 분석해보자. 추상 (인터페이스) 뿐만 아니라 **구체(구현) 클래스에도 의존**하고 있다.
    * 추상(인터페이스) 의존 : `DiscountPolicy` 
    * 구체(구현)클래스 : `FixDiscountPolicy` , `RateDiscountPolicy`
  * OCP : 변경하지 않고 확장할 수 있다고 했는데!
    * **→ 지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다!** 따라서 **OCP를 위반 한다**