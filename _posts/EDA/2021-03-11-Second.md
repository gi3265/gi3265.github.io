---
layout: single
title: "Github Blog에 사진 업로드"
description: "Github Blog에 사진 업로드하면서 동시에 해당 사진 Typora 통해서 볼 수 있도록 하기"
comments: true
published: true
categories: "Practice"
typora-root-url: ../../
---

### Typora로 사진 첨부하여 github blog에 올릴 때

Typora에 사진 로드:  typora md파일이 위치한 디렉토리가 이미지를 찾는 default 디렉토리다. 

Github에 사진 업로드: 로컬 저장소(.git 폴더 위치한 폴더)가 이미지를 찾는 default 디렉토리다.

두 장소에 같은 경로를 통해 동시에 업로드 해야 하는데 서로 이미지를 탐색하기 시작하는 default 디렉토리가

다르다 보니 typora.md의  yaml front matter에서 typora-root-url 속성을 통해 typora의 기본 경로를 github  기본 경로에 맞춘다.

plz...
![2021-03-10-COVID19_Trash](/assets/images/EDA/2021-03-10-COVID19_Trash.png)