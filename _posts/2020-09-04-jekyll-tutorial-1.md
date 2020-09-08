---
title: "[Jekyll] Minimal-Mistakes테마로 Github블로그 만들기! -1"
categories: 
  - jekyll
tags:
  - minimal mistakes
  - jekyll
  - tutorial
  
---

###### Windows10 환경에서 진행했습니다.  

> [1. Ruby+Devkit 설치하기](#step1)  
> [2. Ruby에서 Jekyll 다운로드 받기](#step2)  
> [3. Github 레포지토리 만들기](#step3)  
> [4. Minimal-Mistakes 테마 다운받고 적용하기](#step4)  
> [5. Github에 띄우기](#step5)  

## step1
- Jekyll(지킬)은 정적 사이트를 생성해줍니다. 지킬으로 블로그를 수정하고 즉각 로컬 상에서 확인하기 위해 설치합니다.  
- 지킬은 Ruby를 통해 다운로드 받을 수 있는데 이를 위해 Ruby를 다운로드 받습니다.  
[Ruby 다운로드 페이지](https://rubyinstaller.org/downloads/){: target="_blank"}  

- 위 페이지의 왼쪽에서 **Ruby+Devkit 2.7.1-1(x64)**를 다운로드 받습니다.  
- 모든 체크박스를 **체크**하고 모두 다운로드 받아줍니다.  

## step2
윈도우 검색에서 **Start Command Prompt With Ruby**를 찾아 실행시켜줍니다.  
그리고 그 prompt창에서 아래의 2가지 명령어를 실행합니다.  
1. `gem install bundler`  
2. `gem install jekyll`  

bundler는 의존성을 편하게 관리하게 해주는 역할을 합니다.  

## step3
깃허브에서 레포지토리를 생성합니다. 

{% raw %}![alt](/assets/images/jekyll_tutorial/github_create_repository.png){% endraw %}  

레포지토리 주소를 이렇게 만들면 "http://wichan7.github.io" 형식으로 접근이 가능해집니다. (Github 적용이 느려서 기다렸다가 확인해야합니다)  


## step4
[Minimal-Mistakes 다운로드 페이지](https://github.com/mmistakes/minimal-mistakes){: target="_blank"}  
- 위 페이지에서 초록색 버튼을 눌러 Zip 파일을 다운로드 받습니다.  
- C:밑에 Blog폴더를 하나 생성합니다.  
- 아까 다운로드 받은 Zip파일의 압축을 풀어 내용물을 Blog폴더에 담습니다.  
- _config.yml 파일을 열고, url부분에 **자신의 github 레포지토리 주소**를 적습니다.  

{% raw %}![alt](/assets/images/jekyll_tutorial/jekyll_config.png){% endraw %}  

- Ruby prompt에서 **"cd C:\\Blog"** 명령어로 작업공간을 이동합니다.  
1. `bundle install`을 한번 실행해줍니다.  
2. `bundle exec jekyll serve` 명령어로 이제부터 서버를 구동시킬 수 있습니다.  
- 서버를 구동하게 되면 이제 Chrome에서 localhost:4000으로 접속할 수 있게 됩니다.  
- Minimal-Mistakes 테마 적용까지 완료했습니다.  

## step5
- Blog 폴더를 작업공간으로 만들고, 아까 만든 레포지토리로 Commit, push를 해줍니다.  
- 잠시 후 닉네임.github.io로 접속하게 되면 Minimal-Mistakes테마가 적용된 페이지를 만날 수 있습니다.  

<br><br>
# -끝-
긴 글 읽어주셔서 감사합니다.  

다음 편으로는 Category, Tag, Search 등 여러 기능을 추가하는 포스팅으로 찾아오겠습니다.  

