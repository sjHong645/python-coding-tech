# range 보다는 `enumerate` 사용할 것 

## range를 사용한 루프 

- 1번째 예제 
``` python
from random import randint

random_bits = 0
for i in range(32):
    if randint(0, 1):
        random_bits |= 1 << i
```

위 예제와 같이 `range`는 `정수 집합`을 반복하는 루프가 필요할 때 유용하다.  
여기서는 0 ~ 31까지의 범위에서 1씩 증가하는 값을 가진 정수 집합을 얻기 위해서 사용했다. (0, 1, 2, ..., 31)

- 2번째 예제 
``` python
flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']

for i in range(len(flavor_list)):
    flavor = flavor_list[i]
    print(f'{i + 1}: {flavor}')
```

위 예제는 flavor_list라는 리스트에 하나씩 접근하기 위해서 range 함수를 사용했다.  
하지만, 이를 위해서는 `list의 길이`를 알아야 하고 `인덱스를 사용`해서 배열 원소에 접근한다. 때문에 가독성이 떨어질 수 있다. 

## enumerate 사용하기 

이런 문제를 해결하기 위해서 `enumerate 내장 함수`를 제공한다. 

- 대략적인 동작 내용
  1. 이터레이터를 lazy generator로 감싼다
  2. `루프 인덱스`와 `이터레이터의 다음 값`으로 이뤄진 tuple 쌍을 넘겨준다.
  3. `next 함수`를 이용해서 다음 원소를 가져온다.

- 2번 내용
``` python
flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']

for i, flavor in enumerate(flavor_list):
    print(f'{i + 1}: {flavor}')
```

- 2번 내용에서 추가

enumerate의 2번째 파라미터에 `루프를 시작하는 인덱스`를 지정할 수 있다.  
아래 예시에서는 `1번째 부터` 시작하도록 설정했다. (원래는 0번째 부터 시작함)
``` python 
flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']

for i, flavor in enumerate(flavor_list, 1):
    print(f'{i + 1}: {flavor}')
```  

- 3번 내용
``` python
flavor_list = ['바닐라', '초콜릿', '피칸', '딸기']

it = enumerate(flavor_list)

print(next(it)) # next를 사용해서 다음 원소를 가져온다. 
print(next(it))
```
















