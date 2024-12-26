---
title: 구간 합 알고리즘
author: kymin
date: 2024-11-13 14:29
categories: [Algorithm]
tags: [algorithm, java]
---
## 누적 합(sum)과 구간 합(Prefix sum)

### 누적 합

누적 합은 처음부터 정해진 구간까지 더한 값을 의미하며 누적 합을 이용하여 합 배열을 생성할 수 있다.

합 배열은 주어진 배열에서 0부터 특정 인덱스까지의 데이터가 더해진 값(누적 합)이 배열에 저장된 형태이다.

```java
배열 : [1,2,3,4,5,6,7]
합 배열 : [1,3,6,10,15,21,28]
```

### 구간 합

구간 합은 사간 복잡도를 줄이기 위해 합 배열을 이용하여 주어진 배열을 전처리하는 과정이다.

합 배열의 값을 이용하여 아래의 식으로 특정 구간의 값을 구할 수 있다.

```java
int[] numbers = {1,2,3,4,5,6,7};
int[] sumArray = {1,3,6,10,15,21,28};

// 인덱스 i부터 j까지의 구간 합
int prefixSum = sumArray[j] - sumArray[i - 1];
```

이 때 `arrayIndexOutOfBound`가 발생하지 않도록 0번 인덱스의 값을 0으로 두고 주어진 배열보다 인덱스를 하나씩 증가시켜줘야 한다.

2차원 배열에서도 마찬가지로 합 배열을 이용하여 구간합을 구할 수 있다.

```java
int[][] numbers= {{1,2,3,4},{2,3,4,5},{3,4,5,6},{4,5,6,7}}

// 2차원 배열의 합 배열 구하기 : s(x,y) = s(x,y-1) + s(x-1,y) - s(x-1,y-1) + a(x,y)
int[][] sumArray = {{1,3,6,10},{3,8,15,24},{6,15,27,42},{10,24,42,64}}

// (x1,y1)부터 (x2,y2)까지의 구간 합
int result = sumArray[x2][y2] - sumArray[x2][y1 - 1] - sumArray[x1 - 1][y2] + sumArray[x1 - 1][y1 - 1];
```

마탄가지로 `arrayIndexOutOfBound`가 발생하지 않도록 행과 열의 0번 인덱스의 값을 0으로 두고 주어진 배열보다 인덱스를 하나씩 증가시켜줘야 한다.



-----------------------

##### 참고자료 : 

[LibreWiki](https://librewiki.net/wiki/%EC%8B%9C%EB%A6%AC%EC%A6%88:%EC%88%98%ED%95%99%EC%9D%B8%EB%93%AF_%EA%B3%BC%ED%95%99%EC%95%84%EB%8B%8C_%EA%B3%B5%ED%95%99%EA%B0%99%EC%9D%80_%EC%BB%B4%ED%93%A8%ED%84%B0%EA%B3%BC%ED%95%99/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98_%EA%B8%B0%EC%B4%88)

[https://www.youtube.com/playlist](https://www.youtube.com/playlist?list=PLG7te9eYUi7tAQygBknaTciy8wzLCe-Ll)