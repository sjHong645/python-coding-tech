# 리스트를 반환하기보다는 제너레이터를 사용하자 

ex. 문자열에서 찾은 단어의 인덱스를 반환한다. 
``` python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result

address = '컴퓨터(영어: Computer, 문화어: 콤퓨터, 순화어: 전산기)는 긴급한'
result = index_words(address)
print(result[:10])
# Output: [0, 8, 18, 23, 28, 38]
```

하지만, 2가지 문제점이 있다. 각 문제점을 해결할 수 있는 방법에 대해 살펴보려고 한다. 

## 1. 코드 자체가 지저분하다. 간결하게 무슨 동작을 하는지 알 수 없다.

- 새 결과를 찾을 때 마다 append 메소드 호출해야 한다.
- 하지만, 메서도 호출 덩어리가 커서 `index + 1`의 중요성이 희석된다.

### 개선방법 - `제너레이터(generator)`

`제너레이터`는 `yield 식`을 사용하는 함수에 의해 만들어진다. 

- 앞서 보인 예제와 같은 결과를 만들어내는 제너레이터 함수를 정의한 코드
``` python 

def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

# 함수가 호출될 때 제너레이터 함수는 실제로 실행되지 않고 iterator를 반환한다. 
it = index_words_iter(address)
print(next(it)) # iterator가 next 함수를 호출할 때 마다
                # 제너레이터 함수를 다음 yield 식까지 진행시킨다. 
print(next(it))
# Output: 
# 0
# 8
```

확실히 이전보다 함수를 읽기 편해졌다. 하지만, 결과값이 yield 식에 의해 전달된다는 단점이 존재한다.  

이때, 제너레이터가 반환하는 iterator를 `list 내장 함수`에 전달하면 제너레이터를 쉽게 리스트로 변환할 수 있다. 

``` python
result = list(index_words_iter(address))
print(result[:10])
# Output: [0, 8, 18, 23, 28, 38]
```

## 2. 함수를 반환하기 전에 리스트에 모든 결과를 다 저장하고 나서 반환해야 한다. 

때문에, 전달된 값의 크기가 매우 크면 프로그램이 메모리를 소진해서 중단될 수 있다. 

아래와 같이 `제너레이터 버전`으로 만들면 사용하는 메모리 크기를 어느정도 제한할 수 있어서  
입력 길이가 아무리 길어도 쉽게 처리할 수 있다. 

ex. 파일에서 한 번에 한 줄씩 읽어서 한 단어를 출력하는 제너레이터 
``` python
def index_file(handle):
    offset = 0
    for line in handle:
        if line:
            yield offset
        for letter in line:
            offset += 1
            if letter == ' ':
                yield offset
```

위 함수가 사용하는 메모리는 입력된 데이터 중에서 `가장 긴 줄의 길이`로 제한된다.  
또한 입출력 데이터를 별도로 저장하지 않기 때문에 입력 데이터 크기가 커도 출력 시퀀스를 만들 수 있다. 

- 실행 결과 
``` python
with open('address.txt', 'r', encoding='utf-8') as f:
    it = index_file(f)
    results = itertools.islice(it, 0, 10) # 이렇게 제너레이터를 iterator에서 사용하고 나면
                                          # 재사용할 수 없다.   
    print(list(results))

# 출력 : [0, 8, 18, 23, 28, 38]
```



























