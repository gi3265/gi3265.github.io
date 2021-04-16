---
layout: single
date: 2021-04-15
title: "T아카데미 NLP 모델 BERT"
description: "BERT description"
comments: true
published: true
typora-root-url: ../../
categories: "NLP"
toc: true
toc_sticky: true 
---

# [1강]- NLP

출처: https://www.youtube.com/watch?v=qlxrXX5uBoU

## 1.1 다양한 임베딩 방식들



### 1.1.1 Word2vec 

* 자연어(특히, 단어)의 의미를 벡터 공간에 임베딩.

* 한 단어의 주변 단어들을 통해 해당 단어의 의미를 파악.

* 장점: 
  *  단어간의 유사도 측정하기 용이
  * 단어간 관계 파악에 용이
  * 벡터 연산을 통한 추론 가능(e.g. 한국 - 서울 +도쿄  = ?) 

* 단점:
  * 단어의 subword information무시(이를 보완한 것이 n-gram이용한 fasttext)
  * OOV(out of vocabulary)에서 적용 불가능



### 1.1.2 FASTTEXT

한국어는 다양한 용언 형태를 가진다.(e.g. 모르네, 모르데, 모르지, 모르구나, ***)

Word2Vec의 경우 이러한 다양한 용언 형태를 모두 각각 다른 어휘로 인식한다는 한계를 가짐.

---

<용언이 변화함에도 그 의미는 비슷하다는 것을 어떻게 알 수 있을까?>

* Fasttext는 단어를 n-gram으로 나누어 이를 모두 학습시킴으로서 subtext에 대한 인식 가능성을 높임(e.g. assumption을 학습시킨다고 할 경우 'as', 'ss','su', ***, 'assumption'으로 n-gram vectors로 쪼개어 임베딩. 결과적으로 모델은 assump이라는 어휘와 assumption이 별개의 말이 아니라는 것을 인식할 수 있음)
* Fasttext 장점:
  * 오탈자, OOV, 등장 횟수가 적은 학습 단어에 대해서도 학습 효과가 좋다

* Word2Vec나 FastText의 단점:
  * 동형어, 다의어 등에 대해서는 embedding 성능이 좋지 못함.
  * 주변 ''단어''를 통해 학습이 이루어지기에 '문맥'을 고려할 수 없음.

### 1.1.3 기계 독해와 BERT

Bert로 인해 기계 독해 분야가 빠르게 성장하기 시작.

기계 독해란? 한 마디로 Semantic Analysis. 의미를 분석하는 것. 기계가 글을 읽어 사람이 사전에 질문들에 대한 답을 미리 입력해 주지 않아도 기계가 알아서 질문과 글을 이해하고 질문에 적절한 답을 제공해 줄 수 있음.

의미 해석은 결국 분류의 문제이다. ex) 최초의 컴퓨터가 뭐야? <- 질문: 98%, 요구: 0.5%, 거절: 0.025%, 승낙: 0.025%



## 1.2 워드 임베딩의 활용

* 토픽 키워드 추출할 때 활용
* 다른 자연어처리 모델의 입력으로 사용(e.g. 학습 데이터의 양이 매우 적은 감성분석을 사용할 경우, 학습 데이터만으로는 특성 추출 불가)



# [2강] - 자연어 모델

출처: https://www.youtube.com/watch?v=zia49ZyKiX0

## 1. RNN

### 1.1 RNN의 구조적 문제점

* 입력 sequence의 길이가 매우 긴 경우, 처음에 나온 token에 대한 정보가 희석
* 고정된 conext vector 사이즈로 인해 긴 sequence에 대한 정보를 함축하기 어려움
* 모든 token이 영향을 미치니, 중요하지 않은 token도 영향을 줌



## 2. Attention 모델

### 2.1 Attention모델의 의의

RNN 모델의 단점을 극복하고자 탄생한 모델.

* 인간이 정보처리를 할 때, 모든 sequence를 고려하면서 정보처리를 하는 것이 아님
* 인간의 정보처리와 마찬가지로, 중요한 feature은 더욱 중요하게 고려하는 것이 Attention의 Motiv

### 2.2 Mechanism

핵심 아이디어: hidden state만 다음 유닛으로 넘긴 채, ouput 벡터의 값을 버리던 기존의 RNN 방식 버리고 output벡터의 정보를 최대한 활용해 보자는 것.

![Attention_Mechanism](/assets/images/NLP/Attention_Mechanism.png)

### 2.3 Self-attention 모델의 등장

#### "Attention is all you need!" ← 논문의 제목

IDEA: RNN의 기본 방식 굳이 써야 해? 없애버리자. -> RNN에서 encoder과 decoder 제거

![image-20210416024444456](/assets/images/NLP/image-20210416024444456.png)

-학습 과정-

1. Decoder 결과가 정답과 많이 다름:

* 좋지 못한 context vector -> 좋지 못한 attention weight
* Fully connected feed forward network에서 score 조정 -> s1~s3 조정됨에 따라 Attention weight도 조정

2. 기존 RNN + Attention 방식: decoder가 해석하기에 가장 적합한 weight 찾고자 노력
3. Attention이 decoder이 아니라, input인 값을 가장 잘 표현할 수 있도록 학습하면?

-> 자기 자신을 가장 잘 표현할 수 있는 좋은 임베딩

4. Self-Attention 모델의 탄생!



### 2.4 Self-Attention 모델의 작동 원리

Query, Key, Value로 구성된 Attention layers를 동시에 여러개를 수행(<-효율적인 병렬 처리.GPU 여러대 필요로 함.)

RNN은 이전 state를 기다리고 그 다음에야 다음 state를 예측하는 방식으로 수행되는 점과 대비됨.

![image-20210416030154527](/assets/images/NLP/image-20210416030154527.png)



# BERT

* Self-Attention의 심화 버전: Multi-head Attention을 12개를 달아버림(그림은 3개가 달려있는 것)

![image-20210416030426427](/assets/images/NLP/image-20210416030426427.png)















