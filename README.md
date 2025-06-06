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


## Chapter 3. 애그리거트

### **1\. 애그리거트**

(Chapter 2에서 잠깐 다룸)

\- 관련 객체를 하나로 묶은 군집

\- 예를 들면 '주문' 이라는 도메인 개념 (애그리거트)는 '주문', '배송지 정보', '주문자', '주문 목록' 등의 하위 모델로 구성됨

> 도메인 객체 모델이 복잡해지면 개별 구성요소 위주로 모델을 이해하게 되고 전반적인 구조나 큰 수준에서 도메인 간의 관계를 파악하기 어려워진다. 애그리거트는 관련된 객체를 하나의 군으로 묶어 주는 역할을 한다. 수많은 객체를 애그리거트로 묶어서 바라보면 상위 수준에서 도메인 모델 간의 관계를 파악할 수 있다!

#### **\* 애그리거트의 특징**

1) 복잡한 도메인 객체 모델을 이해하는데 도움을 준다.

2) 일관성을 관리하는 기준이 된다

\- **복잡한 도메인을 단순한 구조로 만들어주기 때문에, 복잡도가 낮아지는 만큼 도메인 기능을 확장하고 변경하는데 용이**하게 해줌

3) 관련된 모델을 하나로 모았기 때문에 한 애그리거트에 속한 객체는 유사하거나 동일한 라이프 사이클을 갖는다.

4) 애그리거트는 경계를 갖는다. **한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않는다.**

\- 경계를 설정할 때 기본이 되는 것은 도메인 규칙과 요구사항이다.

\- 도메인 규칙에 따라 함께 생성되는 구성요소는 한 애그리거트에 속할 가능성이 높다.

### **2\. 애그리거트 루트**

\- 애그리거트에 속한 모든 객체는 일관된 상태를 유지해야 한다.

\- **애그리거트 루트 엔티티**는 **애그리거트 전체가 일관된 상태를 유지하도록 만드는 데 책임을 갖는 주체**이다.

\- 즉, 애그리거트에 속한 객체는 애그리거트 루트 엔티티에 직/간접적으로 속하게 된다.

\* 불필요한 중복을 피하고 애그리거트 루트를 통해서만 도메인 로직을 구현하게 만드려면 다음 두 가지를 습관적으로 적용해야 함

#### **1) 단순히 필드를 변경하는 set 메서드를 공개 (public) 범위로 만들지 않는다.**

\- 공개 범위의 set 메서드는 도메인 로직이 한 곳으로 응집되지 않도록 하는 단점이 있음 > 유지보수 힘듦

#### **2) 밸류 타입은 불변으로 구현한다.**

\- 애그리거트 외부에서 밸류 객체의 상태를 구현할 수 없도록 만들어줌. > 애그리거트 전체의 일관성을 올바르게 유지 가능

**\* 트랜잭션 범위**

\- 작을수록 좋다! > 잠금 대상이 많아질수록 성능은 떨어짐

\- 한 트랜잭션에서는 한 애그리거트만 수정해야함 > 애그리거트에서 다른 애그리거트를 변경하지 않는다는것을 의미

\>> 애그리거트는 서로 최대한 독립적인것이 좋기에 권장하는 사항일 뿐 강제성을 갖지는 않는다!

### **3\. 리포지터리와 애그리거트**

\- 리포지터리는 애그리거트 단위로 존재한다.

\- 보통 save와 findById의 두 메서드를 기본으로 제공한다.

### **4\. ID를 이용한 애그리거트 참조**

\- 애그리거트도 다른 애그리거트를 참조할 수 있다. (마치 객체처럼)

\- 애그리거트에서 다른 애그리거트를 참조한다는 것은 다른 애그리거트의 루트를 참조한다는 것과 같다.

\- **JPA는 @ManyToOne, @OneToOne 같은 애너테이션을 이용하여, 필드를 이용한 다른 애그리거트 참조를 쉽게 할 수 있음.**

\* 필드를 통한 루트 애그리거트 참조는 다음과 같은 문제점을 내포함

**1) 편한 탐색 오용**

\- 애그리거트 내부에서 다른 애그리거트를 수정하기 쉬워짐. > 한 애그리거트가 관리하는 범위는 자기 자신으로 한정해야 함을 명심

**2) 성능에 대한 고민**

\- 지연 로딩 (lazy) 와 즉시 로딩 (eager) 중 무엇을 사용할지 등의 다양한 경우를 고려해서 코딩해야하는 번거로움이 생김

**3) 확장 어려움**

\- 하위 도메인마다 다른 DBMS를 사용하게 되는 경우 더 이상 다른 애그리거트 루트를 참조하기 위해 JPA같은 단일 기술을 사용할  수 없게된다.

\>>> ID를 이용해서 다른 애그리거트를 참조한다면, 위 세 가지 문제점을 완화할 수 있다!!!

**\* ID를 이용한 애그리거트 간접 참조**

\- 모든 객체가 참조로 연결되지 않고 한 애그리거트에 속한 객체들만 참조로 연결됨 > 모델의 복잡도를 낮추고, 응집도를 높여줌

\- 다른 애그리거트를 직접 참조하지 않으므로, 지연 로딩으로 할지 즉시 로딩으로 할지 고민하지 않아도 됨. 구현 복잡도가 낮아짐.

\- 애그리거트별로 다른 구현 기술을 사용하는 것도 가능해짐

### **5\. 애그리거트 간 집합 연관**

\- 애그리거트 간 1-N 연관과 M-N 연관이 있음

1) 1-N 연관

\- Set과 같은 컬렉션을 이용해서 표현 가능

\- 페이징 등을 이용할 때 모든 데이터를 조회해야 되므로 성능이 급격히 떨어진다는 단점이 있음.

```
## 1-N 연관 예시

public class Category {
	private Set<Product> products; // 다른 애그리거트에 대한 1-N 연관
    
    
    
}
```

**2) M-N 연관**

\- 개념적으로 양쪽 애그리거트에 컬렉션으로 연관을 만든다.

\- 주로 사용되는 방식임

```
## M-N 연관 예시

public class Product {

	pricate Set<CategoryId> categoryIds;
    
    
}
```

### **6\. 애그리거트를 팩토리로 사용하기**

\- 애그리거트가 갖고 있는 데이터를 이용해서 다른 애그리거트를 생생해야 할 때 고려해보자.

```
public class Store {
	
    public Product createProduct(Productld newProductld, ProductInfo pi) {
		if (isBlockedO) throw new StoreBlockedException();
		return ProductFactory.create(newProductId, getldQ, pi);
    }
```

```
public class RegisterProductService {

	public Productld registerNewProduct(NewProductRequest req) {
		Store store = storeRepository.findByld(req.getStoreldO);
		checkNull(store);
		Productld id = productRepository.nextldO
		Product product = store.createProduct(id,
		productRepository.save(product);
		return id;
    }
    
    ~~
}
```

Product의 경우 제품을 생성한 Store의 식별자를 필요로 함

\> 따라서, Store에 Product를 생성하는 팩토리 메서드를 추가

\> Product를 생성할 때 필요한 데이터의 일부를 직접 제공하면서 동시에 도메인 로직을 함께 구현 가능

**\>>> Product 생성 가능 여부를 확인하는 도메인은 Store에 위치함. 해당 로직을 변경해도 도메인 영역의 Store만 변경하면 되고, 응용 서비스는 변경하지 않아도 된다!!!!!**


## Chapter 4. 리포지터리와 모델 구현

### **1\. JPA를 이용한 리포지터리 구현**

**1) 모듈 위치**

\- 리포지토리는 애그리거트와 같이 도메인 영역에 속함

\- 리포지토리를 구현한 클래스는 인프라스트럭처 영역에 속함

### **2\. 리포지터리 기본 기능 구현**

\- 리포지터리가 제공하는 두 가지 기본 기능은? >  save(), findById()

\- 리포지터리 인터페이스는 애그리거트 루트를 기준으로 작성한다!

\- JPA의 EntityManager를 이용해 다른 추가적인 기능을 구현 가능

### **3\. 스프링 데이터 JPA를 이용한 리포지터리 구현**

\- 스프링과 JPA를 함께 적용할 때 사용함

\* 스프링 데이터 JPA의 기본적인 규칙

1) 엔티티를 저장하는 메서드

\> save()

2) 엔티티를 조회하는 메서드

\> findById()

3) 특정 프로퍼티를 이용해서 엔티티를 조회하는 메서드

\- findBy(프로퍼티 값)

\> findByOrderer()

4) 중첩 프로퍼티를 통해 엔티티를 조회하는 메서드

\> findByOrdererMemberId()

5) 엔티티를 삭제하는 메서드

\> delete()

\> deleteById()

### **4\. 매핑 구현**

**1) 엔티티와 밸류 기본 매핑 구현**

\- 애그리거트 루트는 엔티티이므로 @Entity로 매핑 설정

\- 한 테이블에 엔티티와 밸류 데이터가 같이 있다면? 밸류는 @Embeddable, 밸류 타입 프로퍼티는 @Embedded

**2) 기본 생성자**

\- JPA에서 @Entity와 @Embeddable 클래스를 매핑하려면 기본 생성자를 제공해야 함. 왜?

\> DB에서 데이터를 읽어와 매핑된 객체를 생성할 때 기본 생성자를 사용해서 객체를 생성하기 때문에!

\> Lombok으로 쉽게 처리하자! @NoArgsConstructor

**3) 필드 접근 방식 사용**

\- JPA는 필드와 메서드의 두 가지 방식으로 매핑을 처리할 수 있음.

\- 메서드 방식은 프로퍼티를 위한 get/set 메서드를 구현하면 됨

\> Lombok의 @Getter, @Setter로 쉽게 처리하자!

\> **set 메서드는 함부로 구현하지 말것,**

앞선 챕터에서 정리했던 것처럼 외부에 set 메서드 대신 의도가 잘 드러나는 기능을 제공해야함

ex) setState() -> cancel()

**4) AttributeConverter를 이용한 밸류 매핑 처리**

\- 두 개 이상의 프로퍼티를 가진 밸류 타입을 한 개 컬럼에 매핑하려고 할 때 사용됨

\- convertToDatabaseColumn() 메서드 : 밸류 타입을 DB 컬럼 값으로 변환

\- convertToEntityAttribute() 메서드 : DB 컬럼 값을 밸류로 변환

```
public interface AttributeConverter<X, Y> {	
	public Y convertToDatabaseColumn (X attribute);
	public X convertToEntityAttribute (Y dbData);
}
```

\- 위 예시에서 타입 파라미터 X는 밸류 타입, Y는 DB 타입

\- AttributeConverter 인터페이스를 구현한 클래스는 @Converter 애너테이션을 적용해야 함!

```
// Money를 위한 AttributeConverter 구현

@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter<Money, Integer〉{

    @Override
    public Integer convertToDatabaseColumn (Money money) {
    	return money = = null ? null : money.getValue();
    }
    
	@Override
    public Money convertToEntityAttribute(Integer value) {
    	return value = = null ? null ： new Money(value);
    }
}
```

\> autoApply 속성이

true : 해당하는 타입의 모든 프로퍼티에 대해 자동으로 적용

```
// autuApply = true 예시

©Entity
@Table(name = "purchase_order")
public class Order {

    @Column(name = "total_amounts")
    private Money totalAmounts; // MoneyConverter를 적용해서 값 변환


}
```

false (기본 값) : 프로퍼티 값을 변경할 때 사용할 컨버터를 직접 지정해야 함

```
// autuApply = false 예시

©Entity
@Table(name = "purchase_order")
public class Order {

    @Column(name = "total_amounts")
    @Convert(converter = MoneyConverter.class) // 사용할 컨버터 직접 지정
    private Money totalAmounts;


}
```

**5) 밸류 컬렉션 : 별도 테이블 매핑**

\- @ElementCollection과 @CollectionTable을 함께 사용

\> @CollectionTable : 밸류를 저장할 테이블을 지정하는 어노테이션

**6) 밸류 컬렉션 : 한 개 컬럼 매핑**

\- AttributeConverter를 사용하면 밸류 컬렉션을 한 개 컬럼에 쉽게 매핑 가능! (4-4 참고)

**7) 밸류를 이용한 ID 매핑**

\- 밸류 타입을 식별자로 매핑하면, @Id 대신 @EmbeddedId 어노테이션을 사용함

\- JPA에서 식별자 타입은 Serializable 이어야 하므로, 식별자로 사용할 밸류 타입은 Serializable 인터페이스를 상속받아야함

**8) 밸류 컬렉션을 @Entity로 매핑하기**

\- 밸류인데 @Entity를 사용해야 할 때도 있음

\- JPA에서 상속 구조를 갖는 밸류 타입을 사용하려면, @Embeddable 대신 @Entity를 사용해야함

### **5\. 애그리거트 로딩 전략**

\- JPA 매핑을 설정할 때는 애그리거트에 속한 객체가 모두 모여야 완전한 하나가 됨!

\> FetchType.EAGER (즉시 로딩)을 이용하면 조회 시점에서 완전한 상태의 애그리거트를 로딩 가능

\> 그러나 컬렉션에 대해서 즉시 로딩을 사용하면 문제가 될 수 있음

\> 애그리거트에 맞게 지연 로딩과 즉시 로딩을 잘 설정해야 함

\* **기본적으로는 LAZY를 사용, 정말 필요한 상황에만 EAGER를 명시적으로 사용**함.

\* 예를 들면, 작은 규모의 애그리거트 또는 항상 함께 조회해야 하는 경우 EAGER를 사용한다.

### **6\. 도메인 구현과 DIP**

\- DIP에 따르면 도메인이 인프라에 의존해서는 안됨 그러나 우리는 JPA와 같은 라이브러리를 사용하며

도메인이 인프라에 의존하는 상황을 만든다. 왜? 이렇게 하면 어노테이션만 붙이면 되서 편하니까!

\> **DIP를 완벽하게 지키면 좋음, 그러나 개발 편의성과 실용성을 챙기는 것 또한 중요함!**

## Chapter 5. 스프링 데이터 JPA를 이용한 조회 기능

### **\* JPA의 Specification (스펙)**

\- 다양한 검색 조건을 조합해야 할 때 **필요한 조합마다 find 메서드를 정의하는것은 좋은 방법이 아님!**

```
List<Order> findByOrdererId(String ordererId);
List<Order> findByOrdererIdAndStatus(String ordererId, OrderStatus status);
List<Order> findByOrdererIdAndFromDateAfter(String ordererId, Date fromDate);
List<Order> findByOrdererIdAndFromDateAfterAndStatus(String ordererId, Date fromDate, OrderStatus status);
// ... 조합이 늘어날수록 메서드가 기하급수적으로 늘어남
```

\- 위와 같은 문제점이 발생하는데, 이런 경우 JPA의 **'Specification (스펙)' 기능을 사용하면 동적으로 조건을 조합 가능!**

```
public interface OrderRepository extends JpaRepository<Order, Long>, JpaSpecificationExecutor<Order> {
}

// 검색 조건을 동적으로 조합
Specification<Order> spec = Specification.where(OrderSpecs.hasOrdererId("user1"))
    .and(OrderSpecs.hasStatus(OrderStatus.PAID))
    .and(OrderSpecs.orderDateAfter(someDate));

List<Order> results = orderRepository.findAll(spec);
```

\- 스펙을 충족하는 엔티티를 검색하고 싶으면 findAll() 메서드를 사용하면 된다.

\- 스펙 인터페이스는 and, or, not, where 등의 메서드를 제공함

### **\* 스펙 빌더 클래스**

\- 스펙 구조가 복잡할 때는, 스펙 빌더 클래스를 사용하면 이를 완화할 수 있다.

\- and(), ifHasText(), ifTrue() 메서드 외 필요한 메서드를 추가해서 사용 가능하다.

```
// 스펙 빌더 클래스
public class SpecBuilder<T> {
    private List<Specification<T>> specs = new ArrayList<>();

    public static <T> SpecBuilder<T> builder() {
        return new SpecBuilder<>();
    }

    public SpecBuilder<T> and(Specification<T> spec) {
        if (spec != null) specs.add(spec);
        return this;
    }

    public SpecBuilder<T> ifHasText(String value, Function<String, Specification<T>> specSupplier) {
        if (value != null && !value.trim().isEmpty()) {
            specs.add(specSupplier.apply(value));
        }
        return this;
    }

    public SpecBuilder<T> ifNotNull(Object value, Function<Object, Specification<T>> specSupplier) {
        if (value != null) {
            specs.add(specSupplier.apply(value));
        }
        return this;
    }

    public Specification<T> toSpec() {
        return specs.stream().reduce(Specification::and).orElse(null);
    }
}
```

```
// 스펙 정의 예시
public class UserSpecs {
    public static Specification<User> nameLike(String name) {
        return (root, query, cb) -> cb.like(root.get("name"), "%" + name + "%");
    }
    public static Specification<User> createdAfter(LocalDateTime dateTime) {
        return (root, query, cb) -> cb.greaterThan(root.get("createdAt"), dateTime);
    }
}
```

```
// 빌더를 활용한 동적 검색 조건 조합
public List<User> searchUsers(String name, LocalDateTime createdAfter) {
    Specification<User> spec = SpecBuilder.<User>builder()
        .ifHasText(name, UserSpecs::nameLike)
        .ifNotNull(createdAfter, dt -> UserSpecs.createdAfter((LocalDateTime) dt))
        .toSpec();

    return userRepository.findAll(spec);
}
```

### **\* 정렬 지정하기**

\- JPA는 두 가지 방법을 사용해서 정렬을 지정할 수 있음

1) 메서드 이름에 OrderBy를 사용해서 정렬 기준 지정

2) Sort를 인자로 전달

```
findByOrdererIdOrderByOrderDateDeseNumberAsc
```

\- 위와 같이 두 가지 이상의 프로퍼티에 대한 정렬 순서도 지정할 수 있음

\- **프로퍼티가 많아지면 메서드 이름이 길어지는 단점이 있는데 이를 해결하기 위해 Sort 타입을 사용함**

Sort 예시 1번

```
Sort sort1 = Sort.by("number").ascending();
Sort sort2 = Sort.by("orderDate").descending();

Sort sort = sortl.and(sort2);
```

Sort 예시 2번 (1번을 짧게 표현)

```
Sort sort = Sort.by("number").ascending().and(Sort.by("orderDate").descending());
```

### **\* 페이징 처리하기**

\- Pageable 타입 파라미터를 사용하면 페이징을 자동으로 처리해줌

### **\* JPA 동적 인스턴스 생성**

\- JPA는 쿼리 결과에서 임의의 객체를 동적으로 생성할 수 있는 기능을 제공함

\- 쉽게 말하면, **쿼리 결과를 DTO로 바로 매핑이 가능**하다!

\- @Query 어노테이션과 new 키워드를 활용하면 된다.

```
@Query("SELECT new com.example.dto.UserDto(u.id, u.name) FROM User u WHERE u.status = :status")
List<UserDto> findUsersByStatus(@Param("status") String status);
```

위와 같이 JPQL을 사용해서 그 결과를 DTO로 바로 매핑이 가능하다.

\> SELECT절의 new 키워드와 @Query 어노테이션 사용

### **\* JPA 하이버네이트 @Subselect 사용**

\- @Subselect는 쿼리 결과를 @Entity로 매핑할 수 있는 기능!

\- 데이터베이스 뷰를 만들기 어려운 상황에서, 임시로 복잡한 쿼리 결과를 엔티티처럼 다루고 싶을 때 주로 사용함

\- **실제 테이블이 생기는 것이 아니기 때문에 읽기 전용 작업에서만 사용이 가능하다 쓰기 작업 불가능!**

\- 보통 대부분의 경우, **복잡한 쿼리 결과는 DTO로 직접 매핑해서 반환하는것이 더 명확하고 유지보수에도 유리함**
