#### [ML with Python] 3. 비지도 학습 알고리즘 (2-1) 차원축소 : PCA(주성분분석)
- 본 포스팅은 PCA에 관한 기본적인 내용에 관하여 다룹니다.
- 주성분 분석(`PCA`)
- 고유얼굴(eigenface) 특성 추출 맛보기

___

필요 라이브러리 import


```python
import mglearn
import matplotlib.pyplot as plt
import numpy as np
from tqdm import notebook

from sklearn.datasets import load_breast_cancer
from sklearn.datasets import fetch_lfw_people

from sklearn.neighbors import KNeighborsClassifier
```

---

####  <u>주성분 분석(PCA)</u>

  책에 따르면, `주성분 분석`이란 <u>통계적으로 상관관계가 없도록 데이터셋을 회전하는 기술</u>이라고 한다. 이 문구를 보고 아! 이런 의미구나 하는 사람이 있는 반면, 나와 같이 무슨 소리일까라는 생각부터 하게 된다. 이는 PCA를 익히면서 점차 알 수 있긴하다. 시작하기에 앞서 이를 간단히 설명하면 통계적 상관관계가 없다는 의미를 다시 나열하면 다음과 같다. `주성분 분석`(즉 `PCA`)는 <u>상관관계가 있는 변수들 사이의 변동</u>을 <u>상관관계가 없는 </u>`주성분`<u>이라 부르는 새로운 변수</u>들로 설명하고자하는 차원 축소 기술을 의미한다. 하지만, 이제 <u>회전하는 기술</u>이라는 점에서 의문을 품을 수 있다. 이에 관해서는 다음의 과정들을 살펴보자.

<br>


```python
mglearn.plots.plot_pca_illustration()
```


![png](ML_PCA_files/ML_PCA_4_0.png)


위의 그림은 원본 데이터 포인트를 색으로 구분해 표시했을 뿐만 아니라, `PCA`의 단계를 step by step으로 보여주는 그래프이다. 

<br>

왼쪽 위에 있는 그래프를 살펴보자. `PCA`알고리즘에서는 우선 특성간의 연관성을 고려하기 위해 <u>(<b>STEP1</b>)분산이 가장 큰 방향</u>을 찾는다. 왜냐하면, 분산이 클 수록 두 특성 간의 연관성이 크다(feature1이 커지면 feature2는 작아지는 등)는 의미이기 때문이다. `PCA` 알고리즘은 <u>첫 번째 방향과 직각인 방향 중에서 가장 많은 정보를 담은 방향을 찾는다.</u> 2차원의 경우 직각 방향은 하나이지만, 고차원의 경우에서는 무한한 직각 방향이 있을 수 있다. 이런 과정을 거쳐 찾은 방향을 데이터에 있는 주된 분산의 방향이라고 해서 `주성분(principal component)`라고 한다. 일반적으로, <u>원본 특성의 갯수만큼 주성분이 존재</u>한다.

<br>

오른쪽 위에 있는 그래프는 <u>(<b>STEP2</b>)</u>`주성분`<u> 1과 2를 각각 x축 y축에 나란하도록 회전</u>시킨 것이다. 또한 x축 y축 중심이 0에 맞춰진 것을 확인할 수 있는데, 이는 회전시키기 전에 평균을 뺐기 때문이다. 이때 `PCA`<u>에 의해 생긴 두 축은 서로 연관되어 있지 않다는 것을 기억하자</u>. `주성분`이란 특성들의 연관성을 표현한 것이지, 종속변수를 발생시키는 독립변수가 아니다. 이는 해당 그래프에서 주성분 1이 변하더라도 주성분 2에 변화가 없는 것을 통해 직관적으로 알 수 있다.

<br>

제목에서 볼 수 있듯이, `PCA`는 <u>주성분의 일부만 남기는 차원 축소 용도로 사용할 수 있다.</u> 왼쪽 아래에 있는 그래프를 살펴보자. 해당 그래프는 <u>(<b>STEP3</b>) 주성분 중 가장 유용한 방향을 찾아서 그 방향의 성분, 즉 첫 번째 주성분을 유지</u>하려고 한다. 이렇게 하면 <u>2차원 데이터셋이 1차원 데이터셋으로 감소</u>한다.

<br>

마지막으로 오른쪽 아래의 그래프를 살펴보자. 여기도 `PCA`의 한 과정 일 수 있지만, 데이터를 원래대로 복원하는 과정이라고 생각하는게 나중에 이해하기 편하다. 여기서는 <u>(<b>STEP4</b>)데이터에 다시 평균을 더하고 반대로 회전시켰다.</u> 데이터 포인트들이 다시 기존의 특성(property)공간에 놓이게 되었지만, 첫 번째 주성분의 정보를 담고 있다. 이처럼 이 변환은 <u>노이즈를 제거</u>하거나 <u>주성분에서 유지되는 정보를 시각화</u>하는데 종종 사용한다.

<br>



---

#### <u>PCA를 적용해 유방암 데이터셋 시각화하기</u>

`PCA`가 가장 널리 사용되는 분야는 `고차원 데이터셋 시각화`이다.

<br>

세 개 이상의 특성을 가진 데이터를 흔히들 많이 사용하는 산점도로 나타내기란 매우 쉽지않다. 나도 이전에 아무 것도 모르고 특성이 많은 데이터를 산점도로 나태내었다가 시간은 시간대로 낭비하고, 노력은 노력대로 낭비하는 사태가 벌어지는 있었다. 지금 여기서 사용하는 유방암 데이터셋도 특성이 30개나 되기 때문에, 산점도로 표시할 경우 지옥을 보기 쉽다(대략 435개의 산점도 그래프를 그리고 이를 살펴봐야하는데 솔직히 말이 안된다). 

<br>

이런 `고차원 데이터`를 산점도로 표시하는 것 보다는 `히스토그램`을 이용하여 각 클래스 레이블을 표시해보는 것이 더 도움이된다. 


```python
plt.rc('font', family='Malgun Gothic')
cancer = load_breast_cancer()

fig, axes = plt.subplots(15, 2, figsize=(10, 20))
malignant = cancer.data[cancer.target == 0]
benign = cancer.data[cancer.target == 1]

ax = axes.ravel()

for i in range(30):
    _, bins = np.histogram(cancer.data[:, i], bins=50)
    ax[i].hist(malignant[:, i], bins=bins, color=mglearn.cm3(0), alpha=.5)
    ax[i].hist(benign[:, i], bins=bins, color=mglearn.cm3(2), alpha=.5)
    ax[i].set_title(cancer.feature_names[i])
    ax[i].set_yticks(())
ax[0].set_xlabel("특성 크기")
ax[0].set_ylabel("빈도")
ax[0].legend(["악성", "양성"], loc="best")
fig.tight_layout()
```


![png](ML_PCA_files/ML_PCA_8_0.png)


위의 `히스토그램` 그래프에서 특성들이 클래스 별로 어떻게 분포되어 있는지를 알려주고, 이를 통해 어떤 특성이 양성과 악성샘플을 구분하는 데 더 좋은지 가늠해볼 수 있다. 예를 들어"smoothness error"특성은 두 그래프가 거의 겹쳐져 별로 쓸모가 없다. 하지만, "worst concave points"의 경우 확실히 구분되어 매우 유용한 특성이란 걸 알 수 있다.

<br>

하지만 이러한 `히스토그램`도 <u>특성 간의 상호작용이나 이 상호작용이 클래스와 어떤 관련이 있는지는 전혀 알려주지 못한다.</u><br>이러한 단점 때문에 `PCA`를 사용하게 되는 것이다.

<br>

`PCA`를 적용하기 전에 <u>특성의 스케일이 서로 다르면 올바른 주성분 방향을 찾을 수 없기 때문에 `PCA`를 사용할 때는 반드시 표준값으로 바꿔야한다.</u> 따라서, `StandardScaler`를 사용해 특성의 분산이 1이 되도록 데이터의 스케일을 먼저 조정해야 한다.


```python
from sklearn.preprocessing import StandardScaler

cancer = load_breast_cancer()

scaler = StandardScaler()
scaler.fit(cancer.data)
X_scaled = scaler.transform(cancer.data)
```

<br>

`PCA` 변환을 학습시키고 적용하는 과정은 데이터 전처리와 비슷하다.
- `PCA` 객체를 생성하기
- `fit` 매서드 호출하여 주성분 찾기
- `transform` 메서드를 호출해 데이터를 회전시키고 차원을 축소하기


```python
from sklearn.decomposition import PCA
# 데이터의 처음 두 개 주성분만 유지시킵니다
pca = PCA(n_components=2)
# 유방암 데이터로 PCA 모델을 만듭니다
pca.fit(X_scaled)

# 처음 두 개의 주성분을 사용해 데이터를 변환합니다
X_pca = pca.transform(X_scaled)
print("원본 데이터 형태:", str(X_scaled.shape))
print("축소된 데이터 형태:", str(X_pca.shape))
```

    원본 데이터 형태: (569, 30)
    축소된 데이터 형태: (569, 2)
    


```python
# 클래스를 색깔로 구분하여 처음 두 개의 주성분을 그래프로 나타냅니다.
plt.figure(figsize=(8, 8))
mglearn.discrete_scatter(X_pca[:, 0], X_pca[:, 1], cancer.target)
plt.legend(["악성", "양성"], loc="best")
plt.gca().set_aspect("equal")
plt.xlabel("첫 번째 주성분")
plt.ylabel("두 번째 주성분")
```




    Text(0, 0.5, '두 번째 주성분')




![png](ML_PCA_files/ML_PCA_13_1.png)


`PCA`는 `비지도 학습`이므로 회전축을 찾을 때 어떤 클래스 정보도 사용하지 않는다.<br>(위의 클래스 표시는 기존의 데이터에서 추출해서 표현했을 뿐이다.)<br>단순히 데이터에 있는 `상관관계`만을 고려한다.

<br>

`PCA`의 단점은 <u>그래프의 두 축을 해석하기가 쉽지 않다는 점</u>이다.<br>
`PCA`의 `주성분`은 원본 데이터에 있는 어떤 방향에 대응하는 <u>여러 특성이 조합</u>된 형태이다. 이러한 조합은 보통 매우 복잡하다.<br>`PCA` 객체가 학습(`fit`)될 때 `components_`속성에 주성분이 저장된다.


```python
print("PCA 주성분 형태:", pca.components_.shape)
```

    PCA 주성분 형태: (2, 30)
    

위의 결과를 보면 `PCA의 주성분`은 2행 30열로 되어있는 것을 확인할 수 있다.
- 행 : 각 주성분을 나타냄
- 열 : 원본 데이터의 특성(property)에 대응하는 값

<br>

이 `components_`값을 숫자로 출력하고 시각화하면 다음과 같다.


```python
print("PCA 주성분:", pca.components_)
```

    PCA 주성분: [[ 0.21890244  0.10372458  0.22753729  0.22099499  0.14258969  0.23928535
       0.25840048  0.26085376  0.13816696  0.06436335  0.20597878  0.01742803
       0.21132592  0.20286964  0.01453145  0.17039345  0.15358979  0.1834174
       0.04249842  0.10256832  0.22799663  0.10446933  0.23663968  0.22487053
       0.12795256  0.21009588  0.22876753  0.25088597  0.12290456  0.13178394]
     [-0.23385713 -0.05970609 -0.21518136 -0.23107671  0.18611302  0.15189161
       0.06016536 -0.0347675   0.19034877  0.36657547 -0.10555215  0.08997968
      -0.08945723 -0.15229263  0.20443045  0.2327159   0.19720728  0.13032156
       0.183848    0.28009203 -0.21986638 -0.0454673  -0.19987843 -0.21935186
       0.17230435  0.14359317  0.09796411 -0.00825724  0.14188335  0.27533947]]
    


```python
plt.matshow(pca.components_, cmap='viridis')
plt.yticks([0, 1], ["첫 번째 주성분", "두 번째 주성분"])
plt.colorbar()
plt.xticks(range(len(cancer.feature_names)),
           cancer.feature_names, rotation=60, ha='left')
plt.xlabel("특성")
plt.ylabel("주성분")
```




    Text(0, 0.5, '주성분')




![png](ML_PCA_files/ML_PCA_18_1.png)


`첫 번째 주성분`의 경우 <u>모든 특성의 부호가 같은 것</u>을 확인할 수 있다.<br> 이는 즉 <u>모든 특성 사이에 공통의 상호관계</u>가 있다는 의미이다.<br> 따라서 한 특성의 값이 커지면 다른 값들도 같이 커질 것이다.<br>하지만, 부호가 모두 다른 두 번째 주성분이나 이처럼 특성이 많을 경우 축이 가지는 의미를 정확히 알기는 쉽지않다.

<br>

---

#### <u>고유얼굴(eigenface) 특성 추출</u>

`PCA`는 `특성 추출`에도 이용가능하다. `특성 추출`은 원본 데이터 표현보다 분석하기에 더 적합한 표현을 찾을 수 있을 것이란 생각에서 출발한다.  이미지를 다루는 애플리케이션은 `특성 추출`이 도움이 될만한 좋은 예시이다.

<br>

`PCA`를 이용하여 LFW데이터셋의 얼굴이미지에 특성을 추출하는 아주 간단한 애플리케이션을 만들어보자.


```python
people = fetch_lfw_people(min_faces_per_person=20, resize=0.7)
image_shape = people.images[0].shape

fig, axes = plt.subplots(2, 5, figsize=(15, 8),
                         subplot_kw={'xticks': (), 'yticks': ()})
for target, image, ax in notebook.tqdm(zip(people.target, people.images, axes.ravel())):
    ax.imshow(image)
    ax.set_title(people.target_names[target])
```


    HBox(children=(HTML(value=''), FloatProgress(value=1.0, bar_style='info', layout=Layout(width='20px'), max=1.0…


    
    


![png](ML_PCA_files/ML_PCA_22_2.png)


LFW 데이터셋에는 (클래스 레이블 cnt)62명의 얼굴을 찍은 이미지가 총 3023개 있으며 각 이미지의 크기는 87 X 65픽셀이다. 


```python
print("people.images.shape:", people.images.shape)
print("클래스 개수:", len(people.target_names))
```

    people.images.shape: (3023, 87, 65)
    클래스 개수: 62
    

데이터셋의 편증을 없애기 위해 사람마다 50개의 이미지만 선택하겠다.(이렇게 하지 않으면 조지 부시의 이미지에 치우친 특성이 추출된다.)


```python
# 각 타깃이 나타난 횟수 계산
counts = np.bincount(people.target)
# 타깃별 이름과 횟수 출력
for i, (count, name) in enumerate(zip(counts, people.target_names)):
    print("{0:25} {1:3}".format(name, count), end='   ')
    if (i + 1) % 3 == 0:
        print()
```

    Alejandro Toledo           39   Alvaro Uribe               35   Amelie Mauresmo            21   
    Andre Agassi               36   Angelina Jolie             20   Ariel Sharon               77   
    Arnold Schwarzenegger      42   Atal Bihari Vajpayee       24   Bill Clinton               29   
    Carlos Menem               21   Colin Powell              236   David Beckham              31   
    Donald Rumsfeld           121   George Robertson           22   George W Bush             530   
    Gerhard Schroeder         109   Gloria Macapagal Arroyo    44   Gray Davis                 26   
    Guillermo Coria            30   Hamid Karzai               22   Hans Blix                  39   
    Hugo Chavez                71   Igor Ivanov                20   Jack Straw                 28   
    Jacques Chirac             52   Jean Chretien              55   Jennifer Aniston           21   
    Jennifer Capriati          42   Jennifer Lopez             21   Jeremy Greenstock          24   
    Jiang Zemin                20   John Ashcroft              53   John Negroponte            31   
    Jose Maria Aznar           23   Juan Carlos Ferrero        28   Junichiro Koizumi          60   
    Kofi Annan                 32   Laura Bush                 41   Lindsay Davenport          22   
    Lleyton Hewitt             41   Luiz Inacio Lula da Silva  48   Mahmoud Abbas              29   
    Megawati Sukarnoputri      33   Michael Bloomberg          20   Naomi Watts                22   
    Nestor Kirchner            37   Paul Bremer                20   Pete Sampras               22   
    Recep Tayyip Erdogan       30   Ricardo Lagos              27   Roh Moo-hyun               32   
    Rudolph Giuliani           26   Saddam Hussein             23   Serena Williams            52   
    Silvio Berlusconi          33   Tiger Woods                23   Tom Daschle                25   
    Tom Ridge                  33   Tony Blair                144   Vicente Fox                32   
    Vladimir Putin             49   Winona Ryder               24   


```python
mask = np.zeros(people.target.shape, dtype=np.bool)
for target in np.unique(people.target):
    mask[np.where(people.target == target)[0][:50]] = 1
    
X_people = people.data[mask]
y_people = people.target[mask]

# 0~255 사이의 이미지의 픽셀 값을 0~1 사이로 스케일 조정합니다.
# (옮긴이) MinMaxScaler를 적용하는 것과 거의 동일합니다.
X_people = X_people / 255.
```

<br>

`얼굴 인식`이라 하면 통상적으로 새로운 얼굴 이미지가 DB에 있는 기존 얼굴 중 하나에 속하는지 찾는 작업이다. 이에 대한 방법 중 하나로 각 사람을 서로 다른 클래스로 구분하는 분류기를 만드는 것이다. 하지만 보통 얼굴 DB에는 사람의 수는 많지만 각 사람에 대한 이미지는 적다(즉 클래스별 훈련 데이터가 너무 적다는 의미). 대규모 모델을 다시 훈련시키지 않고도 새로운 사람의 얼굴을 쉽게 추가할 수 있어야 한다.

<br>

간단한 방법으로, 분류하려는 한 사람(k=1)의 얼굴과 가장 비슷한 얼굴 이미지를 찾는 `1-최근접 이웃 분류기법`이 있다.


```python

# 데이터를 훈련 세트와 테스트 세트로 나눕니다
X_train, X_test, y_train, y_test = train_test_split(X_people, y_people, stratify=y_people, random_state=0)
# 이웃 개수를 한 개로 하여 KNeighborsClassifier 모델을 만듭니다
knn = KNeighborsClassifier(n_neighbors=1)
knn.fit(X_train, y_train)
print("1-최근접 이웃의 테스트 세트 점수: {:.2f}".format(knn.score(X_test, y_test)))
```

    1-최근접 이웃의 테스트 세트 점수: 0.23
    

하지만 정확도는 23%이다. 클래스 62개를 분류하는 문제에서 아주 나쁜 결과는 아니지만 그렇다고 좋은 결과도 아니다. 네 번에 한 번 꼴로 올바르게 인식하는 것이다. 그렇다면 `PCA`를 적용하면 어떨까?

<br>

얼굴의 유사도를 측정하기 위해 `원본 픽셀 공간에서 거리를 계산하는 것`<u>은 좋지 않은 방법</u>이다. 이유는 다음과 같다. 픽셀을 사용해서 두 이미지를 바교할 경우 동일한 위치에 있는 픽셀 값과 비교한다. 이런 방식은 픽셀을 비교할 때 얼굴 위치가 한 픽셀만 오른쪽으로 이동해도 완전히 큰 차이를 만들어내며 다른 얼굴로 인식하게 한다. 즉 인접 픽셀끼리의 상관관계가 매우 매우 매우 높기 때문에 이미지를 그대로 사용하면 `redundant`가 발생하는 것이다. 이런 `redundant`<u>를 감소시키기 위한 솔루션으로 </u>`PCA`<u>의 </u>`화이트닝(whitening)`<u>이 있다.</u>

<br>

여기서는 주성분으로 변환후 거리를 계산하여 정확도를 높이는 방법에 대해서 진행할 것이다.<br> `PCA`의 `화이트닝(whitening)` 옵션을 사용해서 <u>주성분의 스케일이 같아지도록 조정</u>하자.

> <b>(참고) `PCA 화이트닝` VS `PCA변환 후 StandardScaler 적용`</b>
>
> <br>
>
> 화이트닝 옵션은 `PCA` 변환할 때 표준편차로 나누어 적용한다.<br>
> 또한 PCA변환에서 데이터의 평균을 빼서 평균을 0으로 만들어 준다. <br>
> 즉 (X-mean)/std이라 볼 수 있다.<br>
>
> <br>
>
> 이는 `PCA`변환을 적용한 뒤에 `StandardScaler`를 적용하는 것과 같다.


```python
mglearn.plots.plot_pca_whitening()
```


![png](ML_PCA_files/ML_PCA_31_0.png)


`PCA`객체를 훈련 데이터로 학습시켜서 (`n_components`)처음 100개의 주성분을 추출한다. 그런 다음 훈련 데이터와 테스트 데이터를 변환한다. 이에 대한 결과로 기존 1547개의 데이터 샘플의 많은 특성들이 `주성분` 100개로 줄어든 것을 확인할 수 있다. 


```python
pca = PCA(n_components=100, whiten=True, random_state=0).fit(X_train)
X_train_pca = pca.transform(X_train)
X_test_pca = pca.transform(X_test)

print("X_train_pca.shape:", X_train_pca.shape)
```

    X_train_pca.shape: (1547, 100)
    

이를 이용해`1-최근접 이웃` 모델을 변환시킨 훈련 데이터로 학습시킨 뒤에 변환시킨 테스트 데이터에 적용했을 때 23%에서 31%로 크게 향상한 것을 확인할 수 있다. 즉 주성분이 데이터를 더 잘 표현한다고 직관적으로 판단 가능하다.


```python
knn = KNeighborsClassifier(n_neighbors=1)
knn.fit(X_train_pca, y_train)
print("테스트 세트 정확도: {:.2f}".format(knn.score(X_test_pca, y_test)))
```

    테스트 세트 정확도: 0.31
    

<br>

또한, `이미지 데이터`<u>일 경우엔</u>`주성분`<u>을 쉽게 시각화</u> 할 수 있다.<br> 이때 주성분은 무엇을 수치로서 확인하기 보다는 <u>입력 데이터 공간에서의 어떤 방향</u>으로 확인 가능하다.


```python
fig, axes = plt.subplots(3, 5, 
                         figsize=(15, 12), 
                         subplot_kw={'xticks': (), 'yticks': ()})b

for i, (component, ax) in enumerate(zip(pca.components_, axes.ravel())):
    ax.imshow(component.reshape(image_shape), cmap='viridis')
    ax.set_title("주성분 {}".format((i + 1)))
```


![png](ML_PCA_files/ML_PCA_37_0.png)


이 주성분들을 완전하게 이해할 수는 없지만, 몇몇 주성분이 잡아낸 얼굴 이미지의 특징을 짐작해 볼 수 있다.<br>
- 주성분 1 : 얼굴과 배경의 명암 차이를 기록
- 주성분 2 : 오른쪽과 왼쪽 조명의 차이
- 등등...

이 `PCA` 모델은 픽셀을 기반으로 하므로, 얼굴의 배치와 조명이 두 이미지가 얼마나 비슷한지 판단하는 데 큰 영향을 준다.

<br>

`PCA` 모델을 이애하는 또 다른 방법은 <u>몇 개의 </u>`주성분`<u>을 사용해 원본 데이터를 재구성하는 것</u>이다. 맨 처음에 살펴보았던 그래프에서 분산이 작은 주성분을 제거하고, 제거한 상태에서 회전을 반대로 되돌리고 평균을 더해서 원래 데이터 공간에 기존 특성 데이터 포인트로 옮겼었다. 같은 방식으로 얼굴 데이터셋에 적용해 몇 개의 주성분으로 데이터를 줄이고 원래 공간으로 되돌릴 수 있다. <u>원래 특성 공간으로 되돌리는 작업</u>은 `inverse_transform` 메서드를 사용한다.


```python
mglearn.plots.plot_pca_faces(X_train, X_test, image_shape)
```

    ________________________________________________________________________________
    [Memory] Calling mglearn.plot_pca.pca_faces...
    pca_faces(array([[0.539869, ..., 0.243137],
           ...,
           [0.043137, ..., 0.593464]], dtype=float32), 
    array([[0.237908, ..., 0.267974],
           ...,
           [0.401307, ..., 0.254902]], dtype=float32))
    ________________________________________________________pca_faces - 1.8s, 0.0min
    


![png](ML_PCA_files/ML_PCA_40_1.png)


주성분을 적게 사용했을 때는 얼굴의 각도, 조명과 같은 이미지의 기본 요소만 확인되지만, 주성분을 더 많이 사용할수록 이미지가 더욱 선명해지는 것을 확인할 수 있다. 주성분을 픽셀 수만큼 사용하면 변환 후에 어떤 정보도 일지 않게 되므로 이미지를 완벽하게 재구성할 수 있을 것이다.

<br>

cancer데이터셋 처럼 `PCA`의 처음 두 주성분을 이용해 전체 데이터를 누구의 얼굴인지 클래스로 구분해 산점도로 나타낼 수도 있다. 하지만, 주성분 두 개만을 사용했을 때는 전체 데이터가 한 덩어리로 뭉쳐 있어 클래스가 잘 구분되지 않는 것을 확인할 수 있다. 이는 위에서 봤던 그림과 똑같은 개념으로 적용된다!


```python
mglearn.discrete_scatter(X_train_pca[:, 0], X_train_pca[:, 1], y_train)
plt.xlabel("첫 번째 주성분")
plt.ylabel("두 번째 주성분")
```




    Text(0, 0.5, '두 번째 주성분')




![png](ML_PCA_files/ML_PCA_42_1.png)


<br>

---

### References

- 안드레아스 뮐러, 세라 가이도, 『파이썬 라이브러리를 활용한 머신러닝』, 박해선, 한빛미디어(2017)
