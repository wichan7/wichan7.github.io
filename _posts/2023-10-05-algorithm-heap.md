---
title: "[알고리즘] Javascript로 우선순위 큐 구현하기 (Feat. Heap)"
categories: 
  - Algorithm
tags:
  - algorithm
  - javascript
  - heap
toc: true
---

## 우선순위 큐란 무엇인가?
> 우선순위 큐(Priority Queue)는 우선순위가 높은 데이터가 먼저 나가는 형태의 자료구조이다.  
원소가 큐에 방금 들어왔어도, 우선순위가 가장 높으면 가장 먼저 나갈 수 있다.  

## 어떻게 구현할까?
작은 숫자가 더 높은 우선순위를 가진다고 가정하자. 어떻게 구현할 수 있을까?  

### 직관적인 방법
직관적으로 생각해보면, 일단 집어넣고 배열 내 모든 원소를 순회하는 방법이 있다.  
``` javascript
class PQ {
    constructor() {
        this.queue = [];
    }

    push = (e) => this.queue.push(e); 

    pop = () => {
        if (this.queue.length === 0) return;

        let min = Infinity;
        for (let elem of this.queue) {
            if (elem < min) min = elem;
        }
        return min;
    }
}
```  
직관적인 방법으로 요구사항을 구현했다.  

> 이 때, push 함수의 시간 복잡도가 O ( 1 )  
pop 함수의 시간 복잡도가 O ( N ) 임은 자명하다.  

### 힙 자료구조를 사용하는 방법
**힙**은 부모 노드가 자식 노드보다 항상 더 작은 `완전이진트리`이다.  
참고로 완전이진트리는, 위쪽, 왼쪽으로 노드가 채워져있는 트리 자료 구조이다.  
`완전이진트리`  
![structure](/assets/images/algorithm/heap/complete.png)  

`포화이진트리`  
![structure](/assets/images/algorithm/heap/perfect.png)  

완전이진트리와 포화이진트리는 다른 것으로, 포화이진트리는 노드가 꽉꽉 차 있다.  

각설하고, 힙의 '부모 노드가 자식 노드보다 항상 더 작은 특징'을 이용해 우선순위 큐를 만들 수 있다.  
큐 push 연산 시 항상 힙의 구조를 만족하게 삽입하고,  
큐 pop 연산 시 힙의 루트 노드를 pop하면 우선순위 큐가 된다.  

#### 구현
``` javascript
class PQ {
    constructor() {
        this.heap = [null];
    }

    _swap = (a, b) => [this.heap[a], this.heap[b]] = [this.heap[b], this.heap[a]];

    size = () => this.heap.length - 1;

    peek = () => this.heap[1] !== undefined ? this.heap[1] : null;

    push = (value) => {
        this.heap.push(value);
        let curIdx = this.heap.length - 1;
        let parIdx = (curIdx / 2) >> 0;

        while (curIdx > 1 && this.heap[curIdx] < this.heap[parIdx]) {
            this._swap(parIdx, curIdx)
            curIdx = parIdx;
            parIdx = (curIdx / 2) >> 0;
        }
    }

    pop = () => {
        const min = this.heap[1];
        if (this.heap.length <= 2) this.heap = [null];
        else this.heap[1] = this.heap.pop();

        let curIdx = 1;
        let leftIdx = curIdx * 2;
        let rightIdx = curIdx * 2 + 1;

        while (this.heap[leftIdx] < this.heap[curIdx] || this.heap[rightIdx] < this.heap[curIdx]) {
            const minIdx = this.heap[leftIdx] > this.heap[rightIdx] ? rightIdx : leftIdx;
            this._swap(minIdx, curIdx);
            curIdx = minIdx;
            leftIdx = curIdx * 2;
            rightIdx = curIdx * 2 + 1;
        }

        return min;
    }
}
```

> 이 때, push 함수의 시간 복잡도가 O ( logN )  
pop 함수의 시간 복잡도가 O ( logN )임을 알 수 있다.  

## 마무리
기본 javascript에는 heap이나 STL의 priority_queue가 제공되지 않아, 직접 구현해야만 한다.  
요것 뿐만이 아니라, combination, permutation, queue ...등 직접 구현해왔다.  

코딩 테스트 응시 중에 이걸 언제 구현하고 있을까...