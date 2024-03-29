---
title: "Puppeteer 웹 크롤러 만들기 - 1"
categories: 
  - Web
tags:
  - web
  - crawling
  - javascript
toc: true
---

## 웹 크롤러란 무엇일까?
> 웹 크롤러(web crawler)는 조직적, 자동화된 방법으로 월드 와이드 웹을 탐색하는 컴퓨터 프로그램이다.  
>
> 웹 크롤러가 하는 작업을 **웹 크롤링**(web crawling) 혹은 **스파이더링**(spidering)이라 부른다. 검색 엔진과 같은 여러 사이트에서는 데이터의 최신 상태 유지를 위해 웹 크롤링한다. 웹 크롤러는 대체로 방문한 사이트의 모든 페이지의 복사본을 생성하는 데 사용되며, 검색 엔진은 이렇게 생성된 페이지를 보다 빠른 검색을 위해 인덱싱한다. 또한 크롤러는 링크 체크나 HTML 코드 검증과 같은 웹 사이트의 자동 유지 관리 작업을 위해 사용되기도 하며, 자동 이메일 수집과 같은 웹 페이지의 특정 형태의 정보를 수집하는 데도 사용된다.  
>
> 웹 크롤러는 봇이나 소프트웨어 에이전트의 한 형태이다. 웹 크롤러는 **대개 시드(seeds)라고 불리는 URL 리스트에서부터 시작**하는데, 페이지의 모든 **하이퍼링크를 인식하여 URL 리스트를 갱신**한다. 갱신된 URL 리스트는 재귀적으로 다시 방문한다.  
>
>...[wikipedia](https://ko.wikipedia.org/wiki/%EC%9B%B9_%ED%81%AC%EB%A1%A4%EB%9F%AC) 중 발췌...

웹 크롤러의 정의에서 중요한 점은, 단순히 하나의 URL Endpoint에서 데이터를 파싱하고 뽑아내는, 좁은 작업이 아니라는 것이다.  
**하이퍼링크를 인식하여 URL 리스트를 갱신**하고, 이것을 목적에 따라 데이터를 추출하고, 하이퍼링크를 인덱싱 해가며 웹을 떠돌아야 한다.  

## 어떻게 만들 수 있을까?
Java의 HttpConnection 모듈, C#의 HttpWebRequest 클래스, NodeJS의 http 모듈 등 http 통신 핸들링이 가능한 언어라면, 간단한 크롤러를 만들 수 있을 것이다.  

하지만 네이티브 코드로 쌓아나간다면 브라우저 javascript 런타임에 대한 고려, 필요한 브라우저의 기능 구현 등 목적에 따라 여러가지 구현할 요소가 많아질 것이다.  

이런 기초 구현을 신경쓰지 않기 위해 브라우저를 조종해 크롤링을 구현하는 방법이 있다.  

## 크로미움을 알자
크로미움은 오픈 소스 웹 브라우저 이름이며, 모두가 잘 아는 '크롬'은 '크로미움'을 기반으로 만들어졌다.  
크롬 뿐만 아니라 삼성 인터넷 브라우저, 네이버 웨일, 마이크로소프트 엣지 등이 크로미움 기반으로 개발되었다.  

익히 알려진 Selenium과 비교적 최근에 나온 **Puppeteer** 등이 크로미움 제어를 지원하고 있는데, 본 포스팅에서는 Puppeteer를 사용해볼 것이다.  

## Puppeteer 설치
```
npm init
npm i puppeteer
```

## Puppeteer를 실행해보자
프로젝트 폴더에 app.js를 생성하고, 아래 소스코드를 붙여넣어보자.  
``` javascript
// app.js
const puppeteer = require('puppeteer')

(async () => {
    const links = ["https://naver.com", "https://google.com", "https://daum.net"]

    const browser = await puppeteer.launch({ headless: false })
    for (let link of links) {
        const page = await browser.newPage()
        await page.goto(link)
        console.log(page.content())
    }

    await browser.close()
})()
```

puppeteer는 크로미움 브라우저 조종을 지원하는 자바스크립트 라이브러리이다.  
이 라이브러리는 자바스크립트 사상에 맞게, 비동기 처리 되도록 구성되어 non-blocking 하게 구성 가능하다. (필요한 경우 await이나 callback function으로 동기처리 할 수 있다.)  

소스 코드에서 느낌이 오듯 `puppeteer.launch()`로 브라우저를 오픈하고 `browser.newPage()`로 빈 페이지를 생성할 수 있다.  
`page.content()`로 page의 HTML 등 컨텐츠를 문자열로 리턴받을 수도 있다.  

다음 포스팅에서는 Cheerio 라이브러리와 함께 page를 파싱하고 원하는 정보를 얻어보자.