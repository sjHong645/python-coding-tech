# 딕셔너리 삽입 순서에 의존할 때는 조심해라 

## 파이썬 3.5 이전의 딕셔너리는 순서를 저장하지 않는다. 

``` python
# Python 3.5
baby_names = {
    'cat': 'kitten',
    'dog': 'puppy',
}

print(baby_names)

# Output
# {'dog': 'puppy', 'cat': 'kitten'}
```

이런 일이 발생하는 이유는 딕셔너리를 구현하기 위해  
`내장 hash 함수`와 파이썬 인터프리터가 시작할 때 초기화되는 `난수 seed값`을 사용하는 `해쉬 테이블 알고리즘`을 사용했기 때문이다. 

이후에 `파이썬 3.6부터`는 딕셔너리가 삽입 순서를 보존하도록 동작이 개선됐고 파이썬 3.7부터는 아예 파이썬 언어 명세에 이 내용이 포함됐다. 
때문에, 파이썬 3.7부터는 dict 인스턴스에 있는 내용을 이터레이션할 때 `key를 삽입한 순서`대로 값을 돌려받는다는 사실에 의존할 수 있다. 

## 파급효과 : 파이썬 3.7 이후에 dict 인스턴스가 순서를 보장하게 되었다. 

1. `**kwargs` : 함수의 여러 개의 인자들을 `dictionary` 형태로 받는 키워드 
- 파이썬 3.5 이전 : 순서가 보장되지 않았음
- 파이썬 3.7 이후 : 순서가 보장됨


2. 클래스의 인스턴스 딕셔너리 

인스턴스 필드를 대입한 `순서`가 `__dict__에 반영`되었다는 걸 확인할 수 있다.

``` python
class MyClass:
    def __init__(self):
        self.alligator = 'hatchling'
        self.elephant = 'calf'

a = MyClass()
for key, value in a.__dict__.items():
    print(f'{key} = {value}')

# Output
# alligator = hatchling
# elephant = calf
```

## 유사 딕셔너리를 주의하세요 - from collection.abc import MutableMapping 

유사 딕셔너리를 정의할 때 특정 기준으로 정렬하도록 설정했다면 표준 dict를 기준으로 만들었던 여러 기능들이 제대로 동작하지 않을 수 있다. 

- 표준 dict를 사용했을 때 
ex. 가장 귀여운 아기 동물을 뽑는 콘테스트의 결과를 보여주는 프로그램을 작성 
``` python 
votes = {
    'otter': 1281,
    'polar bear': 587,
    'fox': 863,
}

# 득표 데이터를 처리하고
# 각 동물의 이름과 순위를 빈 딕셔너리(ranks)에 저장하는 함수를 정의함 
def populate_ranks(votes, ranks):
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i

# 앞선 함수에서 내용을 채운 딕셔너리(ranks)에서
# 가장 많은 득표를 얻은 값을 출력 
def get_winner(ranks):
    return next(iter(ranks))

# 원하는대로 동작하는 걸 확인 
ranks = {}
populate_ranks(votes, ranks)
print(ranks)
winner = get_winner(ranks)
print(winner)

# Output
# {'otter': 1, 'fox': 2, 'polar bear': 3}
# otter
```

- 유사 딕셔너리를 사용했을 때

프로그램의 요구 사항이 변경되어 `알파벳순`으로 결과를 표시해야 한다고 하자. 

이 경우에 `collections.abc 모듈`을 사용해 딕셔너리와 비슷하지만 내용을 알파벳 순서대로 iteration해주는 클래스를 새로 정의할 수 있다. 

``` python
from collections.abc import MutableMapping

class SortedDict(MutableMapping):
    def __init__(self):
        self.data = {}

    def __getitem__(self, key):
        return self.data[key]

    def __setitem__(self, key, value):
        self.data[key] = value

    def __delitem__(self, key):
        del self.data[key]

    def __iter__(self):
        keys = list(self.data.keys())
        keys.sort()
        for key in keys:
            yield key

    def __len__(self):
        return len(self.data)
```

이렇게 정의한 `SortedDict`는 표준 딕셔너리의 프로토콜을 준수했다. 
앞서 정의한 함수를 호출하면서 SortedDict 인스턴스를 표준 dict 위치에 사용해도 아무런 오류가 발생하지 않는다. 

하지만, 실행 결과는 요구 사항에 맞지 않는다. 

``` python
sorted_ranks = SortedDict() # 표준 dict가 아닌 SortedDict() 클래스를 가지고 빈 dict를 만들었다. 
populate_ranks(votes, sorted_ranks) # 방금 만든 dict에 득표수를 기준으로 순서를 설정해서 key-value를 삽입 
print(sorted_ranks.data) 
winner = get_winner(sorted_ranks) # 당연히 득표수가 가장 많은 key가 출력될 줄 알았지만
                                  # 그렇지 않았다.
                                  # 알파벳 순서가 가장 앞에 있는 fox가 출력되었다.
                                  # 그렇게 정렬되도록 SortedDict 내부에서 설정했기 때문이다. 
print(winner)

# Output
# {'otter': 1, 'fox': 2, 'polar bear': 3} 
# fox
```


- 이 문제를 해결하는 3가지 방법

1. ranks 딕셔너리가 어떤 특정 순서로 iteration된다고 가정하지 않고 get_winner 함수 구현

가장 보수적이고 가장 튼튼한 해법 

``` python
def get_winner(ranks):
    for name, rank in ranks.items():
        if rank == 1: # 순서로 결정하는게 아니라
                      # 등수 값을 가지고 결정 
            return name

winner = get_winner(sorted_ranks)
print(winner)
```

2. 함수 맨 앞에 ranks의 타입이 원하는 타입인지 검사하는 코드를 추가. 그렇지 않으면 예외 

보수적인 접근 방법보다 실행 성능이 더 좋다. 

``` python
def get_winner(ranks):

    # ranks의 타입이 dict가 아니라면 Exception을 출력 
    if not isinstance(ranks, dict):
        raise TypeError('dict 인스턴스가 필요합니다')
    return next(iter(ranks))
```

3. `타입 annotation`을 사용해서 get_winner에 전달되는 값이 dictinary와 비슷한 동작을 하는 MutableMapping 인스턴스가 아니라 dict 인스턴스가 되도록 강제하는 방법 + mypy strict 모드 

``` python
from typing import Dict, MutableMapping

def populate_ranks(votes: Dict[str, int],
                   ranks: Dict[str, int]) -> None:
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i

def get_winner(ranks: Dict[str, int]) -> str:
    return next(iter(ranks))

class SortedDict(MutableMapping[str, int]):
    ... 

votes = {
    'otter': 1281,
    'polar bear': 587,
    'fox': 863,
}

sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks) # mypy strict 모드에서 ranks에 전달된 모드가 정확히 맞지 않음 
print(sorted_ranks.data)
winner = get_winner(sorted_ranks) # mypy strict 모드에서 ranks에 전달된 모드가 정확히 맞지 않음 
print(winner)

# Output
# {'otter': 1, 'fox': 2, 'polar bear': 3}
# otter

# 엄격한 type checking을 위해 mypy 도구를 strict 모드로 사용했다. 
# python -m mypy --strict example.py
# example.py:48: error: Argument 2 to "populate_ranks" has incompatible type "SortedDict"; expected "Dict[str, int]"
# example.py:50: error: Argument 1 to "get_winner" has incompatible type "SortedDict"; expected "Dict[str, int]"

```

정적 타입 안정성과 런타임 성능을 가장 잘 조합해준다. 


