---
layout: single
title: '[React] React를 이용해 간단한 일기장 만들기'
categories: Frontend
tag: [React]
toc: true 
author_profile: false
sidebar:
    nav: "counts"
published: true

---

리액트를 이용하여 간단한 일기장을 만들어보는 프로젝트를 진행했다. 코드는 [한입크기로 잘라먹는 리액트](https://www.inflearn.com/course/%ED%95%9C%EC%9E%85-%EB%A6%AC%EC%95%A1%ED%8A%B8)를 참고했으며, 강의를 완강한 기념으로 관련 내용을 정리하고자한다. 

## React + Vite
감정 일기장은 사용자의 기분을 기록할 수 있는 간단한 일기장 웹 어플리케이션이다. React + Vite를 이용하여 개발했다. 

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-10-19-emotion-diary\vite_react.png" alt="Alt text" style="width: 60%; height: 60%; margin: 30px">
</div>


### React 

전통적인 **MPA(Multi-Page Application) 방식**에서는 서버가 여러 개의 웹 페이지를 가지고 있고, 사용자가 브라우저를 통해 요청을 보내면 서버는 해당 요청에 맞는 HTML 페이지를 SSR(Server-Side Rendering) 방식으로 미리 생성하여 반환한다.

이러한 방식은 초기 웹 개발에서 주로 사용되었으며, 각 페이지 이동 시마다 전체 HTML 문서를 새로 받아와 렌더링했다.

하지만 이 방식에는  페이지 이동 시 화면 깜빡임, 모든 요소의 재렌더링, 서버 부하 증가 등의 단점이 있었다.

React는 전통적인 MPA 방식의 단점을 극복하기 위해 **SPA(Single Page Application) 방식**을 이용한다. 이 방식에서는 단일 HTML 파일의 내용을 자바스크립트를 이용해 재 렌더링 해주는 방식으로 페이지를 구성한다. 


### Vite 
Vite는 JavaScript, CSS, 이미지 등 프로젝트에서 사용하는 모든 파일들을 하나의 번들 파일로 묶어 브라우가 번들파일을 받아 직접 실행하게 도와준다. 

Vite는  CSR 개발 환경에서의 번들링과 개발 속도를 최적화하기 위해 만들어진 도구라고 할 수 있다.


## 감정 일기장 
전체적인 프로젝트 진행방식은 다음과같다. 
 
>라우팅 설정 -> 글로벌 레이아웃 → 공통 컴포넌트 구현 → 개별 페이지 및 복잡한 기능 구현


### 개발환경

감정 일기 프로젝트는 복잡한 데이터 관리가 필요하지 않기 때문에, Web Storage의 데이터 저장 기능을 활용하여 사용자가 감정 일기를 쉽게 기록하고 관리할 수 있도록 설계되었다.

- 프론트엔드: React
- 데이터베이스: Web Storage (localStorage)
- 개발 도구: VSCode, Vite

### 디렉토리 구조 

```
📦 src
 ┣ 📂assets                 # 이미지 및 정적 파일 저장
 ┣ 📂components             # 재사용 가능한 UI 컴포넌트
 ┃ ┣ 📜Button.jsx           
 ┃ ┣ 📜DiaryItem.jsx        
 ┃ ┣ 📜DiaryList.jsx        
 ┃ ┣ 📜Editor.jsx           
 ┃ ┣ 📜EmotionItem.jsx      
 ┃ ┣ 📜Header.jsx           
 ┃ ┗ 📜Viewer.jsx           
 ┣ 📂hooks                  # 커스텀 훅 모음
 ┃ ┣ 📜useDiary.jsx         
 ┃ ┗ 📜usePageTitle.jsx    
 ┣ 📂pages                  # 페이지 단위 컴포넌트
 ┃ ┣ 📜Diary.jsx            
 ┃ ┣ 📜Edit.jsx             
 ┃ ┣ 📜Home.jsx             
 ┃ ┣ 📜New.jsx              
 ┃ ┗ 📜Notfound.jsx         
 ┣ 📂util                   # 유틸리티 함수 모음
 ┃ ┣ 📜constant.js          
 ┃ ┣ 📜get-emotion-image.js 
 ┃ ┗ 📜get-string-date.jsx  
 ┣ 📜App.css                # 전체 앱의 스타일
 ┣ 📜App.jsx                # 앱의 루트 컴포넌트
 ┣ 📜index.css              # 전역 스타일
 ┗ 📜main.jsx               # 앱의 진입점

```
<br>
페이지는 크게 Diary, Edit, Home, New, Notfound 페이지로 나뉜다. 

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-10-19-emotion-diary\home.png" alt="Alt text" style="width: 30%; height: 40%; margin: 10px">
     <img src="{{site.url}}\images\2024-10-19-emotion-diary\write.png" alt="Alt text" style="width: 30%; height: 40%; margin: 10px">
     <img src="{{site.url}}\images\2024-10-19-emotion-diary\diary.png" alt="Alt text" style="width: 30%; height: 40%; margin: 10px">
</div>

 - Home : 일기 리스트 렌더링
 - Diary : 일기 상세 조회 
 - New : 새로운 일기 작성
 - Edit : 기존 일기 수정

### 상태관리 

프로젝트의 상태 관리는 React의 기본적인 상태 관리 도구인 useState, useReducer, useContext를 활용하여 이루어진다. 

#### useState 
간단한 상태 관리

- 데이터 로딩 상태
- 정렬 타입
- 일기 아이템
- 달력 날짜

#### useReducer 
복잡한 상태 관리

- 새로운 일기 생성
- 기존 일기 수정
- 기존 일기 삭제

#### useContext 
전역 상태 관리
- 전체 일기 데이터 관리
- 일기 생성, 수정, 삭제 함수 제공

### 배포 
GitHub Pages를 이용해 배포를 진행했다. gh-pages 패키지는 GitHub Pages에 정적 웹사이트를 배포하기 위한 간편한 도구이다.

#### gh-pages 패키지 
> npm install gh-pages --save-dev

#### package.json  

프로젝트를 빌드하여 dist 폴더에 빌드 파일 생성후 gh-pages 패키지를 사용하여 dist 폴더의 내용을 GitHub Pages에 배포한다. 

```js
  "scripts": {
     "deploy": "gh-pages -d dist",
     "predeploy": "npm run build",
  }
```

## 배포된 페이지 

[감정 일기장](https://jyundev.github.io/emotion-diary/) 에서 프로젝트를 볼 수 있다. 

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-10-19-emotion-diary\emotion_diary.png" alt="Alt text" style="width: 70%; height: 70%; margin: 10px">
</div>

## 후기 

이 프로젝트는 간단한 기능을 가지고 있지만, 공통 컴포넌트와 props를 이용한 구조를 통해 React의 뛰어난 재사용성을 실감할 수 있었다.

또한, useState, useReducer, useContext를 활용한 상태 관리 경험을 통해 효율적인 상태 관리의 중요성을 생각해볼 수 있었고 커스텀 훅을 사용함으로써 코드의 재사용성을 높이는 방법에 대해서도 깊이 있게 고민해볼 수 있었다.

아직 useState, useReducer, useContext를 적절히 사용해야 할 상황에 대한 공부가 필요하며, 앞으로 Redux를 포함한 다양한 리액트 상태 관리 솔루션에 대해서도 학습할 계획이다.

<br>
<br>

----
Reference

- <a href = 'https://trends.builtwith.com/javascript/React'>React Usage Statistics</a>
- <a href = 'https://modulabs.co.kr/blog/react-library/'>리액트(React)를 왜 사용해야 할까? </a>
- <a href = 'https://nomadcoders.co/react-for-beginners/lectures/3256'>Why React</a>
- <a href = 'https://www.inflearn.com/course/%ED%95%9C%EC%9E%85-%EB%A6%AC%EC%95%A1%ED%8A%B8/dashboard'> 한입 크기로 잘라 먹는 리액트(React.js)</a>


