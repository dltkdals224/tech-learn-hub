## 내장 함수

### abs(x)

abs(x)는 어떤 숫자를 입력받았을 때, 그 숫자의 절댓값을 반환하는 함수.

```python
abs(3) # 3
abs(-3) # 3
```

<br/>

### all(iterable)

반복 가능한(iterable) 데이터를 입력 값으로 받으며 이 요소가 모두 참이면 True, 거짓이 하나라도 있으면 False를 반환한다.

```python
all([1, 2, 3]) # True

# 입력 인수가 빈 값인 경우에는 True를 반환
all([]) # True
all([[]]) # False
```

<br/>

### any(iterable)

반복 가능한(iterable) 데이터를 입력으로 받아 하나라도 참이 있으면 True를, 모두 거짓일 때 False를 반환한다.

```python
any([1, [], False, 0]) # True
```

<br/>

### bin(x)

정수를 "0b"로 접두사가 붙은 이진 문자열로 변환한다. 
x가 파이썬 int 개체가 아닌 경우, 정수를 반환하는 __index__() 메서드를 정의해야 한다.

```python
bin(3) #'0b11'
bin(-10) # '-0b1010'
```

<br/>

### chr(i)

유니코드 코드 포인트가 정수 i 인 문자를 나타내는 문자열을 반환한다. <br/>
인자의 유효 범위는 0에서 1,114,111(16진수로 0x10FFFF)까지.

```python
chr(97) # 'a'
chr(44032) # '가'
```

<br/>

### divmod(a, b)

divmod(a, b)는 2개의 숫자를 입력으로 받는다. 그리고 a를 b로 나눈 몫과 나머지를 튜플로 반환하는 함수. <br/>
몫을 구하는 연산자 //와 나머지를 구하는 연산자 %를 각각 사용한 결과로 구성된 것.

```python
result = divmod(17, 5)
print(result)  # (3, 2)

quotient, remainder = divmod(20, 3)
print(f"몫: {quotient}, 나머지: {remainder}")  # 몫: 6, 나머지: 2
```

<br/>

### enumerate(iterable, start=0)

이 함수는 순서가 있는 데이터(리스트, 튜플, 문자열)를 입력으로 받아 인덱스 값을 포함하는 enumerate 객체를 반환한다.

```python
seasons = ['Spring', 'Summer', 'Fall', 'Winter']
list(enumerate(seasons)) # [(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
list(enumerate(seasons, start=1)) # [(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
```

<br/>

### eval(source, /, globals=None, locals=None)

문자열로 구성된 표현식을 입력으로 받아, 해당 문자열을 실행한 결괏값을 반환하는 함수이다.

```python
eval('1+2') # 3
eval("'hi' + 'a'") # 'hia'
```

<br/>

### filter(function, iterable)

filter 함수는 첫 번째 인수로 함수를, 두 번째 인수로 그 함수에 차례로 들어갈 반복 가능한 데이터를 받는다. <br/>
그리고 반복 가능한 데이터(iterable)의 요소 순서대로 함수를 호출했을 때 반환 값이 참인 것만 걸러내 반환한다.

```python
def positive(x):
    return x > 0

print(list(filter(positive, [1, -3, 5, 0, -5, 2]))) # [1, 5, 2]
```

<br/>

### getattr(object, name[, default])

object에 존재하는 속성의 값을 가져온다. <br/>
dictionary는 object가 아님.

```python
class MyClass:
    def __init__(self):
        self.a = 1
obj = MyClass()

value = getattr(obj, 'a')
value2 = getattr(obj, 'b', 100)

print(value) # 1
print(value2) # 100
```

<br/>

### hasattr(object, name)

인자는 객체와 문자열. 문자열이 객체의 속성 중 하나의 이름이면 결과는 True 이고, 그렇지 않으면 False. <br/>
getattr(object, name) 을 호출하고 AttributeError를 발생시키는지를 보는 식으로 구현.

마찬가지로, dictionary는 object가 아님.

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def greet(self):
        print(f"Hello, my name is {self.name}")

person1 = Person("Alice", 30)

print(hasattr(person1, 'name'))    # True
print(hasattr(person1, 'address')) # False
```

<br/>

### hex(x)

hex(x)는 정수를 입력받아 16진수(hexadecimal) 문자열로 변환하여 반환하는 함수이다.

```python
hex(234) # '0xea'
hex(3) # '0x3'
```

<br/>

### int(string, /, base=10)

int(x)는 문자열 형태의 숫자나 소수점이 있는 숫자를 정수로 반환하는 함수이다. 만약 정수가 입력되면 그대로 반환한다. <br/>
int(x, radix)는 radix 진수로 표현된 문자열 x를 10진수로 변환하여 반환한다.

```python
int('3') # 3
int(3.4) # 3

int(123.45) # 123
int('123') # 123
int('   -12_345\n') # -12345
int('FACE', 16) # 64206
int('0xface', 0) # 64206
int('01110011', base=2) # 115

# 변환 결과 기준은 무조건 10진수
int('11', 2) # 3
int('1A', 16) # 26
```

<br/>

### len(s)

len(s)은 입력값 s의 길이(요소의 전체 개수)를 반환하는 함수. <br/>
시퀀스 (문자열, 바이트열, 튜플, 리스트 또는 range 같은) 또는 컬렉션 (딕셔너리, 집합 또는 불변 집합 같은) 일 수 있다.

```python
len("python") # 6
len([1,2,3]) # 3
len((1, 'a')) # 2
```

<br/>

### list(iterable)

list(iterable)는 반복 가능한 데이터(iterable)를 입력받아 리스트로 만들어 반환하는 함수이다.

```python
list('python') # ['p', 'y', 't', 'h', 'o', 'n']
list((1,2,3)) # [1, 2, 3]
```

<br/>

### map(function, iterable, *iterables)

map(f, iterable)은 함수(f)와 반복 가능한 데이터를 입력으로 받는다. <br/>
map 함수는 입력받은 데이터의 각 요소에 함수 f를 적용한 결과를 반환하는 함수.

```python
def two_times(x):
    return x * 2 

list(map(two_times, [1, 2, 3, 4])) # [2, 4, 6, 8]
```

<br/>

### max(iterable, *, default, key=None)

인수로 반복 가능한 데이터를 입력받아 그 최댓값을 반환하는 함수.

default: 제공된 iterable이 비어있는 경우 돌려줄 객체를 지정. <br/>
iterable이 비어 있고 default 가 제공되지 않으면 ValueError 발생. <br/>
key 인자는 list.sort() 에 사용되는 것처럼 단일 인자 순서 함수를 지정.

```python
max([1, 2 ,3]) # 3
max('python') # y
```

<br/>

### min(iterable, *, default, key=None)

인수로 반복 가능한 데이터를 입력받아 그 최솟값을 반환하는 함수. <br/>
인자와 관련한 자세한 설명은 max를 참조.

```python
min([1, 2, 3]) # 1
min('python') # y
```

<br/>

### oct(x)

oct(x)는 정수를 8진수 문자열로 바꾸어 반환하는 함수.

```python
oct(34) # 0o42
oct(-56) # '-0o70'
```

<br/>

### ord(c)

ord(c)는 문자의 유니코드 숫자 값을 반환하는 함수. (chr(i)와 반대)

```python
ord('a') # 97
ord('가') # 44032
```

<br/>

### pow(base, exp, mod=None)

base의 exp 거듭제곱을 반환; <br/>
mod가 있는 경우, base의 exp 거듭제곱의 모듈로 mod를 반환. <br/>
(pow(base, exp) % mod) 보다 더 빠르게 계산됨.

```python
pow(2, 4) # 16
pow(1.5, 3) # 3.375

pow(38, -1, mod=97) # 23
# (38 * 23) % 97 = 1 임을 의미.
```

<br/>

### class range(start, stop, step=1)

for문과 함께 자주 사용하는 함수로, 입력받은 숫자에 해당하는 범위 값을 반복 가능한 객체로 만들어 반환.

> 인수가 1개

```python
list(range(5)) # [1, 2, 3, 4, 5]
```

> 인수가 2개

```python
list(range(5, 10)) # [5, 6, 7, 8, 9]
```

> 인수가 3개

```python
list(range(1, 10, 2)) # [1, 3, 5, 7 ,9]
list(range(1, 10, 3)) # [1, 4, 7]

list(range(0, -10, -1)) # [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
```

<br/>

### reversed(seq)

reversed() 함수는 Python의 내장 함수로, 주어진 시퀀스를 역순으로 순회할 수 있는 이터레이터를 반환. <br/>

```python
my_list = [1, 2, 3, 4, 5]
for item in reversed(my_list):
    print(item)  # 5, 4, 3, 2, 1 순서

my_text = "Hello"
reversed_text = ''.join(reversed(my_text))
print(reversed_text)  # "olleH"
```

<br/>

### round(number, ndigits=None)

round(number[,ndigits]) 함수는 숫자를 입력받아 반올림해 반환하는 함수. <br/>
[,ndigits] 인자는 없을 수도 있다.

```python
round(4.6) # 5
round(4.2) # 4

round(5.678, 2) # 5.68
```

<br/>

### slice(start, stop, step=None)

시퀀스(문자열, 리스트, 튜플 등)를 슬라이싱하는 데 사용되는 슬라이스 객체를 생성.

start: 슬라이싱 시작 인덱스 (선택적, 기본값 None) <br/>
stop: 슬라이싱 종료 인덱스 (필수) <br/>
step: 슬라이싱 간격 (선택적, 기본값 None)

```python
s = slice(1, 5, 2)

my_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(my_list[s])  # [1, 3]
```
<br/>

### sorted(iterable, /, *, key=None, reverse=False)

sorted(iterable) 함수는 입력 데이터를 정렬한 후 그 결과를 리스트로 반환하는 함수. <br/>
두 번째 인자에 key를 두어 list.sort()와 마찬가지로 특정 대상 정렬 기준을 삼을 수 있다.

리스트 자료형의 sort 함수와 달리 객체 자체에 대한 정렬 값을 실제로 반환한다. <br/>
(sort는 원본 배열을 변경하며 정렬하지만 None를 반환)

- `key = ???` 에서의 지정자

len, abs, min, max, sum, int, str.lower, str.upper,
operator.itemgetter, operator.attrgetter

- `key = ???` 에서의 lambda

lambda x: (-x[1], x[0])
앞의 인자부터 정렬 우선순위를 가지며, 음수는 내림차순으로 정렬.

```python
sorted([3, 1, 2]) # [1, 2, 3]
sorted([3, 1, 2], reverse=True) # [3, 2, 1]

sorted(['a', 'c', 'b'])
```

<br/>

### class str(object=b'', encoding='utf-8', errors='strict')

str(object)은 문자열 형태로 객체를 변환하여 반환하는 함수.

```python
str(3) # '3'
str('hi') # 'hi'
```

<br/>

### sum(iterable, /, start=0)

start 및 iterable 의 항목들을 왼쪽에서 오른쪽으로 합하고 합계를 반환. <br/>
iterable 의 항목은 일반적으로 숫자며 시작 값은 문자열이 될 수 없다.

```python
sum([1, 2, 3]) # 6
sum(['hi', 'everyone']) # error
```

<br/>

### tuple(iterable)

tuple(iterable)은 반복 가능한 데이터를 튜플로 바꾸어 반환하는 함수. 만약 입력이 튜플인 경우에는 그대로 반환.

```python
tuple("abc") # ('a', 'b', 'c')
tuple([1, 2, 3]) # (1, 2, 3)
```

<br/>

### type(name, bases, dict, **kwds)

type(object)은 입력값의 자료형이 무엇인지 알려 주는 함수.

```python
type("abc") # <class 'str'>
type([ ]) # <class 'list'>
type(open("test", 'w')) # <class '_io.TextIOWrapper'>
```

<br/>

### zip(*iterables, strict=False)

zip(\*iterable)은 동일한 개수로 이루어진 데이터들을 묶어서 반환하는 함수.

```python
list(zip([1, 2, 3], [4, 5, 6])) # [(1, 4), (2, 5), (3, 6)]
list(zip([1, 2, 3], [4, 5, 6], [7, 8, 9])) # [(1, 4, 7), (2, 5, 8), (3, 6, 9)]
list(zip("abc", "def")) # [('a', 'd'), ('b', 'e'), ('c', 'f')]
```
