[TOC]

## Map, Filter, Reduce

##### 1. Map

* 여러 개의 데이터를 대상으로 한 번에 특정 함수를 적용하기 위해 사용
* Map 함수는 Map 타입을 반환하므로 List, Tuple, Set 등으로 변환해야 함
* `map(Function, iterable)` 형태로 사용

```python
def exp(x):
    return x**2

numList = [1, 2, 3, 4, 5]          # iterable 객체 정의
expList = list(map(exp, numList))  # numList 요소 하나하나에 exp 함수 적용
print(expList)                     # [1, 4, 9, 16, 25]
```



```python
a, b = input().split()    # a = 12, b = 4 (string 타입이므로 각각 int() 캐스팅)

a, b = int(input().split()) 
# split()의 반환값이 List 타입이므로 
# int([12, 4])와 같은 형태가 되어 캐스팅 에러 발생

a, b = map(int, intput().split()) 
# int 캐스팅 된 채로 a, b에 저장됨
# map을 통해 적용되는 함수는 iterable 객체의 요소 하나하나에 적용되기 때문임

# Dictionary type을 전달할 때는 Key 값만 전달함
```



##### 2. Filter

* iterable 객체에서 명시한 조건을 충족하는 요소만 추출하여 반환
* 조건은 함수로 지정, 함수의 반환 값은 True, False로 지정해야 함
* filter함수 또한 filter 타입을 반환하므로 다른 형태로 캐스팅 필요
* `filter(Function, iterable)` 형태로 사용

```python
def getOdd(x):
    return (x % 2) == 1           # 홀수인 것만 True 값을 반환함

numTuple = (1, 2, 3, 4, 5)
oddTuple = tuple(filter(getOdd, numTuple))  # True 값만 filtering
print(oddTuple)      # (1, 3, 5)
```



##### 3. Reduce

* Map 과 동일하나, 이전 단계의 결과 값이 다음 단계의 입력 값 중 하나로 들어가게 됨
* `reduce(Function, iterable, [initializer=None])` 형태로 사용

```python
import functools        # functools 모듈 필요

def sum(x, y):
    return x + y

numSet = {1, 2, 3, 4, 5}
sum = functools.reduce(sum, numSet)
print(sum)     # 15

# 처음에는 1, 2가 sum의 인수로 들어가고 return 값으로 3이 반환
# 다음 단계에서는 이전 단계의 return 값인 3과, 다음 요소인 3이 sum의 인수가 됨
# 다음 단계에서는 이전 단계의 return 값인 6과, 다음 요소인 4가 sum의 인수가 됨
# 반복
```



##### 4. List Comprehension(리스트 내포, 리스트 조건식)

* 특정 iterable 객체로부터 주어진 표현식이 적용된 새로운 iterable 객체를 생성
* `[expression for item in list]` 형태로 사용

```python
numList = [1, 2, 3, 4, 5]
expList = list(i**2 for i in numList)  # i**2 라는 조건을 적용시킴
print(expList)   # [1, 4, 9, 16, 25] 
```



```python
numDict = {'A': 1, 'B': 2, 'C': 3, 'D': 4}
expDict = {key: value**2 for key, value in numDict.items()}
print(expDict)  # {'A': 1, 'B': 4, 'C': 9, 'D': 16}

# dictionary 타입의 경우 위와 같이 사용 가능 
# 반환 값도 dictionary이어야 하니 조건식을 key: value 형태로 만듬
```



* `[expression for item in list if condition]` 의 형태로 조건문 추가 가능(filter)

```python
numTuple = (1, 2, 3, 4, 5)
oddTuple = tuple(i for i in numTuple if i % 2 == 1)
print(oddTuple)      # (1, 3, 5)
# i % 2 == 1을 만족하는 요소만 expression에 적용됨

numDict = {'A': 1, 'B': 2, 'C': 3, 'D': 4}
oddDict = {key: value for key, value in numDict.items() if value % 2 == 1}
print(oddDict)       # {'A': 1, 'C': 3}
# value값 중 value % 2 == 1을 만족하는 value만 key: value에 전달됨
```





##### 5. Lambda Function

* 필요한 시점에 간단하게 생성하고 사용할 수 있는 임시 함수
* 이름을 부여할 수 있으므로 다른 함수의 인수로도 사용이 가능
* `[functionname =] lambda args: expression` 형태로 사용

```python
f = lambda x : x+1
print(f(3))          # 4

# 3이라는 인수를 받아 함수 내에서 x라는 변수에 저장되고
# x+1이라는 조건식에 의해 4라는 return 값 반환
```

* 함수는 함수 객체를 생성하고 메모리에 저장하므로 사용 빈도가 적은 함수들은 메모리 낭비가 존재
* 람다 함수는 사용 후 힙 메모리에서 증발하므로 메모리 낭비가 방지 됨



```python
# 같은 결과를 반환하는 두 가지 방식 비교

# List Comprehension
increasedList = list(x+1 for x in numList)
print(increasedList)            # [2, 3, 4, 5, 6]

# Lambda
increasedList = list(map(lambda x: x+1, numList))
print(increasedList)            # [2, 3, 4, 5, 6]

# lambda가 함수이므로 map()에 적용시킬 수 있음

---
# List Comprehension
numTuple = (1, 2, 3, 4, 5)
oddTuple = tuple(i for i in numTuple if i % 2 == 1)
print(oddTuple)      # (1, 3, 5)

# Lambda
oddTuple = tuple(filter(lambda x: x % 2 == 1, numTuple))
print(oddTuple)            # (1, 3, 5)

# i % 2 == 1을 반환하는 함수를 따로 지정안해도 되니 메모리 낭비가 방지 됨

---
numSet = {1, 2, 3, 4, 5}
sum = reduce(lambda x, y: x+y, numSet)
print(sum)       # 15
```

