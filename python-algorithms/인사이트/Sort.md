# 정렬

## 프로그래머스

### [가장 큰 수](https://school.programmers.co.kr/learn/courses/30/lessons/42746)

#### 구현방식

> 리스트 결합

```python
numbers = ['6', '2', '10']
''.join(numbers) # '6210'
```

> 

#### 아이디어

> 문제를 해결하는 핵심 아이디어

```
numbers.sort(key=lambda x: x*3, reverse=True)의 key=lambda x: x*3 동작 방식.
문자열에 대한 * 연산이 3번 반복되는 값에 대한 비교로써 다음과 같이 동작.

"3" * 3 = "333"
"30" * 3 = "303030"
"333"이 "303030"보다 크므로, "3"이 "30"보다 앞에 오게됨.
```

> '000'을 0으로 만들기

```
return str(int(answer))
```