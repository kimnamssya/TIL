[TOC]

## 반복문

### 1. `while`

##### 1.1 개요

##### 1.2 무한 루프

##### 1.3 break & continue

##### 1.4 while else

##### 1.5 중첩 while

* while 문 내에 while을 중첩하여 사용 가능

```python
>>> num1 = 2
>>> num2 = 1
>>> while num1 <= 9:      # num1이 9 이하인 동안 반복
   		num2 = 1
        while num2 <= 9:  # num2가 9 이하인 동안 반복
            print(f'{num1} * {num2} = {num1*num2}')
            num2 += 1
        num1 += 1
```



### 2. `for` loop

* for의 조건에는 iterable 객체를 사용할 수 있음(반복가능한 객체를 의미)
* Python의 iterable 객체 : String, List, Tuple, Range, Dictionary, Set

```python
>>> for item in Iterable :   # Iterable 객체의 요소 갯수만큼 반복
    	반복할 구문
```



##### 2.1 for in List

```python
numList = [1, 2, 3, 4]
for i in numList:             # i에 순서대로 1, 2, 3, 4가 대입된다.
	print(i, end=' ')

#1 2 3 4

numList = [[1, 'a'], [2, 'b'], [3, 'c'], [4, 'd']]
for i, j in numList:
	print(i, j, end=' ')

# 1 a 2 b 3 c 4 d 

numList = [[1, 'a'], [2, 'b'], [3, 'c'], [4, 'd']]
for i in numList:
	print(i, end=' ')

# [1, 'a'] [2, 'b'] [3, 'c'] [4, 'd']
```



##### 2.2 for in Tuple

* List와 동일



##### 2.3 for in Set

* List와 동일



##### 2.4 for in Range

* range는 숫자 리스트를 자동 생성해주는 함수,  `ex) a = list(range(1, 10, 2))` 

```python
numRange1 = range(1, 10)
for i in numRange1:
	print(i, end=' ')      # 1 2 3 4 5 6 7 8 9

# 1부터 100까지의 합
sum = 0
for i in range(1, 101):
    sum += i
print(sum)
```

* 즉, `range()`는 list로 보면 됨.



##### 2.5 for in Dictionary

* for in dictionary에서 dictionary는 기본적으로 key를 반환함

```python
subjectDic = {'kor': '0100', 'eng': '0101', 'math': '0102'}
for t in subjectDic:
    print(t, end=' ')

# kor eng math
```

* `keys()`, `values()`, `items()` 메소드를 통해 key와 value를 각각 또는 동시에 반환 가능

```python
subjectDic = {'kor': '0100', 'eng': '0101', 'math': '0102'}
for t in subjectDic.keys():
    print(t, end=' ')        # kor eng math

for t in subjectDic.values():
    print(t, end=' ')        # 0100 0101 0102

for t in subjectDic.items():
    print(t, end=' ')        # ('kor', '0100') ('eng', '0101') ('math', '0102')

for t, x in subjectDic.items():
    print(t, x, end=' ')     # kor 0100 eng 0101 math 0102
```



##### 2.6 enumerate

* index와 Iterable 객체의 원소를 묶어서(pair) tuple형태로 반환

```python
numList = [1, 2, 3, 4]
for i in enumerate(numList):
	print(i, end=' ')        # (0, 1) (1, 2) (2, 3) (3, 4)

for index, value in enumerate(numList):
    	print(f'{index}: {value}')
        
'''
0: 1
1: 2
2: 3
3: 4
```



##### 실습 2

```python
# for문을 사용하여 2단에서 9단까지 구구단 출력

for i in range(2, 10):
    for j in range(1, 10):
        print(f"{i} * {j} = {i*j}")
```

##### 실습 3

```python
# data에서 6월 둘째 주의 시가평균, 종가평균 구하기

data =  [["2021.06.04", 1116.50, 1116.00],
         ["2021.06.07", 1111.20, 1112.90],
         ["2021.06.08", 1111.30, 1114.20],
         ["2021.06.09", 1118.50, 1115.40],
         ["2021.06.10", 1117.00, 1115.80],
         ["2021.06.11", 1113.50, 1110.80],
         ["2021.06.14", 1116.30, 1116.70],
         ["2021.06.15", 1117.50, 1117.00]]

targetData = data[1:6]
beginPrice = 0
endPrice = 0

for dailyData in targetData:
    beginPrice += dailyData[1]
    endPrice += dailyData[2]

print(f"시가 평균: {beginPrice / len(targetData):.2f}, 종가 평균: {endPrice / len(targetData):.2f}")

```

