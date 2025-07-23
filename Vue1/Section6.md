# Vue CLI: 뷰 프로젝트 생성 도구

## Vue CLI 소개 및 설치

[Vue.js 프로젝트 생성 도구](https://cli.vuejs.org/)  
CLI: Command-Line Interface

```
npm install -g @vue/cli
```

```
vue create vue3-cli
vue create vue3-cli --packageManager npm
```

yarn에서 에러 발생하여 npm으로 강제 지정

<br/>

## Vue 프로젝트 생성 과정 설명 및 서버 실행

```
cd vue3-cli

npm run serve
```

<br/>

## Vue 프로젝트 폴더 내용 살펴보기

> pakcage.json: dependencies / devDependencies

```
yarn add -D {라이브러리명}
npm install -D {라이브러리명}
```
devDependencies에 포함된 라이브러리는 실제 배포할 때 포함되지 않기 때문에 빌드 시간을 줄일 수 있다.


> vue.config.js

```js
const { defineConfig } = require("@vue/cli-service");
module.exports = defineConfig({
  transpileDependencies: true,
  // ...
});
```

webpack에 대한 설정.  
하나하나의 설정을 모두 작성하지는 않음.

> public/index.html

```html
<!DOCTYPE html>
<html lang="">
  <head>
  <!-- ... -->
  </head>
  <body>
    <!-- ... -->
    <div id="app"></div>
  </body>
</html>
```

app 내부에서 Vue CLI로 빌드된 결과물이 출력.

> main.js

```js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

인스턴스에 마운트 하는 핵심 과정을 포함.

<br/>

## 라이브러리, 파일 임포트 방식 설명

> main.js

```js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

**from** {???}: package.json에 작성된 라이브러리 원본

> node_modules

해당 파일이 라이브러리 함수들을 보관.

ex. runtime-core/dist/runtime-core.cjs.js  
이러한 형태로 작성된 라이브러리 파일을 불러와 사용하는 구조.

> 라이브러리 호출과 파일 호출

```js
// 라이브러리 호출
import { createApp } from 'vue'
// 파일 호출
import App from './App.vue'
```

모듈 기반으로 관심사를 분리하여 파일을 나눠 작성할 수 있는 구조는 리액트와 동일.

<br/>

## 페이지 로딩 과정 분석

localhost:8080 > element

```html
<head>
    <!-- ... -->
    <script defer src="/js/chunk-vendors.js"></script>
    <script defer src="/js/app.js"></script>
</head>
```

<br/>
