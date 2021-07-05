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

##### 3.2 파라미터 기본값 설정

##### 3.3 위치 인수와 키워드 인수

### 4. 가변 함수

### 5. 변수의 범위(Scope)