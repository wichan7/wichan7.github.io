---
title: "[programmers] H-index"
categories: 
  - programmers
tags:
  - algorithm
  - programmers
toc: true
---

# programmers lv2. H-index
~~~ javascript
function solution(str1, str2) {
    str1 = str1.toUpperCase();
    str2 = str2.toUpperCase();

    if (str1 === str2) {
        return 1 * 65536;
    }

    //init arr1, arr2
    let arr1 = [];
    for (let i=0; i<str1.length-1; i++) {
        let s = "" + str1[i] + str1[i + 1];
        if (/[A-Z]{2}/.exec(s)) {
            arr1.push(s);
        }
    }
    let arr2 = [];
    for (let i=0; i<str2.length-1; i++) {
        let s = str2[i] + str2[i + 1];
        if (/^[A-Z]{2}$/.exec(s)) {
            arr2.push(s);
        }
    }
    arr1 = arr1.sort();
    arr2 = arr2.sort();
    let equal = 0;
    let diff = 0;
    while (arr1.length && arr2.length) {
        if (arr1[0] === arr2[0]) {
            arr1.shift();
            arr2.shift();
            equal += 1;
        }
        else if (arr1[0] < arr2[0]) {
            arr1.shift();
            diff += 1;
        } else {
            arr2.shift();
            diff += 1;
        }
    }
    
    diff += arr1.length + arr2.length;
    return Math.floor(equal / (equal + diff)  * 65536);
}
~~~

풀이는 쉬운데.. 어떻게 최적화 할 수 있을까?
