---
title: 스프링에서 REST API와 CRUD 구현
author: kymin
date: 2024-01-13 19:31
categories: [Spring]
tags: [spring, web, java]
---


## REST API

REST(Representational State Transfer) API는 REST 아키텍쳐 스타일을 준수하는 API를 나타내는 말로 REST 아키텍쳐를 적절하게 준수한 API를 RESTful API라고도 부른다.

REST 아키텍쳐는 자원(resource)를 이름으로 구분하여 해당 자원의 상태를 주고받는 것을 의미한다. 즉, HTTP URI(Uniform Resource Identifier)를 통해 자원을 명시하고 HTTP Method(POST, GET, PUT, DELETE, PATCH 등)를 통해 해당 자원(URI)에 대한 CRUD Operation을 적용하는 것을 말한다.

## CRUD

Create, Read, Update, Delete의 약자로 Create는 데이터 생성, Read는 데이터 조회, Update는 데이터 수정, Delete는 데이터 삭제를 의미한다.

> **RESTful API를 위한 CRUD와 HTTP Method의 관계**
>
> | CRUD   | HTTP Method    |
> | ------ | -------------- |
> | Create | `POST`         |
> | Read   | `GET`          |
> | Update | `PUT`, `PATCH` |
> | Delete | `DELETE`       |
>
> `PUT`은 전체 수정, `PATCH`는 부분 수정을 위해 사용하는 Method이다.

## Controller에서의 구현

`@Controller`어노테이션을 사용하면 Controller 객체는 ViewName을 반환하게 되고 DispatcherServlet이 ViewResolver를 통해 ViewName에 해당하는 View를 보여준다.

하지만 REST API를 구현하기 위해서는 Controller 객체가 ViewName이 아닌 JSON형태의 데이터를 반환해야 한다.

이를 위해 Controller 객체에 `@ResponseBody`어노테이션을 추가해주고 데이터에 대한 객체를 반환해주면 객체가 JSON 형태로 반환된다.

```java
@Controller
@RequiredArgsConstructor
public class TestController {
  	private final TestService testService;
  	// viewName을 반환하는 기존 함수
    @GetMapping("/info")
    public String testViewName(Model model, @RequestParam String info) {
        Test testInfo = testService.findInfo(info);
        model.addAttribute("info", testInfo);
        return "/info";
    }
  	//JSON형태의 데이터를 반환
  	@GetMapping("/info")
	  @ResponseBody
  	public ResponseEntity<Test> testJson(@RequestParam String info) {
      	 return ResponseEntity.ok(testService.findInfo(info));
    }
}
```

보통 Controller로 객체를 반환할 때에는 `ResponseEntity`로 감싸서 반환하게 된다.

> ResponseEntity?
>
> ResponseEntity는 HTTP 요청에 대한 응답데이터 뿐만 아니라 HttpHeader와 HttpBody를 포함하는 클래스이다.
>
> ResponseEntity의 생성자에는 상태 코드(status), 응답 헤더(header), 응답 데이터(body)가 있고 필요에 따라 원하는 응답을 출력할 수 있다.

`@Controller`어노테이션과 `@ResponseBody`어노테이션을 통합한 `@RestController`를 사용하면 아래와 같이 더 간결하게 코드를 작성할 수 있다.

```java
@RestController
@RequiredArgsConstructor
public class TestController {
  	private final TestService testService;
  
  	@GetMapping("/info")
  	public ResponseEntity<Test> testJson(@RequestParam String info) {
       	return ResponseEntity.ok(testService.findInfo(info));
    }
}
```

### POST

주로 새로운 리소스를 생성하는 데에 사용되며 requestBody에 key-value형식으로 데이터를 입력하여 전달한다.

GET과 달리 URI에 데이터가 노출되지 않기 때문에 보안상의 이점이 있으며 

```java
@RestController
@RequiredArgsConstructor
public class TestController {
    private final TestService testService;
  	
    @PostMapping("/info")
    public ResponseEntity<Test> testJson(@RequestBody RequestDto info) {
      	 return ResponseEntity.ok(testService.saveInfo(info));
    }
}
```



### GET

리소스를 조회할 때에 사용되며 URI에 query parameter 또는 path variable 방식으로 리소스를 식별하기 위한 데이터를 전달한다.

```java
@RestController
@RequiredArgsConstructor
public class TestController {
   	private final TestService testService;
  	// query parameter방식 -> GET | http://kymin.com/info?id=1
   	@GetMapping(path = "/info", params = id)
  	public ResponseEntity<Test> testQueryParams(@RequestParam String id) {
       	return ResponseEntity.ok(testService.findInfo(id));
    }
    // path variable -> GET | http://kymin.com/info/1
   	@GetMapping("/info/{id}")
  	public ResponseEntity<Test> testPathVariable(@PathVariable String id) {
       	return ResponseEntity.ok(testService.findInfo(id));
    }
}
```

### 

### PUT

기존에 존재하는 리소스를 수정할 때 사용하며 요청을 통해 전달받은 데이터를 기존의 데이터에 덮어쓰는 개념이다.

### PATCH

기존에 존재하는 리소스를 부분적으로 수정할 때 사용하며 

### DELETE

리소스를 삭제하는 데에 사용되며 

```java
@RestController
@RequiredArgsConstructor
public class TestController {
   	private final TestService testService;
  	@DeleteMapping("/info/{id}")
  	void deletePost(@PathVariable int id) {
      	service.deleteInfo(id);
    }
}
```

### 

-----

##### 참고자료 :

[https://junvelee.tistory.com](https://junvelee.tistory.com/107)

[https://velog.io/@leesomyoung](https://velog.io/@leesomyoung/Spring-Boot-RestController%EC%99%80-Controller%EC%9D%98-%ED%8A%B9%EC%A7%95%EA%B3%BC-%EC%B0%A8%EC%9D%B4%EC%A0%90)

[https://memostack.tistory.com](https://memostack.tistory.com/244)

[https://devlog-wjdrbs96.tistory.com](https://devlog-wjdrbs96.tistory.com/182)

[https://youwjune.tistory.com](https://youwjune.tistory.com/42)

[https://inpa.tistory.com/entry](https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-HTTP-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%A2%85%EB%A5%98-%ED%86%B5%EC%8B%A0-%EA%B3%BC%EC%A0%95-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)