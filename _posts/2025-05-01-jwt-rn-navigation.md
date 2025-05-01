---
title: "[React-Native] Navigation Stack을 입맛대로 조작하기"
categories:
  - App
tags:
  - reactnative
---

우리는 React-Native와 React로 하이브리드 앱 서비스를 제공한다.  
요구사항을 구현하던 중 화면을 전환하지 않고 Native App의 화면 스택을 조작해야하는 일이 생겼다.  

## 어떻게?
[RN 공식 문서](https://reactnavigation.org/docs/navigation-actions/)에 친절한 예제가 있다.  

``` typescript
import { CommonActions } from '@react-navigation/native';

navigation.dispatch((state) => {
  // Remove all the screens after `Profile`
  const index = state.routes.findIndex((r) => r.name === 'Profile');
  const routes = state.routes.slice(0, index + 1);

  return CommonActions.reset({
    ...state,
    routes,
    index: routes.length - 1,
  });
});
```

navigation.dispatch의 콜백으로 현재 앱의 routes를 참조할 수 있는데,  
CommonActions.reset과 함께 사용하여 앱의 스크린 스택을 정말 자유롭게 조작할 수 있었다.  


공식문서 스펙을 잘 읽으면 떡이 나온다.