---
title: "[추천시스템] DeepWalk & Node2Vec"
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

> GROVER, Aditya; LESKOVEC, Jure. **node2vec: Scalable feature learning for networks**. In: Proceedings of the 22nd ACM SIGKDD international conference on Knowledge discovery and data mining. 2016. p. 855-864.  
> 
> PEROZZI, Bryan; AL-RFOU, Rami; SKIENA, Steven. **Deepwalk: Online learning of social representations**. In: Proceedings of the 20th ACM SIGKDD international conference on Knowledge discovery and data mining. 2014. p. 701-710.

이 블로그 포스팅은 위 논문을 읽고 공부한 내용을 바탕으로 정리한 글입니다.

## Abstract
오늘 소개해드릴 내용은 다음과 같습니다. 우선 Graph에 머신러닝 기법들을 적용하는 것에 대해 간단하게 알아보고, DeepWalk 알고리즘에 대해 알아보고, 그것의 더 일반화된 모델인 Node2Vec을 알아보고 그 후에 실험 결과에 대해 알아보고 마무리 하도록 하겠습니다.


## Representation Learning on Graph
그래프는 아래 그림과 같이 노드와 엣지로 구성된 데이터 형태를 의미한다.
<center><img src = "https://user-images.githubusercontent.com/40378824/106994622-83599500-67c0-11eb-9b9e-fca56dfe5b1e.png" width="100%" height="100%"></center>
그래프를 보면 사람은 직관적으로 여러가지 task를 수행할 수 있다. 대표적으로 노드가 속한 community를 예측하거나 노드 사이의 link를 예측하는 task가 존재한다. 하지만 컴퓨터는 이러한 그래프 구조를 바로 다루지 못한다. 그래서 그래프에 관련된 다양한 Task를 컴퓨터에게 학습시키기 위해서는 그래프를 벡터 공간에 표현해야한다. 이렇게 벡터 공간에 벡터로 표현한 것을 임베딩 (embedding)이라고 하는데, 이러한 임베딩을 바탕으로 Machine Learning Task를 수행해야 한다.  

김치찌개가 맛있으려면 김치찌개에 넣는 김치가 맛있어야 하듯이, 그래프 구조를 기반으로한 Machine Learning Task를 잘 수행하기 위해서는 그래프 임베딩의 품질이 매우 중요하다. [한국어 임베딩 : ratsgo's blog](https://ratsgo.github.io/natural%20language%20processing/2019/09/12/embedding/)

오늘 알아볼 DeepWalk와 Node2Vec은 그래프의 노드들을 벡터공간에 임베딩하는 알고리즘이다.

## DeepWalk
DeepWalk는 social network상의 멤버들을 하나 이상의 카테고리로 묶으려는 시도에서 시작되었다. 이러한 문제를 Relational Classification 문제라고 하는데 기존에는 network structure가 주어졌을때 label 에 대한 Posterior distribution을 추론하는 방식으로 학습을 했다고 한다. ( 이 부분에 대해서는 추가적인 공부가 필요해보인다!)  

하지만 Deep Walk는 그래프 구조와 라벨의 distribution을 분리하고 오직 그래프 구조만을 unsupervised learning을 통해 학습하는 거이다. 즉, 일단 그래프 노드에 각각 라벨이 주어져 있다고 하더라도, 라벨을 생각하지 않고 그래프 구조만을 파악해서 학습을 하겠다는 것이다. 이럴 경우 한번 representation을 생성하면 다양한 task에 사용할 수 있다는 장점이 있고, 예상치 못한 관계를 도출하는 장점도 존재한다.
<center><img src = "https://user-images.githubusercontent.com/40378824/106995842-1bf11480-67c3-11eb-9a34-fc924239e0bf.png" width="100%" height="100%"></center>


### Language Model
DeepWalk를 더 진행하기 전에 언어 모델에 대해서 간단히 짚어 보아야한다. 갑자기 그래프 얘기하다 말고 왠 자연어처리냐 라고 생각할 수 있다. 하지만 DeepWalk의 핵심 아이디어가 자연어처리 알고리즘 Word2Vec 에 기반한다.  

우선 언어 모델의 목적은 corpus (말뭉치)에 있는 단어들의 sequence가 발생할 likelihood를 측정하는데 있다. 이를 수식으로 표현하면, 말뭉치 속의 어떤 단어들이 주어졌을 때, 이 단어 다음 단어가 나올 확률을 최대화한다고 표현을 할 수 있다.

$$
W_1^n=(w_0,w_1,...,w_n)
$$

$$
P(w_n|w_0,w_1,...,w_{n-1})
$$

 이것을 그래프 분야로 가져오려고 잘 생각을 해보면, 그래프 전체를 corpus(말뭉치)라고 생각을 하고, 그래프 속의 노드 하나를  단어라고 생각을 하면, 아래의 식과 같이 노드 sequence의 발생 확률을 최대화 시키는 방식으로 노드를 학습시킬 것이라는 생각을 할 수 있다.

$$
P(v_i|(v_1,v_2,...,v_{i-1}))
$$

Given Mapping Function
$$
 \Phi : v \in V \rightarrow \mathbb{R}^{|V|\times D}
$$

$$
P(v_i|(\Phi(v_1), \Phi(v_2), ..., \Phi(v_{i-1})))
$$


### Power Law
잠깐 언어 모델을 사용하는 것에 의문을 표시할 수 있다. 이를 위해서 단어의 분포와 노드의 분포가 나타내는 특성을 잠깐 확인해보자. 단어의 분포와 노드의 분포가 나타내는 특성은 power law를 통해서도 비슷함을 파악할 수 있다. Power Law란 하나의 수가 다른 수의 거듭 제곱으로 표현되는 두 숫자의 함수적 관계를 의미한다. 여기서 노드들의 degree 분포가 멱법칙을 따른다는 것은, 쉽게 말하자면 소수의 노드들이 다수의 edge에 포함되어야 한다는 것을 의미하는데, 단어 역시 적은 수의 단어가 절대적으로 많이 사용되는 우리들의 일상에 비추어 보았을때, 언어 모델을 모델링 하기 위해서 만들어진 word2vec을 그래프에 적용해도 될 것 같다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107003090-9162e200-67cf-11eb-9cb8-245e3c6cba80.png" width="100%" height="100%"></center>


### Word2Vec
이제 DeepWalk의 핵심 아이디어를 제공하는 Word2Vec에 대해서 알아보자. Word2Vec은 이름 그대로 Word를 Vector로 바꾸어주는 알고리즘이다. Word2Vec은 기존의 언어모델보다 훨씬 뛰어난 성능을 나타내서 주목을 많이 받은 알고리즘이다.  

자연어 처리에 있어서 모델을 설계하는 데 몇가지 가설들이 존재하는데, 그 중에서 Word2Vec은 Distributional Hypothesis라는 가설을 중심으로 설계 되었다. "분포 가설"은 비슷한 문맥 속에 존재하는 단어들은 비슷한 의미를 가질 것이라는 가설이다.  [(자연어 처리에 관해 궁금하다면 이 블로그를 보라)](https://ratsgo.github.io/from%20frequency%20to%20semantics/2017/03/10/frequency/)

잠깐 아래의 예시를 보면, 단어의 주위만 보았는데도 어떤 단어가 적합하고, 어떤 단어가 부적합한지 어느 정도 드러나는 것을 알 수 있다. 이 빈칸에 들어 갈 수 있는 단어들은 서로 비슷한 맥락을 갖는 단어들, 즉 서로 비슷한 단어들이다. 단어의 주변을 보면 그 단어를 알 수 있기 때문에, 단어의 주변이 비슷하면 비슷한 단어라는 말이된다. 이것이 Word2Vec의 발상의 기본이라고 볼 수 있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/106997407-2c56be80-67c6-11eb-81fd-978a9f7c4b13.png" width="70%" height="70%"></center>  

Word2Vec에 대한 설명은 위키독스의 [딥러닝을 이용한 자연어 처리 입문](https://wikidocs.net/22660) 을 참고했습니다.

Word2Vec은 주어진 문장을 window size만큼 잘라가면서 학습하는 방식이다. 문장을 보면 빨간색 단어를 중심으로 양 옆의 window size만큼 학습에 사용하게 된다. 예를 보면, window 사이즈가 2인 예시이다. Cat을 학습시킨다고 했을 때, 양 옆의 단어 the fat과 sat on 이라는 단어들을 활용해서 학습시키는 방식이다.
<center><img src = "https://user-images.githubusercontent.com/40378824/106997633-8fe0ec00-67c6-11eb-80b2-7670f6814584.png" width="70%" height="70%"></center>  
이렇게 윈도우 사이즈만큼 잘라서 학습을 진행하면서 gradient를 업데이트해주기 때문에, 단어들의 sequence정보 또한 중요하지 않게 되고, 단어의 순서가 아닌 "같이 등장하는 단어" 에 집중해서 학습을 하게 된다.  

Word2Vec을 학습하는 방식에는 두가지가 있다. 우선 CBOW 방식은 그림에서 보는 것처럼 주변의 단어를 통해서 중심 단어를 예측하는 방식이다. Skip Gram방식은 이와 반대로 중심 단어를 통해 주변 단어를 예측하는 방식으로 학습을 진행하게 되는데, SkipGram 방식이 CBOW 방식에 비해서 업데이트 횟수가 윈도우 사이즈만큼 더 많이 때문에 Skip Gram 방식이 통상적으로 더 좋은 성능을 내는 것으로 알려져있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/106998121-58267400-67c7-11eb-9f23-cacbc45fa247.png" width="100%" height="100%"></center>
그림을 보면 헷갈릴 수 있지만, back propagation을 생각해보면 Skip gram 방식이 더 많은 업데이트가 가능하다.  

이제 다시 그래프로 돌아가기 전에 몇가지 기억해야할 핵심적인 내용들이 있다.  
1. 하나의 단어를 통해 주변의 단어를 학습시키게 된다.
2. 문맥은 단어의 양쪽을 모두 들여다 보아야한다.
3. 순서에 대한 제약은 중요하지 않다.


### Random Walk
이러한 핵심들을 그래프에 적용하려면 어떻게 해야할까? 우선은 sequence를 만들어야한다. DeepWalk의 핵심 아이디어이다. Random Walk를 통해서 그래프의 노드들의 sequence를 만들어 내겠다는 것이다.

Random Walk란 말 그래도, 랜덤하게 걸어서 연결된 노드들을 방문하는 것이다. 이렇게 그래프 위를 걸어온 길을 그대로 노드들의 sequence로 표현할 수 있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107001160-85295580-67cc-11eb-8f58-4c09aa92a6d1.png" width="100%" height="100%"></center>  

여기서 가장 중요한 부분은 문장들은 단어들의 순서가 유의미하게 존재하지만, 노드들의 시퀀스에서는 걸어온 순서가 존재하지만 사실 이 순서가 아무런 의미가 없다는 것이다. 하지만 word2vec의 방식을 사용하면 노드들의 순서가 아닌 "같이 등장하는 횟수"에 집중을 하면서 그래프에서도 적합한 알고리즘이라는 것을 확인할 수 있었습니다.



### DeepWalk 알고리즘
가장 중요한 것은 DeepWalk가 두개의 중요 부분으로 나누어져 있다는 것이다. Random Walk Generator를 통해서 노드들의 sequence를 생성해내고, 그 노드들의 집합을 활용해서 SkipGram 방식으로 학습시키는 것이다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107004085-1dc1d480-67d1-11eb-8323-e327d2618273.png" width="70%" height="70%"></center>  


### SkipGram 알고리즘
SkipGram을 그림으로 표현하면 다음과 같다. 우선 input으로 전체 그래프의 노드 중 학습에 사용할 노드를 나타내는 one-hot vector를 받게 된다. 이러한 input을 V matrix를 거쳐서 Score vector가 생성된다. 이 score vector를 softmax 방식으로 각각의 노드가 나타날 확률을 구하게 되고, 그 확률과 실제 노드들의 one-hot vector를 비교해서 loss를 update하는 방식이다. 이 때 흥미로운 점은 이 처음 input으로 받는 벡터를 hidden layer의 벡터로 변환해주는 Matrix가 이 자체로 embedding이 된다는 점이다. 이러한 방식을 embedding lookup이라고 한다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107004806-2cf55200-67d2-11eb-9399-539b777a0963.png" width="70%" height="70%"></center>


### Hierarchical Softmax
이제 DeepWalk를 효율적으로 수행하기 위한 두가지 테크닉을 소개한다. 우선 Hierarchical Softmax이다. 수식에서 보듯이, softmax를 할 때마다 그래프 노드의 개수만큼 연산을 해야한다는 비효율이 존재한다. 그래프의 사이즈가 커지거나, 자연어처리 분야에서 전체 문서의 크기가 클 경우 모든 단어들에 대해서 softmax 연산을 수행하는 것은 매우 비효율 적이다. 왜냐하면 학습에 사용되는 단어와 아주 관련이 없는 경우에는 크게 softmax 연산에 영향을 미치지 못하게 되는데, 절대 다수의 단어가 크게 관련이 없기 때문이다.  

Word2Vec에서는 이러한 문제점을 해결하기 위해서 Negative Sampling 방식과 Hierarchical Softmax 방식을 제안한다. DeepWalk에서는 Hierarchical Softmax를 통해서 이러한 비효율을 해결한다.  
Hierarchical Softmax는 이 그림처럼 모든 노드들을 tree의 leaf로 표현해서, 특정한 path로 갈 확률을 최대화하는 방식으로 학습된다. 이럴 경우 이제 시간 복잡도가 log V 만큼 낮아지기 때문에 효율적인 연산이 가능해진다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107005353-f10ebc80-67d2-11eb-98f7-32e3cf8f9521.png" width="100%" height="100%"></center>


### Asynchronous Stochastic Gradient Descent
DeepWalk를 효율적으로 수행하기 위한 두번째 테크닉은 병렬화이다. 논문에서는 효율적인 학습을 위해서 Asynchronous Stochastic Gradient Descent 알골리즘을 사용한다고 되어 있다. 위의 그림을 보면 W97이 종료되면서 parameter version 100번이 업데이트 되어 101번이 되는 것을 확인할 수 있다. 이러한 방식으로 여러개를 병렬적으로 학습시키고 parameter를 공유하여 업데이트하는 방식으로 효율적으로 학습할 수 있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107006209-25cf4380-67d4-11eb-8492-f1ff0a76da23.png" width="100%" height="100%"></center>


## Node2Vec
Node2Vec은 DeepWalk의 Random Walk 방식을 조금 더 generalize한 방식으로 제안한다.  

### Graph Structure
노드를 구성하는 방식에는 homophily 방식과 Structural Equivalence 방식이 존재한다.  
* Homophily 특성은 노드들이 community를 기반으로 비슷한 것들끼리 묶인다는 것이고, 
* structural equivalence는 노드가 그래프에서 어떤 구조적인 역할을 하는지를 기반으로 비슷한 것들끼리 묶이게 된다.
이러한 다양한 구조들을 탐색하기에는 RandomWalk로는 부족하다는 것이다.  

그렇다면 조금 더 구체적으로 살펴보기 위해서 두개의 극단적인 샘플링 방식을 예로 들을 수 있다. 
* BFS : 시작 노드와의 거리가 1인 노드들만 샘플링하는 방식
* DFS : 하나의 스텝마다 시작노드에서부터의 거리가 1씩 늘어나는 방식
논문을 보면 BFS 방식을 사용했을때, structural role을 잘 반영하여 학습이 된다고 하고, 반대로 DFS 방식을 따르면 구조들의 homophily 특성을 잘 반영하여 학습이 된다고 적혀있다. 하지만 직관적으로 보았을 때, BFS 방식이 homophily 특성을 더 잘 반영할 것 같다. (이 부분에 있어서 도움을 주실 수 있으실 수 있는 분이 있다면 연락부탁드립니다!)  


### Random Walk with Parameters
이런 "두개의 극단적인 샘플링 방식을잘 섞으면 좋은 샘플링 방식을 얻을 수 있겠다"는 것이 node2vec의 핵심 아이디어이다. parameter p, q를 활용한 2차 markov transition probability를 사용하게 된다. 아래의 식을 보면 transition probability를 $$p$$와 $$q$$를 통해서 조절하게 된다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107007846-6cbe3880-67d6-11eb-86ac-c4be1b2faa2e.png" width="100%" height="100%"></center>  
간단히 그림에 대해서 설명을 하면,
1. 이전 상태가 t 였는데, 지금은 v에 있는 상황이다.
2. 이럴 경우 다시 t로 돌아갈 수 있고 ($$d_{tx} = 0$$), x1으로 가서 t와 거리가 그대로 1일 수 있고($$d_{tx} = 1$$), x2나 x3로 가서 t와 더욱 멀어지는 선택($$d_{tx} = 2$$)을 할 수 있다.  

만약 p가 커지게 되면 c인 경우가 적어지게 되고, 자신으로 돌아올 확률이 낮아지게 된다. 또한 q가 커지게 되면 $$d_{tx} = 2$$ 인 경우가 적어지게 되고, 새로운 곳을 탐색할 확률이 낮아지게된다. 

다음으로 p와 q를 설정하고 노드를 본 모습이다. q가 작은 위의 그림을 보면 노드가 탐색을 멀리 갈 확률이 높아지게 되고, 그로 인해서 homophily적인 특성을 잘 파악할 수 있다. 또한 아래의 그림에서는 q를 크게 설정해서, 멀리 갈 확률이 낮아지게 되고 이로 인해서 structural role이 비슷한 노드들끼리 같은 색깔을 띄는 것을 확인할 수 있다.
<center><img src = "https://user-images.githubusercontent.com/40378824/107008623-595f9d00-67d7-11eb-9436-9cb1f5d91059.png" width="100%" height="100%"></center>

## Conclusion
마지막으로 지금까지 한 내용을 모두 정리해보려고 한다.
* DeepWalk와 Node2Vec은 노드를 샘플링하여 SkipGram 방식으로 학습한다.
* DeepWalk는 uniform한 Random Walk를 사용한다.
* Node2Vec은 parameter를 사용해서 flexible하고 controllable한 탐색을 가능하게 했다.
DeepWalk와 Node2Vec의 가장 큰 차이점은 노드의 sequence를 만들 때, 샘플링하는 방식이다.
이 외에도 DeepWalk에서 hierarchical softmax를 사용한 것 처럼 Node2Vec에서는 Negative Sampling 방식으로 계산의 complexity를 조절한다.  

DeepWalk와 Node2Vec 모두 input에서 hidden layer로 가는 매트릭스 자체가 embedding matrix로써 역할을 하는데,
이러한 embedding방식을 direct encoding이라고 한다. 이러한 direct encoding 방식에는 몇가지 단점이 존재한다.
* parameter가 share되지 않는다
* 노드에 attribute가 존재할 경우 이를 반영하지 못한다
* 새로운 노드에 대해서 새로운 학습이 없이는 적용이 불가능하다
