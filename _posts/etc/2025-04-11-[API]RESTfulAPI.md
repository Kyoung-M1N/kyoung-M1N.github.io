---
title: RESTful API
author: kymin
date: 2025-04-11 16:38
categories: [Etc, API]
tags: [api]

---

## **REST(Representational State Transfer)**

### **REST 아키텍쳐?**

Roy Fielding은 REST를 웹이라는 분산 하이퍼미디어 시스템을 위한 아키텍쳐로 정의한다. 이는 소프트웨어 공학 원칙을 적용하고 적용 과정에서 발생 가능한 문제점을 해결하기 위한 제약 조건들과 방법들에 의해 구성된다.

### **REST를 정의 하기 위한 6가지 제약조건**

REST라는 개념을 처음 제시한 Roy Fielding은 REST를 아키텍쳐로 정의하며 이를 충족시키기 위한 6가지 제약 조건을 제시한다.

- **Client-server architecture**
  - 데이터 저장과 사용자 인터페이스에 대한 관심사를 분리
  - 여러 플랫폼에서 같은 인터페이스를 사용하게 되며 사용자 인터페이스의 이식성 향상
  - view와 같은 요소를 구성할 필요없이 데이터만 전달하도록 단순화 되며 서버의 확장성을 향상
  - 클라이언트와 서버의 독립적인 개발, 유지보수, 확장 가능
  - 그러나 서비스가 동작하기 위해 클라이언트와 서버가 통신하여 상호작용이 필요

- **Stateless**
  - 클라이언트와 서버의 통신 과정은 본질적으로 stateless

  - 따라서 클라이언트가 서버에 보내는 요청에는 서버가 해당 요청을 이해하고 동작하는 데에 필요한 모든 정보가 반영

  - 클라이언트는 혼자서 서버나 데이터베이스의 정보에 접근할 수 없으며, 이에 따라 세션에 대한 정보는 전적으로 클라이언트가 관리

  - 요청에 충분한 정보가 포함되어 있기 때문에 요청에 대한 내용과 특징을 파악하기 위해 요청 외에 다른 것들을 고려하거나 참고하지 않아도 되므로 가시성 향상

  - 높은 가시성으로 인해 요청에 대한 부분적인 장애를 복구, 해결하기 위해 고려해야할 요소가 줄어들어 신뢰성 향상

  - 서버가 요청에 대한 정보를 저장하지 않아도 되므로 리소스(메모리)를 충분히 확보할 수 있게 되고, 요청에 대한 내용을 관리하는 과정이 없으므로 비즈니스 로직에만 집중할 수 있기 때문에 기능의 확장성 향상

  - 요청이 저장되지 않으므로 반복되는 요청에 대한 반복적인 동작으로 인해 네트워크 성능 저하가 발생

- **cache**
  - 무상태성에서 네트워크 성능을 향상시키기 위해 캐시 가능 제약조건을 추가
  - 응답 데이터가 클라이언트에서 cacheable인지 non-cacheable인지 명시적으로 지정
  - 응답이 캐싱 가능하면 동일한 요청에 대해 클라이언트가 캐시의 내용을 재사용할 수 있는 권한이 주어져 통신 휫수 감소
  - 이를 통해 네트워크 효율 향상, 확장성 증가, 성능 향상으로 인한 사용자 만족도 증가 등의 효과
  - 새로운 데이터의 추가, 수정, 삭제로 인해 캐시의 데이터와 차이가 발생하면 기능의 신뢰성이 떨어진다는 문제 발생
  
- **Uniform interface**
  - 소프트웨어공학의 원칙인 일반성(generality)를 적용하기 위해 일관된 인터페이스를 적용
  - 인터페이스를 일관된 형태로 정의하고 응답의 공통적인 형태를 만들어서 한 시스템 내에서 표준화
  - 응답마다 변환과정이 통일되므로 전체적인 시스템 아키텍쳐가 단순화, 가시성 향상
  - 이를 통해 서버와 클라이언트의 결합도가 더욱 약해지고 응집도가 향상
  - 기능이 수정되거나 추가되어도 공통된 인터페이스의 형태로 반환되기 때문에 진화가능성 향상
  - 하지만 각 응답이 표준화된 인터페이스의 형태로 변환되는 과정이 필요하기 때문에 효율성이 저하

- **Layered system**
  - 서버와 클라이언트의 시스템이나 중간 컴포넌트에 계층적 스타일을 적용
  - 다른 컴포넌트가 계층 너머의 동작을 알 수 없도록 기능을 분리
    - 예를 들어 클라이언트와 서버는 응답을 요청을 보내고 응답을 받아 처리하는 것을 별도의 계층으로 분리
    - 중간 컴포넌트 분산 서버 환경에서의 gateway나 proxy를 별도의 계층으로 분리
  - 기존 레거시 기능을 캡슐화, 신기능을 기존 클라이언트로부터 보호
  - 계층적 분리로 인해 각 계층의 인터페이스를 정의해야 하는 문제가 발생하지만, uniform interface로 인해 해결
  - 계층의 추가로 네트워크 성능의 저하가 발생할 수 있지만, cache를 이용하여 신뢰성이 떨어지지 않는 선에서 해결
  - 이때 메시지는 uniform interface에 의해 자기 서술적이므로 계층의 추가에도 의미가 변질되지 않음

- **Code-on-demand(optional)**
  - 클라이언트가 스크립트 등의 형태로 코드를 다운로드, 실행할 수 있도록 하여 클라이언트의 기능 확장 가능
  - 클라이언트를 단순화, 확장성을 향상시키지만 클라이언트의 기능이 외부에 존재하므로 가시성 저하
  - REST아키텍쳐의 선택적 조건

위의 내용이 REST가 적용되기 위한 아키텍쳐이며 현대 웹에서 사용하는 http통신의 형태와 거의 동일하다.

우리가 RESTful API를 만들기 위해서 집중해야 할 것은 여기서 uniform한 interface를 구성하는 것이다.

### **REST 아키텍쳐의 구성 요소**

Roy Fielding은 uniform interface의 구성 요소를 아래와 같이 정의한다.

- **Component**

  시스템을 구성하는 요소들을 의미한다. component에는 리소스를 제공하는 서버를 의미하는 origin-server, 중계 역할을 하는 proxy와 gateway, 클라이언트를 의미하는 user-agent가 포함된다.

- **Connector**

  시스템을 구성하는 component를 연결하기 위한 요소들을 의미한다. connector에는 서버와 클라이언트도 포함되며, 이외에도 데이터를 임시 저장하는 cache, 리소스의 식별자를 네트워크 주소로 변환하는 resolver, 프로토콜 변환을 위해 사용되는 tunnel 등이 포함된다.

- **Data**

  component가 connector를 통해 주고받는 정보를 의미하며 특성(nature)과 상태(state)를 가지는 요소입니다. data에는 아래의 내용들이 포함된다.

  - resource

    클라이언트가 서버에 요청하는 자원에 대한 표현을 의미한다. 이름을 붙일 수 있는 모든 정보가 자원에 해당되며, 자원에는 빈 값이 매핑될 수도 있다.

    ex. /post/1

  - resource identifier

    특정 자원을 식별하기 위한 경로와 자원의 이름을 모두 포함하는 식별자를 의미한다. 식별자는 웹 상에서 유일하게 존재하며, 식별자만으로 자원을 식별할 수 있어야 하며 url, urn등이 해당된다.

    ex. http://localhost:8080/post/1

  - resource meta-data

    클라이언트가 서버에 전달하는 요청에 대한 상태와 특성에 대한 내용을 의미한다.

    ex. vary, alternates 헤더

  - representation

    서버가 클라이언트로 반환하는 응답의 내용을 의미한다. 응답으로는 html문서, 이미지, 텍스트 등의 데이터가 전달될 수 있다.

    ex. {"title" : "제목", "content" : "본문"}

  - representation meta-data

    서버가 클라이언트로 반환하는 응답에 대한 상태와 특성에 대한 내용을 의미한다.

    ex. content-type, date 헤더

  - control data

    서버가 반환한 응답에 대해 클라이언트가 수행 가능한 행위를 명시한 것을 의미한다. 보통 클라이언트의 캐시 가능 여부와 관련된 내용이 포함된다.

    ex. cache-control, if-modified-since 헤더

## **RESTful API**

결국 RESTful API는 uniform interface의 구성요소를 모두 포함하며 균일한 인터페이스를 제공하는 API이다. 따라서 우리가 RESTful API를 만들기 위해서는 uniform interface 제약조건을 중점적으로 지켜주면 된다.

### **uniform interface를 위한 4가지 조건**

- identification of resource(자원에 대한 식별)
  - 자원을 식별할 수 있는 정보를 의미
  - 자원의 상태는 변화 가능하지만 식별자는 불변
  - 예를들어 /user/1은 식별자로 변하면 안되지만 상태인 이름, 전화번호는 변할 수 있음
- manipulation of resources through representation(표현을 통한 자원의 조작)
  - 자원의 조작에 대한 내용이 표현되어야 하며 이는 http메서드로 주로 표현
- Self-descriptive message(자기 서술적 메시지)
  - 일단 자원을 식별 가능해야하며 body가 있는 경우 어떤 데이터를 담고 있는지 명확히 표현
  - 파라미터에 약어를 너무 많이 쓰면 안되고 conent-type등을 명시
- HATEOAS: hypermedia as the engine of application state(애플리케이션의 상태 관리를 위한 하이퍼미디어 컨트롤)
  - 애플리케이션의 상태 전이는 링크를 통해 발생해야함
  - 서버의 관점에서 보면 게시클 상세보기에 접근한 경우 댓글달기, 게시글 수정하기, 게시글 삭제하기, 좋아요 등의 링크를 포함해서 보내주어야 한다.



## REST API

REST의 특징을 기반으로 서비스 API를 구현한 것으로, 웹 애플리케이션이 제공하는 각각의 데이터를 리소스, 즉 자원으로 간주하고 각각의 자원에 고유한 URI를 할당함으로써 이를 표현하는 API를 정의하기 위한 소프트웨어 아키텍처 스타일이다.

최근 OpenAPI를 제공하는 기관들은 대부분 REST API를 제공한다.

각 요청이 어떤 동작이나 정보를 제공하기 위한 것인지 요청의 모습만으로 파악이 가능한 것이 특징이다.

### 디자인 가이드

- URI는 정보의 자원을 표현해야한다.
- 자원에 대한 행위는 HTTP Method로 표현하며 행위에 대한 정보는 URI에 포함시키지 않는다.

### 규칙

- URI에는 명사를 사용한다.

  리소스를 나타내기 위한 이름은 동사가 아닌 명사가 사용되어야 한다.

  ```
  ex) /getMemberList -> /members/list
  ```

- `/`는 계층관계를 나타내기 위해 사용한다.

  리소스의 계층관계을 표현할 때에는 `/`로 표현한다.

  ```
  ex) weathers/rain
  ```

- URI의 마지막 문자로 `/`를 사용하지 않는다.

  URI의 모든 문자는 리소스의 식별자로 사용되기 때문에 마지막에 `/`가 있는 URI와 없는 URI는 의미상으로는 같은 리소스를 나타낼 수 있지만 실제로는 다른 리소스를 나타내야한다.

- 밑줄`_`을 사용하지 않으며 하이픈`-`을 통해 가독성을 높인다.

  ```
  ex) /members/authentified_user -> /members/authentified-user
  ```

- 대문자를 사용하지 않는다.

  CamelCase 대신 `-`을 통해 가독성을 높여준다.

  ```
  ex) /members/authentifiedUser -> /members/authentified-user
  ```

- 파일의 확장자는 URI에 포함하지 않는다.

  ```
  ex) http://kymin.com/members/list.html (X)
  ```

- HTTP 응답 상태 코드를 사용한다.

  클라이언트에게 요청에 대한 피드백을 제공해야한다.



### **HATEOAS (Hypermedia As The Engine Of Application State)**

HATEOAS는 서버가 클라이언트 요청에 대해 응답을 할 때, 추가적인 정보를 제공하는 링크를 포함하는 것을 의미한다.

HATEOAS는 서버와 클라이언트가 동적인 상호작용을 할 수 있도록 링크를 제공하며 링크를 통해 데이터와 관련된 작업에 대한 정보를 응답과 함께 제공받는다.

### HATEOAS 적용

아래와 같이 `build.gradle`을 수정하여 의존성 추가

```
dependencies {
	...
	implementation 'org.springframework.boot:spring-boot-starter-hateoas'
	...
}
```

클라이언트의 요청에 대한 응답에 링크를 추가하여 반환

```java
@RestController
@RequestMapping("/api")
public class MemberContorller {
    private MemberService memberService;

    public MemberContorller(MemberService memberService) {
        this.memberService = memberService;
    }

    @PostMapping(value = "/my-info")
    @ResponseBody
    public EntityModel<MemberEntity> showInfo(@RequestBody MemberDto dto) {
        EntityModel<MemberEntity> response = EntityModel.of(memberService.signUp(dto),
                linkTo(methodOn(this.getClass()).signUp(dto)).withSelfRel(),	// 현재 자원의 URL을 링크로 추가
                linkTo(methodOn(this.getClass()).logout()).withRel("logout"));	// 로그아웃을 위한 URL을 링크로 추가
        return response;
    }
  	...
}
```

```
// 기존 응답
{
	"userId" : "kymin@email.com",
	"username" : "kymin"
}
```

```
// HATEOAS 적용
{
	"userId" : "kymin@email.com",
	"username" : "kymin"
	"_links" : {
		"self" : {
			"href" : "http://localhost:8080/api/my-info"	// 현재 자원의 URL
		},
		"logout" : {
			"href" : "http://localhost:8080/api/logout"	// 로그아웃을 위한 URL
		}
	}
}
```

-----

##### 참고자료 :

[https://velog.io/@somday](https://velog.io/@somday/RESTful-API-이란)

[https://dev-coco.tistory.com/97](https://dev-coco.tistory.com/97)

[https://www.boostcourse.org/](https://www.boostcourse.org/web326/lecture/58986?isDesc=false)

[https://m.blog.naver.com/aservmz](https://m.blog.naver.com/aservmz/222234406469)

[https://haesummy.tistory.com](https://haesummy.tistory.com/entry/REST-API%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC-feat-%EB%A1%9C%EC%9D%B4-%ED%95%84%EB%94%A9-%EB%85%BC%EB%AC%B8)

[https://joomn11.tistory.com](https://joomn11.tistory.com/26)