[TOC]

## 예외 처리

##### 1. 예외 처리 개요

* exception이란 코드 실행 중 발생하는 에러를 의미
* 문법적으로 옳아도 예외 발생 가능(ex> 0으로 나눌 때, 존재하지 않는 파일에 접근할 때)
* 따라서 예측할 수 없는 사용자의 입력 또는 행동에 대처하도록 프로그램 작성



##### 2. try, except

* try 블록 수행 중 예외가 발생하면 except 블록이 실행됨(try 블럭의 예외 이후 코드는 skip)
* 예외가 발생하지 않으면 except는 실행되지 않음

```python
try:
	...
except [Exception [as Error_Variable]]:
	...
```



```python
numList = [0, 1, 2, 3]
while True:
	num1, num2 = map(int, input('숫자 2개를 입력하세요: ').split())   # 5 3
	try:
			print(int(num1) / int(num2))  # PASS
			print(numList[4])             # Error
	
	except ZeroDivisionError:
			print('예외 발생')
            
# except 뒤에 exception type을 적어주면 해당 예외만 처리됨
# except: 만 있으면 모든 에러에 대해 예외 처리함

---
	except (ZeroDivisionError, IndexError):
		print('예외 발생')
        
# 위와 같이 하나로 묶어 exception type을 지정 가능

---
	except ZeroDivisionError:
		print('ZeroDivisionError 발생')
	except IndexError:
		print('IndexError 발생')
        
# 위와 같이 서로 다른 과정으로 예외 처리 가능(exception handler)

---
	except ZeroDivisionError as e:
		print(f'ZeroDivisionError 발생: {e}')
        
# as 키워드를 사용하여 에러 내용 출력 가능 
# 위 예시의 경우 division by zero 출력

---
	except ZeroDivisionError:
		pass
    
# exception handler를 pass로 지정하여 별도의 처리 없이 코드 진행 가능

---
	except Exception as e:
		print(e)
        
# 에러 타입을 모를 경우 Exception으로 대체 가능
```

