# 이터레이터나 제너레이터를 다룰 때 itertools를 사용하자 

itertools 내장 모듈에는 이터레이터를 조직화하거나 사용할 때 쓸모 있는 여러 함수가 들어있다. 

복잡한 이터레이션 코드를 작성하고 있다고 생각될 때 쓸 만한 기능을 itertools에서 찾아보자. 

## 여러 이터레이터 연결하기 
itertools 내장 모듈에는 여러 이터레이터를 하나로 합칠 때 쓸 수 있는 여러 함수가 있다. 

### chain 
여러 이터레이터를 하나의 순차적인 이터레이터로 합치고 싶을 때 사용한다. 

``` python
it = itertools.chain([1, 2, 3], [4, 5, 6])
print(list(it))
# 출력: [1, 2, 3, 4, 5, 6]
```

### repeat
하나의 값을 반복해서 전달하고 싶을 때 사용한다. 이때, 반복횟수는 2번째 인자로 지정하면 된다. 

``` python
it = itertools.repeat('안녕', 3)
print(list(it))
# 출력: ['안녕', '안녕', '안녕']
```

### cycle 
어떤 이터레이터가 내놓는 원소들을 계속 반복하고 싶을 때 사용한다. 

``` python
it = itertools.cycle([1, 2])
result = [next(it) for _ in range(10)]
print(result)
# 출력: [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]
```

### tee 
하나의 이터레이터를 병렬적으로 2번째 인자로 지정된 개수의 이터레이터로 만들고 싶을 때 사용한다. 

해당 함수로 만들어진 각각의 이터레이터를 소비하는 속도가 같지 않으면  
처리가 되지 않은 이터레이터의 원소를 큐에 담아둬야 하기 때문에 메모리 사용량이 늘어난다. 

``` python
it1, it2, it3 = itertools.tee(['하나', '둘'], 3)
print(list(it1))
print(list(it2))
print(list(it3))
```

### zip_longest
여러 이터레이터 중 짧은 쪽 이터레이터의 원소를 모두 사용한 경우 fillvalue로 지정한 값을 채워넣어준다.  
(fillvalue로 지정한 값이 없으면 None을 넣는다) 

``` python
keys = ['하나', '둘', '셋']
values = [1, 2]

normal = list(zip(keys, values))
print('zip:', normal)
# 출력 => zip: [('하나', 1), ('둘', 2)]

it = itertools.zip_longest(keys, values, fillvalue='없음')
longest = list(it)
print('zip_longest:', longest)

# 출력 => zip_longest: [('하나', 1), ('둘', 2), ('셋', '없음')]
```

## 이터레이터에서 원소 필터링

### islice
이터레이터를 복사하지 않으면서 원소 인덱스를 이용해 슬라이싱하고 싶을 때 사용하는 함수  

동작은 기존에 알고 있는 슬라이싱 방법과 동일하다. 

``` python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

first_five = itertools.islice(values, 5) # [:5]
print('앞에서 다섯 개:', list(first_five))

middle_odds = itertools.islice(values, 2, 8, 2) # [2:8:2]
print('중간의 홀수들:', list(middle_odds))
# 출력:
# 앞에서 다섯 개: [1, 2, 3, 4, 5]
# 중간의 홀수들: [3, 5, 7]
```

### takewhile

이터레이터에서 주어진 predicate이 False를 반환하는 첫 원소가 나타날 때까지 원소를 돌려준다. 

``` python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7

it = itertools.takewhile(less_than_seven, values)
print(list(it))
# 출력: [1, 2, 3, 4, 5, 6] # 주어진 이터레이터에서 7보다 크거나 같은 값이 나올 때까지 계속 반환 
```

### dropwhile 

`takewhile의 반대`다.  
다만, 주어진 조건을 만족하지 않는 원소가 있으면 멈추지 않고 건너뛴다는 차이점이 있다. 

``` python
it = itertools.dropwhile(less_than_seven, values)
print(list(it))
# 출력: [7, 8, 9, 10]
```

### filterfalse 

`filter 내장 함수의 반대`다.  
주어진 이터레이터에서 predicate가 False를 반환하는 모든 원소를 돌려준다. 

``` python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = lambda x: x % 2 == 0

filter_result = filter(evens, values)
print('Filter:', list(filter_result))

filter_false_result = itertools.filterfalse(evens, values)
print('Filter false:', list(filter_false_result))
# 출력:
# Filter: [2, 4, 6, 8, 10]
# Filter false: [1, 3, 5, 7, 9]
```

## 이터레이터에서 원소의 조합 만들어내기 

### accumulate 

매개변수를 2개 받는 함수를 반복 적용하면서 이터레이터 원소를 하나의 값으로 줄여준다.  
이 함수가 반환하는 이터레이터는 원본 이터레이터의 각 원소에 대해 누적된 결과를 내놓는다. 

``` python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sum_reduce = itertools.accumulate(values) # 이항 함수를 전달하지 않아서 주어진 입력 데이터의 원소의 합계 계산 
print('합계:', list(sum_reduce))
# 합계: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]

def sum_modulo_20(first, second):
    output = first + second
    return output % 20

modulo_reduce = itertools.accumulate(values, sum_modulo_20)
print('20으로 나눈 나머지의 합계:', list(modulo_reduce))
# 20으로 나눈 나머지의 합계: [1, 3, 6, 10, 15, 16, 18, 21, 25, 5]

이 과정을 구체적으로 살펴보면:

초기값: 1
1 + 2 = 3 (3 % 20 = 3) # 3 % 20의 결과값이 다음 값으로 넘어감
3 + 3 = 6 (6 % 20 = 6) # 아래 모든 식들이 동일한 방식으로 다음 값으로 넘김 
6 + 4 = 10 (10 % 20 = 10)
10 + 5 = 15 (15 % 20 = 15)
15 + 6 = 21 (21 % 20 = 1)
1 + 7 = 8 (8 % 20 = 8)
8 + 8 = 16 (16 % 20 = 16)
16 + 9 = 25 (25 % 20 = 5)
5 + 10 = 15 (15 % 20 = 15)
```

한 번에 한 단계씩 결과를 내놓는다는 점을 제외하면  
기본적으로 functools 내장 모듈에 있는 reduce 함수의 결과와 동일하다. 

accumulate에 이항 함수를 넘기지 않으면 주어진 입력 데이터 원소의 합계를 계산한다. 

### product 

하나 이상의 이터레이터에 들어 있는 아이템들의 `Cartesian product`(데카르트 곱)`을 반환한다.  

이걸 사용하면 list comprehension을 깊게 내포하지 않아도 된다.

``` python
single = itertools.product([1, 2], repeat=2)
print('리스트 한 개:', list(single))

multiple = itertools.product([1, 2], ['a', 'b'])
print('리스트 두 개:', list(multiple))
# 출력:
# 리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]
# 리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
```

### permutations 

이터레이터가 내놓은 원소들로부터 만들어낸 길이 N인 순열을 반환한다. 

``` python
it = itertools.permutations([1, 2, 3, 4], 2)
print(list(it))
# 출력: [(1, 2), (1, 3), (1, 4),
         (2, 1), (2, 3), (2, 4),
         (3, 1), (3, 2), (3, 4),
         (4, 1), (4, 2), (4, 3)]

```

### combinations

이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 조합을 반환한다. 

``` python
it = itertools.combinations([1, 2, 3, 4], 2)
print(list(it))
# 출력: [(1, 2), (1, 3), (1, 4),
         (2, 3), (2, 4),
         (3, 4)]
```

### combinations_with_replacement 

이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 중복 조합을 반환한다. 

``` python
it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
print(list(it))
# 출력: [(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]
```















