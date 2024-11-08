# 문제해결 아이디어

## 바로가기

[Hash](#hash)

[Queue, Stack](#queue-stack)

[Heap](#heap)

[Sort](#sort)

[BFS](#bfs)

<br/>
<br/>

## Hash

### [전화번호 목록](https://school.programmers.co.kr/learn/courses/30/lessons/42577)

> 탐색이 필요한 대상을 dic의 key로 설정하여 big-O를 최소화

```
Q)
전화번호부에 적힌 전화번호 중, 한 번호가 다른 번호의 접두어인 경우가 있는지 확인.

A)
전화번호부 리스트를 순회하며 dictionary의 key에 할당.
다시 전화번호부 리스트를 순회하며 별도의 문자열 조작 없이 dicionary에 조회(O(1)).

ex)
for number in phone_book:
    dic[number] = -1
    
for number in phone_book:
    for s in range(len(number)):
        if(number[:s] in dic):
```

<br/>

### [의상](https://school.programmers.co.kr/learn/courses/30/lessons/42578)

> (key를 기준으로 value의 중복 선택을 막는) 멱집합

```
Q)
각 키별로 최대 1가지 value만 선택할 수 있습니다.
선택한 value의 일부가 중복되더라도,
다른 value가 겹치지 않거나 value를 추가로 더 선택한 경우에는 서로 다른 집합으로 간주합니다.
최소 한 개의 value는 선택되어야 합니다.

A)
각 (values + 1)의 모든 곱 - 1

ex)
얼굴	동그란 안경, 검정 선글라스
상의	파란색 티셔츠
하의	청바지
겉옷	긴 코트

(2+1) * (1+1) * (1+1) * (1+1) - 1 = 23
```

> 임의의 리스트에 대한 멱집합 구하는 방법

파이썬에서 임의의 리스트에 대한 멱집합(power set)을 구하는 가장 효율적인 방법은 비트 연산을 이용하는 것. <br/>
(단순한 멱집합의 개수는 2^(인자의 개수))

시간 복잡도는 O(2^n). (n은 입력 리스트의 길이)

```python
def powerset(lst):
    result = []
    n = len(lst)
    for i in range(1 << n):
        subset = [lst[j] for j in range(n) if (i & (1 << j))]
        result.append(subset)
    return result
```

<br/>
<br/>

## Queue, Stack

### [다리를 지나는 트럭](https://school.programmers.co.kr/learn/courses/30/lessons/42583)

> 문제를 접근하는 방식

```
truck_weights의 length가 < 10000으로 제한되어있기 때문에,
reversed와 같은 방식으로 추출하는 방식에 신경써야할 이유가 없었다. (그냥 pop(0)때리면 됨)

보다 중요한 것은 2중 if문의 구조 설정 방식.
```

<br/>
<br/>

## Heap

### [디크스 컨트롤러](https://school.programmers.co.kr/learn/courses/30/lessons/42627#)

> 문제를 접근하는 방식

```
문제의 핵심은 (작업가능한 대상 중)소요시간이 짧은순서를 우선 처리하는 것.
```

<br/>
<br/>

## Sort

### [가장 큰 수](https://school.programmers.co.kr/learn/courses/30/lessons/42746)

> 문제를 해결하는 핵심 아이디어

```
numbers.sort(key=lambda x: x*3, reverse=True)의 key=lambda x: x*3 동작 방식.
문자열에 대한 * 연산이 3번 반복되는 값에 대한 비교로써 다음과 같이 동작.

"3" * 3 = "333"
"30" * 3 = "303030"
"333"이 "303030"보다 크므로, "3"이 "30"보다 앞에 오게됨.
```

<br/>
<br/>

## BFS

### [소수 찾기](https://school.programmers.co.kr/learn/courses/30/lessons/42839?language=python3)

> 임의의 숫자가 소수인지 판별하는 방법

모든 합성수 n은 √n보다 작거나 같은 약수를 반드시 갖는다. <br/>
√n까지만 확인하여, 검사해야 할 숫자의 범위를 줄일 수 있다.

시간 복잡도는 O(√n)

```python
import math
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0:
            return False
    return True
```

> 임의의 수가 가지는 모든 약수 알아내는 방법

```python
import math

def find_divisors(n):
    divisors = set() # 중복인자 방지
    for i in range(1, int(math.sqrt(n)) + 1):
        if n % i == 0:
            divisors.add(i)
            divisors.add(n // i)
    return sorted(divisors)
```

> 임의의 숫자 리스트에서 가능한 모든 숫자 조합을 만들어내는 방법

set()으로 중복을 방지(add로 추가되는 것들에도 모두 적용).

```python
from itertools import permutations

def generate_combinations(nums):
    result = set()
    for i in range(1, len(nums) + 1):
        for p in permutations(nums, i):
            result.add(int(''.join(map(str, p))))
    return sorted(result)
```

<br/>

### [피로도](https://school.programmers.co.kr/learn/courses/30/lessons/87946)

> [DP를 사용하는 풀이](https://school.programmers.co.kr/questions/52569)

(나중에 작성)

<br/>
<br/>
