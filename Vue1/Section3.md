# Vue.js 기본 개념과 문법

## Vue Instance

모든 Vue 앱은 createApp 함수를 사용하여 새로운 앱 인스턴스를 생성하는 것으로 시작.

```html
import { createApp } from 'vue'
<script>
const app = createApp({
    // 최상위 컴포넌트 옵션
})
</script>
```

```html
<div id="app"></div>

<script>
Vue.createApp({
        data() {
            return {
                message: '10'
        }
    }
}).mount('#app');
// 앱 인스턴스는 .mount() 메서드가 호출될 때까지 아무 것도 렌더링하지 않는다
</script>
```

<br/>

## Vue Methods

```typescript
Vue.createApp({
  template: ,
  data: ,
  methods: ,  // 이 중 methods를 알아본다.
  created: ,
  watch: ,
});
```

```html
<!-- html -->
<div id="app">
    <p>{{ count }}</p>
    <button v-on:click="addCount">+</button>
</div> 

<!-- javascript -->
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
const app = createApp({
    data() {
        return {
            count: 0
        }
    },
    methods: {
        addCount() {
            this.count++  // this <- createApp 내부 값에 접근
        }
    }
}).mount('#app');
</script>
```

<br/>

## Vue Directive: v-for

```html
<!-- html -->
<div id="app">
    <ul>
        <li v-for="item in items">
            {{ item }}
        </li>
    </ul>
</div>

<!-- javascript -->
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
const app = createApp({
    data() {
        return {
            items: [1, 2, 3]
        }
    },
}).mount('#app');
</script>
```

<br/>
