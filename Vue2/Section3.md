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
  //   ...

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
