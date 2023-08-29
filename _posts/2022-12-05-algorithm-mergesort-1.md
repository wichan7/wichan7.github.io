---
title: "[알고리즘] Javascript로 병합 정렬 구현하기"
categories: 
  - Algorithm
tags:
  - algorithm
  - javascript
  - merge sort
toc: true
---
## 병합 정렬이란?
배열을 가장 작은 단위부터 합칩니다.<br>
분할, 병합 하다 보면(nlogn) 최종적으로 원본 배열이 정렬됩니다.<br>

## 구현 전체 코드 (Javascript)
~~~ javascript
let array = [0, 2, 1, 3, 7, 6, 4, 8, 5];

function merge(array) {

    if (array.length === 1) {
        return [array[0]];
    }

    let l = merge(array.slice(0, array.length / 2));
    let r = merge(array.slice(array.length / 2, array.length));
    let m = [];

    while( l.length !== 0 && r.length !== 0 ) {
        if ( l[0] < r[0] ) {
            m.push(l.shift());
        } else {
            m.push(r.shift());
        }
    }
    
    return [...m, ...l, ...r];
}

console.log(merge(array));
~~~