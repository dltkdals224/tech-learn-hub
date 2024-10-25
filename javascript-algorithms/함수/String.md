## chartAt(index)

문자열에서 특정 위치에 있는 문자를 반환

```javascript
let str = "Hello";

console.log(str.charAt(1)); // "e"
```

<br/>

## concat(str [, ...strN])

두 개 이상의 문자열을 결합

```javascript
let str1 = "Hello";
let str2 = "World";

console.log(str1.concat(" ", str2)); // "Hello World"
```

<br/>

## includes(searchString [, position])

문자열에 특정 문자열이 포함되어 있는지 확인

```javascript
let str = "Hello World";

console.log(str.includes("World")); // true
console.log(str.includes("World", 7)); // false
console.log(str.includes("world")); // false
```

<br/>

## startsWith(searchString [, length]), endsWith(searchString [, length])

어떤 문자열이 임의의 문자로 시작하는지, 혹은 끝나는지를 확인하여 결과를 리턴

```javascript
let str = "Hello World";

console.log(str.startsWith('llo')); // false
console.log(str.startsWith('llo', 2)); // true

console.log(str.endsWith('Worl')); // false
console.log(str.endsWith('Worl',str.length - 1)); // true
```

<br/>

## indexOf(searchValue[, fromIndex]), lastIndexOf(searchValue[, fromIndex])

특정 위치부터 탐색하여 특정 문자열의 첫 번째 (인덱스)위치를 반환 <br/>

```javascript
let str = "Hello World";

console.log(str.indexOf("World")); // 6
```

<br/>

## padStart(targetLength [, padString]), padEnd(targetLength [, padString])

각각 문자열의 시작 혹은 끝을 다른 문자열로 채워, 주어진 길이를 만족하는 새로운 문자열을 반환

```javascript
let str = "5";

console.log(str.padStart(3, "0")); // "005"
console.log(str.padEnd(3, "0")); // "500"
```

<br/>

## repeat(count)

문자열을 주어진 횟수만큼 반복해 붙인 새로운 문자열을 반환

```javascript
let str = "abc";

"abc".repeat(2); // "abcabc"
```

<br/>

## replace(searchFor, replaceWith), replaceAll(searchFor, replaceWith)

패턴에 일치하는 대상을 치환하여 새로운 패턴이 들어간 문자열을 반환 <br/>
원본 문자열은 변환되지 않는다.

```javascript
const str =  "apple, Banana, banana, orange, banana";

console.log(str.replace("banana", "mango")); // "apple, Banana, mango, orange, banana"
console.log(str.replace(/banana/g, "mango")); // "apple, Banana, mango, orange, mango"
console.log(str.replace(/banana/gi, "mango")); // "apple, mango, mango, orange, mango"

console.log(str.replaceAll("banana", "mango")); // "apple, Banana, mango, orange, mango"

// searchFor pattern이 빈 문자열인 경우 문자열의 시작 부분에 대체 문자열이 추가
"abc".replace("", "_"); // "_abc"
```

<br/>

## slice(beginIndex [, endIndex])

문자열의 일부를 반환 <br/>
(substring()과 기능적으로 동일하게 동작하지만, 음수 인덱스 처리에서 차이가 존재한다.)

```javascript
let str = "Hello World";

console.log(str.slice(0, 5)); // "Hello"
```

<br/>

## split([sep [, limit]])

문자열을 인자에 해당하는 문자를 기준으로 배열 분할

```javascript
let str = "Hello World";

console.log(str.split(" ")); // ["Hello", "World"]
console.log(str.split(" "), 1); // ["Hello"] // 배열 최대길이 제한
console.log(str.split("")); // ["H", "e", "l", "l", "o", " ", "W", "o", "r", "l", "d"]

```

<br/>

## toLowerCase(), toUpperCase()

각각 문자열을 소문자와 대문자로 변환

```javascript
let str = "Hello World";

console.log(str.toLowerCase()); // "hello world"
console.log(str.toUpperCase()); // "HELLO WORLD"
```

<br/>

## toString()

객체의 문자열 표현을 반환

```javascript
const stringObj = new String('foo');

console.log(stringObj); // String { "foo" }
console.log(stringObj.toString()); // "foo"
```

<br/>

## trim(), trimStart(), trimEnd()

문자열 끝의 공백을 제거하면서 원본 문자열을 수정하지 않고 새로운 문자열을 반환

```javascript
const greeting = '   Hello world!   ';

console.log(greeting.trim()); // "Hello world!"
console.log(greeting.trimStart()); // "Hello world!   "
console.log(greeting.trimEnd()); // "   Hello world!"
```

<br/>

## length

문자열의 길이를 반환

```javascript
let str = "Hello";

console.log(str.length); // 5
```

<br/>