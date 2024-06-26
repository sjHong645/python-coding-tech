# `__missing__`을 사용해 key에 따라 다른 default값을 생성하는 방법 

앞서 살펴봤던 setdefault, defaultdict를 모두 사용하기 적당하지 않은 경우가 있다. 

- 예시 : 파일 시스템에 있는 SNS 프로필 사진을 관리하는 프로그램

1. in, KeyError를 사용한 방법 
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

이렇게 구현하면 딕셔너리를 더 많이 읽게 되고 내포되는 블록 깊이가 더 깊어진다는 단점이 있다. 

2. setdefault를 사용한 방법
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


