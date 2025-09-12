# Vue 싱글 파일 컴포넌트

## Vue Single File Component 소개

[싱글 파일 컴포넌트](https://joshua1988.github.io/vue-camp/vue/sfc.html#%E1%84%89%E1%85%B5%E1%86%BC%E1%84%80%E1%85%B3%E1%86%AF-%E1%84%91%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%AF-%E1%84%8F%E1%85%A5%E1%86%B7%E1%84%91%E1%85%A9%E1%84%82%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3%E1%84%8B%E1%85%B4-%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8C%E1%85%A1%E1%86%A8-%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5)

다른 프레임워크는 가지지 않는 Vue의 특이점.  
싱글 파일 컴포넌트는 화면의 특정 영역에 대한 HTML, CSS, JS 코드를 한 파일에서 관리하는 방법.

```html
<!-- .vue 파일 구조 -->
<template>
  <!-- html (뷰 컴포넌트의 표현단, 템플릿 문법) -->
</template>

<script>
  // 자바스크립트 (뷰 컴포넌트 내용)
</script>

<style>
  /* CSS (뷰 템플릿의 스타일링) */
</style>
```

브라우저에서는 .vue 파일이 인식될 수 없어 별도의 변환 과정이 존재.

<br/>

## App 컴포넌트

```html
<!-- html 영역 -->
<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
  <!-- <hello-world msg="Welcome to Your Vue.js App"/> -->
</template>
```
single file component level에서 컴포넌트는 무조건 Pascal case로 작성.  
'hello-world'로 작성해도 문제는 없으며 일관되게 작성하는 것이 중요.

```html
<!-- javascript 영역 -->
<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>
```

```html
<!-- css 영역 -->
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

<br/>

## 싱글 파일 컴포넌트 코드 작성 팁

-

<br/>

## Vue 컴포넌트 등록 방법 및 명명 규칙

```html
<!-- App.vue -->
<template>
  <AppHeader />
  <div>
    {{ message }}
  </div>
  <button @click="showAlert">경고</button>
</template>

<script>
import AppHeader from "./components/AppHeader.vue";

export default {
  components: {
    AppHeader,
  },
  data() {
    return {
      message: "hello",
    };
  },
  methods: {
    showAlert() {
      alert("경고");
    },
  },
};
</script>
```

```html
<!-- AppHeader.vue -->
<template>
  <div>
    <h1>AppHeader</h1>
  </div>
</template>

<script>
export default {
  name: "AppHeader",
};
</script>
```

<br/>

## 싱글파일 컴포넌트의 props, event emit

```html
<template>
  <AppHeader :appTitle="message" @change="changeMessage" />
</template>

<script>
import AppHeader from "./components/AppHeader.vue";

export default {
  components: {
    AppHeader,
  },
  data() {
    return {
      message: "앱 헤더 컴포넌트",
    };
  },
  methods: {
    changeMessage() {
      this.message = "변경됨";
    },
  },
};
</script>
```

```html
<template>
  <h1>{{ appTitle }}</h1>
  <button @click="changeTitle">click</button>
</template>

<script>
export default {
  props: ["appTitle"],
  methods: {
    changeTitle() {
      this.$emit("change");
    },
  },
};
</script>

<style scoped></style>
```

<br/>
