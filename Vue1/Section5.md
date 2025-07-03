# Vue.js 템플릿 문법

## Vue 템플릿 문법(Template Syntax) 소개

[뷰의 템플릿 문법](https://joshua1988.github.io/vue-camp/vue/template.html)

<br/>

## Vue 디렉티브: v-if, v-show

```html
<!-- data-binding.html -->
<div id="app">
  <!-- v-if -->
  <p v-if="login">로그인 성공</p>
  <p v-else>로그인 실패</p>
  <button @click="loginUser">로그인</button>

  <hr />

  <!-- v-show -->
  <p v-show="login">로그인 성공</p>
  <button @click="loginUser">로그인</button>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  Vue.createApp({
    data() {
      return {
        login: false,
      };
    },
    methods: {
      loginUser() {
        this.login = !this.login;
      },
    },
  }).mount("#app");
</script>
```

<br/>

## Vue 데이터 바인딩: id, class, style

```html
<!-- class-styling -->
<style>
  .primary {
    color: red;
  }
</style>

<div id="app">
  <h1>클래스 바인딩</h1>
  <!-- 색상을 동적으로 조정할 수 있게 함 -->
  <div :class="textClass">클래스 데이터 바인딩 예제</div>

  <h1>아이디 바인딩</h1>
  <section :id="tab" :style="sectionStyle">아이디 데이터 바인딩 예제</section>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  Vue.createApp({
    data() {
      return {
        textClass: "primary",
        sectionId: "tab",
        sectionStyle: { color: "red" },
      };
    },
  }).mount("#app");
</script>
```

<br/>