---
title: 외부 API
author: kymin
date: 2024-01-12 14:51
categories: [Spring]
tags: [spring, web, java, api]

---

## 외부 API 불러오기

서버는 경우에 따라서 데이터베이스 뿐만 아니라 외부 서버와 통신을 해야하는 경우가 존재한다.

외부에서 제공하는 데이터를 가져오거나 외부에서 제공하는 서비스를 사용해야 하는 경우가 대표적이다.

스프링 프레임워크에서 외부 API를 호출하는 방법은 대표적으로 RestTemplate과 WebClient가 있다.

RestTemplate은 비동기처리와 non-blocking이 불가능한 반면에 WebClient는 코드 작성을 통해 자유롭게 사용 가능하며 메소드 체이닝만으로도 WebClient를 생성하거나 요청을 보낼 수 있다.

스프링 5.0에서 WebClient가 나오면서 공식 문서는  WebClient를 추천하고 있다.



## WebClient 사용

### 의존성 추가

아래와 같이 스프링 프로젝트에 포함된 `build.gradle`의 `dependencies`에 webflux의 라이브러리에 대한 의존성을 추가한다.

```java
dependencies {
	...
	implementation 'org.springframework.boot:spring-boot-starter-webflux'
	...
}
```

### WebClient 인스턴스 생성

Config파일을 생성하고 HTTP method를 호출할 WebClient를 bean으로 등록한다.

```java
@Configuration
public class WebClientConfig {
    @Bean
    public WebClient webClient(){
    return WebClient.builder()
            .baseUrl("https://dapi.kakao.com/v2/local/search/category.json?")
      .defaultHeader("Authorization", "api-key")
      .build();
  }
}
```

### API 호출

필요한 기능(HTTP method)에 따라 api를 호출하는 기능을 구현한다.

```java
@Service
public class RestaurantService {
    WebClient webClient;

    public RestaurantService(WebClient webClient) {
        this.webClient = webClient;
    }	//bean으로 등록된 WebClientConfig의 webClient의 의존성 주입

    public Map<String, Object> getRestaurants(RequestDto dto) {
        Map<String, Object> response = webClient.get()
                .uri(uriBuilder -> uriBuilder
                        .queryParam("x", dto.getLongitude())
                        .queryParam("y", dto.getLatitude())
                        .queryParam("radius", dto.getRadius())
                        .build())
                .retrieve()
                .bodyToMono(Map.class)
                .block();
        return response;
    }	// get을 통한 api호출
  
  	public Map<String, Object> postRestaurants(RequestDto dto) {
    	Map<String, Object> bodyMap = new HashMap<>();
      bodyMap.put("x", dto.getLongitude());
      bodyMap.put("y", dto.getLatitude());
    	bodyMap.put("radius", dto.getRadius());
    	// request body 생성 후 값 설정
    	Map<String, Object> response =
              webClient
                      .post()
                      .bodyValue(bodyMap)
                      .retrieve()
                      .bodyToMono(Map.class)
                      .block();
      return response;
  } // post를 이용한 api 호출
}
```

GET의 경우 `queryParam()`을 통해 URL에 쿼리로 파라미터를 추가할 수 있고, POST의 경우 request body를 생성하여 `bodyMap.put()`을 통해 파라미터를 추가할 수 있다.

이후 `bodyMap.put()`를 통해 response entity에 대한 디코딩을 진행하고 `bodyToMono()`를 통해 원하는 형태로 변환한다.

마지막으로 webClient는 기본적으로 non-blocking방식을 지원하기 때문에 `block()`을 통해 요청에 대한 응답이 전달될 때까지 다른 로직을 실행하지 않도록 한다.

---

##### 참고자료 :

[https://velog.io/@da_na](https://velog.io/@da_na/WebClient-WebClient-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C-%EC%99%B8%EB%B6%80-API-%ED%98%B8%EC%B6%9C%ED%95%98%EA%B8%B0)

[https://jforj.tistory.com/319](https://jforj.tistory.com/319)

[https://gngsn.tistory.com/154](https://gngsn.tistory.com/154)

[https://myvelop.tistory.com/m/217](https://myvelop.tistory.com/m/217)