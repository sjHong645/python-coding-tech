# 복잡한 기준을 사용해 정렬할 때는 `key 파라미터`를 사용하자 

## list 내장 타입에서 제공하는 sort 메서드 

1. default 설정 : 원소 타입에 맞춘 `오름차순` 정렬
  - 문자열, 숫자에 대해서 잘 동작한다. 
``` python 
numbers = [93, 86, 11, 68, 70] 
numbers.sort()
print(numbers)

# Output
[11, 68, 70, 86, 93] # 숫자들을 오름차순 정렬함 
```

2. 특정한 객체를 정렬하는 방법

- 아래와 같이 특정 객체에 sort 메소드를 적용하면 당연히 오류가 발생한다.
  - 서로 다른 Tool 인스턴스 간에 어떤 기준으로 순서를 비교해야 하는지 알 수 없기 때문이다. 

``` python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight

    def __repr__(self):
        return f'Tool({self.name!r}, {self.weight!r})'

tools = [
    Tool('수준계', 3.5),
    Tool('해머', 1.25),
    Tool('스크루드라이버', 0.5),
    Tool('끌', 0.25),
]

tools.sort()

# Output
# Traceback ...
# TypeError: '<' not supported between instances of 'Tool' and 'Tool'
```

- 객체를 정렬하는 기준은 크게 2가지 방법을 이용해 제시할 수 있다.
  - 정렬을 위한 특별 메서드 정의
  - sort 메서드의 key 매개변수 활용하기 - 여기서는 이 부분에 대해서 다룰 것이다. 실제로는 이 방법이 더 유용하기 때문이다.

- 예시 1 : `Tool 객체의 name을 기준`으로 정렬 

``` python

print('미정렬:', repr(tools))
tools.sort(key=lambda x: x.name) # sort 메소드에 lambda 식을 정렬 기준을 설정했다.
                                 # 여기서는 Tool 객체의 name을 기준으로 정렬하도록 했다. 
print('\n정렬:', tools)

# 미정렬: [Tool('수준계', 3.5), Tool('해머', 1.25), Tool('스크루드라이버', 0.5), Tool('끌', 0.25)]
# 정렬: [Tool('끌', 0.25), Tool('수준계', 3.5), Tool('스크루드라이버', 0.5), Tool('해머', 1.25)]
```

- 예시 2 : `Tool 객체의 weight를 기준`으로 정렬 

``` python
tools.sort(key=lambda x: x.weight)
print('무게로 정렬:', tools)

# 무게로 정렬: [Tool('끌', 0.25), Tool('스크루드라이버', 0.5), Tool('해머', 1.25), Tool('수준계', 3.5)]
```

- 예시 3 : 문자열과 같은 기본 타입 정렬

``` python
places = ['home', 'work', 'New York', 'Paris']
places.sort() # 일반적인 문자열 오름차순으로 정렬
              # 이때, 대소문자가 구분됨 
print('대소문자 구분:', places)

places.sort(key=lambda x: x.lower()) # 대소문자를 구분하지 않기 위해 모든 문자열을 소문자로 변경
                                     # 이후에 문자열 정렬 
print('대소문자 무시:', places)

# 대소문자 구분: ['New York', 'Paris', 'home', 'work']
# 대소문자 무시: ['home', 'New York', 'Paris', 'work']
```






