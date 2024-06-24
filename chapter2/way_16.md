# in을 사용하거나 딕셔너리 키가 없을 때 KeyError를 처리하기보다는 `get을 사용`하자. 

## 간단한 타입의 값이 들어있는 딕셔너리 
ex. 샌드위치 가게에서 고객들이 가장 좋아하는 빵에 투표한 결과를 dictionary로 저장함 
``` python
counters = {
    '폼퍼니켈': 2,
    '사워도우': 1,
}
```

투표할 때 투표수를 증가시키려면 먼저 key가 딕셔너리에 존재하는지 살펴봐야 한다.  
key가 없다면 기본 카운터 값인 0을 딕셔너리에 넣고 그 카운터를 증가시킨다. 

- `in`을 사용하여 원하는 동작하는 코드
``` python
key = '밀'
if key in counters:
    count = counters[key]
else:
    count = 0

counters[key] = count + 1
```

- `KeyError 예외`를 활용한 코드
``` python
try:
    count = counters[key]
except KeyError:
    count = 0
counters[key] = count + 1
```

딕셔너리에서는 이런 식으로 `key가 존재하면` 그 값을 가져오고 `key가 존재하지 않으면` 기본 값을 반환하는 동작을 자주 실행한다. 
이러한 기능을 지원하기 위해 `get 메소드`가 있다.

``` python
count = counters.get(key, 0) # counter라는 딕셔너리에 key값에 해당하는 value가 있다면 그걸 반환
                             # 없다면 0을 반환 
counters[key] = count + 1
```

앞서 소개한 in과 KeyError를 사용하는 방식보다 훨씬 짧고 가독성 있는 코드가 만들어졌다. 

따라서 간단한 타입의 값이 들어있는 딕셔너리의 경우 `get 메서드`를 사용하는 방법이 가장 좋다. 

cf) 카운터로 이뤄진 딕셔너리를 유지해야 하는 경우에는 collections 내장 모듈에 있는 Counter 클래스를 고려해보자. 

## 보다 복잡한 값이 들어있는 딕셔너리 

ex. 어떤 사람이 어떤 유형의 빵에 투표했는지 저장한 딕셔너리

- 예시 자료 소개 & in을 사용한 코드 
``` python
votes = {
    '바게트': ['철수', '순이'],
    '치아바타': ['하니', '유리'],
}
key = '브리오슈'
who = '단이'

# in을 사용한 부분 
if key in votes:
    names = votes[key]
else:
    votes[key] = names = []

names.append(who)
print(votes)

# Output
# {'바게트': ['철수', '순이'], '치아바타': ['하니', '유리'], '브리오슈': ['단이']}

```

- KeyError 예외를 이용한 부분
``` python
try:
    names = votes[key]
except KeyError:
    votes[key] = names = []

names.append(who)
```

- get 메서드를 사용한 부분
``` python
names = votes.get(key) # key값에 해당하는 value가 있으면 그 값을 반환
                       # key값이 없다면 None 값을 반환

if names is None:
    votes[key] = names = [] # 새로운 key-value를 딕셔너리에 대입 

names.append(who)
```

- get 메서드 사용 & if문 안에 대입식 사용

``` python
if (names := votes.get(key)) is None:
    votes[key] = names = []
```

- setdefault 메서드 사용
``` python
names = votes.setdefault(key, []) # key값에 해당하는 value가 있으면 그 값을 반환
                                  # key값이 없다면 그 key값을 가지고 key-value를 딕셔너리에 직접 대입. 이때, value는 []이다.

                                  # 정리하면, key값이 있을 때는 해당 key값에 대응되는 value를 반환
                                  # key값이 없다면 []를 반환한다. 
names.append(who) 
```

이 책의 저자는 setdefault를 사용하는 걸 추천하지 않음. defaultdict를 사용하는 것을 권장함. (way_17에서 다룬다)









