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

이 경우에 `collections.abc 모듈`을 사용해 딕셔너리와 비슷하지만 내용을 알파벳 순서대로 iteratino해주는 클래스를 새로 정의할 수 있다. 

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
sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks)
print(sorted_ranks.data)
winner = get_winner(sorted_ranks)
print(winner)

# Output
# {'otter': 1, 'fox': 2, 'polar bear': 3} 
# fox
```

- 문제점 : get_winner를 populate_ranks의 삽입 순서에 따라 딕셔너리를 iteration한다고 가정함 
  - 이 코드는 dict 대신 SortedDict를 사용해서 이 가정은 더 이상 성립하지 않음 


