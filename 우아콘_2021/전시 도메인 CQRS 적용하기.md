# 전시 도메인 CQRS 적용하기

도메인을 데이터로 집어넣으려고 하면 정책이나 외부에 노출하면 안되는 데이터가 같이 들어갈 때도 있는데 이러한 것들은 복잡하게 만든다.

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd5c1c36d-8469-4b65-9156-a69cc6e9b7fb%2FUntitled.png?id=cfefe953-369b-47a8-a598-22d55348395b&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=1060&userId=&cache=v2)

이로 인해서 도메인과 모델이 훨씬 복잡해지게 된다. 

이렇게 하다 보면 명령 로직과 조회 로직이 하나의 모델만 보게 된다.

이거를 분리한다..

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F1350d551-d334-4087-b0ab-0b39f887cae0%2FUntitled.png?id=eca20425-b5b3-460c-8017-cd35e81c8824&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=2000&userId=&cache=v2)

조회가 목적이면 엔티티보다는 dto를 사용하는 것을 추천한다. 그리고 조회를 하면서 성능상 문제가 생길 수도 있다. 우리는 이걸 저장하는 시점에 조회 모델을 생성해서 저장하였다. DB에 있는 값 그대로 가져와 이용가능 하도록 설계하는 것을 추천하고 있다. (여기서는 json 데이터로 저장하였다) 그래서 nosql을 사용하였고 데이터를 그대로 읽었고 조회해서 캐시를 이용하지 않고 보다 근본적으로 해결하였다.

```java

CatalogReadOnly {
	id,
	parent_id,
	children_ids,
	...
}
이런 식으로 비정규화된 데이터를 NoSql에 저장한다.
```

조회에 최적화된 데이터베이스를 사용할 수도 있다. (redis, 엘라스틱서치, 몽고디비) 우리는 레디스를 선택해서 사용하였다.

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F31bc0c63-e413-48a9-af2a-469b7678ad8d%2FUntitled.png?id=a9126b3b-63ab-423f-b199-8ba67d216603&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=1340&userId=&cache=v2)

하지만 네모박스처럼 어색한 부분이 아직 남아있었다. 조회 모델을 생성하는 리소스가 많은데 저장하는데가 리소스를 소비하게 돼서 부담이 된다.

### 이벤트 소싱

데이터가 저장이 되었을 때 이벤트로 만들어 event store에 저장하고 consumer에서 데이터를 만든다.

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcc86893f-9952-41bd-8b45-74ff3e163e6d%2FUntitled.png?id=4c7eda4c-4e35-4707-a14f-a11f5355778d&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=1340&userId=&cache=v2)

 정리하면

1. 명령 로직으로 데이터를 저장한다.
2. 저장과 동시에 이벤트를 발생시켜 이벤트 스토어에 저장
3. 발행된 이벤트를 consumer가 소비하여 조회 모델을 만들어서 저장한다.
4. 조회시에 read model을 읽어와서 보여준다.

### CQRS 적용 적기

- UX와 비즈니스 요구 사항이 복잡해질 때
- 조회 성능을 보다 높이고 싶을 떄 (사용자가 많아지고, 트래픽 부담이 생길 때)
- 데이터를 관리하는 영역과 이를 뷰로 전달하는 여역의 책임이 나뉘어져야 할 때
- 시스템 확장성을 높이고 싶을 때

> CQRS는 서비스 전체에 도입하는 것이 아니라 필요한 부분에 도입하면 된다.
> 

### 의존성 해결

한 클래스에 여러 비즈니스 로직을 모아서 처리를 먼저 하였다. 근데 양방향으로 있는 것들이 있었고 우리들은 이것을 정리하였다. (단방향으로 변경) 그리고 한 클래스가 1개의 책임 이상을 가지기도 하였고… 그래서 정리하였다. 

전체 트리 모델이 카탈로그 목록, 카탈로그, 상품으로 나뉘게 되었다.

- 카탈로그 목록
- 카탈로그
- 상품

> 조회 모델은 조회 상에 성능 이점이 가능하다
> 

도메인 로직에 필요한 데이터를 그 자체를 조회 모델로 만들면 된다.

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fd4f77f2b-97bb-4a26-803b-a8936d2fd781%2FUntitled.png?id=04e887ac-6698-4624-9f4f-6dfbd75bfb5c&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=1340&userId=&cache=v2)

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F6d0ba95d-d97e-48c9-a0a2-29fa5a1d8bf8%2FUntitled.png?id=a66a3139-e208-41f9-b40f-c4460d00b613&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=1340&userId=&cache=v2)

### 최종그림

우리팀은 명령하는 팀과 조회하는 팀으로 나뉘어져 있고 조회하는 팀이 명령하는 팀의 로직을 알 필요가 없었기 때문에 분리를 시켜 운영하게 되었기 때문에 아래와 같은 그림을 고려하였다.

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F516da674-0ff5-4a45-98be-4d9462dd19bf%2FUntitled.png?id=21f17504-8fee-48b8-a1e9-1059a859fbc2&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=1340&userId=&cache=v2)

### 이벤트 스토어에 저장되는 내용

- 엔티티 업데이트에 따라 갱신 해야할 대상 id
- 변경 감지할 property 지정 - Optional
- 실행 메서드 지정 (선택해서 보내기 떄문에 성능상 이점)
- 데이터 변경자 이름

![Untitled](https://wyjax.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F59d135f6-bf25-48c0-82cf-2d1875d090dc%2FUntitled.png?id=67b5ae85-b234-4d29-bd31-2d54d620155d&table=block&spaceId=bbed102e-42c3-40a8-b277-7e1ab60cf7eb&width=2000&userId=&cache=v2)

### 변경 감지 방법

- Jpa EntityListeners
    - 가장 추상화 잘되어 있고 쓰기 간편하다
    - 콜백함수 지정 가능 prepersist, postpersist
- Hibernate Eventlistener
    - 26가지 디테일한 상황에 콜백이 가능
    - 보세 상세한 정보 전달 (변경된 프로퍼티, 이전 상태, 현재 상태 등)
    - 모든 엔티티 변경 사항이 전달됨
    - 다만 1개의 엔티티가 특정되지 않고 변경된 엔티티가 전부 전달된다.
- Hibernate Interceptor
    - session, interceptor에 등록가능
    - 저장될 데이터 조작 가능
- Spring AOP
    - methodd에만 설정 가능
    - 메서드 실행 전후, 반환 후, 예외 상황, 어노테이션 붙은 경우
    - pointcut 문법으로 동작
    - 특정 케이스만 사용했음

결과적으로 Hibernate EventListener, Spring AOP