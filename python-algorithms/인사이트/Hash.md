# 해시

해시에 관한 내용이 아닌, 해시와 관련된 내용을 풀다가 깨닳은 내용을 정리. <br/>
따라서 인사이트 폴더의 글을 읽는 경우는 모든 파일의 내용을 훑어야 할 필요가 있음.

## 프로그래머스

### [전화번호 목록](https://school.programmers.co.kr/learn/courses/30/lessons/42577)

#### 아이디어

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

#### 구현방식

> dictionary 내 values 할당 방식 (최초 할당까지 커버)

```python
dic = {}

# List
dic.setdefault(key, []).append(item)

# Number
dic[key] = dic.get(key, 0) + 1
```

> dictionary 인자 추출 3종

```python
dic = {}

dic.keys()
dic.values()
dic.items()
```

#### 아이디어

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

<br/>
<br/>

### [베스트앨범](https://school.programmers.co.kr/learn/courses/30/lessons/42579)

#### 구현방식

> for문 내 index를 접근하기 위한 표현식

```python
for index, value in enumerate(list):
```

> sort와 sorted의 차이

```
sort
- 오직 list 만을 대상으로 동작
- list 원본을 변경 후 None을 반환

sorted
- 모든 이터러블에 사용이 가능
- list 원본의 변경 없이 할당으로 사용 (추가 메모리를 사용)

모두 외우는게 귀찮다면 그냥 sorted만 사용하는게 문제가 생길 상황이 적다.
```

> 정렬 조건

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
<br/>