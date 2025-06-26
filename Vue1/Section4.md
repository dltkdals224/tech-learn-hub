# Vue.js 컴포넌트

## Vue Component 소개

[뷰 컴포넌트](https://joshua1988.github.io/vue-camp/vue/components.html#%E1%84%8F%E1%85%A5%E1%86%B7%E1%84%91%E1%85%A9%E1%84%82%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3-%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC-%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3-%E1%84%92%E1%85%A7%E1%86%BC%E1%84%89%E1%85%B5%E1%86%A8)

React와 마찬가지로 UI 영역을 컴포넌트 단위로 분리하여 개발을 수행한다.

컴포넌트 생성하는 기본적인 코드 형식

```javascript
var app = Vue.createApp();

app.component('컴포넌트 이름', {
  // 컴포넌트 내용
});
```

<br/>

## Vue Component 등록과 표시

```html
<!-- html -->
<div id="app">
    <app-header></app-header>
</div>

<!-- javascript -->
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
    Vue.createApp({
        components: {
            'app-header': {
                template: `<h1>컴포넌트 등록</h1>`
            }
        }
    })
</script>
```

Vue의 공식 스타일 가이드에서도 HTML에서는 항상 kebab-case를 사용하도록 권장하고 있다.

<br/>

## Vue Component 통신 방식

[뷰 컴포넌트](https://joshua1988.github.io/vue-camp/vue/components.html#%E1%84%8F%E1%85%A5%E1%86%B7%E1%84%91%E1%85%A9%E1%84%82%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3-%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC-%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3-%E1%84%92%E1%85%A7%E1%86%BC%E1%84%89%E1%85%B5%E1%86%A8)

뷰 컴포넌트는 각각 고유한 데이터 유효 범위를 갖습니다.  
따라서, 컴포넌트 간에 데이터를 주고 받기 위해선 아래와 같은 규칙을 따라야 합니다.

- 상위에서 하위로는 데이터를 내려줌 (props)  
- 하위에서 상위로는 이벤트를 올려줌 (event)

<br/>

## Vue Component Props

프롭스 속성은 컴포넌트 간에 데이터를 전달할 수 있는 컴포넌트 통신 방법.  
프롭스 속성을 기억할 때는 상위 컴포넌트에서 하위 컴포넌트로 내려보내는 데이터 속성으로 기억.

```html
<div id="app">
  <app-header v-bind:title="appTitle"></app-header>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  Vue.createApp({
    data() {
      return {
        appTitle: "props 전달",
      };
    },
    components: {
      "app-header": {
        template: "<h1>{{ title }}</h1>",
        props: ["title"],
      },
    },
  }).mount("#app");
</script>
```

<br/>

## Event Emit 소개 / Event Emit 구현

```html
<div id="app">
  <p>갱신 횟수: {{ refreshCount }}</p>
  <app-contents @refresh="handleRefresh"></app-contents>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  const appContents = {
    template: `
                <p>
                    <button v-on:click="sendEvent">갱신</button>
                </p>
            `,
    methods: {
      sendEvent() {
        this.$emit("refresh"); // 상위 컴포넌트에 이벤트 발생
      },
    },
  };

  // root
  Vue.createApp({
    data() {
      return {
        refreshCount: 0,
      };
    },
    methods: {
      handleRefresh() {
        this.refreshCount++;
        alert("부모 컴포넌트에서 이벤트 받음. 갱신 횟수: " + this.refreshCount);
      },
    },
    components: {
      "app-contents": appContents,
    },
  }).mount("#app");
</script>
```

<br/>

## 같은 레벨의 컴포넌트간 데이터 전달 방법

같은 레벨의 컴포넌트간 직접적으로 통신할 수 없다.  
하위로 컴포넌트의 깊이가 깊어질 수록 문제를 마주할 가능성이 높아진다.

```html
<div id="app">
  <!-- view 내부적으로 kebab-case로 처리 필요 (=appTitle과 연결) -->
  <app-header :app-title="message"></app-header> <!-- props -->
  <app-contents @login="receive"></app-contents> <!-- event -->
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  const appHeader = {
    props: ["appTitle"],
    template: "<h1>{{ appTitle }}</h1>",
  };

  const appContents = {
    template: `
      <p>
        <button @click="sendEvent">로그인</button>
      </p>
    `,
    methods: {
      sendEvent() {
        this.$emit("login");
      },
    },
  };

  // root
  Vue.createApp({
    data() {
      return {
        message: "",
      };
    },
    methods: {
      receive() {
        this.message = "로그인 성공";
      },
    },
    components: {
      "app-header": appHeader,
      "app-contents": appContents,
    },
  }).mount("#app");
</script>
```

<br/>
