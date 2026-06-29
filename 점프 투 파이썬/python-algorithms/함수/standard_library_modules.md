## 표준 라이브러리

### datetime

> datetime.date()

현재의 그레고리력이 앞으로도 적용될 것이라는 가정하에 이상적인 나이브 날짜. <br/>

```python
import datetime
day1 = datetime.date(2024, 1, 12)
day2 = datetime.date(2024, 10, 16)

diff = day2 - day1
print(diff.days) # 278
```
<br/>

> date.isoweekday() <br/>
> date.isocalendar() <br/>
> date.isoformat()

```python
import datetime
day = datetime.date(2024, 12, 25)

print(day.isoweekday()) # 3  # 수요일(index 1을 월요일로 표기)
print(day.isocalendar()) # datetime.IsoCalendarDate(year=2024, week=52, weekday=3)
print(day.isoformat()) # '2024-12-25'
```

<br/>
<br/>

### time

> time.time()

time.time()은 UTC(Universal Time Coordinated 협정 세계 표준시)를 사용하여 현재 시간을 실수 형태로 리턴하는 함수. <br/>
1970년 1월 1일 0시 0분 0초를 기준으로 지난 시간을 초 단위로 돌려준다.

```python
import time
print(time.time()) # 1729054773.6623979
```
<br/>

> time.localtime()

time.localtime은 time.time()이 리턴한 실수 값을 사용해서 연도, 월, 일, 시, 분, 초, ... 의 형태로 바꾸어 주는 함수이다.

```python
import time
print(time.localtime(time.time()))
# time.struct_time(tm_year=2024, tm_mon=10, tm_mday=16, tm_hour=14, tm_min=2, tm_sec=52, tm_wday=2, tm_yday=290, tm_isdst=0)
```
<br/>

> time.asctime()

time.localtime에 의해서 반환된 튜플 형태의 값을 인수로 받아서 날짜와 시간을 알아보기 쉬운 형태로 리턴하는 함수이다.

```python
import time
print(time.asctime(time.localtime(time.time()))) # Wed Oct 16 14:03:36 2024
```
<br/>

> time.ctime()

time.asctime(time.localtime(time.time()))은 time.ctime()을 사용해 간편하게 표시할 수 있다. <br/>
asctime과 다른 점은 ctime은 항상 현재 시간만을 리턴할 수 밖에 없다.

```python
import time
print(time.ctime()) # Wed Oct 16 14:04:55 2024
```

<br/>

### math

> math.gcd() <br/>
> math.lcm()

math.gcd 는 최대공약수(gcd, greatest common divisor)를 구할때 사용하는 함수이다. <br/>
math.lcm 은 최소공배수(lcm, least common multiple)를 구할때 사용하는 함수이다.

```python
import math
print(math.gcd(60, 100, 80)) # 20
print(math.lcm(15, 25)) # 75
```
<br/>

> math.ceil() <br/>
> math.floor()

```python
import math
print(math.ceil(3.5)) # 4
print(math.ceil(-3.5)) # -3
print(math.floor(3.5)) # 3
print(math.floor(-3.5)) # -4
```
<br/>

> math.pow(x,y)

x의 y승을 반환.

```python
import math
print(math.pow(3, 4)) # 81.0
```
<br/>

> math.comb(n, k) <br/>
> math.perm(n, k)

nCk 조합, nPk 순열을 계산.

```python
import math
print(math.comb(5, 2)) # 10(=5C2)
print(math.perm(5, 2)) # 20(=5P2)
```
<br/>

> math.factorial() <br/>
> math.sqrt() <br/>
> math.pi() <br/>
> math.e()

```python
import math
print(math.factorial(5)) # 120
print(math.sqrt(7)) # 2.6457513110645907
print(math.pi) # 3.141592653589793
print(math.e) # 2.718281828459045
```

<br/>
<br/>

### heapq

힙을 활용한 우선순위 큐 기능을 구현할 때 사용. <br/>
최소힙(오름차순 정렬)의 시간복잡도는 O(NlogN). <br/>
(파이썬의 힙은 최소힙이므로, 단순히 원소를 힙에 전부 넣었다가 빼는 것 만으로도 시간복잡도 O(NlogN)의 오름차순 정렬)

PriorityQueue 라이브러리가 따로 있지만, 코딩테스트 환경 내에서는 heapq가 더 빠르게 동작한다.

```
힙(heap)이란?

완전 이진 트리의 일종으로 우선순위 큐를 위하여 만들어진 자료구조.
여러 개의 값들 중에서 최댓값이나 최솟값을 빠르게 찾아내도록 만들어진 자료구조이다.

힙은 일종의 반정렬 상태(느슨한 정렬 상태) 를 유지한다.
- 큰 값이 상위 레벨에 있고 작은 값이 하위 레벨에 있다는 정도
- 간단히 말하면 부모 노드의 키 값이 자식 노드의 키 값보다 항상 큰(작은) 이진 트리를 말한다.
힙 트리에서는 중복된 값을 허용한다. (이진 탐색 트리에서는 중복된 값을 허용하지 않는다)

> 최대 힙(max heap)
부모 노드의 키 값이 자식 노드의 키 값보다 크거나 같은 완전 이진 트리
key(부모 노드) >= key(자식 노드)

> 최소 힙(min heap)
부모 노드의 키 값이 자식 노드의 키 값보다 작거나 같은 완전 이진 트리
key(부모 노드) <= key(자식 노드)
```
> heapq.heappush() <br/>
> heapq.heappop()

각각 힙에 원소를 삽입하고 꺼내는 함수.

```python
## 최소힙
import heapq

# 리스트 생성
heap = []

# 원소 추가
for value in [1, 3, 5, 7, 9, 2, 4, 6, 8, 0]:
    heapq.heappush(heap, value)

# 정렬된 결과 출력
result = []
while heap:
    result.append(heapq.heappop(heap))

print(result)  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

```python
## 최대힙
import heapq

# 리스트 생성
heap = []

# 원소 추가 (음수로 변환하여 추가)
for value in [1, 3, 5, 7, 9, 2, 4, 6, 8, 0]:
    heapq.heappush(heap, -value)

# 정렬된 결과 출력 (다시 양수로 변환)
result = []
while heap:
    result.append(-heapq.heappop(heap))

print(result)  # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

<br/>
<br/>

### bisect

'정렬된 배열'에서 특정한 원소를 찾아야할 때 효과적으로 사용됨.

> bisect_left(a, x[, lo=0, hi=len(a)]) <br/>
> bisect_right(a, x[, lo=0, hi=len(a)])

각각 정렬된 순서를 유지하면서 리스트 a에 데이터 x를 삽입할 가장 왼쪽 인덱스를 찾는 메서드, <br/>
정렬된 순서를 유지하도록 리스트 a에 데이터 x를 삽입할 가장 오른쪽 인덱스를 찾는 메서드. <br/>
(정렬된 순서는 `'오름차순'`에 대해서만 정상적으로 동작)

시간 복잡도는 모두 O(logN)이며, <br/>
lo와 hi는 각각 검색을 시작할 리스트의 최소 인덱스, 검색을 끝낼 리스트의 최대 인덱스.

```python
import bisect
a = [1, 2, 2, 2, 3, 4, 5]

print(bisect.bisect_left(a, 2)) # >>> 1
print(bisect.bisect_right(a, 2)) # >>> 4
```
<br/>

> bisect.insort_left(a, x[, lo=0, hi=len(a)]) <br/>
> bisect.insort_right(a, x[, lo=0, hi=len(a)])

bisect.bisect와 달리 실질적으로 x를 정렬된 배열에 삽입하는 함수. <br/>
(마찬가지로 정렬된 순서는 `'오름차순'`에 대해서만 정상적으로 동작)

삽입이 필요하기 때문에 시간 복잡도는 모두 O(n)

```python
import bisect
a = [1, 2, 2, 2, 3, 4, 5]

bisect.insort_right(a, 2)
print(a)  # 출력: [1, 2, 2, 2, 2, 3, 4, 5]
bisect.insort_left(a, 2)
print(a)  # 출력: [1, 2, 2, 2, 2, 2, 3, 4, 5]
```

bisect는 이차원 배열에 대해서도 적용이 가능하다. <br/>
자동적으로 첫 번째 요소를 기준으로 정렬하며, key를 통해 정렬 순서를 강제할 수도 있다.

```python
import bisect
alphabet = [['a', 1], ['c', 3], ['z', 26]]

bisect.insort_right(alphabet, ['y', 25], key=lambda x: x[1])
print(alphabet)
# [['a', 1], ['c', 3], ['y', 25], ['z', 26]]
```

<br/><br/>

### operator

> operator.itemgetter()

주로 sorted와 같은 함수의 key 매개변수에 적용하여 다양한 기준으로 정렬할 수 있도록 도와주는 모듈. <br/>
시퀀스(리스트, 튜플 등)나 딕셔너리에서 특정 인덱스나 키의 값을 추출하는 함수를 생성.

```python
import operator

# 리스트의 정렬
students = [
    ('John', 'A', 15),
    ('Jane', 'B', 12),
    ('Dave', 'B', 10)
]

sorted_by_grade_age = sorted(students, key=operator.itemgetter(1, 2))
print(sorted_by_grade_age) 
# [('John', 'A', 15), ('Dave', 'B', 10), ('Jane', 'B', 12)]

# 딕셔너리 리스트 정렬
people = [
    {'name': 'John', 'age': 30},
    {'name': 'Jane', 'age': 25},
    {'name': 'Bob', 'age': 35}
]

sorted_people = sorted(people, key=operator.itemgetter('age'))
print(sorted_people)
# 출력: [{'name': 'Jane', 'age': 25}, {'name': 'John', 'age': 30}, {'name': 'Bob', 'age': 35}]
```

<br/>
<br/>

### functools

> functools.reduce()

function을 반복 가능한 객체(iterable)의 요소에 왼쪽에서 오른쪽으로 누적 적용하며 누적합 구하는 함수.

```python
import functools

data = [1, 2, 3, 4, 5]
result_by_add = functools.reduce(lambda x, y: x + y, data)
result_by_mul = functools.reduce(lambda x, y: x * y, data)
print(result_by_add) # 15 <= (((((1+2)+3)+4)+5))
print(result_by_mul) # 120 <= (((((1*2)*3)*4)*5))
```
<br/>

> lru_cache()

함수의 결과를 캐싱하여 반복적인 계산을 피할 수 있다. <br/>
동적 프로그래밍 문제나 재귀 함수의 성능을 크게 향상시킬 수 있다.

```python
import functools

# 큰 maxsize는 더 많은 결과를 저장할 수 있지만 메모리 사용량이 증가.
@functools.lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

<br/>
<br/>

### itertools

> itertools.chain(\*iterables)

이터러블에서 다음 이터러블로 넘어가며 모든 이터러블이 소진될 때까지 이터레이터에 추가. <br/>
여러 시퀀스를 단일 시퀀스처럼 처리하는 데 사용.

```python
import itertools
print(list(itertools.chain('ABC', 'DEF')))  # ['A', 'B', 'C', 'D', 'E', 'F']
```
<br/>

> itertools.product(\*iterables, repeat=1)

입력 이터러블들(iterables)의 카테시안 곱을 계산.

대략 제너레이터 표현식에서의 중첩된 for-루프와 동등. <br/>
예를 들어, product(A, B)는 ((x,y) for x in A for y in B)와 같은 결과를 반환.

```python
import itertools
print(list(itertools.product('AB', '12')))  
# [('A', '1'), ('A', '2'), ('B', '1'), ('B', '2')]
```
<br/>

> itertools.permutations(iterable, r=None)

```python
import itertools
print(list(itertools.permutations('ABC', 2)))
# [('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]
```

순열을 직접 생성할 필요가 없고 순열의 개수만 필요한 상황이라면, math.perm(n,k) 사용.

<br/>

> itertools.combinations(iterable, r)

```python
import itertools
print(list(itertools.combinations('ABC', 2)))
# [('A', 'B'), ('A', 'C'), ('B', 'C')]
```

마찬가지로 조합을 생성하는게 아닌 조합의 개수만 필요한 상황이라면, math.comb(n,k) 사용.

<br/>

> itertools.combinations_with_replacement(iterable, r)

입력 iterable에서 요소의 길이 r 서브 시퀀스들을 반환하는데, 개별 요소를 두 번 이상 반복할 수 있다.

```python
from itertools
print(list(itertools.combinations_with_replacement('ABC', 2)))
# [('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]
```

<br/>
<br/>

### collections

> collections.deque([iterable[, maxlen]])

iterable의 데이터로 왼쪽에서 오른쪽으로 (append()를 사용해서) 초기화된 새 데크(deque) 객체를 반환. <br/>
iterable을 지정하지 않으면, 새 데크는 비어있는 상태.

데크는 스택과 큐를 일반화 한 것. <br/>
(이름은 “deck”이라고 발음하며 “double-ended queue”의 약자) <br/>
데크는 스레드 안전하고 메모리 효율적인 데크의 양쪽 끝에서의 추가(append)와 팝(pop)을 양쪽에서 거의 같은 O(1) 성능으로 지원.

list 객체는 유사한 연산을 지원하지만, 빠른 고정 길이 연산에 최적화되어 있으며, <br/>
하부 데이터 표현의 크기와 위치를 모두 변경하는 pop(0)과 insert(0, v) 연산에 대해 O(n) 메모리 이동 비용이 발생.

deque 객체는 다음 메서드를 지원:

- append(x)

    데크의 오른쪽에 x를 추가합니다.

- appendleft(x)

    데크의 왼쪽에 x를 추가합니다.

- clear()

    데크에서 모든 요소를 제거하고 길이가 0인 상태로 만듭니다.

- copy()

    데크의 얕은 복사본을 만듭니다.

- count(x)

    x 와 같은 데크 요소의 수를 셉니다.

- extend(iterable)

    iterable 인자에서 온 요소를 추가하여 데크의 오른쪽을 확장합니다.

- extendleft(iterable)

    iterable에서 온 요소를 추가하여 데크의 왼쪽을 확장합니다. 일련의 왼쪽 추가는 iterable 인자에 있는 요소의 순서를 뒤집는 결과를 줍니다.

- index(x[, start[, stop]])

    데크에 있는 x의 위치를 반환합니다. (인덱스 start 또는 그 이후, 그리고 인덱스 stop 이전). <br/>
    첫 번째 일치를 반환하거나 찾을 수 없으면 ValueError를 발생시킵니다.

- insert(i, x)

    x를 데크의 i 위치에 삽입. <br/>
    삽입으로 인해 제한된 길이의 데크가 maxlen 이상으로 커지면, IndexError가 발생합니다.

- pop()

    데크의 오른쪽에서 요소를 제거하고 반환합니다. 요소가 없으면, IndexError를 발생시킵니다.

- popleft()

    데크의 왼쪽에서 요소를 제거하고 반환합니다. 요소가 없으면, IndexError를 발생시킵니다.

- remove(value)

    value의 첫 번째 항목을 제거합니다. 찾을 수 없으면, ValueError를 발생시킵니다.

- reverse()

    데크의 요소들을 제자리에서 순서를 뒤집고 None을 반환합니다.

- rotate(n=1)

    데크를 n 단계 오른쪽으로 회전합니다. n이 음수이면, 왼쪽으로 회전합니다. <br/>
    데크가 비어 있지 않으면, 오른쪽으로 한 단계 회전하는 것은 d.appendleft(d.pop())과 동등하고, 왼쪽으로 한 단계 회전하는 것은 d.append(d.popleft())와 동등합니다.

- maxlen

    데크의 최대 크기 또는 제한이 없으면 None.

<br/>

> collections.Counter([iterable-or-mapping])

Counter는 해시 가능한 개체를 카운트하는 딕셔너리의 하위 클래스. <br/>
요소가 사전 키로 저장되고 요소의 개수가 사전 값으로 저장되는 컬렉션.

카운트는 0 또는 음수 카운트를 포함한 모든 정수 값이 될 수 있다. <br/>
Counter 클래스는 다른 언어의 bags 또는 multisets과 유사.

Counter 개체는 모든 딕셔너리에 사용할 수 있는 메서드 이외의 추가 메서드를 지원.

- elements()

    개수만큼 반복되는 요소에 대한 이터레이터를 반환합니다. <br/>
    요소는 처음 발견되는 순서대로 반환됩니다. <br/>
    요소의 개수가 1보다 작으면 elements()는 이를 무시합니다.

```python
c = Counter(a=4, b=2, c=0, d=-2)
sorted(c.elements())
for target in c.elements():
    print(target)  # ['a', 'a', 'a', 'a', 'b', 'b']
```

- most_common([n])

    n 개의 가장 흔한 요소와 그 개수를 가장 흔한 것부터 가장 적은 것 순으로 나열한 리스트를 반환. <br/>
    n이 생략되거나 None이면, most_common()은 계수기의 모든 요소를 반환.

    개수가 같은 요소는 처음 발견된 순서를 유지합니다:

```python
print(Counter('abracadabra').most_common(3))  # [('a', 5), ('b', 2), ('r', 2)]
```

- subtract([iterable-or-mapping])

    이터러블이나 다른 매핑 (또는 계수기)으로부터 온 요소들을 뺍니다. <br/>
    dict.update()와 비슷하지만 교체하는 대신 개수를 뺍니다. 입력과 출력 모두 0이나 음수일 수 있습니다.

```python
c = Counter(a=4, b=2, c=0, d=-2)
d = Counter(a=1, b=2, c=3, d=4)
c.subtract(d)
print(c)  # Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6})
```

- total()

    카운트의 합을 계산합니다.
    일반적인 딕셔너리 메서드를 Counter 객체에 사용할 수 있습니다만, 두 메서드는 계수기에서 다르게 동작합니다.

```python
c = Counter(a=10, b=5, c=0)
print(c.total())  # 15
```

- update([iterable-or-mapping])

    요소는 이터러블에서 세거나 다른 매핑(또는 계수기)에서 더해집니다. <br/>
    dict.update()와 비슷하지만, 교체하는 대신 더합니다. <br/>
    또한, 이터러블은 (key, value) 쌍의 시퀀스가 아닌, 요소의 시퀀스일 것으로 기대합니다.

<br/>
