# 강의 오리엔테이션

## 컴포넌트 생성 및 등록하기

일반적으로 src/components 를 두어 컴포넌트를 관리.  
작명은 PascalCase로 한다.

파일 내 vue 입력 + `tab`을 통해 single file component template를 불러올 수 있다.

```html
<template></template>

<script>
  export default {};
</script>

<style></style>
```

작성 이후 App.vue 파일에서 컴포넌트 등록을 다음과 같은 형태로 한다.

```html
<!-- App.vue -->
<template>
  <div id="app">
    <TodoHeader></TodoHeader>
  </div>
</template>

<script>
  import TodoHeader from "./components/TodoHeader.vue";
  // ...

  export default {
    components: {
      // 'TodoHeader': TodoHeader,
    },
  };
</script>

<style></style>
```

<br/>

## 파비콘, 아이콘, 폰트, 반응형 태그 설정하기

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <!-- favicon -->
    <link
      rel="shortcut icon"
      href="src/assets/favicon.ico"
      type="image/x-icon"
    />
    <link rel="icon" href="src/assets/favicon.ico" type="image/x-icon" />
    <!-- awesome font -->
    <link
      rel="stylesheet"
      href="https://use.fontawesome.com/releases/5.0.9/css/all.min.css"
      integrity="sha384-5SOiIsAziJl6AWe0HWRKTXlfcSHKmYV4RBF18PPJ173Kzn7jzMyFuTtk8JA7QQG1"
      crossorigin="anonymous"
    />
    <!-- google font -->
    <link
      href="https://fonts.googleapis.com/css?family=Ubuntu"
      rel="stylesheet"
    />
  </head>
  <body></body>
</html>
```

favicon generator을 통해 생성한 favicon.ico를 추가 (link 코드 포함)

font awesome > get started > Web Fonts with CSS의 link 코드 추가

google font ubuntu > 하단 탭의 link 코드 추가

<br/>

## TodoHeader 컴포넌트 구현

```html
<template>
  <header>
    <h1>TODO LIST</h1>
  </header>
</template>

<!-- scoped: 해당 컴포넌트 내부에서만 유효한 속성으로 지정 -->
<style scoped>
  h1 {
    color: #2F3852;
    font-size: 24px;
    font-weight: 900;
  }
</style>
```

<br/>

## TodoInput 컴포넌트의 할 일 저장 기능 구현

```html
<template>
  <header>
    <h1>TODO LIST</h1>
    <button v-on:click="addTodo">add</button>
  </header>
</template>

<script>
export default{
  data: function() {
    return {
      newTodoItem: ""
    }
  },
  methods: {
    addTodo: function() {
      localStorage.setItem('key', this.newTodoItem)  // 내장 함수
    }
  }
}
</script>

<style scoped>
/* ... */
</style>
```

<br/>

## TodoInput 컴포넌트 코드 정리 및 UI 스타일링  

생략

<br/>

## TodoList 컴포넌트의 할 일 목록 표시 기능 구현  

```html
<template>
  <div>
    <ul>
      <li v-for="todoItem in todoItems" v-bind:key="todoItem">
        {{todoItem}}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      todoItems: []
    }
  },
  created: function() {
    // for문 돌면서 storage value 가져오기
  }
}
</script>
```

> [vue의 lifecycle](https://ko.vuejs.org/guide/essentials/lifecycle).

> [컴포지션 API: 라이프사이클 훅](https://ko.vuejs.org/api/composition-api-lifecycle.html)


각 Vue 컴포넌트 인스턴스는 생성될 때 일련의 초기화 단계를 거친다.  
데이터 관찰 -> 템플릿 컴파일 -> DOM에 마운트 -> DOM 업데이트

- onMounted()

컴포넌트가 마운트된 후 호출될 콜백을 등록합니다.

이 훅은 일반적으로 컴포넌트의 렌더링된 DOM에 접근해야 하는 부수 효과를 수행하거나, 서버 렌더링 애플리케이션에서 DOM 관련 코드를 클라이언트로 제한할 때 사용됩니다.

- onUpdated()

반응형 상태(reactives state, props, computed 등) 변경으로 인해 컴포넌트의 DOM 트리가 업데이트된 후 호출될 콜백을 등록합니다.  
최초 렌더링(onMounted()가 수행되는 상황)에서는 살향되지 않습니다.

이 훅은 컴포넌트의 모든 DOM 업데이트 후에 호출됩니다.  
이는 여러 상태 변경이 성능상의 이유로 하나의 렌더 사이클로 묶일 수 있기 때문입니다.

- onUnmounted()

컴포넌트가 언마운트된 후 호출될 콜백을 등록합니다.  
말 그대로 "이 컴포넌트가 더 이상 필요 없어져서 화면(=DOM)에서 제거될 때" 호출되는 훅입니다.

이 훅은 타이머, DOM 이벤트 리스너, 서버 연결 등 수동으로 생성한 부수 효과를 정리할 때 사용하세요.

<br/>

- onBeforeMount()

컴포넌트가 마운트되기 직전에 호출될 훅을 등록합니다.

이 훅이 호출될 때, 컴포넌트는 반응형 상태 설정을 마쳤지만 아직 DOM 노드가 생성되지 않았습니다.  
곧 처음으로 DOM 렌더 효과를 실행할 예정입니다.

- onBeforeUpdate()

반응형 상태 변경으로 인해 컴포넌트의 DOM 트리가 업데이트되기 직전에 호출될 훅을 등록합니다.

이 훅은 Vue가 DOM을 업데이트하기 전에 DOM 상태에 접근할 때 사용할 수 있습니다. 이 훅 내에서 컴포넌트 상태를 변경해도 안전합니다.

- onBeforeUnmount()

컴포넌트 인스턴스가 언마운트되기 직전에 호출될 훅을 등록합니다.

이 훅이 호출될 때, 컴포넌트 인스턴스는 여전히 완전히 동작 가능한 상태입니다.

- onErrorCaptured()

하위 컴포넌트에서 전파된 오류가 포착되었을 때 호출될 훅을 등록합니다.

<br/>

## TodoList 컴포넌트 UI 스타일링  

<br/>

## TodoList 컴포넌트 할 일 삭제 기능 구현  

<br/>

## TodoList 컴포넌트의 할 일 완료 기능 구현  

<br/>

## TodoFooter 컴포넌트 구현  

<br/>

## 깃헙 브랜치 안내

<br/>