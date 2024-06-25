# 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault보다 defaultdict를 사용하자 

내가 직접 만들지 않은 딕셔너리를 다룰 때 key가 없는 경우를 처리하는 방법에는 여러 가지가 있다. 

[앞서](chapter2/way_16.md) 잠깐 살펴봤지만
- `in`과 `KeyError`를 사용하는 방법보다는 낫지만
- 경우에 따라서는 `setdefault`가 더 나을 수도 있다.

## setdefault vs defaultdict

### 예시 데이터 : 방문했던 세계 각국의 도시 이름 저장
``` python
visits = {
    '미국': {'뉴욕', '로스앤젤레스'},
    '일본': {'하코네'},
}
```

### setdefault 적용 

딕셔너리 안에 나라 이름이 들어있는지 여부와 관계없이 각 집합에 새 도시를 추가할 때 `setdefault`를 사용할 수 있다. 

아래 코드를 보면 알겠지만 `setdefault`를 사용했을 때 훨씬 더 코드가 간결하다. 
``` python
visits.setdefault('프랑스', set()).add('칸')  # setdefault를 사용한 경우
                                             # '프랑스'라는 key값이 있었다면 그에 대응되는 value를 반환
                                             # key값이 없었다면 기본값으로 지정한 빈 집합(set())이 반환

if (japan := visits.get('일본')) is None:  # get을 사용한 경우 
    visits['일본'] = japan = set()
japan.add('교토') 
```

### 클래스 내부에서 dictionary 인스턴스를 사용한 경우 

``` python
class Visits:
    def __init__(self):
        self.data = {}

    # 이렇게 설정함으로써 명시적으로 setdefault 호출의 복잡도를 감춰준다. 
    def add(self, country, city):
        city_set = self.data.setdefault(country, set())
        city_set.add(city)

visits = Visits()
visits.add('러시아', '예카테린부르크')
visits.add('탄자니아', '잔지바르')
```

### setdefault를 이용해서 구현했을 때 아쉬움 

add 메소드를 통해 전달된 `나라`가 data 딕셔너리의 key값으로 존재하는지 관계없이  
새로운 set 인스턴스를 생성하기 때문에 그닥 효율적인 구현은 아니다. 

### defaultdict 적용

collections 내장 모듈에 있는 defaultdict 클래스는 key가 없을 때 자동으로 default 값을 저장해서 원하는 동작을 간단히 처리할 수 있다.  
여기서 원하는 동작은 key가 없을 때 default 값을 만들기 위해 호출할 함수를 제공하는 것 뿐이다. 

- Visits 클래스를 defaultdict를 사용해 재작성 
``` python
from collections import defaultdict

class Visits:
    def __init__(self):
        self.data = defaultdict(set) # 모든 key에 대한 value의 자료형을 set으로 지정
                                     # key값이 없다면 기본 자료형인 set()을 반환
                                     # 있다면 이에 대응되는 value를 반환
                                     # 이렇게 되면 key에 대응되는 value의 자료형은 무조건 set이여야 한다.  

    def add(self, country, city):
        self.data[country].add(city) # defaultdict를 가지고 data를 정의함으로써
                                     # data 딕셔너리에 있는 키에 접근하면 항상 기존 set 인스턴스가 반환된다고 가정할 수 있음 

visits = Visits()
visits.add('영국', '바스')
visits.add('영국', '런던')
print(visits.data)

# Output
# defaultdict(<class 'set'>, {'영국': {'바스', '런던'}})
```

















