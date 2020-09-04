---
title: "[Jekyll] Minimal-Mistakes테마로 Github블로그 만들기! -2"
categories: 
  - jekyll
tags:
  - minimal mistakes
  - jekyll
  - tutorial
toc: true
---
이번 시간에는 블로그에 **Category**와 **Tag**기능을 넣어보도록 하겠습니다.  

## 1. navigation.yml 수정하기  
먼저 C:Blog 밑에 있는 **navigation.yml**을 열어주고 아래와 같이 수정해줍니다.
~~~ ruby
# main links
main:
  # - title: "Quick-Start Guide"
  #   url: https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/  
  - title: "Category"
    url: /categories/
  - title: "Tag"
    url: /tags/
  # - title: "About"
  #   url: https://mmistakes.github.io/minimal-mistakes/about/
  # - title: "Sample Posts"
  #   url: /year-archive/
  # - title: "Sample Collections"
  #   url: /collection-archive/
  # - title: "Sitemap"
  #   url: /sitemap/
~~~

## 2. _pages 폴더 추가하기
우선 C:\\Blog 밑에 **_pages**라는 이름으로 폴더를 생성합니다.  

그 후 C:\\Blog\\docs\\_pages 에서

- 404.md
- category-archive.md
- tag-archive.md  

파일을 새로 만든 _pages 폴더로 이동시킵니다.

## 3. _config.yml 수정하기
C:\\Blog\\_config.yml을 열고 맨 아래로 내립니다.  

~~~ ruby
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: # true
      share: true
      related: true

  #>>>>>>>>>>>>>>>>>>> 여기서부터 추가 됨
  # _pages 부분입니다.
  - scope:
      path: "_pages"
      type: pages
    values:
      layout: single
      author_profile: true
  # >>>>>>>>>>>>>>>>>>>> 여기까지 추가 됨
~~~

#으로 주석된 부분을 추가해줍니다.  

## 4. _post에서 포스팅하기
이제 포스팅 할 때 아래 예제처럼 작성합니다.  
~~~ ruby

---
title: "[Jekyll] Minimal-Mistakes테마로 Github블로그 만들기! -2"
categories: 
  - jekyll
  - babo
tags:
  - minimal mistakes
  - jekyll
  - tutorial
toc: true
---

밑에 대충 글 내용.

~~~

완성!  
