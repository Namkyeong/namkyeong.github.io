---
title: "[추천시스템] SoRec: Social Recommendation Using Probabilistic Matrix Factorization"
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

> Ma, Hao, et al. "Sorec: social recommendation using probabilistic matrix factorization." Proceedings of the 17th ACM conference on Information and knowledge management. 2008.

이 블로그 포스팅은 위 논문을 읽고 공부한 내용을 바탕으로 정리한 글입니다.

## Abstract
전통적인 추천 시스템 알고리즘들은 사용자들이 independent and identically distributed하다는 가정하에 모델링되었다. 하지만 현실 세계에서는 친구들끼리 음악을 추천하고, 최근 인기있는 넷플릭스의 작품들을 공유한다. 즉, 현실 세계 속에서의 사용자들은 서로 dependency가 존재하게 된다.   
이 논문에서는 사용자의 **Social Network**와 **평점 기록 데이터**를 활용한 Probabilistic Matrix Factorization을 기반의 모델을 소개한다. 기존의 PMF 모델이 사용자의 latent factor matrix $$U$$를 Rating matrix $$R$$ 로만 학습을 시켰다면, 이 논문에서는 Social Graph Matrix $$C$$를 통해서도 $$U$$를 학습시키게 된다.


## Social Recommendation Framework
이 논문에서 Social Network를 표현할 때 trust라는 데이터를 사용하게 된다. 이때, trust는 단어 뜻 그대로 사용자 $$u_i$$가 사용자 $$ u_j$$를 얼마나 신뢰하는지를 나타낸 지표이다.
<center><img src = "https://user-images.githubusercontent.com/40378824/105324403-b885b500-5c0e-11eb-8967-d991736850ba.PNG" width="100%" height="100%"></center>
그래프를 보면 각각의 사용자의 관계는 $$w_ij$$를 통해 사용자 $$u_i$$가 사용자 $$ u_j$$를 얼마나 신뢰하는지 나타낸다.
예를 들어, 사용자 $$u_3$$가 사용자 $$u_1$$을 0.8만큼 신뢰하는 것을 알 수 있다. 이때 trust의 특징은 양방향이 아니라 단방향이라는 것이다.  
[양방향의 social networking을 다룬 논문도 존재한다.](https://dennyzhou.github.io/papers/RSR.pdf)  

Probabilistic Matrix Factorization에서 사용자의 latent factor matrix U와 아이템의 latent factor matrix V를 이용해서 User의 관측되지 않은 Rating에 대한 예측을 수행했다.  
하지만 SoRec에서는 Social Network Graph의 latenet factor matrix Z를 추가하여 $$U^T V$$와 $$U^T Z$$를 통해 User의 Rating을 예측한다.

<center><img src = "https://user-images.githubusercontent.com/40378824/105329926-0ac9d480-5c15-11eb-9bab-448991afb7c5.PNG" width="70%" height="70%"></center>

### Social Network Matrix Factorization
social network를 표현한 그래프 (그림)를 $$C$$ matrix로 표현할 수 있다. 이 때 matrix를 구성하는 각각의 $$c_{ik}$$는 사용자 i가 사용자 k를 얼마나 trust하는지 나타낸다고 볼 수 있다. Rating을 matrix Factorization 하는 것과 마찬가지로 Social Network 역시 $$U$$와 $$Z$$로 factorize할 수 있다. Rating Matrix 뿐만 아니라 **Social Network Graph를 나타낸 matrix를 factorize하여 사용자에 대한 latent matrix $$U$$를 학습시키는 것**이 이 논문의 핵심 주제라고 할 수 있다.
관측된 Social network relationship에 대한 조건부 확률은 다음과 같이 정의할 수 있다.

$$
p(C|U,Z,\sigma_C^2) = \prod_{i=1}^{m}\prod_{k=1}^{m}N[(c_{ik}|g(U_i^TZ_k), \sigma_C^2)]^{I_{ik}^C}
$$

이때 $$g(x)$$는 logistic function으로 trust값을 0과 1 사이로 제약한다. PMF 논문에서와 마찬가지로 matrix $$U$$, $$Z$$ 각각 Gaussian Prior를 가정한다. 즉,  

$$
p(U|\sigma_U^2) = \prod_{i=1}^{m}N(U_i|0, \sigma_U^2I)
$$  

$$
p(Z|\sigma_Z^2) = \prod_{i=1}^{m}N(Z_k|0, \sigma_Z^2I)
$$  

과 같이 각각의 latent factor matrix에 Gaussian Prior를 부여한다.

간단한 베이즈 추론을 통해 다음과 같은 결과를 얻을 수 있다.   

$$
p(U,Z|C, \sigma_C^2, \sigma_U^2, \sigma_Z^2)
$$  

$$
 \propto p(C|U,Z,\sigma_C^2)p(U|\sigma_U^2)p(Z|\sigma_Z^2)
$$  
 
$$
= \prod_{i=1}^{m}\prod_{k=1}^{m}N[(c_{ik}|g(U_i^TZ_k), \sigma_C^2)]^{I_{ik}^C}
\prod_{i=1}^{m}N(U_i|0, \sigma_U^2I)\prod_{i=1}^{m}N(Z_k|0, \sigma_Z^2I)
$$

즉, U와 Z 의 Posterior 확률을 U와 Z의 Prior와 C에 대한 likelihood로 표현할 수 있다. (이 부분에 대해서는 Bayesian에 관련해서 더 공부가 필요해 보인다.)  


### User-Item Matrix Factorization
User와 Item에 관련한 Matrix Factorization은  Social Network Matrix Factorization 방식과 동일하다. 다만 Social Network Matrix $$C$$가 아닌 사용자가 아이템에 대해 평점을 매긴 Rating Matrix 에 대해서 사용자의 latent matrix $$U$$와 아이템의 latent matrix $$I$$로 factorization한 것이다.


### Matrix Factorization for Social Recommendation
이제 최종적인 모델을 볼 차례이다. 종합적으로 보았을 때, Social Network matrix $$C$$를 $$U$$와 $$Z$$에 관한 factorization과 Rating matrix $$R$$을 $$U$$와 $$V$$에 관한 factorization이 함께 이루어지게 된다.  
이를 log posterior로 표현하면 다음과 같다.

$$
\ln p(U,V,Z|C,R, \sigma_C^2,\sigma_R^2, \sigma_U^2,\sigma_V^2, \sigma_Z^2) = \\
$$

$$
-\frac{1}{2\sigma_R^2} \sum_{i=1}^{m}\sum_{j=1}^{n} I_{ij}^{R}(r_{ij}-g(U_{i}^{T}V_{j}))^2 \\
$$

$$
-\frac{1}{2\sigma_C^2} \sum_{i=1}^{m}\sum_{k=1}^{m} I_{ij}^{C}(c_{ik}^*-g(U_{i}^{T}Z_{k}))^2 \\
$$

$$
-\frac{1}{2\sigma_U^2} \sum_{i=1}^{m}U_{i}^{T}U_{i} -\frac{1}{2\sigma_V^2} \sum_{i=1}^{n}V_{j}^{T}V_{j} -\frac{1}{2\sigma_Z^2} \sum_{k=1}^{m}Z_{k}^{T}Z_{k} \\ 
$$

$$
-\frac{1}{2}((\sum_{i=1}^{m}\sum_{j=1}^{n}I_{ij}^{R})\ln\sigma_R^2 + (\sum_{i=1}^{m}\sum_{k=1}^{m}I_{ik}^{C})\ln\sigma_C^2) \\
$$

$$
-\frac{1}{2}(ml\ln\sigma_R^2 + nl\ln\sigma_V^2 + ml\ln\sigma_Z^2) + C
$$

이 log posterior를 최대화하는것은 아래의 loss function을 최소화하는 것과 동일한 문제가 된다.

$$
\mathfrak{L}(R, C, U, V, Z) = \\
$$

$$
\frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{n} I_{ij}^{R}(r_{ij}-g(U_{i}^{T}V_{j}))^2 + \frac{\lambda_{C}}{2}\sum_{i=1}^{m}\sum_{k=1}^{m} I_{ik}^{C}(c_{ik}^*-g(U_{i}^{T}Z_{k}))^2 \\
$$

$$
+\frac{\lambda_{U}}{2}\left \| U \right \|_{F}^{2} +\frac{\lambda_{V}}{2}\left \| V \right \|_{F}^{2} + \frac{\lambda_{Z}}{2}\left \| Z \right \|_{F}^{2} 
$$
