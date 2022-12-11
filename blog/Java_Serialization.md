## 자바 직렬화 



#### 자바 직렬화란?

자바 직렬화란 자바 시스템에서 사용되는 객체나 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 byte형태로 데이터를 변환하는 기술과 byte로 변환된 데이터를 다시 객체로 변환하는 기술을 아울러서 말한다.



#### 직렬화vv 

자바 기본 타입과 ```java.io.Serializable``` 인터페이스를 상속받은 객체는 직렬화 할 수 있는 기본 조건을 가지게 된다.

```java
public class User implements Serializable {
    private String uid;
    private String name;
    
    @Override
    public String toString() {
        return String.format("User{uid='%s', name='%s'}", uid, name);
    }
}
```



#### 직렬화 방법

자바에서 직렬화하는 방법은 ```java.ioObjectOutputStream``` 을 이용하는 것이다.

```java
Member member = new Member("wyjax", "wyjax", 27);
byte[] = serializedMember;
try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
    try (ObjectOutputStream oos = new ObjectOutputStream()) {
        oos.writeObject(member);
        // serializedMember -> 직렬화된 member 객체
        serializedMember = baos.toByteArray();
    }
}
// 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
System.out.println(Base64.getEncoder().encodeToString(serializeMember));
```



#### 역직렬화 방법

```java
String base64Member = "Member {name='good', uid='good'}";
byte[] serializedMember = Base64.getDecoder().decode(base64Member);

try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
        // 역직렬화된 Member 객체를 읽어온다.
        Object objectMember = ois.readObject();
        Member member = (Member) objectMember;
        System.out.println(member);
    }
}
```

직렬화 관련된 문제입니다. 아직은 아무것도 보고된 것이 없습니다.ㄴㄴ

#### 역직렬화시 클래스 구조 변경 문제

```java
public class Member implements Serializable {
    private String name;
    private String email;
    private int age;
    // ...
}
```

이 클래스를 역직렬화하면?

```java
Base64.getEncoder().encodeToString(serializedMember);
```

```
rO0ABXNyABp3b293YWhhbi5ibG9nLmV4YW0xLk1lbWJlcgAAAAAAAAABAgAESQADYWdlSQAEYWdlMkwABWVtYWlsdAASTGphdmEvbGFuZy9TdHJpbmc7TAAEbmFtZXEAfgABeHAAAAAZAAAAAHQAFmRlbGl2ZXJ5a2ltQGJhZW1pbi5jb210AAnquYDrsLDrr7w=
```

이런 식으로 나온다. 이 문자열을 바로 역직렬화 시키면 바로 Member 객체로 변한다.

​                      

##### Member 클래스의 구조 변경에 대한 문제

```java
public class Member implements Serializable {
    private String name;
    private String email;
    private int age;
    // phone 속성 추가
    private String phone;
   //...
}
```

제일 좋은 것은 'phone' 필드를 추가했어도 역직렬화 했을 때 phone이 없어도 자동으로 채워졌으면 좋겠다는 것이다. 하지만 이것을 역직렬화 시키면...

```java
java.io.InvalidClassException: // exception이 나온다 
```

위와 같은 예외가 발생하는 이유는 ```serialVersionUID``` 가 맞지 않아서다. SUID는 필수 값은 아니고 호환가능한 클래스는 SUID값이 고정되어 있다. suid가 선언되어 있지 않으면 클래스의 기본 해쉬값을 사용한다. 그렇기 때문에 내부에 SerialVersionUID를 직접 기술하지 않아도 내부적으로 serialVersionUID 정보가 추감되며, 내부 값도 자바 직렬화 스펙 그대로 생성된 클래스의 해쉬 값으로 되어 있다.



##### 그렇다면...

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private String email;
    private String phone;
    // 생략  
}
```

조금이라도 역질렬화 할 때 대상 클래스에 에러가 발생되지 않으려면 ```serialVersionUID```를 관리 해줘야 한다. 하지만 suid가 동일하다고 해서 신경써야할 부분이 없는 것이 아니다 !



##### 멤버 변수명은 같은데 변수 타입으 변경될 때

```java
private String age; // 원래 변수

private int age; // 변경된 변수
```

```java
 java.lang.ClassCastException: cannot assign instance of java.lang.String to field woowahan.blog.exam1.Member.email of type java.lang.StringBuilder in instance of woowahan.blog.exam1.Member // 에러내용
```

직렬화 예외가 발생한 것을 볼 수 있다. 그러면 필드가 추가되거나 삭제됐을 때에는 어떻게 변경이 될까?



##### 직렬화된 자바 데이터에 존재하는 멤버 변수가 없애거나 추가했을 때

```java
public class Member {
    private static final long serialVersionUID = 1L;
    private String name;
    private String email;
    // private int age; // 제거   
}

// 결과 :     Member{name='wyjax', email='wyjax.dev@gmail.com'}
```

예외는 발생하지 않는다. 그냥 값 자체만 없어졌다. 그렇다면 멤버 변수를 추가할 경우에는?

```java
public class Member implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private String email;
    private int age;
    private String nickName; // 추가한 변수
}

// 결과 : Member{name='wyjax', email='wyjax.dev@gmail.com', age=27, nick=null}
```

필드를 추가해도 예외가 발생하지 않는다. 그 이유는 ```serialVersionUID```를 넣어줬기 때문이다.



#### 자바 직렬화시 확인해야할 점

- 특별히 문제없으면 자바 직렬화 버전은 개발시 직접 관리되어야 한다.
- serialVersionUID의 값이 동일하면 멤버 변수 및 메서드 추가는 크게 문제될 것이 없다.
- 역직렬화 대상의 클래스의 멤버 변수 타입 변경을 지양해야 한다. 자바 역직렬화시에 타입으 엄격하다. 나중에 타입이 변경이 되면 직렬화된 데이터가 존재하는 상태라면 발생할 예외의 경우의 수를 다 신경 써야 한다.
- 외부(DB, 캐시 서버, NoSQL 서버)에 **장기간 저장될 정보**는 자바 직렬화 사용을 **지양**해야 한다. 역직렬화 대상의 클래스가 언제 변경이 일어날지 모르는 환경에서 긴 시간 동안 외부에 존재했던 직렬화된 데이터는 쓰레기가 될 가능성이 높다.
- 개발자가 직접 컨트롤이 가능한 클래스의 객체가 아닌 경우에는 직렬화를 지양해야 한다. (개발자가 직접 컨트롤하기 힘든 객체는 프레임워크 또는 라이브러리에서 제공하는 클래스의 객체를 이야기한다.)

> 자바 직렬화를 사용할 때는 될 수 있으면 자주 변경되는 클래스의 객체는 사용 안하는 것이 좋다. 변경이 취약하기 때문에 어떠한 상황에서 예외가 발생할 지 모르기 때문이다. 특히 역직렬화가 되지 않을 때와 같은 상황을 예외처리하는 것이 좋다.



#### 용량 문제

자바 직렬화시에는 기본적으로 타입에 대한 정보같은 클래스의 메타 정보도 가지고 있기 때문에 다른 포맷에 비해서 용량이 커진다. 이건 클래스의 덩치가 커지게 되면 비례해서 커진다. 
예를 들면 클래스 안에 List<>, Map<> 인터페이스를 가지고 있는 경우에 직렬화를 하게 되면 class 뿐만 아니라 내부의 List, Map에 대한 클래스 정보도 포함하고 있기 때문에 용량이 비대해진다. 최소 2배의 크기를 더 가지게 된다.



#### 직렬화를 사용하는 이유

자바 직렬화 형태의 데이터 교환은 자바 시스템 간의 데이터 교환을 위해서 존재한다.

- JVM의 메모리에서 상주하는 객체 데이터를 그대로 영속화(persistence)할 때 사용된다. (시스템이 종료되어도 사라지지 않으며, 영속화된 데이터이기 때문에 네트워크로 전송도 가능하다)
- Servlet Session : servlet 기반의 WAS들은 대부분 세션의 Java 직렬화를 지원한다.
- Cache : 캐시할 부분을 직렬화된 데이터를 저장해서 사용
- Java RMI (Remote Method Invocation)
    - 원격 시스템의 메서드를 호출할 때 전달하는 메세지를 직렬화하여 사용
    - 메시지를 전달받은 원격 시스템에서는 메시지를 역직렬화하여 사용
- 객체가 세션에 저장하지 않는 단순한 데이터 집합이고, 컨트롤러에서 생성되어 뷰에서 소멸하는 데이터의 전달체라면 객체 직렬화를 고려하지 않아도 된다.



#### 직렬화의 장점

자바 직렬화는 자바 시스템에서 개발에 최적화되어 있다. 복잡한 데이터 구조의 클래스의 객체라도 직렬화 기본 조건만 지키면 큰 작업 없이 바로 직렬화가 가능하다. 데이터가 자동으로 타입에 맞게 맞춰준다.





***출처 : https://woowabros.github.io/experience/2017/10/17/java-serialize.html [우아한형제들 기술블로그]***