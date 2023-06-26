**시작하기에 앞서 나는 매일 매일 공부할 것을 나 자신에게 약속한다**
---


# JavaSpring
스프링공부 (인프런 김영한의 스프링 핵심 원리 - 기본편)

## 비즈니스 요구사항과 설계
* 회원
  * 회원을 가입하고 조회할 수 있다
  * 회원은 일반과 VIP 두 가지 등급이 있다.
  * 회원 데이터는 자체의 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다.
* 주문과 할인 정책
  * 회원은 상품을 주문할 수 있다.
  * 회원 등급에 따라 할인 정책을 적용할 수 있다.
  * 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라.
  * 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수도 있다.


<br>
    요구사항을 보면 회원 데이터, 할인 정책 같은 부분은 지금 결정하기 어려운 부분이다. 그렇다고 이런 정책이 결정될 때까지 개발을 무기한 기다릴 수도 없다. 우리는 앞에서 배운 객체 지향설계 방법이 있지 않은가!

<br>

인터페이스를 만들고 구현체를 언제든지 갈아끼울 수 있도록 설계하면 된다. 그럼 시작해보자

> **참고** : 프로젝트 환경설정을 편리하게 하려고 스프링 부트를 사용한 것이다. 지금은 스프링 없는 순수한 자바로만 개발을 진행한다는 점을 꼭 기억하자! 스프링 관련은 한참 뒤에 등장한다.


## 회원 도메인 설계
* 회원 도메인 요구사항
  * 회원을 가입하고 조회할 수 있다.
  * 회원은 일반과 VIP 두 가지 등급이 존재한다.
  * 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다.



`회원 도메인 협력 관계`
<img src="https://file.notion.so/f/s/0184d32f-0b87-4c74-b0a4-0a79dba87c6b/Untitled.jpeg?id=58f9beab-b977-4521-8071-1bd06828e907&table=block&spaceId=66565165-181a-42cf-902d-d809eb755c8d&expirationTimestamp=1687852167627&signature=H8OBRuGHNOBTqaCxzijjjQw3deh77vs4cN9Uphhb7as&downloadName=Untitled.jpeg">

