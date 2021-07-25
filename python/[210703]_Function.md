[TOC]

## 함수

### 1. 개요

* 특정한 작업을 수행하는 독립적인 코드 블록
* 하나의 큰 프로그램 => 여러 개의 분리된 구조



### 2. 함수의 사용법

##### 2.1 함수 정의

##### 2.2 함수 호출

```python
# []는 optional
def 함수명([파라미터]): 
	수행할 구문
	[return 반환 값]

# 함수 호출 시
[변수명 =] 함수명([인수])
```



* 다수의 값 반환 가능
* 하나의 변수로 받으면 Tuple 형태로 받음
* Unpacking하여 반환 값의 개수만큼 받을 수도 있음

```python
def myFunc():
    return 1, 2, 3, 4

a = myFunc()
print(a)                 # (1, 2, 3, 4)  (하나로 받으면 Tuple)

a, b, c, d = myFunc()
print(a, b, c, d)        # 1 2 3 4       (개수가 일치하면 Unpacking)

a, b = myFunc()
# ValueError: too many values to unpack (expected 2)
```



* 실습 1

```python
# 자판기 구현

def myFunc():
    menu = input('메뉴를 선택하세요:')
    if menu == '콜라':
        print('콜라는 1000원입니다.')
        print('콜라를 전달합니다.')

    elif menu == '사이다':
        print('사이다는 900원입니다.')
        print('사이다를 전달합니다.')

    else:
        print('물은 500원입니다.')
        print('물을 전달합니다.')

# main
print('가게를 엽니다.')
print('손님이 도착했습니다.')
myFunc()
print('가게를 청소합니다.')
print('손님이 도착했습니다.')
myFunc()
print('정산합니다.')
print('손님이 도착했습니다.')
myFunc()

```



* 실습 2

```python
# n을 입력받아 n!을 계산 => factorial은 함수로 구현

def factorial(a):
    if a == 1:
        return 1
    return a * factorial(a-1)

n = input("양수를 입력하세요: ")
result = factorial(int(n))
print(result)

```





### 3. 인수(Argument)와 파라미터

##### 3.1 일반적인 인수와 파라미터

```PYTHON
def Sum(a, b):           # 인수 a, b 
    return a + b		 # 반환 값 a + b 

# 인수와 반환 값은 없을 수도 있음
```



##### 3.2 파라미터 기본값 설정

```python
def Sum(a, b = 99, c = 10):
    return a + b + c

print(Sum(1, 2, 3))   # 6
print(Sum(1, 2))      # 13
print(Sum(1))         # 110

# 인수를 전달하면 해당 인수 사용
# 인수를 전달하지 않으면 기본 값 사용

def Sum(a, b = 99, c):
    return a + b + c

# 기본 값이 위와 같이 처음 또는 중간에 위치할 수 없음
# 즉, 기본 값을 할당 하려면 맨 마지막에 전달해야함
# SyntaxError: non-default argument follows default argument
```



##### 3.3 위치 인수와 키워드 인수

* 위 예시는 모두 위치 인수
* 전달된 인수가 순서대로 함수에 전달됨



* 반면에 키워드 인수는 인수에 이름을 붙여 전달하므로 전달 순서와는 상관 없음

```python
def Time(min, sec):
    print(f"{min}분 {sec}초 입니다")
    
Time(min = 8, sec = 10) 
Time(sec = 10, min = 8) 
# 위의 두 함수는 같은 결과를 출력함
```



### 4. 가변 함수

* 함수 호출 시 넘겨줘야 할 인수의 수가 가변적일 때 사용



##### 4.1 컨테이너를 인수로 사용

* List, Tuple, Set을 사용하는 방법

```python
def mySum(a):
    sum = 0
    for i in a:        # for문을 통해 모든 인수가 합에 반영됨
        sum += i
    return sum

numlist = [1, 2, 3]
sum = mySum(numlist)
print(sum)    # 6
```



##### 4.2 `*args` 사용

* 임의의 개수의 위치 인수를 받을 때 사용

```python
# *args는 argument의 줄임말이며 꼭 args를 사용할 필요는 없음
# 단지 *를 통해 여러개의 위치 인수를 받겠음을 알리는 것

def mySum(*args):      # 임의의 개수의 위치 인수가 들어올 것임을 알림
    sum = 0
    for i in args:     # 전달된 인수는 Tuple 타입으로 args에 저장됨
        sum += i
    return sum

sum = mySum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
print(sum)     # 55
```



##### 4.3 `**kwargs` 사용

* 임의의 개수의 키워드 인수를 받을 때 사용

```python
# **kwargs는 keyward argument의 줄임말이며 꼭 kwargs를 사용할 필요는 없음
# 단지 **를 통해 여러개의 키워드 인수를 받겠음을 알리는 것

def mySum(**kwargs):      # 임의의 개수의 키워드 인수가 들어올 것임을 알림
    
    sum = 0
    for i in kwargs.values():  # 전달된 인수는 Dictionary 타입으로 kwargs에 저장
        sum += i               
    return sum

# kwargs.keys를 통해 num1, num2, num3에 접근도 가능함
sum = mySum(num1=1, num2=2, num3 = 3)
print(sum)      # 6
```



### 5. 변수의 범위(Scope)

##### 5.1 지역 변수(Local variable)

* 함수 내부에서 선언된 변수 
* 함수 밖에서는 사용이 불가능



##### 5.2 전역 변수(Global variable)

* 함수 밖에서 선언된 변수 
* 함수 내에서 사용 가능

```python
def Func():
    a = 5      # 함수 내의 local variable인 a가 새롭게 생성된 것임 
    print(a)   # 즉, 함수 외부의 a = 10과는 전혀 다른 변수이며 함수 종료시 삭제
    
a = 10
Func()

print(a)   # 10 
# 함수 밖에서 선언된 a = 10은 함수 내에서 값을 바꾸어도 값이 변경되지 않음

# Func()에서 a 변수에 값을 5로 할당했지만 
# 이는 단지 local 영역에 생성된 새로운 변수 a 이다(함수 밖의 a와는 다른 변수)
```



* 하지만 mutable(=changeable) 타입의 변수라면 함수 내에서도 값 변경이 가능
* mutable object (List, Dictionary)
* immutable object (Numeric, String, Tuple)

```python
def Func():
    a[0] = 99   

a = [1, 2, 3]
Func()
print(a) # [99, 2, 3]

# List는 mutable object이므로 함수 내에서도 값 변경이 가능하다
```



* immutable object도 함수의 인수로 넘겨준다면 값 변경이 가능함

```python
def myFunc(a):
    a = 6
    return a

a = 10
a = myFunc(a)          # Numeric object를 인수로 전달
print(a)               # 6

# immutable object이지만 인수로 넘겨주어 값 변경이 가능해짐
```



* 또는 global 키워드를 사용하면 값 변경이 가능함

```python
def Func():
    global a     # global 키워드를 통해 함수 내부의 a가 
                 # 함수 밖의 전역 변수 a와 동일함을 재선언
    a = 6 

a = 10
Func()
print(a)         # 6
```

