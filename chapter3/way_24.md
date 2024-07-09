# None과 docstring을 사용해 동적인 default 인자를 지정하자 

ex. 로그 메시지와 시간을 함께 출력하고 싶음

``` python
from time import sleep
from datetime import datetime

# 디폴트 인자는 함수가 정의되는 시점에 딱 한 번만 실행된다.
# 그래서 함수가 정의된 시점의 시간값이 when에 default값으로 정의되기 때문에
# log 함수는 계속 똑같은 시간값을 출력한다. 
def log(message, when=datetime.now()):
    print(f'{when}: {message}')

log('안녕!')
sleep(0.1)
log('다시 안녕!')

# default값을 None을 지정 & 실제 동작을 docstring으로 문서화
# 이러면, 함수를 호출할 때 마다 시간값이 달라진다. 
def log(message, when=None):
    """메시지와 타임스탬프로 로그에 남긴다.

    Args:
        message: 출력할 메시지.
        when: 메시지가 발생한 시간(datetime).
            디폴트 값은 현재 시간이다.
    """
    if when is None:
        when = datetime.now()
    print(f'{when}: {message}')

```

ex. JSON 데이터로 인코딩된 값을 읽으려고 한다. 디코딩에 실패하면 default로 빈 딕셔너리를 반환한다. 

``` python
import json

def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default

# 위와 같이 함수를 정의할 때
# 아래와 같은 문제가 발생한다.

# default 값이 모듈을 로드하는 시점에 단 한 번만 설정되기 때문에  
# default 매개변수에 지정한 딕셔너리가 decode를 호출한 모든 부분에서 공유된다. 

foo = decode('잘못된 데이터')
foo['stuff'] = 5
bar = decode('또 잘못된 데이터')
bar['meep'] = 1
print('foo:', foo)
print('bar:', bar)

# Output
# foo: {'stuff': 5, 'meep': 1}
# bar: {'stuff': 5, 'meep': 1}

assert foo is bar # 동일함 


#### None과 docstring을 사용한 경우 ####

def decode(data, default=None):
    """문자열로부터 JSON 데이터를 읽어온다.

    Args:
        data: 디코딩할 JSON 데이터.
        default: 디코딩 실패 시 반환할 값이다.
            디폴트 값은 빈 딕셔너리다.

    """
    try:
        return json.loads(data)
    except ValueError:
        if default is None:
            default = {}
        return default
```

- 타입 애너테이션을 사용한 경우

``` python
from typing import Optional

# when 인자에 datetime인 Optional 값이라는 타입 애너테이션을 추가함
# 때문에 when에서 사용할 수 있는 값은 None, datetime개체 밖에 없다. 
def log_typed(message: str,
              when: Optional[datetime]=None) -> None:
    """메시지와 타임스탬프로 로그에 남긴다.

    Args:
        message: 출력할 메시지.
        when: 메시지가 발생한 시간(datetime).
            디폴트 값은 현재 시간이다.
    """
    if when is None:
        when = datetime.now()
    print(f'{when}: {message}')

```











