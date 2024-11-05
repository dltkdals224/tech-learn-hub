# 힙

## 프로그래머스

### [디크스 컨트롤러](https://school.programmers.co.kr/learn/courses/30/lessons/42627#)

#### 구현방식

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

#### 아이디어

> 문제를 접근하는 방식

```
문제의 핵심은 (작업가능한 대상 중)소요시간이 짧은순서를 우선 처리하는 것.
```


