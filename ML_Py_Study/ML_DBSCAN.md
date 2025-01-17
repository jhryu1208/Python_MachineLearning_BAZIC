#### [ML with Python] 3. 비지도 학습 알고리즘 (3-2) 병합 군집
- 본 포스팅은 k-평균 군집에 관한 기본적인 내용에 관하여 다룹니다.
- 병합 군집(`agglomerative clustering`) 
- 계층형 군집과 덴드로그램


___

필요 라이브러리 import


```python
import mglearn
import matplotlib.pyplot as plt
import numpy as np
from sklearn.datasets import make_blobs
from sklearn.datasets import make_moons
from sklearn.cluster import DBSCAN
plt.rc('font', family='Malgun Gothic')
```

---

<br>


#### <u>DBSCAN(density-based spatial clustering of applications with noise)</u>

`DBSCAN`
- `밀도 기반 데이터 클러스터링 알고리즘`
- Big 장점 : <u>클러스터의 개수를 미리 지정할 필요가 없다</u>
- 복잡한 형상의 데이터셋에서도 무리없이 적용 가능
- 어떤 클래스에도 속하지 않는 포인트를 구분
- (k-평균/병합군집 보다는 다소 느리지만) 비교적 큰 데이터셋에도 적용 가능

<br>

`DBSCAN`의 원리 순서를 알기 위해서는 아래의 용어와 매개변수들을 간단히 기억해둘 필요가 있다.
- DBSCAN의 주 매개변수
    - <b>min_samples</b> : 핵심 포인트를  중심점으로 간주하는 주변 지역의 표본 수
    - <b>eps</b> : 핵심 포인트를 중심으로 측정되는 유클리디언 거리값
- <b>밀집지역(dense region)</b> : 특성 공간에서 (거리가 가까워서) 데이터가 붐비는 지역
- <b>핵심 샘플(or 핵심 포인트)</b> :  `eps`거리 안에 데이터가 지정한 `min_samples`개수를 만족시키는 `밀집지역`에 있는 데이터 포인트
- <b>잡음(noise)</b> : `eps`거리 안에 들어오는 포인트 수가 지정한 `min_sample`보다 적을 경우 어디에도 속하지 않는 `잡음`으로 레이블됨


<br>

`DBSCAN` 어떤 식으로 적용되는가?
- <b>STEP 1</b> : 특성 공간에서 데이터가 붐비는 `밀집지역`을 찾고, 그 범위안에서 `핵심 샘플`이될 포인트를 지정한다.
- <b>STEP 2</b> : 어느 데이터 포인트에서 `eps`거리 안에 데이터가 `min_samples`개수 만큼 들어 있으면 이 데이터 포인트를 `핵심 샘플`로 지정한다. 이 경우 해당 데이터 포인트는 새로운 클러스터 레이블로 할당된다. (`min_samples`개수를 충족시키지 않으면 `잡음`으로 분류)
- <b>STEP 3</b> : 새롭게 할당된 `핵심 샘플`로 부터 `eps`거리 안의 포인트가 만약 어떤 클러스터에도 할당되지 않았다면 해당 클러스터 레이블로 할당시킨다. (만약 핵심 샘플이면 그 포인트의 이웃을 차례로 방문)
- <b>STEP 4</b> : STEP 1~3을 반복하며 `eps`거리 안에 더 이상 핵심 샘플이 없을 때까지 자란다.

![사진1](https://user-images.githubusercontent.com/53929665/103149384-39869380-47ac-11eb-8b8c-649036630985.png)

<center>출처 : https://medium.com/@agarwalvibhor84/lets-cluster-data-points-using-dbscan-278c5459bee5</center>

<br>

`DBSCAN`을 <u>한 데이터셋에 여러 번 실행하면 핵심 포인트의 군집은 항상 같고 매번 같은 포인트를 잡음으로 레이블</u>한다. 그러나 경계 포인트는 한 개 이상의 클러스터 핵심 샘플의 이웃이 될 수 있기 때문에 경계 포인트가 어떤 클러스터에 속할지는 포인트를 방문하는 순서에 따라 달라진다. 보통은 경계 포인트는 많지 않으며 포인트 순서 때문에 받는 영향도 적어 중요한 이슈에 해당하지 않는다.

<br>

이전에 사용했던 버블데이터셋에 `DBSCAN`을 적용해보자.


```python
from sklearn.cluster import DBSCAN

X, y = make_blobs(random_state = 0, n_samples = 12)

dbscan = DBSCAN()
clusters = dbscan.fit_predict(X)
print('클러스터 레이블:\n', clusters)
```

    클러스터 레이블:
     [-1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1]
    

여기서는 모든 포인트에 `잡음 포인트`를 의미하는 `-1 레이블`이 할당되었다. 이는 작은 샘플 데이터셋에`eps`와 `mins_samples`의 기본값이 해당 데이터셋에 부적합하기 때문이다. 여러가지 `min_samples`와 `eps`에 대한 클러스터링 할당은 다음과 같다.


```python
mglearn.plots.plot_dbscan()
```

    min_samples: 2 eps: 1.000000  cluster: [-1  0  0 -1  0 -1  1  1  0  1 -1 -1]
    min_samples: 2 eps: 1.500000  cluster: [0 1 1 1 1 0 2 2 1 2 2 0]
    min_samples: 2 eps: 2.000000  cluster: [0 1 1 1 1 0 0 0 1 0 0 0]
    min_samples: 2 eps: 3.000000  cluster: [0 0 0 0 0 0 0 0 0 0 0 0]
    min_samples: 3 eps: 1.000000  cluster: [-1  0  0 -1  0 -1  1  1  0  1 -1 -1]
    min_samples: 3 eps: 1.500000  cluster: [0 1 1 1 1 0 2 2 1 2 2 0]
    min_samples: 3 eps: 2.000000  cluster: [0 1 1 1 1 0 0 0 1 0 0 0]
    min_samples: 3 eps: 3.000000  cluster: [0 0 0 0 0 0 0 0 0 0 0 0]
    min_samples: 5 eps: 1.000000  cluster: [-1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1 -1]
    min_samples: 5 eps: 1.500000  cluster: [-1  0  0  0  0 -1 -1 -1  0 -1 -1 -1]
    min_samples: 5 eps: 2.000000  cluster: [-1  0  0  0  0 -1 -1 -1  0 -1 -1 -1]
    min_samples: 5 eps: 3.000000  cluster: [0 0 0 0 0 0 0 0 0 0 0 0]
    


![png](ML_DBSCAN_files/ML_DBSCAN_8_1.png)


위의 그래프에서 클러스터에 속한 포인트는 색이 칠해져있고, `잡음`인 경우는 색이 칠해져 있지 않으며 `핵심 샘플`은 크게 `경계 포인트`는 작게 표시되었다. 이를 통해 알 수 있는 `eps`와 `min_samples`의 특징은 다음과 같다.

<br>

- `eps`
    - 가까운 포인트의 범위를 결정하는데 중요한 역할을 함
    - eps ⇧ ∝ 클러스터에 포함되는 데이터 개수 ⇧ (즉, 클러스터 사이즈 ⇧) ∝ 전체 클러스터 개수 ⇩
    - eps 매우 ⇩ ∝ (min_samples의 조건을 충족하기 어려워지므로) 핵심 포인트 ⇩ & 잡음 포인트 ⇧
    - eps 매우 ⇧ ∝ 모든 데이터 포인트가 하나의 클러스터에 속하게 됨
    
- `min_samples`
    - 클러스터의 최소 크기를 결정
    - 덜 조밀한 지역의 경우 포인트를 잡음으로 분류할지 클러스터로 분류할지 결정하는데중요한 역할을 함
    - min_samples ⇧ ∝ (조건을 충족시키기 어려워지므로) 핵심 데이터 포인트 ⇩ ∝ 잡음 포인트 ⇧

<br>

`DBSCAN`은 이처럼 클러스터의 개수를 지정할 필요는 없지만 `eps`의 값은 간접적으로 몇 개의 클러스터가 만들어질지 제어한다. 따라서, 적절한 `eps`값을 쉽게 찾으려면 `StandardScaler`나 `MinMaxScaler`로 모든 특성의 스케일을 비슷한 범위로 조정해주는 것을 권장한다 ㅠㅠ...

<br>

아래는 복잡한 two_moons 데이터셋에 `DBSCAN`을 적용한 결과이다. 두개의 반달 모양을 매우 정확하게 구분한 것을 확인할 수 있다.


```python
X, y = make_moons(n_samples=200, noise=0.05, random_state=0)

# 평균이 0, 분산이 1이 되도록 데이터의 스케일을 조정합니다
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
scaler.fit(X)
X_scaled = scaler.transform(X)

dbscan = DBSCAN()
clusters = dbscan.fit_predict(X_scaled)
# 클러스터 할당을 표시합니다
plt.scatter(X_scaled[:, 0], X_scaled[:, 1], c=clusters, cmap=mglearn.cm2, s=60, edgecolors='black')
plt.xlabel("특성 0")
plt.ylabel("특성 1")
```




    Text(0, 0.5, '특성 1')



    C:\Users\jhryu\AppData\Local\Programs\Python\Python37\lib\site-packages\matplotlib\backends\backend_agg.py:238: RuntimeWarning: Glyph 8722 missing from current font.
      font.set_text(s, 0.0, flags=flags)
    C:\Users\jhryu\AppData\Local\Programs\Python\Python37\lib\site-packages\matplotlib\backends\backend_agg.py:201: RuntimeWarning: Glyph 8722 missing from current font.
      font.set_text(s, 0, flags=flags)
    


![png](ML_DBSCAN_files/ML_DBSCAN_11_2.png)


<br>

---

#### <u> DBSCAN 장단점 정리</u>

- <b>장점</b>
    - `잡음`지점도 걸러내기 때문에 이상치를 구별해내기 유용하다.
    - 저밀도 클러스터에서 고밀도 클러스터를 분리하는데 유용하다.
    - `k-평균`과 `병합 군집`처럼 클러스터 수를 미리 지정할 필요 없다.
    - 복잡한 데이터셋에서도 강하다!
    

- <b>단점</b>
    - 부분적으로 비슷한 밀도를 가진 데이터셋의 경우 해당 알고리즘은 연약하다.
    - 데이터 포인트 처리 순서가 매번 다르기 때문에 해당 알고리즘을 시행할 때 마다 다른 결과가 도출된다.
    - 데이터의 차원이 높아질수록 `eps`매개변수의 값을 지정하기 어려워진다.

<br>

---

### References

- 안드레아스 뮐러, 세라 가이도, 『파이썬 라이브러리를 활용한 머신러닝』, 박해선, 한빛미디어(2017)
- [scikit-learn document for DBSCAN](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html)
- [Vibhor Agarwal's medium](https://medium.com/@agarwalvibhor84/lets-cluster-data-points-using-dbscan-278c5459bee5)
