# 키워드 인자로 선택적인 기능을 제공하자 

- 예시 함수 정의 
``` python
def remainder(number, divisor):
    return number % divisor
```

- `딕셔너리`를 사용하여 함수를 호출하고 싶다. 이럴 때 `** 연산자`를 사용할 수 있다.
  - 딕셔너리에 있는 값을 함수에 전달하고
  - 각 값에 대응하는 key를 keyword로 사용하도록 한다. 

``` python
my_kwargs = {
    'number': 20,
    'divisor': 7,
}
assert remainder(**my_kwargs) == 6

# ** 연산자를 위치 인자나 키워드 인자와 섞어서 함수를 호출할 수 있다. 
my_kwargs = {
    'divisor': 7,
}
assert remainder(number=20, **my_kwargs) == 6

# 여러 개의 ** 연산자를 사용할 수도 있다.
# 물론, 각 딕셔너리가 갖고 있는 key값이 서로 겹치면 안 된다.
my_kwargs = {
    'number': 20,
}
other_kwargs = {
    'divisor': 7,
}
assert remainder(**my_kwargs, **other_kwargs) == 6
```

모든 키워드의 인자를 받는 함수를 만들고 싶다면 `**kwargs 파라미터`를 사용한다. 

``` python
def print_parameters(**kwargs):
    for key, value in kwargs.items():
        print(f'{key} = {value}')

print_parameters(alpha=1.5, beta=9, gamma=4)
```

### 키워드 인자가 제공하는 유연성의 3가지 장점

1. 코드를 처음 보는 사람들에게 함수 호출의 의미를 명확히 알려줄 수 있다.
   
   - remainder(20, 7) 보다는 remainder(number = 20, divisior = 7)라고 입력한 게 파라미터의 의미를 좀 더 명확하게 나타낸다.

2. 키워드 인자를 이용하여 함수를 정의할 때 `default 값`을 지정할 수 있다





















