---
title: "[추천시스템] Factorization Machines"
layout: single
author_profile: true
read_time: false
comments: true
share: true
related: true
categories:
- Recommender System
use_math: true
toc: true
toc_sticky: true
toc_label: 목차
description: Factorization Machines 논문 리뷰
---

> Rendle, Steffen. "Factorization machines." 2010 IEEE International Conference on Data Mining. IEEE, 2010.  

이 블로그 포스팅은 위 논문을 읽고 공부한 내용을 바탕으로 정리한 글입니다.


## Abstract
Factorization Machine은 기존의 추천시스템에서 많이 사용되는 Matrix Factorization 기반의 모델과 Support Vector Machine(이하 SVM)의 장점을 모두 가진 모델이다.  

Factorization Machine은 SVM과 같이 범용성이 높은 모델에 속하지만 SVM과 달리 변수 사이의 관계를 factorized된 parameter를 이용하여 표현하기 때문에,  
dense 하지 않은 데이터셋에서 좋은 성능을 발휘하지 못하는 SVM과 달리 좋은 성능을 발휘할 수 있다.

또한 기존에 추천시스템에서 자주 사용되는 Matrix Factorization 기반의 모델은 범용적인 사용이 어렵고, 특정한 데이터에 대해서만 성능을 발휘한다는 문제를 가지고 있는데, Factorization Machine은 이러한 범용성에 대한 향상을 가지고 왔다.

## Introduction
이 논문에서는 SVM과 같이 범용적이 prediction task에 사용할 수 있고, 추천 시스템 데이터셋의 sparse한 특성에도 잘 대응할 수 있는 Factorization Machine이라는 알고리즘을 소개한다.

논문에서 제시하는 Factorization Machine의 장점은 다음과 같다.
1. SVM과 달리 Sparse한 데이터셋에서도 parameter의 측정이 가능하다.
2. Factorization Machine은 Linear한 복잡도를 갖는다.
3. 어떠한 데이터에 대해서도 대응할 수 있는 범용성을 갖는다.

## Prediction Under Sparsity
<center><img src = "https://user-images.githubusercontent.com/40378824/105278745-78f8a200-5be9-11eb-9906-b69521d591d6.PNG" width="90%" height="90%"></center>  


데이터셋의 X(1)은 각각 데이터의 feature vector를 의미한다. 하나의 데이터안에는 어떤 사용자(User)가 어떤 영화(Movie)를 감상했으며, 그 사용자가 지금까지 어떤 영화를 감상했고(Others Movies rated), 그 영화를 언제 감상했고(Time), 마지막으로 본 영화가 무엇인지(Last Movie rated) 나타나 있다.


## Factorization Machine
### Factorization Machine Model
<center><img src = "https://user-images.githubusercontent.com/40378824/105279241-86625c00-5bea-11eb-88d9-8f542d794c47.PNG" width="90%" height="90%"></center>  
degree가 2인 Factorization Machine은 위의 식과 같이 정의된다.  

<center><img src = "https://user-images.githubusercontent.com/40378824/105279413-d4775f80-5bea-11eb-9381-733a3c5828d3.PNG" width="50%" height="50%"></center>  
우리가 학습시켜야하는 모델의 parameter는 위의 3개인데, w_0는 실수, w_0 는 n차원의 vector, V는 (n x k) 행렬이다.    

w_0는 전체적인 y 추정값을 나타낼 때 사용되는 global bias를 의미하고,  
w는 위의 데이터셋에 존재하는 각각의 변수가 얼마나 영향력을 끼치는지 의미한다.  
V  행렬을 통해 변수를 벡터 공간에 임베딩하게 되는데, Vi, Vj 벡터의 내적을 통해서 i번째 변수와 j번째 변수의 관계를 모델링하게 된다.  

**i번째 변수와 j번째 변수의 interaction을 V라는 matrix를 통해서 Factorization 시킨 것**이 이 논문의 핵심적인 아이디어이다.
두 변수 사이의  interaction을 나타내는 matrix W (n x n)는 각각의 변수를 벡터공간에 임베딩한 latent matrix인 V로 표현하게 되는 것이다.

Sparse한 데이터셋에서는 변수 사이의 관계를 직접적으로 나타내는 데이터가 많이 존재하지 않는다. 따라서 이러한 문제를 해결할 필요가 있는데, Factorization Machine은 이러한 관계를 Factorization함으로써 "관측되지 않은 변수 사이의 관계"를 "관측된 변수 사이의 관계"를 통해서 예측할 수 있게 된다.


Factorization Machine은 계산복잡도 측면에서도 linear하다는 장점을 가지게 된다. 계산복잡도가 linear 할 경우 입력 데이터의 크기에 비례해서 처리시간이 들게 되는데, 다른 알고리즘에 비해서 계산적인 측면에서 장점을 갖는다고 볼 수 있다.

### Factorization Machines as Predictors
Factorization machine은 다양한 예측 문제에 적용 가능하다.

1. Regression
2. Binary Classification
3. Ranking

### Learning Factorization Machines
Factorization Machine 모델은 Gradient Descent 알고리즘을 통해서 학습될 수 있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/105281442-84e76280-5bef-11eb-9cda-cbe22117e5bf.PNG" width="70%" height="70%"></center>
각각의 parameter에 대한 편미분은 위의 식과 같다.

### d-way Factorization Machine
지금까지 다룬 모델은 2-way (degree = 2)인 Factorization Machine을 다루었다. degree가 2라는 것은 추정치를 추정할 때 변수 조합의 후보를 2개로 하겠다는 의미이다.
이것을 degree d로 쉽게 일반화 할 수 있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/105282317-64b8a300-5bf1-11eb-9792-dc14683282ed.PNG" width="70%" height="70%"></center>


## FMs vs. SVMs
SVM에서 input x는 feature space에서 더 복잡한 공간인 F로 매핑되는데, 이때 변형된 공간에서의 변형된 x와 모델의 parameter w의 내적으로 y의 추정치를 산출하게 된다.
이때 매핑하는 함수는 커널(Kernel)과 관련이 되는데, 이 부분은 SVM을 공부하면서 자세히 공부해 볼 필요가 있다. ([참고: ratsgo's blog](https://ratsgo.github.io/machine%20learning/2017/05/30/SVM3/))

그중 가장 간단한  linear kernel은 degree=1인 Factorization Machine과 동일하다.
Polynomial Kernel은 SVM이 변수들간의 더 높은 interaction을 모델링할 수 있게 한다. 2차원의 Polynomial SVM과 degree=2인 Factorization Machine을 비교해보면 두 모델 모두 변수 사이의 interaction을 2차원으로 모델링하였다. 하지만 두 모델의 차이는 **parametrization**에 있었다. SVM에서 변수들 사이의 관계는 각각 독립적이지만, FM에서는 두 변수의 관계를 factorize하여 서로 dependent하게 조정한 것이다.

## FMs vs. other Factorization Models
Matrix Factorization은 추천 시스템에서 전통적으로 사용되는 모델이다. Matrix Factorization은 두개의 categorical 변수를 factorize하는데,  추천시스템에서는 Rating Matrix를 User latent matrix와 Item latent matrix로 factorize하게 된다. Factorization Machine은 Matrix Factorization 기반 모델의 일반화된 버전이라고 볼 수 있다.


## Conclusion
Factorization Machine은 SVM의 범용성과 sparsity에 대응하는 능력을 모두 갖춘 모델이다.
SVM과 달리 FM은 sparse한 데이터에서도 reliable하게 모델을 학습시킬 수 있으며, 
Linear한 시간복잡도를 가진다. 이러한 성격 덕분에 SVM과 달리 Duality를 통한 optimization이 아닌 Primal을 통한 optimization이 가능하다.

또한 다른 Tensor Factorization 기반의 방법론에 비해서 범용성이 높다는 장점을 가지고 있다는 것을 알 수 있다.


## Implementation
다음은 공부를 하면서 Pytorch로 Matrix Factorization을 구현한 내용입니다.  
많이 부족하므로 문제가 있는 경우 알려주시면 감사하겠습니다.  
[Implementation](https://github.com/Namkyeong/RecSys_paper/tree/main/FactorizationMachine)
