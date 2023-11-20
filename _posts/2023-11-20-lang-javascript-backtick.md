---
title: "자바스크립트, 백틱의 신기한 활용"
categories: 
  - Lang
tags:
  - lang
  - javascript
toc: true
---

## 백틱
윈도우 키보드 물결 표시에 있는 이 것. **`** 백틱이다.  
백틱에는 템플릿 리터럴 이외에, 함수를 호출하는 기능도 있다. (ㄷㄷ!)  

## Template Literal
``` javascript
const name = 'wichan'
const str = `hello ${name}!`

// result: hello wichan!
console.log(str)
```
Template Literal 이라고 부른다.  

## Tagged Template
각설하고, 이 [Tagged Template](https://tc39wiki.calculist.org/es6/template-strings/) 때문에 포스팅했다.  
``` javascript
function fun(arg) {
  console.log(arg)
}

// result: funny javascript
fun('funny javascript')
// result: [ 'funny javascript' ]
fun`funny javascript`
```
함수를 백틱으로 호출하는데 성공했다. 근데 왜 arg가 배열로 들어왔을까?  

### 이럴 때 쓴다.
사실, 백틱으로 함수를 호출할 때 넘어오는 인자가 더 있다.  
다음 예제를 보자.  
``` javascript
const name = 'wichan'
const age = 5

function intro(strings, ...values) {
    console.log(strings)
    console.log(values)
}

/** result:
 *   strings: ['my name is ', ' and i am ', ' years old.']
 *   values: ['wichan', 25]
 */
intro`my name is ${name} and i am ${age} years old.`
```

백틱으로 함수를 호출하는 경우 첫번째 인자로 raw string, 그 이후로 template들을 리턴해준다.  
함수에 문자열을 입력할 때, 템플릿 변수와 문자열을 분리할 때 사용할 수 있다.  