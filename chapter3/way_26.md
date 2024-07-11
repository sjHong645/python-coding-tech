# functools.wrap을 사용해 함수 데코레이터를 정의하자 

`데코레이터`는 자신이 감싸고 있는 함수가 호출되기 전과 후에 코드를 추가로 실행해준다.  
이는 데코레이터가 자신이 감싸고 있는 함수의 입력 인자, 반환 값, 함수에서 발생한 오류에 접근할 수 있다는 뜻이다.  
함수의 의미를 강화 or 디버깅 or 등록하는 작업에 이런 기능을 유용하게 쓸 수 있다. 

ex. 함수를 호출할 때 마다 `매개변수 값`과 `반환 값`을 출력하고 싶음 

``` python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args}, {kwargs}) -> {result}')
        return result
    return wrapper

@trace
def fibonacci(n):
    """n번째 피보나치 수를 반환한다."""
    if n in (0, 1):
        return n
    return fibonacci(n - 2) + fibonacci(n - 1)


```
