# functools.wrap을 사용해 함수 데코레이터를 정의하자 

## 데코레이터 소개 
`데코레이터`는 자신이 감싸고 있는 함수가 호출되기 전과 후에 코드를 추가로 실행해준다.  
이는 데코레이터가 자신이 감싸고 있는 함수의 입력 인자, 반환 값, 함수에서 발생한 오류에 접근할 수 있다는 뜻이다.  
함수의 의미를 강화 or 디버깅 or 등록하는 작업에 이런 기능을 유용하게 쓸 수 있다. 

ex. 함수를 호출할 때 마다 `매개변수 값`과 `반환 값`을 출력하고 싶음 

``` python
# 데코레이터 정의 
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args}, {kwargs}) -> {result}')
        return result
    return wrapper

# 데코레이터 적용
# trace 앞에 @를 붙이면 된다.
@trace
def fibonacci(n):
    """n번째 피보나치 수를 반환한다."""
    if n in (0, 1):
        return n
    return fibonacci(n - 2) + fibonacci(n - 1)

```

@ 기호를 사용한 것은
1. fibonacci 함수에 대해 데코레이터를 호출한 후
2. 데코레이터가 반환한 결과를
3. 원래 함수가 속해야 하는 영역에 원래 함수와 같은 이름으로 등록하는 것과 같다.

``` python
@trace
def fibonacci(n):
    .... 

위/아래는 같은 동작을 실행하는 코드이다. 

fibonacci = trace(fibonacci)
```

즉, fibonacci 함수를 매개변수로 받아 trace 함수를 실행하기 때문에 아래와 같은 결과가 출력된다.

```
fibonacci((0,), {}) -> 0
wrapper((0,), {}) -> 0
fibonacci((1,), {}) -> 1
wrapper((1,), {}) -> 1
fibonacci((2,), {}) -> 1
wrapper((2,), {}) -> 1
fibonacci((1,), {}) -> 1
wrapper((1,), {}) -> 1
fibonacci((0,), {}) -> 0
wrapper((0,), {}) -> 0
fibonacci((1,), {}) -> 1
wrapper((1,), {}) -> 1
fibonacci((2,), {}) -> 1
wrapper((2,), {}) -> 1
fibonacci((3,), {}) -> 2
wrapper((3,), {}) -> 2
fibonacci((4,), {}) -> 3
wrapper((4,), {}) -> 3

```

## 부작용 

데코레이터가 반환하는 함수의 이름이 fibonacci가 아니게 된다. (`fibonacci = trace(fibonacci)`에서 새롭게 반환된 좌항의 fibonacci)

``` python
print(fibonacci)
# Output: <function trace.<locals>.wrapper at 0x108955dc0>
```

trace 함수는 정의한대로 `wrapper 함수를 반환`한다. 즉, 반환된 wrapper 함수가 fibonacci 라는 이름으로 등록된 것이다. 

이러한 동작은 디버거와 같은 `introspection`을 하는 도구에서 문제가 된다. 
인트로스펙션(introspection)은 실행 시점에 프로그램이 어떻게 실행되는지 관찰하는 것을 뜻한다

ex. help 내장 함수로 확인 

예상 동작 : fibonacci 함수의 docstring 출력 
tlfwp ehdwkr : trace 함수에서 정의한 wrapper 함수 관련 내용 
``` python
help(fibonacci)

# Output:
# Help on function wrapper in module __main__:
# wrapper(*args, **kwargs)
```

또한 데코레이터가 감싼 원래 함수의 위치를 찾을 수 없다. 그래서 `객체 직렬화`도 깨진다. 

``` python
import pickle
pickle.dumps(fibonacci)
# Traceback ...
# AttributeError: Can't pickle local object 'trace.<locals>.wrapper'
```

## 해결 방법 : functools 내장 모듈에 정의된 `wraps 도우미 함수` 사용

``` python
from functools import wraps

def trace(func):
    @wraps(func) # wraps를 wrapper 함수에 적용
                 # wraps는 데코레이터 내부에 들어가는 함수에서 중요한 메타데이터를 복사해서
                 # wrapper 함수에 적용해준다. 
    def wrapper(*args, **kwargs):
        # ...
        return wrapper

@trace
def fibonacci(n):
    # ...
```

help 함수를 실행해보면 알 수 있다. 

``` python
help(fibonacci)
# Output:
# Help on function fibonacci in module __main__:
# fibonacci(n)
#    n번째 피보나치 수를 반환한다.

```

pickle 객체 직렬화도 제대로 작동한다. 

``` python
print(pickle.dumps(fibonacci))
# Output: b'\x80\x04\x95...\x94.'
```

파이썬 함수에는 이 예제에서 살펴본 것 보다 더 많은 표준 attribute가 있다.  
파이썬에서 함수의 인터페이스를 처리하려면 이런 attribute도 보존되어야 한다.  
wraps를 사용하면 이 모든 attribute를 제대로 복사해서 함수가 제대로 작동하도록 해준다. 



