# 구현에서 자주 사용되는 코드

## 바로가기

[deque](#deque)<br/>
[dictionary](#dictionary)<br/>
[heap](#heap)<br/>
[list](#list)<br/>
[number](#number)<br/>
[sort](#sort)<br/>
[반복문](#반복문)


<br/>
<br/>

## deque

> deque의 인자를 index와 함께 구성하는 방식

```python
from collections import deque

# 일반적
queue = deque(enumerate(List))

# 순서를 바꾸기
queue = deque([(v, i) for i, v in enumerate(List)])
```

<br/>
<br/>

## dictionary

> dictionary 내 values 할당 (최초 할당까지 커버)

```python
dic = {}

# List
dic.setdefault(key, []).append(item)

# Number
dic[key] = dic.get(key, 0) + 1
```

<br/>

> dictionary 인자 추출 3종

```python
dic = {}

dic.keys()
dic.values()
dic.items()
```

<br/>
<br/>

## heap

> heapq 이진트리 정렬에서 list와 tuple에 대한 정렬 기준

```python
import heap

original_lsit = [[0, 1], [2, 3], [1, 2], [2, 3], [3, 4]]

heap = []
for list in original_lsit:
    heapq.heappush(heap, list)

result = []
while heap:
    result.append(heapq.heappop(heap))

print(result)   # [[0, 1], [1, 2], [2, 3], [2, 3], [3, 4]]
```

앞의 인덱스부터 오름차순 기준으로 정렬

<br/>
<br/>

## list

> default List 선언

```python
from collections import deque

## 하나의 값으로 설정
list = [0] * len(parameter)
queue = deque([0] * len(parameter)) # 동일

## 임의의 값으로 간격을 두는 설정
n = 10 # 리스트의 길이
start = 1 # 시작 값
step = 2 # 차이
arithmetic_seq_by_comprehension = [start + i * step for i in range(n)]
arithmetic_seq_by_map = list(map(lambda x: start + x * step, range(n)))
```

<br/>

> 배열 뒤집기

```
reverse
- list 원본을 변경

reversed
- list 원본의 변경 없이 할당으로 사용 (추가 메모리를 사용)
```

<br/>

> 리스트 결합

```python
numbers = ['6', '2', '10']
''.join(numbers) # '6210'
```

<br/>
<br/>

## number

> '000'을 0으로 만들기

```python
return str(int(answer))
```

<br/>
<br/>

## sort

> sort와 sorted의 차이

```
sort
- 오직 list 만을 대상으로 동작
- list 원본을 변경 후 None을 반환

sorted
- 모든 이터러블에 사용이 가능
- list 원본의 변경 없이 할당으로 사용 (추가 메모리를 사용)
```

<br/>

> 정렬의 조건 지정

```
기본적으로 sorted_list = sorted(list, key=?) 형태로 사용

- 지정자
len, abs, min, max, sum, int, str.lower, str.upper,
operator.itemgetter, operator.attrgetter

- lambda
lambda x: (-x[1], x[0])
앞의 인자부터 정렬 우선순위를 가지며, 음수는 내림차순으로 정렬.
```

<br/>

> 중복을 제거하는 정렬

```python
sorted_unique_list = sorted(set(original_list))
```

<br/>
<br/>

## 반복문

> for문 내 index를 접근하기 위한 표현식

```python
for index, value in enumerate(list):
```
