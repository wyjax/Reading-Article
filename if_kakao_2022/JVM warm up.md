# JVM warm up

JVM 컴파일러와 warm up

소스코드 → 바이트코드 → jvm - interprete → machine code

c++ 에 비해서 성능이 떨어진다. 왜냐하면 컴파일을 통해서 기계어를 만들기 때문이다. 하지만 단점은 다른 운영체제에서 실행하기가 힘들다

### JIT(just in time) Compiler

적시에 기계어를 만들어낸다.

바이트코드를 캐시에저장하고 기계어로 변환한다. JIT는 자바 컴파일 단계에서는 성능이 좋지만 런타임에서는 도움을 주지 않는다. 그래서 warm up 단계가 필요하다.

CPU - 평균 10% 이하

메모리 - 60% 이하

네트워크 - 10 ~ 20 mb

TPS - transaction per second 초당 트랜잭션 개수

> 레디스는 싱글 노드로 운영
> 

문제해결

### JIT

- method
- profiling
- tiered compliation
    - 단계별로 코드 최적화
        - c1 : 간략한 최적화
        - c2 : 최대 최적화, 코드이
    - complication level (level 0 ~ level 4까지 존재함)
        - level 0 : interpreted code
        - level 1 : simple C1 compiled code
        - level 2 : limited C1 complied code
        - level 3 : full C1 compiled code
        - level 4 : C2 compiled code
        

reference → [https://www.youtube.com/watch?v=CQi3SS2YspY](https://www.youtube.com/watch?v=CQi3SS2YspY)