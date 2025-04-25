## Chapter 1. 도메인 모델 시작하기

**도메인이란? 소프트웨어로 해결하고자 하는 문제 영역**

### **도메인의 핵심 개념과 의도**

1\. 도메인 모델은 소프트웨어가 해결하고자 하는 비즈니스 영역의 중요한 개념과 규칙(= 비즈니스 로직)을 코드로 표현한 것

2\. 실제 비즈니스 동작과 요구사항을 정확하게 반영하고,

도메인 전문가와 개발자가 공통의 언어와 이해를 가질 수 있도록 설계되어야 함

3\. 즉, **도메인의 핵심 개념과 의도란 "해결해야 할 비즈니스의 본질적 규칙과 의미"를 코드에 정확히 담는 것**을 의미함!!

### **개념 모델과 구현 모델**

\- 개념모델? 순수하게 문제를 분석한 결과물. DB, 트랜잭션 처리, 성능, 구현 기술과 같은 것을 고려 X

\- 처음부터 완벽하게 만들 필요 X, 불가능 하기도 함. 보통 개발하면서 도메인에 대한 지식과 이해가 쌓이면서 점진적으로 개념 모델을 구현 모델로 발전시켜 나간다.

### **DDD에서의 엔티티와 밸류**

#### **1\. 엔티티**

\- 가장 큰 특징은 식별자를 가진다는 것

\- 식별자는 다양한 방법으로 생성함 (일련번호 사용, UUID, 직접 값 입력 등)

#### **2\. 밸류**

\- 개념적으로 완전한 하나를 표현할 때 사용함

\- 도메인에서 특별한 의미를 지닌다.

\- 주로 엔티티의 속성이나 여러 데이터의 조합으로 사용된다.

ex) 01012345678로 전화번호를 입력했을 때 이를 010-1234-5678로 만들어주는 클래스가 VO의 예시 중 하나.

\-> **특정 값을 도메인에서 중요한 의미를 가지는 값으로 격상시켜주는 클래스**!

```
public class PhoneNumber {
    private final String value;

    public PhoneNumber(String raw) {
        // 검증 메서드 호출
        this.value = formatAndValidate(raw);
    }

    // 검증 + 포맷팅 메서드
    private String formatAndValidate(String raw) {
        String digits = raw.replaceAll("\\D", "");
        if (digits.length() != 11) {
            throw new IllegalArgumentException("휴대폰 번호는 11자리여야 합니다.");
        }
        return digits.replaceFirst("(\\d{3})(\\d{4})(\\d{4})", "$1-$2-$3");
    }

    public String getValue() {
        return value;
    }
}
```

\> 위 코드가 VO 클래스의 전형적인 예시라고 볼 수 있다.

\> 11자리의 숫자를 중요한 의미를 가지는 값(전화번호)으로 격상 시켜줌

### **도메인 모델에 set 메서드 넣지 않기!**

\- **도메인 get/set 메서드를 무조건 넣는 것은 좋지 않은 버릇임**

\- 특히 set 메서드는 도메인의 핵심 개념이나 의도를 심각하게 저해한다!

\> 단순히 값을 바꾸는 기능만 제공하는 set 메서드는 **"왜" 값이 바뀌고 어떤 규칙이 적용되는지 코드에 드러나지 않음!**

\> set 메서드를 사용하면 **필수 값이 빠진 상태로 객체가 생성**될 수 있어 도메인 규칙이 깨질 수 있음

\> 외부에서 아무때나 set 메서드를 통해 값을 바꿀 수 있으니 **일관성이 떨어지게됨**

\* DTO는 도메인 로직을 담고 있지는 않기 때문에, get/set 메서드를 사용해도 도메인 객체의 데이터 일관성에 영향을 줄 가능성이 높지 않다!

\* 프레임워크가 DTO 필드에 직접 값을 할당하는 기능을 제공한다면, set 메서드 대신 이를 적극적으로 활용하자. ( = Lombok의 @Setter)

### 도메인 용어와 유비쿼터스 언어

\- 코드를 작성할 때 도메인에서 사용하는 용어는 매우 중요함

\- 실제 주문 상태는 '결제 대기', '상품 준비', '출고 완료', '배송 중', '배송 완료', '주문 취소'인데 아래와 같이 코드를 작성하면 해당 코드가 어떤 상태를 표현하는지 알아햐 하는 번거로움이 생긴다. 따라서, 'PAYMENT\_WAITING', 'SHIPPED' 처럼 도메인 용어를 사용해서 OrderState를 구현하면 불필요한 변환 과정을 거치지 않아도 된다.

```
public OriderState {
	STEP1, STEP2, STEP3, STEP4, STEP5, STEP6
}
```

\- 유비쿼터스 언어란? 문서, 도메인 모델, 코드, 테스트 등 모든 곳에서 관련 직군의 사람들이 같은 용어를 사용함으로써 소통 과정에서 발생하는 오류를 줄이는 것에 대한 중요성을 표현하기 위한 용어이다.

\> DDD에서 언어의 중요함을 강조하기 위해 만든 용어!!


## Chapter 2. 아키텍처 개요

### **1\. 아키텍처의 네 가지 영역**

**1) 표현 영역**

\- 사용자의 요청을 해석해서 응용 서비스에 전달하고 응용 서비스의 실행 결과를 사용자가 이해할 수 있는 형식으로 변환하여 응답

\- Controller 클래스가 이에 해당된다고 이해하자

**2) 응용 영역**

\- 표현 영역을 통해 사용자의 요청을 전달받아서 시스템이 사용자에게 제공해야 할 기능을 구현

\- 예를 들면 '주문 등록', '주문 취소', '상품 상세 조회'와 같은 기능 구현

\- Service 클래스가 이에 해당된다고 이해하자

**3) 도메인 영역**

\- 도메인 모델을 구현한다.

\- 응용 영역에서 기능을 구현하기 위해 도메인 영역의 도메인 모델을 사용함.

\- 엔티티, 밸류 오브젝트 (VO), 도메인 서비스, 리포지토리 등이 이에 해당됨

**4) 인프라스트럭처 영역**

\- 구현 기술에 대한 것을 다룬다.

\- RDBMS 연동, Redis와 몽고DB등의 데이터 연동, SMTP, REST API 호출 등을 처리함

\- 예를 들면 WebSocketConfig 같은것들

### **2\. 계층 구조 아키텍처**

[##_Image|kage@bDcuzf/btsNATd3uey/JMHXpurYaEmYVOEHvASWSk/img.png|CDM|1.3|{"originWidth":468,"originHeight":899,"style":"alignCenter","width":183,"height":352,"caption":"계층 구조의 아키텍처 구성"}_##]

\- 계층 구조는 **특성상 상위 계층에서 하위 계층으로의 의존만 존재**한다.  
\- 상위 계층은 바로 아래의 계층에만 의존을 가져야 하지만 **계층 구조를 유연하게 적용하기도 함!**

**\- 고수준 모듈이 저수준 모듈을 사용하면 구현 변경과 테스트가 어렵다는 문제가 발생함 (의존성 때문에)**

### **3\. DIP**

\- 계층 구조 아키텍처에서 발생한 의존성 문제를 해결하기 위해 의존성을 역전시킴

**\- 추상화한 인터페이스를 통해 이를 구현!**

\- Service 인터페이스와 ServiceImpl 구현체를 생각하면 이해가 쉽다.

\- 구현 교체가 어려웠던 문제와 테스트가 어려웠던 문제를 해결 가능하다.

```
// Notifier 인터페이스 (추상화)
public interface Notifier {
    void notify(String message);
}

// EmailNotifier 구현체
public class EmailNotifier implements Notifier {
    @Override
    public void notify(String message) {
        System.out.println("이메일 발송: " + message);
        // 실제 이메일 발송 로직
    }
}

// SmsNotifier 구현체
public class SmsNotifier implements Notifier {
    @Override
    public void notify(String message) {
        System.out.println("SMS 발송: " + message);
        // 실제 SMS 발송 로직
    }
}

// SlackNotifier 구현체
public class SlackNotifier implements Notifier {
    @Override
    public void notify(String message) {
        System.out.println("슬랙 메시지 발송: " + message);
        // 실제 슬랙 메시지 발송 로직
    }
}

// 클라이언트 코드 예시
public class NotificationService {
    private final Notifier notifier;

    // 생성자에서 Notifier 구현체를 주입받음 (DIP)
    public NotificationService(Notifier notifier) {
        this.notifier = notifier;
    }

    public void sendAlert(String message) {
        notifier.notify(message);
    }
}

// 사용 예시
public class Main {
    public static void main(String[] args) {
        Notifier emailNotifier = new EmailNotifier();
        Notifier smsNotifier = new SmsNotifier();
        Notifier slackNotifier = new SlackNotifier();

        NotificationService emailService = new NotificationService(emailNotifier);
        NotificationService smsService = new NotificationService(smsNotifier);
        NotificationService slackService = new NotificationService(slackNotifier);

        emailService.sendAlert("주문이 접수되었습니다.");
        smsService.sendAlert("주문이 발송되었습니다.");
        slackService.sendAlert("긴급 공지사항입니다.");
    }
}
```

ai를 통해 만든 예시 코드를 살펴보면, **NotificationService 에서 단지 Notifier 인터페이스에만 의존**한다.

이렇게 하면 새로운 알림 방식이 추가되거나 기존 구현체가 변경되더라도 NotificationService 코드는 전혀 건드리지 않아도 된다.

### **4\. 도메인 영역의 주요 구성요소**

**\* 엔티티와 밸류**

\- DB 테이블의 엔티티와 도메인 모델의 엔티티는 다르다! 1장에서도 강조함.

\- 도메인 모델의 엔티티는 데이터와 함께 기능을 제공하는 객체이다!

\- 도메인 모델의 엔티티는 두 개 이상의 데이터가 개념적으로 하나인 경우 밸류 타입을 사용해서 표현 가능하다!

\- 밸류는 불변으로 구현할 것이 권장된다.

**\* 에그리거트(Aggregate)**

\- 관련 객체를 하나로 묶은 군집

\- 예를 들면 '주문' 이라는 도메인 개념은 '주문', '배송지 정보', '주문자', '주문 목록', '총 결제 금액'의 하위 모델로 구성됨

**\* 리포지토리**

\- 구현을 위한 도메인 모델

\- 에그리거트 단위로 도메인 객체를 저장하고 조회하는 기능을 정의한다.

### **5\. 요청 처리 흐름**

\- 사용자가 애플리케이션에 기능 실행을 요청하면, 그 요청을 처음 받는 영역은 '표현 영역'

\- 사용자가 전송한 데이터에 문제가 없다면 '응용 영역'에서 기능 실행

\- '응용 영역'에서는 도메인 모델을 이용해서 기능을 구현함. 여러 도메인 객체를 사용하기도 하는데 이 때 @Transactional을 활용

### **6\. 인프라스트럭처 개요**

\- 인프라스트럭처는 표현 영역, 응용 영역, 도메인 영역을 지원함

\- 도메인 영역과 응용 영역에 정의한 인터페이스를 인프라스트럭처 영역에서 구현하는것이 DIP

\- 인프라스트럭처에 대한 의존을 없애는 것이 오히려 구현을 어렵게 한다면 꼭 없애지 않아도 된다.

\- 예를 들면, @Transactional, Lombok의 여러 어노테이션들 사용
