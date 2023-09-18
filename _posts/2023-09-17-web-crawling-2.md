---
title: "Puppeteer 웹 크롤러 만들기 - 2"
categories: 
  - Web
tags:
  - web
  - crawling
  - javascript
toc: true
---
## 개요
[👉 웹 크롤러 만들기 - 1 바로가기](https://wichan7.github.io/web/web-crawling-1/)  

Puppeteer, Cheerio 모듈을 활용해, 웹페이지에서 이메일 주소를 수집하는 크롤링 봇을 구성했다.  
`Trigger Page` 로부터 이메일 주소를 수집하고, `<a href=...>`를 탐색해 중복 없이 방문한다.  

## 모듈 Documentation
[👉 Puppeteer Documentation](https://pptr.dev/)  
[👉 Cheerio](https://cheerio.js.org/docs/intro)  

## 소스 코드
``` javascript
// crawl.js
// 3rd party declaration
import * as puppeteer from 'puppeteer';
import * as cheerio from 'cheerio';
// own libraries declaration
import Queue from './queue.js';

/**
 * Concept:
 *  1. visitQueue에서 url dequeue
 *  2. url 방문
 *  3. 이메일 주소 리스트 추출
 *  4. href 추출, visitQueue에 enqueue
 *  6. 1 ~ 4 반복
 */
(async() => {
    const emails = new Set();
    const histories = new Set();
    const visitQueue = new Queue();

    const browser = await puppeteer.launch();
    const workPage = await browser.newPage();

    // Trigger Page 설정
    visitQueue.enqueue("https://www.naver.com");
    while (!visitQueue.isEmpty()) {
        // 1.
        const url = visitQueue.dequeue();
        // 2.
        await workPage.goto(url);
        // 3.
        const $ = cheerio.load(await workPage.content());
        for (let email of extractEmails($)) {
            emails.add(email);
        }
        // 4.
        for (let href of validateHrefs(extractHrefs($), histories)) {
            visitQueue.enqueue(href);
        }
        // logging
        console.log(emails);
    }
})();

function extractEmails($) {
    /**
     * RFC2822 Email Validation
     */
    return $('body').text().match(/[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?/g) || [];
}

function extractHrefs($) {
    const hrefs = [];

    $('a').each( (i, a) => {
        const href = $(a).attr('href') || "";
        if (href.startsWith("https://") || href.startsWith("http://")) {
            hrefs.push(href);
        }
    } );

    return hrefs;
}

function validateHrefs(hrefs, histories) {
    return hrefs.filter( href => !histories.has(href) )
}

async function wait(seconds) {
    return new Promise( resolve => setTimeout(resolve, seconds * 1000) )
}
```

``` javascript
// queue.js
export default class Queue {
    constructor() {
        this.arr = []
    }

    enqueue(element) {
        this.arr.push(element)
    }

    dequeue() {
        return this.arr.shift()
    }

    isEmpty() {
        return this.arr.length === 0 ? true : false
    }
}
```

## 개선 방향
1. Query, Path Parameter에 대한 trim 후 중복체크를 수행해야, 의미없는 페이지에 중복 방문하지 않는다. (Canonical Tag를 활용하면 될 듯?)  
2. href에 Full URL이 아닌 `'/path'`, `'./path'` 등 경로가 들어오는 경우 폐기하고 있는데, baseUrl을 얻을 수 있다면 path와 base_url의 합성이 가능하다.  