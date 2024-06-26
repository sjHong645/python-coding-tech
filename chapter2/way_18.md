# `__missing__`을 사용해 key에 따라 다른 default값을 생성하는 방법 

앞서 살펴봤던 setdefault, defaultdict를 모두 사용하기 적당하지 않은 경우가 있다. 

## 예시 : 파일 시스템에 있는 SNS 프로필 사진을 관리하는 프로그램

### in, KeyError를 사용한 방법 
``` python
pictures = {}
path = 'profile_1234.png' # 프로필 사진 경로 

# get 메서드와 대입식을 이용해
# key값이 딕셔너리에 있는지 검사함
if (handle := pictures.get(path)) is None:
    try:
        handle = open(path, 'ab')
    except OSError:
        print(f'경로를 열 수 없습니다: {path}')
        raise
    else:
        pictures[path] = handle

handle.seek(0)
image_data = handle.read()
```

- 파일 핸들이 이미 딕셔너리 안에 있으면, 딕셔너리를 한 번만 읽는다. (if문 만족하지 않으니 지나감)
- 파일 핸들이 없으면 `get`을 사용해 딕셔너리를 한 번 읽고
  - try/except 블록의 `else 절`에서 핸들을 딕셔너리에 대입한다.

이렇게 구현하면 `if문`에서 딕셔너리를 반복적으로 많이 읽게 되고  
`try/except`과 `else` 블록이 추가되면서 코드 블록 깊이가 더 깊어진다는 단점이 있다. 

### setdefault를 사용한 방법
``` python
try:
    handle = pictures.setdefault(path, open(path, 'ab'))
except OSError:
    print(f'경로를 열 수 없습니다: {path}')
    raise
else:
    handle.seek(0)
    image_data = handle.read()
```

문제점
1. `open 내장 함수`가 딕셔너리에 경로가 있는지 여부와 관계없이 반복적으로 호출된다.
   a. 이로 인해 같은 프로그램상에 있던 열려있는 파일 핸들과 혼동될 수 있는 새로운 파일 핸들이 생길 수 있다.
2. 예외 처리 구분 모호
   - open 메서드에 의한 예외가 발생할 수 있지만
   - setdefault가 던지는 예외가 발생할 수 있다.
   - 위 코드를 보면 이러한 2가지 예외를 구분할 수 없다.

### defaultdict를 사용한 방법
``` python
from collections import defaultdict

def open_picture(profile_path):
    try:
        return open(profile_path, 'ab')
    except OSError:
        print(f'경로를 열 수 없습니다: {profile_path}')
        raise

pictures = defaultdict(open_picture)
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

문제점
- defaultdict 생성자에 전달한 함수는 `인자를 받을 수 없음`
  - 때문에 defaultdict가 호출하는 도우미 함수가 처리 중인 key를 알 수 없다.
 
### `__missing__`을 활용하는 방법 

dict 타입의 하위 클래스를 만들고 `__missing__` 메서드를 구현하면 key가 없는 경우를 처리하는 로직을 원하는대로 만들 수 있다. 

``` python
class Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key)
        self[key] = value
        return value

pictures = Pictures()
handle = pictures[path] # 이와 같은 딕셔너리 접근에서 path가 딕셔너리에 없다면
                        # __missing__ 메서드가 호출됨
                        # 즉, key가 없을 때 __missing__ 메서드를 호출한다. 
handle.seek(0)
image_data = handle.read()
```


























