---
layout: single
date: 2021-04-30
title: "Detection of Propaganda Techniques in News"
description: "Detection of Propaganda Techniques in News"
comments: true
published: true
typora-root-url: ../../
categories: "NLP"
toc: true
toc_sticky: true 
---

(Giovanni Da San Martino , Alberto Barron-Cede , Henning Wachsmuth, Rostislav Petrov and Preslav Nakov)



## Abstract

* 본 논문은 신문 기사에서 나타나는 선전 기술을 탐지해내는 'SemEval-2020 Task 11'대회의 결과 얻어낸 주요 통찰들을 보여주는 것이다.
* 'SemEval-2020 Task 11'는 다음 두 가지 하위 과제들로 구성된다.
  * Span Identification(SI):   주어진 평문(plain-text)에서, 선전 기술이 하나라도 녹아 있는 텍스트 부분을 찾아내는 것. 선전 기술의 유무만을 판단하므로 이진 분류 과제에 속한다.
  * Techinque Classification(TC): 선전 기술이 들어있다고 판단된 텍스트 조각과 그 맥락이 담긴 문서가 주어졌을 때, 구체적으로 어떤 선전 기술이 적용되었는지 파악하는 것. 14개의 선전 기술 중 하나를 정해 판단하는 것이므로 다중 분류 과제에 속한다.
  
  ![image-20210430161442821](/assets/images/NLP/propaganda_detection/image-20210430161442821.png)
* 본고에서는 과제를 소개하고 경진대회 결과를 분석한다. 특히 높은 점수를 기록한 상위 10개 팀이 어떤 방법을 썼는지 살펴본다.
* 결과부터 말해보자면, 두 하위 과제에서의 최고 시스템들은 모두 pre-trained Transformers와 Ensemble기법을 사용했다.





## 1. 소개

### 1.1 연구의 필요성

  선전은 대중의 의견이나 행동에 대해 미리 정해진 방향으로 영향을 미치기 위해 의도적으로 설계된 의견이나 행동들을 일컫는다. 여기서 '다른 이들에게 영향력을 행사하려 한다는  것'과 '의도적으로 그렇게 한다는 것'이 선전의 가장 본질적인 특징이다.

  선전은 독자가 선전의 존재를 알아차리지 못했을 때 가장 큰 효과를 발휘하며 사람들이 선전의 존재를 의식적으로 알아차리기 위해서는 훈련이 필요한 경우가 많다. 

  특히 인터넷 시대에 이러한 선전은 매우 넓은 범위의 대중들에게 도달하곤 하는데 이러한 맥락에서  최근에는 '가짜 뉴스(fake news)'를 탐지해 내는 기술에 대한 관심이 새로운 연구 분야로 부상하고 있다.





##  2. 선전 기술들

  다양한 선전 기술들을 예시를 통해 설명하자면 다음과 같다.

![image-20210430213140911](/assets/images/NLP/propaganda_detection/image-20210430213140911.png)

* 무거운 용어: 도널드 트럼프가 바이러스를 죽이기 위해 살균제를 주입하자고 제안하자 ***광분했다***

* 라벨링: 코로나 바이러스 비상은 ***'첫 번째 공공의 적'*** 이다.
* 반복
* 과장
* 의심
* 두려움/선입견에 호소하기:
* 애국심을 부추기는 표현
* 지나친 단순화
* 슬로건
* 권위에 호소하기
* 흑백논리
* Thought-terminating-cliche
* 비난의 화살 돌리기(whataboutism)
* 허수아비 공격의 오류
* 주의를 다른 곳으로 돌리기(red-herring)
* 유행에 호소하기(Bandwagon)
* Reductio ad hitlerum

여기서 단독으로는 잘 쓰이지 않는 whatabouttism, straw man, red herring 그리고 Bandwagon, reductio ad hitlerum을 각각 한 종류로 묶었다. 그리하여 총 14 종류의 선전 기술들 범위에서 분류가 이루어지도록 하였다.





## 3. 프레임워크 평가

  SemEval 2020 Task 11의 평가 프레임워크는 PTC-SemEval20 말뭉치와 SI와 TC과제들에 대한 측정 지표로 구성되어 있다. 본 장에서는 데이터셋, 측정 지표 그리고 구성 setup들에 대해 집중적으로 설명할 것이다.



### 3.1 PTC-SemEval20

  PTC-SemEval20 말뭉치를 만들기 위해 2017년 중반부터 2019년 후반까지의 기간동안 발행된 신문들 속의 신문 기사들을 수집했다. 주석을 다는 작업은 선전 문구를 발견하고 동시에, 선전 기술들 중 어떤 것에 해당되는지 라벨링을 달아주는 것으로 이루어졌다. 이는 두 단계로 진행되었는데, (i).주석 전문가들 각각 독립적으로 기사들에 라벨을 다는 단계와, (ii).이후 한 주석 전문가만 선전 문구로 라벨링을 한 기사에 대해 의견을 나누고 각자의 결과들을 통합하는 단계가 그것이다.

![image-20210430170545657](/assets/images/NLP/propaganda_detection/image-20210430170545657.png)

(원본 자료와(왼쪽) 주석을 단 결과물(오른쪽))

![image-20210430170927360](/assets/images/NLP/propaganda_detection/image-20210430170927360.png)

  위의 표는 말뭉치 통계값들을 보여준다. 서로 다른 구분에 속해 있는 선전 토막(이하 스니펫) 들이 겹쳐지는 것을 피해야 하기에 SI과제를 위해 주어진 test set 선전 스니펫 수(1405)가 TC과제를 위한 선전 스니펫 수(1790)보다 적었다(무슨 말이지). 

![image-20210430172857437](/assets/images/NLP/propaganda_detection/image-20210430172857437.png)

  536개의 기사들은 8981개의 선전 스니펫들을 담고 있었고 각각의 스니펫들은 14가지의 선전 기술의 종류들에 속해 있었다. 가장 빈번하게 사용된 선전 기술은 '무거운 용어(Loaded Language)'였다. 두번째로 빈번하게 사용된 기술인 '라벨링'보다 2배 가량 더 빈번하게 사용되었음을 알 수 있었다. 가장 빈번하게 사용된 이 두 기술은 비교적 짧은 길이로 표현이 많이 된 반면, '과장', '지나친 단순화', '슬로건'등의 선전 기술들은 길게 표현이 되었음을 위의 그래프로 부터 확인할 수 있다.



### 3.2 평가 지표 분석

* **Subtask SI**: SI과제 수행 성능을 평가하기 위해서는 선전 문구의 길이를 비교하는 과정이 요구된다. 여기서 사용된 SI평가 함수는 실제 선전 문구의 존재 범위와 모델에 의해 예측된 범위를 부분적으로 비교하는 것에서 얻은 점수만으로도 전반적인 예측에 대해 믿을만한 평가 점수를 반환해 준다.

  * d를 D세트 안에 있는 뉴스 기사라고 해 보자. 실제 선전 문구의 존재 범위(이하 gold span)  t는 연속된 글자 인덱스 범위로 표현된다. 

    ![image-20210430174814602](/assets/images/NLP/propaganda_detection/image-20210430174814602.png)

    예를 들어,  위 그림에서 [4, 19]가 실제 선전 문구로부터 뽑아낸 t라고 할 수 있다. 이렇듯 d기사로부터 모든 선전문구들의 인덱스 범위들을 모두 뽑아 Td 셋에 담고(Td = {t1, t2, ***, tn}) 그러한 과정을 다른 기사들에서도 똑같이 반복하여 얻어낸 Tc, Tf, Te 등등을 T set에 담는다.(T = {Td}d). 

    비슷한 과정을 모델이 예측한 gold span에 대해서도 반복하여 기사 d에서 뽑아낸 gold span 들로 이루어진 Sd set과 Sd의 모인 Dset을 만든다. 그런 이후, 정확도 P와 재현율 R을 다음 공식을 적용함으로써 계산해 낸다.

    ![image-20210430175653741](/assets/images/NLP/propaganda_detection/image-20210430175653741.png)

  * 식 (1)의 경우 *|*S*|* = 0이  되는 경우 결과값으로 0을 반환하도록 정의하고, 식 (2)의 경우 *|*T*|*가 0이 되는 경우 결과값을 0으로 반환하도록 정의한다. 예측된 범위가 겹쳐질 수 있음에 유의한다(Figure 4에서  s3과 s4). 그러므로 (1)과 (2)가 1보다 비슷하거나 작은 값들을 반환하도록 하기 위해서는 모든 중첩된 주석들이 그들의 선전 기술 분류와는 독립적으로 우선 병합이 되어야 한다(병합하되 선전 기술에 대한 분류 정보는 남기도록 한다는 말인 듯). 이렇게 병합된 예가 s4이다. 마지막으로 SI에 대한 평가 지표는 F1 score로 다음 식과 같이 P(S,T)와 R(S,T)의 조화평균으로 정의된다.

  ![image-20210430180457325](/assets/images/NLP/propaganda_detection/image-20210430180457325.png)

* **Subtask TC**: 기사의 선전 스니펫이 주어졌을 때, 그 곳에서 어떤 선전 기술이 활용되었는가를 인식해 내는 것이 TC의 내용이다. 동일한 스니펫 범위에 대해 서로 다른 선전 기술이 주석으로 달려 있는 경우가 있기 때문에(전체 주석의 1.8% 정도...) TC는 원래 multi-label multi-class classification(한 분류대상이 여러 개의 카테고리에 속하는 경우에서의 분류)에 속한다. 하지만 본고에서는 다음 조정을 거침으로써 해당 문제를 single-label multi-class 문제(한 분류대상이 하나의 카테고리에 속하는 분류)로 바라보고자 한다.

  * (i).선전 기술이 쓰인 범위(이하 span)가 여러 개의 선전 기술 카테고리와 연관되어 있을 경우, 입력 파일은 그렇게 여러 기술이 중첩되어 있는 span에 대해 여러개의 복사본을 가지고, 
  * (ii).평가 함수는 실제 라벨링과 가장 높은 일치를 보이는 선전 기술에 대한 예측값이 평가에 활용되도록 한다. 즉, 평가 점수는 동일한 spans에서 (어떤 선전 기술이 쓰였는지에 대한) 예측들이 제출된 순서에 의해 영향받지 않는다. 

  TC에 대한 평가 지표는 micro-average F1이다. 우리가 해당 과제를 single-label task로 전환함에 따라 micro-average F1은 Accuracy(as well as to Precision and to Recall)의 값과 정확하게 같아졌음을 참고해야 한다.



### 3.3 과제 구성

  논문의 바탕이 된 대회는 다음과 같이 3단계로 진행되었다.

- **Phase 1**. Development set(Training set이랑 비슷한 개념인 듯)을 이용. Public Leaderboard에서 다른 참가자들보다 더 나은 성능을 내기 위해 경쟁한다.
- **Phase 2**. Test set이 풀리고 난 이후 최종 예측값을 제출하기까지 며칠이 주어진다. 마지막으로 제출된 하나의 submission파일만이 평가되고 이에 기반하여 우승자를 선정한다. 이 단계에서는 모델의 성능에 대한 즉각적인 피드백이 이루어지지 않는다. 
- **Phase 3**. 모든 참가자들은 그들의 system(모델)을 설명하고 다른 이들의 모델을 리뷰하는 내용을 담은 논문을 제출한다. 인용된 논문들은 [International Workshop on Semantic Evaluation - SemEval-2020](http://www.alt.qcri.org/semeval2020) (co-located with [COLING 2020](https://coling2020.org/))에 등재된다.

250팀이 참여하여 44팀만이 test set에 대한 예측값을 공식적으로 제출했다. 

참고) 실제 대회는 끝났지만 여전히 해당 대회에 참여 가능: https://propaganda.qcri.org/semeval2020-task11/  





## 4. 상위 참여자들의 모델 소개

 이번 장에서는 SI와 TC 과제에 참여한 참가자들의 모델을 개괄적으로 소개한다. 성공적인 접근에 특히 집중했다.



### 4.1 Span Identification Subtask(SI)

![image-20210430183039042](/assets/images/NLP/propaganda_detection/image-20210430183039042.png)

위 표는 SI과제에 참여한 상위 시스템들의 구성을 개략적으로 보여준다. 상위 10개의 시스템 모두 LSTM이나 CRF등과 같이 활용하는 등, Transformer을 최소한 어느 정도 활용했다. 거의 모든 경우, Transformer에 의해 추출된 특징들은 개체명들, 주관적 단서, 정서 등의 도메인 지식을 활용하여 추출한 특징들(engineered features) 에 의해 보충되었다.

[^CRF]: MEMM은 시퀀스 데이터 x에 대해서 앞부분 부터 적절한 라벨을 찾아간다.   즉, n번의 순차적인 분류를 수행하지만, CRF는 가능성이 있는 시퀀스 후보를 몇 개 선택하고, 가장 적합한 하나의 라벨을 고른다.
[^engineered features]: Feature engineering is the process of using domain knowledge to extract features from raw data.
[^misc]: miscellaneous: 이것저것 다양한, 여러가지 종류의

[^]: **XLM**-R (**XLM**-**RoBERTa**, Unsupervised Cross-lingual Representation Learning at Scale) is a scaled cross lingual sentence encoder. It is trained on 2.5T of data across 100 languages data filtered from Common Crawl. **XLM**-R achieves state-of-the-arts results on multiple cross lingual benchmarks.



1. Hitachi: SI에서 가장 좋은 성과를 내었다. 연관 부분 세분화와 라벨링 작업들에 있어 보편적으로 활용되는 BIO인코딩을 사용했다(e.g 개체명 인식). 처음부터 끝까지 훈련된 복잡한 heterogeneous 다층 신경망을 활용했다. 이 신경망은 각 입력 토큰마다 특징들을 만들어주는 사전 학습된 언어 모델들을 활용했다. 여기다 Pos(Part of Speech)와 개체명(Named Entity) 임베딩이 추가되었다. 그 결과 각 토큰마다 3개의 특징들이 존재하게 되었는데 이들은 모두 concatenate된 뒤 양방향 LSTM에 입력값으로 활용되었다.  이 순간에 신경망 가지들은 메인 (1)BIO tag를 예측하는 것을 주요 목적으로 하면서 동시에 2가지의 보조 tag:(2)단어 수준의 선전 기술 분류 및 (3)문장 수준의 선전 기술 분류)를 예측하는 것 또한 목적으로 가지기에, 첫 번째와 두 번째를 위한 양방향 LSTM이 있고 세 번째 목적을 위한 또다른 양방향 LSTM이 있다. 전자의 경우 추가로 CRF 층을 가져 산출값에서의 일관성을 높인다.  BERT, GPT-2, XLNet, XLM, RoBERTa, or XLM-RoBERTa와 같이 서로 다른 아키텍쳐들이 각각 독립적으로 훈련된 이후에 마지막에 앙상블을 통해 결합된다.
2. ApplicaAI



### 4.2 Technique Classification Subtask(TC)



## 5. 결과와 이에 대한 해석



### 5.1 SI과제의 결과

  표 5는 SI과제의 테스트 및 개발에 참여하는 시스템의 성능을 보여준다. 
SI 과제의 베이스라인은 무작위적으로 spans를 만드는 단순한 시스템이다. 4장에서 언급되었듯, 실질적으로 모든 접근들이 특징들을 추출하기 위해 Transformers에 의지하였고 만들어진 토큰 수준의 특징들을 sequential model(layers들이 층층이 쌓인 모델)에 입력값으로 넣었다. 

 상위 5개 모델들 중 단지 3 모델만이 test set에 대해서도 상위 5위에 랭크되었다는 점이 눈여겨 볼 만한데, 그것은 test셋에서 순위가 크게 떨어진 팀들의 경우 training set을 학습하는 과정에서 과적합이 발생했을 수 있음을 의미한다. 반면, 순위를 지킨 나머지 3개 팀(Hitachi, ApplicaAI 그리고 aschern)은 보다 일반화(generalize)에 능한 견고한 시스템을 가졌음을 반영한다. 

![image-20210430191538003](/assets/images/NLP/propaganda_detection/image-20210430191538003.png)

  위의 도식 5는 테스트셋에서 좋은 성능을 보인 모델들을 결합했을 경우의 성능 향상을 보여준다. 모든 예측은 글자 단위로 이루어졌다. 여기서 union은 모델간 예측값들의 합집합(한 모델이라도 해당 글자를 선전 스니펫의 일부라고 판단했다면 해당 글자를 선전 문구의 일부라 판단), intersection은 교집합(모든 모델이 해당 글자를 선전 문구라고 인정한 경우에만 flag로 표시), 그리고 majority Voting은 앙상블 기법의 일종으로 각 모형을 통해 나온 분류값들을 투표를 통해 우세한(과반) 분류값을 선정하는 모델을 말한다. 

  이러한 앙상블의 결과는 예상한 대로 나왔다. union에서는 정확도는 낮게, 재현율은 높게 나왔고 intersection에서는 그 반대의 결과가 나왔다. 

[참고. 정밀도와 재현률]

![image-20210430192930011](/assets/images/NLP/propaganda_detection/image-20210430192930011.png)

---

![image-20210430192951796](/assets/images/NLP/propaganda_detection/image-20210430192951796.png)

  만약 우리가 높은 정확도의 모델을 만들고자 한다면 상위 1,2위의 모델들을 결합했을 경우 정확도가 66.95까지 오르는 intersection을 적용하는 것이 좋을 것이다. 하지만 그에 따라 소실되는 스니펫의 양또한 상당하기에 재현률 또한 급감한다(실제 양성인 자료들까지 다수 삭제가 되므로). Majority voting은 정밀도와 재현율 모두에서 신뢰할만한 값을 유지하며 그 사이 어딘가에 있다.



### 5.2 TC과제의 결과

  ![image-20210430193405130](/assets/images/NLP/propaganda_detection/image-20210430193405130.png)

  표6은 TC과제에서 test set에 대해 보인 성능을 나타낸다. 베이스라인에서는 스니펫의 길이만을 특성으로 활용한 로지스틱 회귀 분류 모델을 활용했다. 흥미롭게도, SI과제에서와 유사하게도 development set(이하 train set)에서 상위 5개의 성능을 보인 팀들 중 정작 test set에서는 단지 2개 팀만이 상위 5위 안에 랭크되었다. 자연스럽게도 train set에서는 무난한 성능을 보인 Hitachi(public 리더보드 8등)와 같은 몇몇 모델들이 test set에 대해서는 오히려 더 나은 성능을 보인 것이 관찰되었다. 

  다른 한편, 가장 빈번하고 짤막하게 등장한 선전 기술들인 1(Loaded Language)와 2(Labeling)의 경우 전반적으로 모델들이 잘 예측하였지만 반대로 가장 길고 가끔씩 등장하는 13 (Straw man, red herring)과 14 (Bandwagon, reduction ad hitlerum, whataboutism)은 모델들이 가장 예측하기 어려워한다는 사실을 확인할 수 있다.





## 6. Conclusion

  지금까지 SemEval-2020 Task 11 on **Detection of Propaganda Techniques in News Articles**를 소개했다. 전반적으로 SI이 더 쉬웠고 모든 시스템들이 베이스라인에 비해 성능을 끌어올린 것을 확인할 수 있었다. 반면, TC의 경우는 훨신 어려웠고 몇몇 팀들은 베이스라인보다 성능을 끌어올리지 못했다.

  향후 연구에서는 더 많은 선전 기술들은 물론이고 더 많은 예시들을 포함하도록 데이터셋을 확장해보려고 계획하고 있다. 나아가 다른 언어들에 대해서도 비슷한 데이터셋을 만들어보려 계획 중이다.

  마지막 제언으로 자동 선전 문구 탐지를 사용하는 것에 대한 윤리적인 결과에 대해 경고하고 싶다. 각각의 모델들은 선전을 잘못 탐지할 수 있다. 우리는 자동 선전 탐지를 사용자로 하여금 뉴스에서 사용된 선전을 '스스로' 찾아내는데 도움이 되고 선전에 대한 경각심을 고양하는  도구로 본다. 본고를 마치며, 시스템은 또한 선전처럼 보이는 것들 모두가 실제 선전은 아닐 수 있다는 것을 확실하게 해야 한다는 점을 짚고 싶다.

