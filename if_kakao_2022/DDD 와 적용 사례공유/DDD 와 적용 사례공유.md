# DDD 와 적용 사례공유

레거시 서비스에 ddd를 적용하여 MSA에 적용했던 사례 공유

### 파트너 사이트?

통계나 그런 사이트를 간단하게 제공하는 것

### 레거시 서버

1. 모놀리틱
2. 기술부채
3. 유지보수의 어려움
4. 그능의고착화

  > MSA와 DDD를 진행하게 되었다.

Domain Driven Design

1. 도메인의 모델과 로직에 집중
2. 유비쿼터스 언어, 보편적 언어 사용
3. 소프트웨어 엔티티와 도메인간 개념의 일치
    1. 분석 모델과 설계가 다른게 아니라, 도메인 모델과 코드가 모두 동일하게 가게 하는 것이다

왜 TDD 대신에 DDD를 사용했을 

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled.png)

TDD는 테스트 주도 개발

BDD는 비즈니스와 협업 중시

 필요한 기능을 구분하고 필터로 개선하는 점진적인 방법을 선택했다. 리소스를 유연하게 가동하고자 레거시와 신규서버를 동시 가동하게 하기 위해서 ㄷㄷㄷ를 선택

### DDD 적용에 필요했던 것들

중요한 개념

- bounded context
- context map : 바운디드의 관계를 나타냄
- aggregate : 라이프 사이클이 같은 도메인들을 모아놓은 집합

바운디드 컨텍스트

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%201.png)

각 영역들을 관리하여 의존성을 줄이기 위해 경계를 놓는다. msa상 각각의 서비스로 나뉘게 된다.

컨텐츠 제작, 메타정보 관리, 판매가 묶임

정산, 통계까 묶임 

### 컨텍스트 Map

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%202.png)

### 어그리거트 예시

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%203.png)

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%204.png)

상품 관리시에 사용을 하게 된다. 

DDD 대표적인 아키텍처

레이어트 아키텍처 (계층형)

- user interface
- application
- domain
- infrastructure

### 클린 아키텍처

- External interface
- interface adaptor
- use case
- entity

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%205.png)

핵사고날 아키텍처

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%206.png)

플스 게임기와 같이 이해하면 된다. 알맞게 구현만 해주면 어떤 유틸리티든 사용할 수 있다. 비즈니스 로직이나 표현로직이 데이터 접근 로직에 의존하지 않도록 하는 것이 중요한 목표

### 선택한 방식?

레이어트 아키텍처 → 거대해지면 애플리케이션이 오염되는 문제가 있을 수 있음

클린 아키텍처보다는 포텐 어댑터라는 명확한 구현 개념이 포함되어 있는 핵사고날을 표현하였다.

### 예시) 파트너 사이트

![Untitled](DDD%20%E1%84%8B%E1%85%AA%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%20%E1%84%89%E1%85%A1%E1%84%85%E1%85%A8%E1%84%80%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B2%20ac9825a7498549e498d518df5bf208ce/Untitled%207.png)

```java
public class Product {
	private ProductSlesInfo productSalesInfo;
	private ProductMetaInfo productMetaInfo;
	private Content content;

	/*
		판매시작
	*/
	public void startSale(LocalDateTime startSaleDt) {
		validateProduct();
		productSalesInfo = ProductSalesInfo.startSale(startSaleDt);
	}
}
```

내부에서 작품을 판해마히기 위해서 필요한 유효성 검사와 판매시장 날짜에 대해서 처리할 수 있도록 하는 관련 로직이 들어가 있음 → 이것은 도메인 로직

```java
// 코드 예시
public void salesProduct(ProductSalesCommand cmd) {
	Product product = loadProductPort. loadProduct(cmd.getProductId());
	// 로드 프러덕트 기능을 ㄹ수행하고 
	Product.startSale(LocalDateTime.now());
	// start sale

	saveProductPort.saveProduct(product);
	// 성공하면 save port를 사용
	eventPublishPort.publisherEvent(product);
	// 알림 시스템으로 메시지 전송을 위한 event port
}
```

CRUD중 R을 제외한 나머지를 분리한 이유는? cqrs 패턴 커맨드와 쿼리 책임을 분리하기 위해서 그런것이다. 

추후에는 조회 성능 개선을 위해서 rabbitmq를 사용하거나 아마존 sqs 를 사용하여 성능 개선을 하기 위해서다

```java
class ProductPersistenceAdapter implements LoadProductPort, SaveProdctPort {

	@Override
	public Product load~~() {
		ProductJpEntity product = productRepository.findById();
		ContentJpaEntity content = contentRepository.findByProductId(productId);
		return mapper.mapToDomainEntity(...);
}

	@Overrive
	public vlod saveProduct(Product product) {
		productRepository.save(mapper.mapToJpaEntity(product));
	}
}
```

하나의 어그리거트는 여러 개의 테이블이나 데이터와 연관이 있을수 있기 떄무넹 각각의  데이터를 하나의 도메인 집합으로 만들어줄 때 변환해주는 것이 필요함

### Mapper

```java
class ProductMapper {
	Product mapToDomainEntity(...) {
		return Product.withId(
			new Product.ProductId(productJpaEntity.getId())
						.ProductSalesInfo...
						ProductMetaInfo...
						Content...);
	}

	ProductJpaEntity mapToJpaEntity(Product product) {
		return ProductJpaEntity...
				.build();
	}
}
// 도메인과 맵퍼를 모두 구현하여 해결을 하였다.
```

### 안좋은 점

1. msadㅔ서 오는 단점은 별개로
2. 아키텍처 구현에서 생성되는 생각보다 많은 코드
3. 각 도메인에 대한 높은 이해도가 필요

### 좋은 점

1. 보편적인 언어 사용에 따른 빠른 커뮤니케이션
2. 도메인간 관계가 복잡한 경우 큰 틀에서 정리가 가능
3. 도메인의 분리에 따른 유지보수에 대한 편의성
    1. 새로운 요구사항에 관한 유연성도 갖추게 되었다.
4. 새로운 기능 및 요규 사항에 대한 유연성
5. 캡슐화가 자동으로 되었다. 
6. loose zㅓ플링, high cohesion
    1. 유즈 케이스나 포트를 사용하여 분리를 시켜 의존도를 낮춰고 도메인과 서비스를 필요한 곳에 모아서 사용했기 때문에 응집도를 높였다.
7. 도메인 로직의 분리로 비즈니스 로직에 집중할 수 있었다.
8. 코드 가독성 향상