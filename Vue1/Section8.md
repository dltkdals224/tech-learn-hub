# Vue.js 최종 프로젝트

## 프로젝트 생성 및 로그인 폼 UI 구성

```html
<!-- App.vue -->
<template>
  <form action="">
    <div>
      <label for="email">email:</label>
      <input id="email" type="text" v-model="email" />
    </div>
    <div>
      <label for="password">password:</label>
      <input type="password" v-model="password" />
    </div>
    <button type="submit">로그인</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      email: "",
      password: "",
    };
  },
  methods: {},
};
</script>
```

- v-model

양방향 데이터 바인딩을 제공하는 Vue의 디렉티브(value + onChange).  
email이라는 데이터 속성과 \<input\>의 value가 연결.

사용자가 텍스트를 입력하면 email 값이 자동으로 업데이트되고,  
반대로 email 값을 코드에서 바꾸면 input의 값도 자동으로 갱신.

input 요소뿐 아니라 여러 form 요소, 커스텀 컴포넌트에서도 사용할 수 있다.

<br/>

## 폼 이벤트 제어 및 서버로 데이터 전송

```html
<template>
  <form action="" @submit.prevent="handleSubmit">
    <div>
      <label for="email">email:</label>
      <input id="email" type="text" v-model="email" />
    </div>
    <div>
      <label for="password">password:</label>
      <input type="password" v-model="password" />
    </div>
    <button type="submit">로그인</button>
  </form>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      email: "",
      password: "",
    };
  },
  methods: {
    handleSubmit() {
      const data = {
        email: this.email,
        password: this.password,
      };
      axios
        .post("https://jsonplaceholder.typicode.com/users/", data)
        .then((res) => {
          console.log(res);
        });
    },
  },
};
</script>
```

- @submit.prevent

```js
handleSubmit(event){
    event.preventDefault();
}
```
기능적으로 동일.  
form 제출 함수의 기본 동작(input value 초기화)를 막는다.

<br/>

## Vue Composition API 코드로 변환하기

```html
<template>
  <form action="" @submit.prevent="handleSubmit">
    <div>
      <label for="email">email:</label>
      <input id="email" type="text" v-model="email" />
    </div>
    <div>
      <label for="password">password:</label>
      <input type="password" v-model="password" />
    </div>
    <button type="submit">로그인</button>
  </form>
</template>

<script>
import axios from "axios";
import { ref } from "vue";

export default {
  name: "App",
  setup() {
    // data
    const email = ref("");
    const password = ref("");

    // methods
    const handleSubmit = () => {
      axios
        .post("https://jsonplaceholder.typicode.com/users/", {
          email: email.value, // composition api 특징
          password: password.value,
        })
        .then((res) => {
          console.log(res);
        });
    };

    return { email, password, handleSubmit }; // <template> 표현식에서 사용 가능
  },
};
</script>
```

<br/>