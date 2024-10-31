# 큐, 스택

해시에 관한 내용이 아닌, 해시와 관련된 내용을 풀다가 깨닳은 내용을 정리. <br/>
따라서 인사이트 폴더의 글을 읽는 경우는 모든 파일의 내용을 훑어야 할 필요가 있음.

## 프로그래머스

### [프로세스](https://school.programmers.co.kr/learn/courses/30/lessons/42587)

#### 구현방식

> deque의 인자를 index와 함께 구성하는 방식

```python
from collections import deque

# 일반적
queue = deque(enumerate(List))

# 순서를 바꾸기
queue = deque([(v, i) for i, v in enumerate(List)])
```

> 중복을 제거하는 정렬

```python
sorted_unique_list = sorted(set(original_list))
```

#### 아이디어

> 문제를 접근하는 방식

```
deque을 통해 요구사항대로 구현했을 경우 big-O는 O(n^2).
priorities가 [1, 2, 3, 4, 5]일 때를 생각해보면 이해할 수 있다.
```

<br/>

### [다리를 지나는 트럭](https://school.programmers.co.kr/learn/courses/30/lessons/42583)

#### 구현방식

> default List 선언

```python
from collections import deque

list = [0] * len(parameter)
queue = deque([0] * len(parameter)) # 동일
```

#### 동작 원리

> 배열 뒤집기

```
reverse
- list 원본을 변경

reversed
- list 원본의 변경 없이 할당으로 사용 (추가 메모리를 사용)

모두 외우는게 귀찮다면 그냥 sorted만 사용하는게 문제가 생길 상황이 적다.
ex) reversed_list = reversed(original_list)
```

#### 아이디어

> 문제를 접근하는 방식

```
truck_weights의 length가 < 10000으로 제한되어있기 때문에,
reversed와 같은 방식으로 추출하는 방식에 신경써야할 이유가 없었다. (그냥 pop(0)때리면 됨)

보다 중요한 것은 2중 if문의 구조 설정 방식.
```
