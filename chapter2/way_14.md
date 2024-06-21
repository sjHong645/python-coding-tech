# 복잡한 기준을 사용해 정렬할 때는 sort 메서드의 `key 파라미터`를 사용하자 

## list 내장 타입에서 제공하는 sort 메서드 

### default 설정 : 원소 타입에 맞춘 `오름차순` 정렬
  - 문자열, 숫자에 대해서 잘 동작한다. 
``` python 
numbers = [93, 86, 11, 68, 70] 
numbers.sort()
print(numbers)

# Output
[11, 68, 70, 86, 93] # 숫자들을 오름차순 정렬함 
```

### 특정한 객체를 정렬하는 방법

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

#### sort 메서드의 key 매개변수 활용하기

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

- 예시 4 : Tool 인스턴스의 오른쪽 값(weight)으로 먼저 정렬 => 왼쪽 값(name)으로 정렬

``` python 
power_tools = [
    Tool('드릴', 4),
    Tool('원형 톱', 5),
    Tool('착암기', 40),
    Tool('연마기', 4),
]
```

사전 지식 - `튜플` 타입에서의 정렬 

``` python
saw = (5, '원형 톱')
jackhammer = (40, '착암기')

assert (jackhammer > saw) # 아무런 에러가 발생하지 않는다.
                          # 즉, 이 부등식은 올바르게 성립하는 것이다.
                          # 두 튜플의 첫 번째 위치에 있는 값으로 먼저 비교
                          # 두 값이 동일하면 두 번째 위치에 있는 값으로 비교
                          # 여기서는 40 > 5가 성립하므로 위 부등식은 올바른 식이 된다.

# 이 내용을 좀 더 명확하게 보여주는 python 식
drill = (4, '드릴')
sander = (4, '연마기')

assert drill[0] == sander[0]  # 무게가 같다(첫 번째 값이 동일하다)
assert drill[1] < sander[1]   # 알파벳순으로 드릴이 더 작다 (2번째 값으로 비교 => 부등식 확인)
assert drill < sander         # 그러므로 드릴이 더 먼저다 (결론)
```

이러한 `튜플의 비교 동작 방식`을 이용해서 원하는 방식으로 정렬할 수 있다. 

``` python
# 정렬에 사용할 값들의 우선순위를 weight -> name으로 설정 
power_tools.sort(key=lambda x: (x.weight, x.name))
print(power_tools)
[Tool('드릴', 4), Tool('연마기', 4), Tool('원형 톱', 5), Tool('착암기', 40)]

# 앞선 정렬과 동일 & 내림차순으로 정렬 
power_tools.sort(key=lambda x: (x.weight, x.name), reverse=True)
print(power_tools)
# [Tool('착암기', 40), Tool('원형 톱', 5), Tool('연마기', 4), Tool('드릴', 4)]

# 특별한 점은 weight에 -를 추가했다는 점이다.
# weight을 기준으로 내림차순 => name을 기준으로 오름차순 정렬하도록 설정되었다.
# 그래서 2번째 정렬 결과와 달리 '드릴'과 '연마기'의 정렬 결과가 다르다
power_tools.sort(key=lambda x: (-x.weight, x.name))

[Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]

# 하지만, 모든 값에 -를 붙일 수 없다. weight가 int값이였기 때문에 가능한 것이다.
# name값은 str이기 때문에 오류가 발생했다. 
>>> power_tools.sort(key=lambda x: (x.weight, -x.name))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 1, in <lambda>
TypeError: bad operand type for unary -: 'str'
```

- 예시 5 : sort 메서드 자체를 여러 번 실행

`리스트 타입의 sort 메서드`는 key 함수가 반환하는 값이 서로 같으면 리스트에 들어있던 원래 순서를 그대로 유지한다. 

``` python
# 아래 두 suite의 코드는 동일한 기능을 수행한다. 
# 1.
power_tools.sort(key=lambda x: (-x.weight, x.name))

# 2.
power_tools.sort(key=lambda x: x.name) # name 기준 오름차순

# 중간 결과
# 보다시피, name을 기준으로 오름차순 정렬이 되었다. 
[Tool('드릴', 4), Tool('연마기', 4), Tool('원형 톱', 5), Tool('착암기', 40)]

# 위와 같이 정렬된 상태에서 weight를 기준으로 내림차순 정렬된다.
# Tool('드릴', 4), Tool('연마기', 4)로 weight값이 동일하기 때문에 두 개의 값의 순서는 그대로 유지된다. 
power_tools.sort(key=lambda x: x.weight, reverse=True) # weight 기준 내림차순
# 결국 아래의 결과가 출력된다. 


# 결과
[Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
```

위와 같은 접근법으로 정렬하면 여러 다른 타입의 정렬 기준을 원하는 기준으로 조합할 수 있다. 

다만, 최종적으로 리스트에서 얻어내고 싶은 `정렬 기준 우선순위의 역순`으로 정렬을 수행해야 한다는 사실을 꼭 기억해야 한다. 

때문에 이 방법은 꼭 필요할 때만 사용할 것을 권장한다. 









