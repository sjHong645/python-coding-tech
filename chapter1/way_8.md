# 여러 이터레이터에 대해 나란히 loop를 수행하려면 `zip`을 사용해라 

``` python

names = ['Cecilia', '남궁민수', '김태지']
counts = [len(n) for n in names]

```

만들어진 list의 각 원소는 같은 인덱스 위치에 있는 원소와 관련이 있다.  
- names[0] & names[0]값의 길이
- names[1] & names[1]값의 길이
- names[2] & names[2]값의 길이

## range를 사용해서 두 개의 리스트에 접근하기 

두 리스트를 동시에 이터레이션 하기 위해서 `names 리스트의 길이`를 사용한 인덱스를 이용해서 접근했다. 

``` python 
longest_name = None
max_count = 0

for i in range(len(names)):
    count = counts[i]
    if count > max_count:
        longest_name = names[i]
        max_count = count

print(longest_name)

```

names와 counts의 원소를 찾는 과정에서 가독성이 떨어진다. 

## enumerate를 사용해서 두 개의 리스트에 접근하기 

``` python
for i, name in enumerate(names):
    count = counts[i]
    if count > max_count:
        longest_name = name
        max_count = count

```

약간은 나아졌지만 인덱스를 사용해서 이터레이터에 접근하는 코드가 여전한 한계점이다. 

## `zip`을 사용해서 두 개의 리스트에 접근하기 

``` python
for name, count in zip(names, counts): # 둘 이상의 이터레이터를 지연 계산 제너레이터를 사용해 묶어줌
                                       # zip 제너레이터는 각 이터레이터의 다음 값이 들어있는 튜플을 반환한다
                                       # 이 튜플을 for문에서 name, count라는 이름의 변수로 언패킹했다. 
    if count > max_count:
        longest_name = name
        max_count = count
```

확실히 이전 코드들에 비해 가독성이 좋아진 걸 확인할 수 있다. 

zip은 자신이 감싼 이터레이터 원소를 하나씩 소비한다.  
이터레이터의 모든 원소를 메모리에 한 번에 로드하지 않고 하나씩 꺼내서 처리한다는 뜻이다. 그러면 당연히 메모리 사용량이 최소화된다.  

때문에 메모리를 다 소모해서 프로그램이 중단되는 위험없이 아주 긴 입력도 처리할 수 있다. 

## zip을 사용할 때 주의할 점 

입력한 이터레이터의 길이가 서로 다를 때 zip이 어떻게 동작하는지에 주의해야 한다. 

- 첫 번째 예제
`names` 리스트에는 항목 추가 & `counts` 리스트에는 추가하지 않음 
``` python 
names.append('Rosalind')

for name, count in zip(names, counts):
    print(name)

Cecilia
남궁민수
김태지 
```

names에서 새로 추가한 원소가 출력되지 않는 걸 확인할 수 있다. 

`zip`은 자신이 감싼 이터레이터 중 어느 하나가 끝날 때까지 튜플을 생성한다.  
즉, names 리스트와 counts 리스트 중에서 `길이가 짧은 counts 리스트`가 `2번째에서 끝나`기 때문에  
names 리스트의 2번째 값까지만 접근한 것이다. 

- 두 번째 예제

길이가 긴 이터레이터의 뒷부분을 버리지 않도록 할 수 있다. 이때, `itertools 모듈의 zip_longest`를 사용해보자. 

``` python
import itertools

for name, count in itertools.zip_longest(names, counts):
    print(f'{name}: {count}')

Cecilia: 7
남궁민수: 4
김태지: 3
Rosalind: None # counts의 3번째 값은 존재하지 않아서 None으로 출력됨 

```

zip_longest는 존재하지 않는 값을 자신에게 전달된 `fillvalue` 값으로 대체한다. (default값은 None)











