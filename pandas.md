# pandas

판다스는 파이썬 프로그램 환경에서 사용하기 편리한 데이터 구조 및 분석 도구를 제공하는 라이브러리

*한 줄 요약*

우리가 익숙한 **스프레드 시트** 형태로 데이터를 관리, 접근할 수 있음

> ![img07](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img07-spreadsheet.png)

- 바로 사용해보기

```python
import pandas as pd

df = pd.read_csv("https://drive.google.com/uc?export=download&id=10VjV0oJXBTJfBBYXsMYxOupPluP4rHhW")
df.head(3)
```

- 행(column)과 열(row)이 존재하는 테이블 형태로 데이터를 관리할 수 있다

survived | pclass | sex | age | sibsp | parch | fare | embarked | class | who | adult_male | deck | embark_town | alive | alone
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
0 | 3 | male | 22.0 | 1 | 0 | 7.2500 | S | Third | man | True | NaN | Southampton | no | False
1 | 1 | female | 38.0 | 1 | 0 | 71.2833 | C | First | woman | False | C | Cherbourg | yes | False
1 | 3 | female | 26.0 | 0 | 0 | 7.9250 | S | Third | woman | False | NaN | Southampton | yes | True

판다스의 사용법은 **10 minutes to pandas**에서 꼭 익혀보자! [[link]](https://pandas.pydata.org/pandas-docs/stable/getting_started/10min.html)

## Element-wise Operation

파이썬에서 데이터를 다룰 때 element-wise로 접근해야하는 경우가 종종 있다.

예를 들어 다음과 같은 상황을 생각해보자.

```python
import numpy as np

name = ["Ted", "Max", "Nana"]
score1 = [90, 50, 80]
score2 = [80, 60, 20]
```

`Ted`의 점수는 `(90, 80)`이고, `Max`의 점수는 `(50, 60)`이다.

만약 세 사람의 평균 점수를 계산한다면 다음과 같은 코드를 떠올리기 쉽다.

```python
assert (len(name) == len(score1) == len(score2))
avg = []
for i in range(len(name)):
  avg.append((score1[i]+score2[i])/2)

print (avg)
```

이처럼 두 개 이상의 배열의 순서(index)가 같은 의미를 가질 때 element-wise로 접근하는 것이 효율적일 때가 많다.

> ![img08](https://github.com/jaehwan-dev/study-in-mis/blob/master/imgs/img08-element-wise%20op.png)

```python
import numpy as np

name = ["Ted", "Max", "Nana"]
score1 = [90, 50, 80]
score2 = [80, 60, 20]

avg = np.add(score1, score2) / 2
print (avg)
```

## 별개의 팁: 같은 기능을 하지만 연산 속도가 현저히 차이날 수 있다

Python의 기본 자료형인 리스트에는 다양한 메쏘드가 구현되어있다.

위의 예에서도 사용하였듯이 이미 선언된 `list`에 원소를 추가하는 메쏘드는 `some_list.append()`이다.

그런데 대량의 데이터를 다룰 때에 이 `some_list.append()` 메쏘드는 굉장히 느리다.

다음의 두 코드를 비교해보자.

```python
def appendLoop(n):
  aList = []
  for i in range(0, n):
    aList.append(i)

  return aList

def assignLoop(n):
  aList = [None] * n
  for i in range(0, n):
    aList[i] = i

  return aList
```

`appendLoop(n)` 함수는 `append()` 메쏘드를 사용하였고, `assignLoop(n)` 함수는 주어진 길이의 배열을 먼저 선언한 후 원소를 채워넣는 방식을 사용한다.

두 함수의 성능 차이는 magic function `%%timeit`을 이용해 비교할 수 있다.

```python
%%timeit
appendLoop(100_000)
```
> 100 loops, best of 3: 8.6 ms per loop

```python
%%timeit
assignLoop(100_000)
```
> 100 loops, best of 3: 5.61 ms per loop


## pandas를 다룰 때 element-wise를 강조한 이유

pandas에서 데이터를 가공할 때, 행(row) 별로 생각하기가 쉽다.

`pandas.DataFrame.iterrows()` 같은 메쏘드는 주어진 데이터프레임의 데이터를 행(row) 별로 순회할 수 있게 처리한다.

하지만 이런 방식의 접근은 매우 느리기 때문에 열(column) 별로 접근하여 element-wise의 연산으로 코드를 짜는 것을 추천한다.

- [예시] 위의 예제 파일에서 성별이 남자인 경우 요금을 2배로 적용한 `new_fare` 열(column)을 추가하는 코드

```python
import pandas as pd

df = pd.read_csv("https://drive.google.com/uc?export=download&id=10VjV0oJXBTJfBBYXsMYxOupPluP4rHhW")
df["new_fare"] = [fare*2 if sex == "male" else fare for sex, fare in zip(df["sex"].values, df["fare"].values)]
df.head(3)
```

survived | pclass | sex | age | sibsp | parch | fare | embarked | class | who | adult_male | deck | embark_town | alive | alone | new_fare
--- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
0 | 3 | male | 22.0 | 1 | 0 | 7.2500 | S | Third | man | True | NaN | Southampton | no | False | 14.5000
1 | 1 | female | 38.0 | 1 | 0 | 71.2833 | C | First | woman | False | C | Cherbourg | yes | False | 71.2833
1 | 3 | female | 26.0 | 0 | 0 | 7.9250 | S | Third | woman | False | NaN | Southampton | yes | True | 7.9250

## 코드 
