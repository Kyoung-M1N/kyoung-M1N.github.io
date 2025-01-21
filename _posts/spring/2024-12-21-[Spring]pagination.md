---
title: 페이징 적용하기
author: kymin
date: 2024-12-21 18:23
categories: [JPA]
tags: [spring, web, java, jpa]


---

## **페이지네이션(Pagination)**

페이지네이션은 여러 개의 데이터를 일정한 크기로 나누어 제공하는 것을 의미한다.

사용자의 요청에 따라 데이터를 반환할 때 데이터의 양이 많은 경우에는 서버의 부하가 커지고 전달 시간이 오래걸리기 때문에 여러 개의 데이터를 나누어 전송한다.

클라이언트의 관점에서는 일반적인 페이지네이션과 무한 스크롤의 두 가지 방식으로 페이지를 나누어 처리할 수 있다.

Spring에서 JPA만을 이용하여 페이지네이션을 구현하기 위해서는 JPQL을 이용하여 직접 쿼리문을 작성한 뒤에 MySQL의 `offset`에 해당하는 `setFirstResult()`와 `limit`에 해당하는 `setMaxResult()`등을 이용하여 구현해야 한다.

```java
List<Data> datas = entityManager.createQuery("select d from Data d", Data.class)
				.setFirstResult(0)	// 0번째 데이터부터 조회
  			.setMaxResult(10)		// 10개씩 조회
  			.getResultList();
```

JPA만을 가지고 페이지네이션을 구현하는 과정에서 전체 데이터의 수를 가져와 계산하거나 몇번째 페이지인지 계산하는 등의 과정이나 범위를 벗어난 요청에 대해 예외 처리를 직접 구현해야 한다.

Spring Data JPA는 위의 페이지네이션을 추상화하여 제공하기 때문에 이를 이용하여 페이지네이션을 편리하게 구현할 수 있다.



### **Pageable과 PageRequest**

`Pageable`은 페이지네이션에 대한 정보를 담기 위한 인터페이스로 Spring Data JPA의 Repository의 파라미터로 전달된다.

Repository는 `Pageable`이 전달한 페이지네이션 정보를 통해 데이터베이스를 조회하고 엔티티 컬렉션을 페이징, 정렬하여 반환한다.

`PageRequest`는 `Pageable`인터페이스의 구현체로 Repository의 파라미터로 전달할 `Pageable`을 생성하기 위해 사용된다.

```java
Pageable pageable = PageRequest.of(pageNumber, pageSize);
```

기본적인 페이지네이션 정보를 담는 것 뿐만 아니라  `Sort`클래스나 `Direction`enum 클래스를 이용하여 정렬을 설정할 수 있다.

```java
PageRequest.of(0, 10, Sort.by("createdAt").descending());
PageRequest.of(0, 10, Sort.by(Direction.DESC, "createdAt"));
PageRequest.of(0, 10, Sort.by(Order.desc("createdAt")));
PageRequest.of(0, 10, Direction.DESC, "createdAt");
// createdAt은 정렬 기준이 되는 entity의 필드 이름을 그대로 작성해주면 된다.
```

Spring Data JPA의 Repository에 `Pageable`을 전달하여 데이터를 조회하면 Repository는 `Slice` 또는 `Page`타입으로 데이터를 반환하게 된다.



## **페이지네이션 결과 반환**

### **Slice**

`Slice`는 `pageable`의 정보에 따라 Repository가 데이터 베이스를 조회한 결과를 반환하는 인터페이스 형태이다.

Repository가 데이터 베이스를 조회한 결과로 생긴 `Slice`는 자기 자신이 가지고 있는 엔티티와 `pageable`의 정보와 이전, 다음 `Slice`의 `pageable`정보 등을 제공한다.

```java
public interface Slice<T> extends Streamable<T> {

	int getNumber(); // 현재 Slice 번호(인덱스) 반환

	int getSize(); // 현재 Slice 크기(엔티티 갯수) 반환

	int getNumberOfElements(); // 현재 Slice가 가지고 있는 엔티티의 갯수 반환(마지막 Slice는 Slice의 크기와 가지고 있는 엔티티의 수가 다를 수 있음)

	List<T> getContent(); // 엔티티 목록을 List로 반환

	boolean hasContent(); // 엔티티 목록을 가지고 있는지 여부를 반환

	Sort getSort(); // Slice의 Sort 객체(정렬 정보) 반환

	boolean isFirst(); // 현재 Slice가 첫번째인지 여부 반환

	boolean isLast(); // 현재 Slice가 첫번째인지 여부 반환

	boolean hasNext(); // 다음 Slice의 존재 유무 반환

	boolean hasPrevious(); // 이전 Slice의 존재 유무 반환

  // 현재 Slice에 대한 Pageable을 생성해서 반환
	default Pageable getPageable() {
		return PageRequest.of(getNumber(), getSize(), getSort());
	}

	Pageable nextPageable(); // 다음 Slice의 Pageable 반환

	Pageable previousPageable(); // 이전 Slice의 Pageable 반환
  
	// Slice 내의 엔티티를 다른 객체로 매핑
	<U> Slice<U> map(Function<? super T, ? extends U> converter);

  // 다음 Slice가 있다면 다음 Slice의 Pageable, 현재 Slice가 마지막이면 현재 Pageable 반환
	default Pageable nextOrLastPageable() {
		return hasNext() ? nextPageable() : getPageable();
	}
  
  // 이전 Slice가 있다면 이전 Slice의 Pageable, 현재 Slice가 첫번째면 현재 Pageable 반환
	default Pageable previousOrFirstPageable() {
		return hasPrevious() ? previousPageable() : getPageable();
	}
}
```

Repository에서 `Slice`를 반환하는 함수를 선언하는 방법은 아래와 같다.

```java
public interface DataRepository extends JpaRepository<Data, Long> {
  Slice<Data> findSliceByName(String name, Pageable pageable);
}
```



### **Page**

`Page`는 `pageable`의 정보에 따라 Repository가 데이터 베이스를 조회한 결과를 반환하는 인터페이스 형태로 `Slice`를 상속 받아 선언된 인터페이스이다.

`Slice`를 상속 받았기 때문에 `Slice`가 제공하는 기능을 모두 사용할 수 있고, 추가적으로 현재 페이지의 정보가 아닌 조회 결과 전체에 해당하는 정보를 제공하는 기능이 존재한다.

```java
public interface Page<T> extends Slice<T> {
  // 빈 페이지 반환
	static <T> Page<T> empty() {
		return empty(Pageable.unpaged());
	}
  
  // Pageable을 받아 빈 페이지 반환
	static <T> Page<T> empty(Pageable pageable) {
		return new PageImpl<>(Collections.emptyList(), pageable, 0);
	}

	int getTotalPages(); // 조회 결과 만들어지는 전체 페이지 개수 반환

	long getTotalElements(); // 조회 결과에 해당하는 전체 엔티티 개수 반환
  
  // Page 내의 엔티티를 다른 객체로 매핑
	<U> Page<U> map(Function<? super T, ? extends U> converter);

}
```

Repository에서 `Page`를 반환하는 함수를 선언하는 방법은 아래와 같다.

```java
public interface DataRepository extends JpaRepository<Data, Long> {
  Page<Data> findPageByName(String name, Pagable pagable);
}
```



### **Slice vs Page**

`Slice`와 `Page`의 가장 큰 차이는 Repository를 이용하여 조회한 결과에 대해 전체 데이터의 수에 대한 정보의 제공 여부이다.

`Page`는 조회 결과에 대한 전체 엔티티의 수나 전체 페이지의 수를 반환하므로 Spring Data JPA가 데이터 베이스에 `select`쿼리 뿐만 아니라 `select count`쿼리를 추가적으로 전달하여 실행하게 된다.

반면에 `Slice`는 `select`만 전달하고 이때 limit을 페이지의 크기보다 1만큼 더 크게 보내서 쿼리문의 실행 결과를 페이지의 크기와 비교하여 다음 `Slice`의 존재 여부를 파악하게 된다.

따라서 쿼리문 전송이 덜한 `Slice`를 사용하는 것이 조금이나마 더 좋은 성능을 가져올 수 있으며, 결론적으로 페이징을 구현하는 과정에서 전체 데이터의 수나 페이지의 수가 필요한 경우라면 `Page`를 사용하고 전체 데이터에 대한 정보가 필요없다면 `Slice`를 사용하는 것이 바람직한 방법인 것 같다.

> 클라이언트에서 일반적인 게시판과 같이 페이지를 나누어서 사용자에게 정보를 제공한다면 전체 데이터나 페이지에 대한 정보가 필요하기 때문에 `Page`를 사용하고, 무한 스크롤과 같이 전제 데이터나 페이지에 대한 정보가 필요 없다면 `Slice`를 사용한다.
>
> `Slice`와 `Page`는 둘 다 엔티티를 원하는 객체로 변환하는 `map()`을 제공하지만 `Slice`와 `Page`에 담아 그대로 반환하게 되면 해당 객체의 `pageable` 까지 전달되기 때문에 클라이언트에게 불필요한 정보를 반환하지 않기 위해 적절한 응답 DTO를 설정하는 것이 중요하다.



-----

##### 참고자료 :

[https://velog.io/@dani0817](https://velog.io/@dani0817/Spring-Boot-%ED%8E%98%EC%9D%B4%EC%A7%95Paging-%EC%A0%81%EC%9A%A9)

[https://hudi.blog](https://hudi.blog/spring-data-jpa-pagination/)