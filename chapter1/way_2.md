# PEP 8 스타일 가이드를 따르자. 

PEP(Python Enhancement Proposal) 8 : 파이썬 코드를 어떤 형식으로 작성할지 알려주는 스타일 가이드 

이 가이드를 따름으로써 `일관된 스타일`을 사용하게 되어 서로 다른 코드를 더 친숙하고 쉽게 읽을 수 있다.  
똑같은 가이드를 준수한 다른 프로그래머의 코드에 더 쉽게 협력할 수 있다. 

## 공백 (whitespace) 
: 탭, 스페이스, 새 줄(newline)을 통칭하는 말 

- 탭 대신 `스페이스`를 사용해서 들여쓰기할 것
- 문법적으로 중요한 들여쓰기는 4칸 띄어쓰기(스페이스)를 사용할 것
  ``` python
    foo = long_function_name(var_one, var_two,
                             var_three, var_four)
    
    def long_function_name(
            var_one, var_two, var_three,
            var_four):
        print(var_one)
  ```
- `한 줄`에 `79개 문자 이하`여야 한다
- 길이가 긴 식을 다음 줄에 이어서 써야 한다면 일반적인 들여쓰기보다 `4번 더 띄어쓰기`를 해서 들여쓴다.

- 파일 안에 있는 `각각의 함수와 클래스 사이`에는 `2줄의 빈 줄`을 넣어라
  ``` python

    import math

    def hello():
        print("Hello, World!")
    
    
    def area_circle(radius):
        """
        Returns the area of a circle with the given radius.
        """
        return math.pi * radius ** 2
    
    
    class Circle:
    
        pass
  ```
- `클래스 안에 있는 메서드와 메서드 사이`에는 `1줄의 빈 줄`을 넣어라
  ``` python
    class Circle:
    
        def __init__(self, radius):
            self.radius = radius
    
        def area(self):
            return area_circle(self.radius)
  ```

- `딕셔너리`에서 `key`와 `콜론(:)` 사이에는 공백을 넣지 않고 한 줄 안에 key와 value를 같이 넣는다면 `콜론(:)` 다음에 `1칸만 띄어쓴다`. 
  ``` python
    dot_com = {'key_str': 14} 
  ```

- 변수 대입식에서 `=` 전후에는 `1칸만 띄어쓴다`.
  ``` python
     a = 1
  ```
- 타입 표기를 덧붙이는 경우에 `변수 이름`과 `콜론(:)` 사이에 공백을 넣지 말 것. `타입 정보`와 `콜론` 사이에는 `1칸만 띄어쓸 것`
  ```
    def func(para_okay: list) :
      pass
  ```

## Naming Convention(명명 규약) 

PEP 8에서는 Python의 여러 부분에서 이름을 어떻게 붙여야 하는지에 대한 스타일을 제시한다. 

- 함수, 변수, 속성 : `lowercase_under`과 같이 `소문자 & 밑줄`을 사용하자
- 보호되어야 하는(protected) 인스턴스의 속성
  1. 앞서 얘기한 일반적인 이름 규칙을 따른다.
  2. 추가적으로 `_leading_underscore`와 같이 해당 속성은 `_ 1개`로 시작한다.
- private 인스턴스 속성
  1. 앞서 얘기한 일반적인 이름 규칙을 따른다.
  2. 추가적으로 `__leading_underscore`와 같이 해당 속성은 `_ 2개`로 시작한다.

- 클래스(예외도 포함) :  `OneTwoThree`와 같이 `CamelCase` 방식을 따른다.
- 모듈 내에서 정의된 변수(모듈 수준의 상수) : `GRAVITY_ALL`과 같이 `대문자 & 밑줄`을 사용한다. 
- 클래스에 있는 인스턴스 메서드
  - 인스턴스 메서드 : 특정 인스턴스에 대해 작업을 수행하는 메서드. 해당 인스턴스의 속성 및 메서드에 접근할 수 있다. 
  - 호출 대상 객체를 가리키는 첫 번째 인자 이름으로 `self`를 사용한다.
- 클래스 메서드 : 클래스를 가리키는 첫 번째 인자 이름으로 `cls`를 사용한다.
   - 클래스 메서드 : 클래스 자체에 대해 작업을 수행하는 메서드입니다. 인스턴스와 독립적으로 동작한다.
``` python
class Circle:
    pi = 3.14159  # 클래스 속성

    def __init__(self, radius):
        self.radius = radius  # 인스턴스 속성

    # 인스턴스 메서드
    def area(self):
        """인스턴스의 면적을 계산"""
        return Circle.pi * self.radius ** 2

    # 클래스 메서드
    @classmethod
    def from_diameter(cls, diameter):
        """직경을 받아 인스턴스를 생성"""
        radius = diameter / 2
        return cls(radius)
```

## Expression(식)과 Statements(문)


  
