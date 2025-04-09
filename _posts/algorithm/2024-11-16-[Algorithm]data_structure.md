---
title: 자료 구조
author: kymin
date: 2024-11-16 15:23
categories: [Algorithm]
tags: [algorithm, java]
---
## **배열(Array)과 리스트(List)**

### **배열**

메모리의 연속된 공간에 값이 채워져있는 형태의 자료구조를 의미하며 각 원소들은 인덱스를 이용하여 참조할 수 있다.

메모리의 연속된 공간에 값이 존재하기 때문에 특정 인덱스의 값을 삭제하거나 삽입할 때에 나머지 값들이 메모리 상에서 이동해야 한다.

배열의 크기는 선언 시에 결정하며 선언이 끝나면 해당 배열의 크기를 조정할 수 없다.

> 데이터의 크기가 변하지 않는 경우, 데이터에 자주 접근해야할 경우, 데이터가 추가, 삭제되는 경우가 거의 없는 경우에 사용하면 좋다.

```java
public class array {
  // 크기가 10인 int형 배열
  int[] intArray = new int[10];
  // 크기가 10*5인 String형 2차원 배열
  String[][] StringArray = new String[10][5];
}
```



### **리스트**

값과 포인터(메모리 상의 주소)를 하나로 묶은 노드를 포인터를 이용하여 연결한 형태의 자료구조를 의미하며 배열과 달리 메모리 상에 연속적으로 값을 저장하지 않는다.

메모리의 연속된 공간에 값을 저장하지 않으므로 인덱스를 통해 각 원소를 참조할 수 없으며 첫번째 원소인 head에서 부터 순차적으로 접근해야 한다.

포인터를 이용하여 연결을 나타내기 때문에 삽입하고 삭제할 때에 값의 이동이 일어나지 않는다.

선언 시에 크기를 별도로 지정하지 않으며 선언 후에도 크기가 조정될 수 있다.

저장할 데이터 외에도 해당 값이나 다음, 이전 노드 등에 대한 메모리 주소를 함께 저장해야 한다.

> 데이터의 크기가 변해야 하는 경우, 데이터의 삽입과 삭제가 자주 일어나야 하는 경우에 사용하면 좋다.

```java
// 직접 구현
public class Node {
  String name;
  int age;
  Node previous;
  Node next;
}

// 자바에서는 기본적으로 제공
List<Object> list = new ArrayList<>();
```



## **스택(Stack)과 큐(Queue)**

### **스택**

배열이 발전된 형태의 자료구조로 삽입과 삭제 연산이 후입선출(LIFO : Last In First Out)로 발생하는 배열이다.

`push`를 통해 값을 삽입하고 `pop`을 통해 값을 삭제하며, 가장 마지막에 삽입된 데이터를 가리키는 `top`이라는 값이 존재한다.

`top`이 현재 가리키는 위치에서만 `push`나 `pop`이 발생한다.

`peek`를 통해 스택의 구조상에서 가장 위에 있는 데이터를 확인할 수 있다. 

> 후입 선출의 개념이 재귀 함수 알고리즘의 원리와 거의 동일하기 때문에 깊이 우선 탐색(DFS : Depth First Searsh)이나 백트래킹 등에 효과적인 자료구조이다.

```java
public class Stack {
  int[] intArray = new int[10];
  int top = 0;
  
  public void push(int data) {
    this.intArray[this.top] = data;
    this.top++;
  }
  
  // pop은 값을 삭제하면서 삭제된 값을 반환한다.
  public int pop() {
    this.top--;
    int popped = this.intArray[this.top];
    this.intArray[this.top] = 0;
    return popped;
  }
  
  public int peek() {
    return this.intArray[this.top - 1];
  }
}
```



### **큐**

배열이 발전된 형태의 자료구조로 삽입과 삭제 연산이 선입선출(FIFO : First In First Out)로 발생하는 배열이다.

`add`를 통해 값을 삽입하고 `poll`을 통해 값을 삭제하며, 삽입은 `rear`에서, 삭제는 `front`에서 일어난다.

`peek`를 통해 큐의 구조상에서 가장 뒤에 있는 데이터를 확인할 수 있다. 

> 너비 우선 탐색(BFS : Breadth First Search)에서 자주 사용되는 자료구조이다.
>
> 데이터가 추가된 순서와 상관 없이 우선 순위에 따라 값이 먼저 반환되는 우선순위 큐는 트리의 일종인 힙(heap)을 이용하여 구현한다.

```java
public class Queue {
  int[] intArray = new int[10];
  int front = 0;
  int rear = 0;
  
  void add(int data) {
    this.intArray[this.rear] = data;
    rear++;
  }

  int poll() {
    int polled = this.intArray[this.front];
    this.intArray[this.front] = 0;
    front++;
    return polled;
  }

  int peek() {
    return this.intArray[this.rear - 1];
  }
}
```



## **그래프(Graph)와 트리(Tree)**

### **그래프**

데이터를 표현하는 단위인 `node`와 `node`를 연결하는 edge를 이용하여 여러 개의 node가 edge로 연결된 형태의 자료구조를 의미하며 데이터나 객체의 연결성에 중점을 둔 자료구조이다.

그래프의 node를 연결하는 edge는 방향성을 가질 수 있으며 방향이 있는 edge에서는 방향에 따라서만 node에 접근할 수 있다.

그래프를 구현하는 방법은 에지 리스트, 인접 행렬, 인접 리스트 세 가지가 있다.

- ### **에지 리스트(edge list)**

  그래프 자료구조를 edge를 중심으로 표현하는 방식으로 배열의 원소에 startNode와 endNode를 저장하여 edge를 표현하고 edge에는 가중치(cost)가 있을 수도 있다.

  2차원 배열상에서 하나의 열이 하나의 edge를 표현한다.

  ```java
  public class EdgeList {
    // edge의 가중치가 없는 그래프, 맨 처음 행이 startNode, 다음 행이 endNode
    // 방향이 없는 그래프에서는 [1,2]와 [2,1]이 같은 표현이지만 방향이 있는 그래프에서 양방향을 표현하기 위해서는 둘 다 필요
    int[][] edgeList = new int[2][10];
    
    // edge의 가중치가 있는 그래프, 맨 처음 행이 startNode, 다음 행이 endNode, 마지막 행이 cost
    int[][] edgeListWithCost = new int[3][10];
  }
  ```

  에지 리스트는 특정 노드와 관련이 있는 에지를 탐색하는 것이 어렵기 때문에(배열 전체에 대한 순회 과정이 필요) 노드 중심 알고리즘에는 잘 사용하지 않는다.

  Bellman-ford 알고리즘이나 kruskal 알고리즘과 같이 edge의 가중치가 중심이 되는 알고리즘에서 사용된다.

- ### **인접 행렬(adjacency matrix)**

  2차원 행렬의 인덱스를 노드와 대응시키고 행렬의 원소에 가중치를 저장하는 방식으로 가중치가 존재하지 않을 경우에는 행렬의 원소를 1로 설정하여 연결 여부를 표현한다.

  ```java
  public class AdjacencyMatrix {
    // 행과 열의 크기는 노드의 수와 동일 -> 정방형
    // 행(세로 인덱스)이 출발노드, 열(가로 인덱스)이 도착노드에 해당
    // 행렬의 원소는 cost를 나타내며 cost가 없는 그래프의 경우 모든 원소를 1로 설정하여 연결 여부를 표현(배열을 초기화하면 원소의 기본 값이 0이므로 0으로 표현할 수 없다.)
    int[][] edgeListWithCost = new int[6][6];
  }
  ```

  특정node가 어느 node와 연결되어 있는지 파악하기 용이하며 특정 edge의 cost에 접근하는 것도 용이하다.

  특정 startNode나 endNode에 해당하는 edge정보를 알아내기 위해 n번의 탐색과정이 필요하므로, 노드의 수에 비해 edge의 수가 적을 경우 불필요한 공간을 많이 사용하게 되며, 노드 갯수가 너무 많은 경우에는 2차원 행렬 자체를 선언할 수 없다.(node의 수가 3만개를 넘어설 경우 `java.lang.OutOfMemoryError: Java heap space`가 발생)

- ### **인접 리스트(adjacency list)**

  `ArrayList`의 배열을 이용하여 그래프를 구현하는 방식으로 그래프를 이용한 알고리즘에서 가장 많이 사용되는 구현 방법이다.

  배열의 크기는 노드의 수와 동일하며 배열의 원소인 `ArrayList`의 원소는 현재 인덱스의 노드를 startNode로 하는 endNode를 나타낸다.

  가중치가 존재하는 경우 Node의 필드에 cost를 추가로 만들어야 하며 해당 노드와 startNode를 잇는 edge의 cost를 나타내게 된다.

  ```java
  public class adjaceencyList {
    ArrayList<Integer>[] adjacencyList = new ArrayList[5];
    
    ArrayList<Node>[] adjacencyListWithCost = new ArrayList[5];
  }
  
  class Node {
    String address;
    int cost;
  }
  ```

  구현 과정은 다소 복잡하지만 시작노드를 통해 연결된 edge에 빠르게 접근할 수 있고, edge의 수에 따라 가변적으로 `ArrayList`의 크기가 변하므로 공간 사용도 효율적이다.



### **트리**

노드와 에지로 이루어진 그래프의 일종으로 일반적인 그래프와 달리 순환 구조를 가지지 않는 것이 특징이다.

루트 노드를 제외한 모든 노드들은 단 하나의 부모 노드를 가지며 트리의 일부인 서브 트리는 트리의 모든 성질을 모두 만족한다.

한 트리 내에서 임의의 두 서브 트리의 루트 노드를 연결하는 경로는 무조건 1개이다.

트리를 구성하는 요소들은 아래와 같다.

|   요소    | 설명                                                    |
| :-------: | :------------------------------------------------------ |
| 부모 노드 | 두 노드 사이의 관계에서 상대적으로 상위에 위치하는 노드 |
| 자식 노드 | 두 노드 사이의 관계에서 상대적으로 상위에 위치하는 노드 |
| 루트 노드 | 트리에서 가장 상위에 있는 노드(부모 노드가 없음)        |
| 리프 노드 | 트리에서 가장 하위에 있는 노드(자식 노드가 없음)        |
| 서브 트리 | 전체 트리에 속하는 부분 트리                            |

일반적으로 트리를 표현하는 경우는 아래와 같다.

```java
public class TreeNode {
  String data;
  TreeNode lNode;
  TreeNode rNode;
}

```

하지만 트리는 그래프의 일종이기 때문에 에지 리스트, 인접 행렬, 인접 리스트 등의 방법으로도 구현이 가능하다.

DFS, BFS를 이용하여 트리를 탐색해야 하는 경우에는 인접 리스트를 이용하여 트리를 구현하는 것이 효과적이고, 이진 트리, 세그먼트 트리, 최소 공통 조상 찾기와 같은 경우에는 1차원 배열을 이용하여 트리만을 표현 가능한 구조로 구현하고 인덱스를 통해 접근하는 것이 효과적이다.

```java
public class Tree {
  // 1차원 배열로 이진트리를 표현
  // 0번은 사용하지 않는 것이 더 편하다
  // 부모 노드에 접근 -> (자식노드 인덱스 / 2)
  // 자식 노드에 접근 -> 왼족 자식 노드 : (부모노드 * 2), 왼족 자식 노드 : (부모노드 * 2 + 1)
  int[] binaryTree = new int[7];
}
```



-----------------------

##### 참고자료 : 

[LibreWiki](https://librewiki.net/wiki/%EC%8B%9C%EB%A6%AC%EC%A6%88:%EC%88%98%ED%95%99%EC%9D%B8%EB%93%AF_%EA%B3%BC%ED%95%99%EC%95%84%EB%8B%8C_%EA%B3%B5%ED%95%99%EA%B0%99%EC%9D%80_%EC%BB%B4%ED%93%A8%ED%84%B0%EA%B3%BC%ED%95%99/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98_%EA%B8%B0%EC%B4%88)

[https://www.youtube.com/playlist](https://www.youtube.com/playlist?list=PLG7te9eYUi7tAQygBknaTciy8wzLCe-Ll)