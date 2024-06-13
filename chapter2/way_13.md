# 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하자

이전에 다뤘던 언패킹의 단점은 `언패킹할 시퀀스의 길이를 미리 알아야한다`는 것이다. 
언패킹을 통해 저장할 변수의 개수와 시퀀스의 길이가 서로 다르면 오류가 발생한다. 

## 예시 : 2개 이상의 원소가 있는 리스트에서 1,2번째로 오래된 자동차의 나이를 알아내기 

- 데이터 조건
``` python
car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
car_ages_descending = sorted(car_ages, reverse=True)
```

- 슬라이스를 이용하는 방법
``` python 
oldest = car_ages_descending[0]
second_oldest = car_ages_descending[1]
others = car_ages_descending[2:]
```

슬라이스를 사용함에 따라 가독성이 떨어지고 인덱스 값을 잘못 입력할 수 있는 오류를 쉽게 범할 수 있다. 

이런 상황을 더 잘 다룰 수 있도록 python에서는 `별표 식`(starred expression)을 지원한다. 

## 별표 식을 이용하는 방법
``` python
# 별표식을 사용해서
# 맨 앞에 있는 2개의 원소는 각각 oldest, second_oldest에 저장되고
# 나머지 값들은 리스트 형태로 others에 저장된다. 
oldest, second_oldest, *others = car_ages_descending 
```

### 다양한 방식의 별표 식 사용
``` python
# 맨 앞에 있는 원소는 oldest 변수에 저장
# 맨 뒤에 있는 원소는 youngest 변수에 저장
# 그 중간에 있는 값들은 리스트 형태로 others에 저장 
oldest, *others, youngest = car_ages_descending
print(oldest, youngest, others)
>>> 20 0 [19, 15, 9, 8, 7, 6, 4, 1]

# 뒤에 있는 1,2번째 원소는 각각 second_youngest, youngest 변수에 저장
# 나머지 앞에 있는 값들은 others에 저장된다.
*others, second_youngest, youngest = car_ages_descending
print(youngest, second_youngest, others)
>>> 0 1 [20, 19, 15, 9, 8, 7, 6, 4]
```

### 별표 식을 잘못 사용하는 경우 

``` python
# 오류의 내용처럼 별표 식을 사용하기 위해서는
# 대상이 되는 변수가 리스트 또는 튜플 "안에" 있어야 한다.

# 현재 상태는 리스트 "자체" 값을 저장하려고 하기 때문에 오류가 발생한다. 
*others = car_ages_descending

>>> Traceback ...
SyntaxError: starred assignment target must be in a list or tuple

# 2개의 별표 식을 사용해서 오류가 발생했다.
# 정확히 말하면 하나의 객체에 2개 이상의 별표 식을 사용해서 오류가 발생했다. 
first, *middle, *second_middle, last = [1, 2, 3, 4]

>>> Traceback ...
SyntaxError: two starred expressions in assignment
```

### 별표 식의 반환 타입 - list 인스턴스 

- 언패킹하는 시퀀스에 남는 원소가 없어서 별표 식 변수 값에 `빈 리스트`가 출력된 걸 볼 수 있다.
``` python
short_list = [1, 2]
first, second, *rest = short_list
print(first, second, rest)

>>> 1 2 []
```

이처럼 별표 식이 저장되는 타입은 `리스트`이다. 

### 이터레이터에서 별표 식 사용하기 

- 데이터
``` python
def generate_csv():
    yield ('날짜', '제조사', '모델', '연식', '가격')
    ...
```

- 똑같은 동작을 하는 두 코드 블록
  - 어떤 header가 있는지와 row의 길이를 알고 싶음
``` python
# 인덱스와 슬라이스를 사용한 경우

all_csv_rows = list(generate_csv())
header = all_csv_rows[0]
rows = all_csv_rows[1:]
print('CSV 헤더:', header)
print('행 수:', len(rows))

# 출력:
# CSV 헤더: ('날짜', '제조사', '모델', '연식', '가격')
# 행 수: 200

# 별표 식을 사용한 경우
# 확실히 인덱스와 슬라이스를 사용할 때 보다 더 가독성이 좋아진 걸 확인할 수 있다. 
it = generate_csv()
header, *rows = it
print('CSV 헤더:', header)
print('행 수:', len(rows))

# 출력:
# CSV 헤더: ('날짜', '제조사', '모델', '연식', '가격')
# 행 수: 200

```

하지만, 별표 식은 `리스트`를 반환하기 때문에 이터레이터를 별표 식을 이용해서 언패킹한다면  
메모리를 필요 이상으로 사용할 수 있어서 프로그램이 멈출 수 있다는 걸 유념하자.













