# 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들자 

- 키워드 인자를 사용하면 명확하게 각 매개변수의 의미를 이해할 수 있다.
``` python
def safe_division(number, divisor, ignore_overflow, ignore_zero_division):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

# overflow를 무시하고 0을 반환 
result = safe_division(1.0, 10**500, True, False)
print(result)  # Output: 0

# 0으로 나눈 경우 발생하는 오류를 무시하고 무한대를 반환 
result = safe_division(1.0, 0, False, True)
print(result)  # Output: inf
```

위와 같이 함수를 사용하면 어떤 예외를 무시할 지 결정하는 두 변수 (ignore_overflow, ignore_zero_division)의 위치를 혼동하기 쉽다. 

이러한 코드의 가독성을 향상시키기 위해 `키워드 인자`를 사용한다. 

``` python
def safe_division_b(number, divisor,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    ...
```

위와 같이 함수를 정의하고 나면 내가 무시하고 싶은 예외를 매개변수를 통해 선택할 수 있게 된다. 

``` python
result = safe_division_b(1.0, 10**500, ignore_overflow=True)
print(result)  # Output: 0

result = safe_division_b(1.0, 0, ignore_zero_division=True)
print(result)  # Output: inf
```

문제는 이러한 방법은 키워드 인자를 사용하는 것이 `선택적인 사항`이라는 것이다.  

이와 같이 복잡한 함수의 경우 호출자가 `키워드만 사용하는 인자`를 통해 의도를 명확히 밝히도록 요구하는 편이 좋다. 

``` python
# safe_division 함수가 키워드만 사용하는 인자만 받도록 만든 코드
# * 기호는 위치 인자의 마지막과 키워드만 사용하는 인자의 시작을 구분해준다. 
def safe_division_c(number, divisor, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
...

safe_division_c(1.0, 10**500, True, False)
TypeError: safe_division_c() takes 2 positional arguments but 4 were given

# 키워드 인자를 사용 & 함수에서 정의된 default값을 사용해서
# 정상적으로 동작한다. 
result = safe_division_c(1.0, 0, ignore_zero_division=True)
assert result == float("inf")
```

하지만, `safe_division_c`에도 문제가 있다.  
호출하는 쪽에서 함수의 맨 앞에 있는 두 인자를 호출할 때 위치와 키워드를 혼용할 수 있다. 

ex) 
``` python
assert safe_division_c(number=2, divisor=5) == 0.4
assert safe_division_c(divisor=5, number=2) == 0.4
assert safe_division_c(2, divisor=5) == 0.4
```

또한, 나중에 요구사항이 바뀌거나 원하는 스타일이 바뀌어서 맨 앞의 두 매개변수의 이름이 변경될 수 있다. 

ex) 
``` python
def safe_division_d(numerator, denominator, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    # ...

# 이렇게 이름이 변경되면 키워드로 호출하는 기존 코드들은 전부 오류가 발생한다.
safe_division_d(number=2, divisor=5)
# Traceback ...
# TypeError: safe_division_d() got an unexpected keyword argument 'number'
```

이름이 변경될 때 오류가 문제가 되는 이유는  
맨 앞에 있는 두 매개변수는 구현에 사용하기 위해 임의로 이름 붙였을 뿐 다른 호출자들이 두 변수의 이름에 의존할 것이라 생각하지 않았기에 문제가 된다. 

애초에 위치 인자로만 사용되도록 의도한 매개변수들이기 때문이다. 

python 3.8에는 이 문제에 대한 해법이 들어있다. `위치로만 지정하는 인자`라는 개념이 추가되었기 때문이다. 

`위치로만 지정하는 인자`는 반드시 `위치`만 사용해서 인자를 지정해야 하고 키워드 인자로는 쓸 수 없다. 

`/` 기호를 사용하여 위치로만 지정하는 인자의 끝을 표시한다. 

``` python
def safe_division_d(numerator, denominator, /, *,
                    ignore_overflow=False,
                    ignore_zero_division=False):
    # ...

assert safe_division_d(2, 5) == 0.4

# 위치로만 지정하는 인자를 키워드를 사용해 지정할 때 오류 발생 
safe_division_d(numerator=2, denominator=5)
# Traceback
# TypeError: safe_division_d() got some positional-only arguments passed as keyword arguments: 'numerator, denominator'

```

이로써 맨 앞에 있는 2개의 매개변수는 호출하는 쪽과 분리되었다. 함수 정의에서 파라미터의 이름을 바꿔도 아무런 영향을 끼치지 않는다. 

- `/` 기호와 `*` 기호 사이에 있는 매개변수는 위치 또는 키워드를 사용해 전달 가능
``` python
def safe_division_e(numerator, denominator, /,
                    ndigits=10, *, # /와 * 사이에 있는 변수 
                    ignore_overflow=False,
                    ignore_zero_division=False):
    try:
        fraction = numerator / denominator
        return round(fraction, ndigits)
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

# ndigits 변수 값을 따로 전달하지 않았음
result = safe_division_e(22, 7)
print(result)  # Output: 3.1428571429

# ndigits 변수 값으로 5를 전달 (위치 인자) 
result = safe_division_e(22, 7, 5)
print(result)  # Output: 3.14286

# ndigits 변수 값으로 2 전달 (키워드 인자)
result = safe_division_e(22, 7, ndigits=2)
print(result)  # Output: 3.14
```















