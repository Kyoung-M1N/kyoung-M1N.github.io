---
title: RESTful API
author: kymin
date: 2024-01-05 16:38
categories: [Etc, API]
tags: [api]

---

## REST(Representational State Transfer)?

### API(Application programming Interface)?

API는 서로 다른 소프트웨어들이 통신하기 위한 방법을 정의하는 규칙을 말한다.

### REST?

REST는 API의 작동 방식에 대한 조건을 부과하는 소프트웨어 아키텍처로 HTTP 프로토콜을 통해 API를 설계하기 위한 아키텍쳐 스타일이다.

자원을 이름으로 구분하여 해당 자원에 대한 정보를 주고 받으며, URI를 통해 자원을 명시하고 HTTP Method(POST, GET, PUT, DELETE)를 통해 해당 자원에 대한 CRUD가 이루어진다.

> URI와 URL
>
> URI(Uniform Resource Identifier)
>
> 네트워크상에 존재하는 자원(Resource)을 식별하는 고유 문자열을 의미한다.
>
> `ex) studylog.kym1n.com`
>
> URL(Uniform Resource Location)
>
> 네트워크 상에 존재하는 자원의 위치를 나타내기 위한 규약으로 식별자와 위치를 동시에 보여준다.
>
> 즉, 어떻게 위치를 찾고 도달할 수 있는지까지 포함되어야 하기 때문에 URL은 프로토콜 + 이름(또는 번호)의 형태여야 한다.
>
> `ex) https://studylog.kym1n.com`
>
> URI가 더 포괄적인 개념이며 URL은 이 안에 포함된다.

### REST의 특징

-  Server-Client구조

  자원을 가지고 있는 쪽이 서버, 자원을 요청하는 쪽이 클라이언트가 되는 구조를 가진다.

  역할의 구분으로 상호간의 의존성이 줄어든다.

- Uniform(인터페이스 일관성)

  URI로 지정한 Resource에 대한 요청을 통일되고, 한정적으로 수행하는 아키텍처 스타일을 의미한다.

  HTTP표준 프로토콜을 따르는 모든 플랫폼에서 사용이 가능하므로 특정 언어나 기술에 종속되지 않는다.

- Stateless(무상태성)

  HTTP 프로토콜이 Stateless이므로 REST역시 무상태성이다.

  즉, 서버가 클라이언트의 세션과 쿠키 등의 정보를 신경쓰지 않아도 되며 각각의 요청이 별개로 인식되어 처리된다.

- Cacheable(캐시 가능)

  웹 표준 HTTP 프로토콜을 따르기 때문에 웹에서 사용하는 기존 인프라를 그대로 사용할 수 있으며 따라서 캐싱 기능을 적용할 수 있다.

- 계층형 구조

  클라이언트는 REST API 서버만을 호출하지만 서버는 다중 계층으로 구성될 수 있으며, 클라이언트는 서버와 직접 통신하는지 중간 서버와 통신하는지 알 수 없다.

- Self-Descriptiveness(자체표현구조)

  요청메시지만 보고도 내용을 쉽게 이해할 수 있다.

> 캐시(cache)
>
> 캐시는 데이터나 값을 미리 복사해놓는 임시 장소를 말하며 캐시에 접근하는 시간보다 원래 데이터에 접근하는 시간이 오래 걸리거나 값을 다시 계산하는 시간을 절약하기 위해 사용한다.

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

## RESTful API

RESTful API는 REST의 설계 규칙을 잘 지켜서 설계된 API를 말하며 이해하기 쉽고 사용하기 쉬운 REST API를 만드는 것을 목적으로 한다.

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

https://dev-coco.tistory.com/97

[https://www.boostcourse.org/](https://www.boostcourse.org/web326/lecture/58986?isDesc=false)

https://joomn11.tistory.com/26