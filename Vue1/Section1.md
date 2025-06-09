# Vue.js 소개

## Vue.js란?

- 간단한 화면 UI 개발부터 라우팅, SSR 등의 애플리케이션 레벨의 개발을 지원하는 프레임워크.

<br/>

## Vue 2와 Vue 3의 차이점

- typescript 기반 라이브러리 내부 로직 재작성
- 주요 개발 도구 변경  
ex) 뷰 개발자 도구, VSCode 플러그인, Vite 기반 프로젝트 생성
- 컴포지션 API, Teleport 등 새로운 문법 지원
- Reactivity 시스템 기반 API 변경

<br/>

## Vue 3 코드 작성 방법

> 옵션 API

```typescript
<div id="app">{{ message }}</div>

<script>
    Vue.createApp({
        data() {
            return {
                message: 'hi',
            };
        },
    }).mount('#app');
</script>
```

> 컴포지션 API

```typescript
<div id="app">{{ message }}</div>

<script>
    Vue.CreateApp({
        setup() {
            const message = ref('hi');

            return {
                message
            }
        }
    }).mount('#app');
</script>
```

Vue 3 에서는 모두 작성 가능한 문법.  
Vue의 사용에 익숙한 사람은 컴포지션 문법이 익숙할 것.  
(클린 코드 관점에서도 컴포지션 문법이 더 유리)

<br/>

## 개발 환경 구성

- nodejs LTS 버전 다운로드
- vscode extenstion 다운로드  
ex) vue vscode snippets, live server, material icon theme, night owl, volar(Vue3 한정)

<br/>

## Vue.js 개발자 도구 안내

- chrome extension Vue.js devtools 다운로드

Vue 애플리케이션에 한해서, 개발자 도구 지원을 받을 수 있다.

<br/>

## 강의 교안과 소스 코드 안내

- [Vue.js 공식 문서](https://v3-docs.vuejs-korea.org/guide/quick-start.html)
- [Cracking Vue.js](https://joshua1988.github.io/vue-camp/)
- [lecture github repository](https://github.com/joshua1988/learn-vue-js)

<br/>

## Hello World(Vue.js 인스턴스)

```typescript
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script>
  const { createApp } = Vue

  createApp({
    data() {
      return {
        message: 'Hello World'
      }
    }
  }).mount('#app')
</script>
```

<br/>
