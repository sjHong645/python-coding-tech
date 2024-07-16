# 매개변수에 대해 이터레이션할 때는 조심하자 

ex. 미국 텍사스 주의 여행자 수를 분석하고 싶다. 

전달받은 데이터 집합은 도시별 방문자 수라고 하자. 

``` python
def normalize(numbers): # 리스트를 매개변수로 받고

    total = sum(numbers) # 리스트 안에 있는 값의 총합 total 
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent) # 각 도시별 퍼센트 값을 반환할 리스트에 추가 
    return result

visits = [15, 35, 80]
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0

# Output: [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

위 코드를 좀 더 확장하려면 텍사스의 모든 도시에 대한 여행자 정보가 들어있는 파일에서 데이터를 읽어야 한다.  
더 나아가 전 세계를 대상으로 여행자 분석을 할 수도 있다. 

그렇게 되면 위 코드는 너무 많은 메모리를 소모하게 될 것이다. 그래서 `제너레이터를 정의`할 필요가 있다. 

- 제너레이터를 이용

``` python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('my_numbers.txt') # my_numbers.txt 파일을 가지고 이터레이터를 만들었다. 
percentages = normalize(it)
print(percentages)
# Output: [] # 아무런 값도 반환되지 않는다. 
```

왜 이런 현상이 발생할까. `normalize 메서드`와 함께 다시보자. 

``` python
def normalize(numbers): # 이터레이터를 매개변수로 받음
    total = sum(numbers) # 여기서 이터레이터를 이미 소진함 
    result = []
    for value in numbers: # 앞서 이터레이터를 모두 소진했기 때문에
                          # 더 이상 남아있는 값이 없으니까 반복문이 작동되지 않음
        percent = 100 * value / total
        result.append(percent) 
    return result # 결국 빈 리스트만 출력됨 
```

위와 같이 이터레이터는 결과를 한 번 만들고나면 다시 사용할 수 없다.  
헷갈리는 점은 다시 사용할 수 없는 이터레이터를 다시 사용하려고 하면 오류가 발생하지 않는다는 점이다. 

이 문제를 해결하기 위한 여러가지 방법을 생각해볼 수 있다. 

## 해결책 1. 입력 이터레이터를 명시적으로 소진시켜서 전체 내용을 list에 저장하기 

``` python
def normalize_copy(numbers):
    numbers_copy = list(numbers)  # 이터레이터를 복사해서 리스트에 저장함
                                  # 이터레이터는 소진되었지만
                                  # 복사한 결과는 남아있으니까 그걸 가지고 앞으로의 원하는 동작 수행   
    total = sum(numbers_copy)
    result = []
    for value in numbers_copy:
        percent = 100 * value / total
        result.append(percent)
    return result

it = read_visits('my_numbers.txt')
percentages = normalize_copy(it)
print(percentages)
assert sum(percentages) == 100.0
# Output: [11.538461538461538, 26.923076923076923, 61.53846153846154]

```

이 방식은 문제점이 있다.  
입력된 이터레이터의 내용을 복사할 때 입력된 값의 크기에 따라 메모리를 엄청 많이 사용할 수 있다. 그로인한 메모리 부족으로 인해 프로그램이 중단될 수 있다.  

앞서 문제가 되었던 상황과 똑같은 문제를 반복한다. 

## 해결책 2 : 호출될 때 마다 이터레이터를 새롭게 반환받기 

``` python
def normalize_func(get_iter):
    total = sum(get_iter())  # 새로운 이터레이터
    result = []
    for value in get_iter():  # (들어있는 값은 동일하지만) 새로운 이터레이터
        percent = 100 * value / total
        result.append(percent)
    return result

path = 'my_numbers.txt'
percentages = normalize_func(lambda: read_visits(path))
print(percentages)
assert sum(percentages) == 100.0

# Output: [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

잘 작동한다. 하지만, 위와 같이 람다 함수를 전달함으로써 가독성이 떨어진다. 

## 해결책 3. 이터레이터 프로토콜을 구현한 새로운 컨테이너 클래스 제공 

### 사전 설명 

`이터레이터 프로토콜`은 python의 for문 또는 그와 연관된 식들이 컨테이너 타입의 내용을 방문할 때 사용한다. 

Python에서 `for x in foo` 구문을 사용  
⇒ 내부적으로 `iter(foo)`를 호출  
⇒ iter 내장함수는 `foo.__iter__`라는 특별 메서드를 호출  
⇒ `__iter__` 메서드는 반드시 `이터레이터 객체를 반환`해야 한다.  
⇒ for문은 반환받은 이터레이터 객체가 데이터를 소진할 때 까지 반복적으로 이터레이터 객체에 대해 next 함수를 호출한다. 

바로 예시를 살펴보자. 

ex. 여행 데이터가 들어 있는 파일을 읽는 iterable 컨테이너 클래스를 정의한 코드 
``` python
class ReadVisits:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

def normalize(numbers): # 앞서 정의한 normalize 함수와 동일함
    total = sum(numbers) # ReadVisits.__iter__를 호출해서 이터레이터 할당 
    result = []
    for value in numbers: # ReadVisits.__iter__를 호출해서 이터레이터 할당 
        percent = 100 * value / total
        result.append(percent)
    return result

path = 'my_numbers.txt'
visits = ReadVisits(path)
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
# Output: [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

normalize 함수 내에 있는 주석부분을 보면 알 수 있듯이 두 이터레이터는 서로 독립적으로 진행되고 소진된다.  
때문에 입력 데이터를 여러 번 읽어야 하는 단점이 존재한다. 

### 단순한 이터레이터인지 컨테이너 타입안에 정의된 이터레이터인지 구분하기

`이터레이터`가 `iter 내장 함수`에 전달되는 경우 `전달받은 이터레이터가 그대로 반환`된다.  
반대로 `컨테이너 타입`이 `iter 내장 함수`에 전달되면 `매번 새로운 이터레이터 객체가 반환`된다. 

따라서 입력값이 이런 동작을 하는지 검사해서 반복적으로 이터레이션 할 수 없는 인자인 경우에 TypeError를 발생시켜서 인자를 거부할 수 있다. 

``` python
from collection.abc import Iterator

def normalize_defensive(numbers):

    # if iter(numbers) in numbers : # 전달받은 값이 그냥 이터레이터라면 반복적으로 호출할 수 없다. 
    if isinstance(numbers, Iterator): # 전달받은 값이 컨테이너 타입안에서 __iter__ 메소드를 정의했기 때문에
                                      # 반복적으로 호출할 수 있다.
        raise TypeError("컨테이너를 제공해야 합니다")
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
```

위와 같이 컨테이너를 사용하는 방법은 앞서 언급했던 것 처럼 입력 데이터를 여러 번 이터레이션해야 한다. 

- normalize_defensive 함수는 리스트, ReadVisits에 대해 모두 제대로 동작한다.

리스트나 ReadVisits 모두 이터레이터 프로토콜을 따르는 iterable 컨테이너이기 때문이다. 
``` python
visits = [15, 35, 80]
percentages = normalize_defensive(visits)
assert sum(percentages) == 100.0

visits = ReadVisits(path)
percentages = normalize_defensive(visits)
assert sum(percentages) == 100.0

visits = [15, 35, 80]
it = iter(visits) 
normalize_defensive(it) # 단순 이터레이터를 전달했기 때문에 예외를 발생시킨다.
# Output:
# Traceback ...
# TypeError: 컨테이너를 제공해야 합니다
```









