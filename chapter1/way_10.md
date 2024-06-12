# 대입식을 사용해 반복을 피하자 

## 대입식 또는 왈러스 연산자

- Python에서 코드 중복 문제를 해결하기 위해 3.8 버전부터 도입된 구문이다.
- 일반 대입문 : a = b 형식으로 작성
- 왈러스 연산자 : a := b 형식으로 작성 (`a 왈러스 b`라고 읽음)

### 예시 1

- 데이터
``` python
fresh_fruit = {
    '사과': 10,
    '바나나': 8,
    '레몬': 5,
}
```

- 일반 대입문만 사용한 경우

``` python
def make_lemonade(count) :
  ...

def out_of_stock() :
  ... 

# 어차피 count 변수는 if 문의 첫 번째 블록 안에서만 쓰인다.
# 하지만, if문 앞에 count 변수를 정의해서 그 이후에도 count 변수에 접근해야 하는 것 처럼 보인다.
# 단순히 원하는 값을 가져와서 그 값이 0인지 아닌지만 판단하기 위해 사용될 뿐이다. 
count = fresh_fruit.get('레몬', 0)
if count:
    make_lemonade(count)
else:
    out_of_stock()

```


- 왈러스 연산자(대입식)을 사용한 경우

위와 같은 상황을 해결하기 위해서 `대입식`을 사용한다. 

``` python

# count 변수가 if문의 첫 번째 블록에서만 의미가 있다는 것을 명확하게 표현한다.
# 동작 방식은 위 예시와 동일하다.
# count 변수에 값을 대입 => if 문에서 count 변수에 따라 조건문 실행여부 판단
if count := fresh_fruit.get('레몬', 0):
    make_lemonade(count)
else:
    out_of_stock()

```

### 예시 2

``` python
def make_cider(count):
    ...

# 일반 대입문을 사용한 경우     
count = fresh_fruit.get('사과', 0)
if count >= 4:
    make_cider(count)
else:
    out_of_stock()

# 왈러스 연산자(대입식)을 사용한 경우
# count 변수에 대입한 결과값이 4 이상인지 판단하도록 햇다.

# count 변수와 관련된 대입식 자체를 ()로 감싸서 4와 비교했다. 
if (count := fresh_fruit.get('사과', 0)) >= 4:
    make_cider(count)
else:
    out_of_stock()

```

### 예시 3 

``` python
# 일반 대입문을 사용한 경우 
bottles = []
while True:  # 무한 루프
    fresh_fruit = pick_fruit()
    
    if not fresh_fruit:  # 중간에서 끝내기
        break
    
    for fruit, count in fresh_fruit.items():
        batch = makejuice(fruit, count)
        bottles.extend(batch)

# 대입식을 사용한 경우
# 어차피 fresh_fruit 값에 의해 반복문이 멈추는지 판단하는 거니까
# 그 값을 그대로 사용할 수 있도록 바꿔줄 수 있다. 
bottles = []
while fresh_fruit := pick_fruit():
    for fruit, count in fresh_fruit.items():
        batch = makejuice(fruit, count)
        bottles.extend(batch)

```



