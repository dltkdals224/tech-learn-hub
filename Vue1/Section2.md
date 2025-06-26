# Vue.js 핵심 동작 원리

## Vue 3 Reactivity - Proxy 소개

```html
<!-- reactivity.html -->
<div id="app"></div>

<script>
    const data = {
        a: 10
    };

    const app = new Proxy(data, {
        get() {
            console.log('값 접근');
        }
        set() {
            console.log('값 갱신');
        }
    })
</script>
```

Proxy를 통해 data 객체를 모방한 뒤 동작을 추가하는 개념.

<br/>

## Vue 3 Reactivity - 동작 원리 구현

```html
<!-- reactivity.html -->
<div id="app">
    <!-- 렌더링 -->
</div>

<script>
    const data = {
        message: 10
    };

    function render(sth) {
        const div = document.querySelector('#app');
        div.innterHTML = sth;
    }

    const app = new Proxy(data, {
        get() {
            console.log('값 접근');
        }
        set(target, prop, newValue) {
            console.log('값 갱신');
            target[prop] = newValue;
            render(newValue);
        }
    })
</script>
```

<br/>

## Reactivity 차이점 - Vue 2 & Vue 3

[Vue.js 공식 문서](https://v3-docs.vuejs-korea.org/guide/quick-start.html)내 Reactivity Fundamentals(반응형 기초) 읽어보는 것을 추천

Vue2: Object.defineProperty()  
Vue3: Proxy

Proxy는 객체 자체를 mocking하여 다루기때문에,  
사전에 완벽히 정의되지 않아도 문제없이 값에 대한 접근과 수정이 가능하다.

<br/>