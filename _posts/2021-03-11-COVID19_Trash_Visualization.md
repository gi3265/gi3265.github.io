---
layout: page
title: "COVID19_Trash_Data_Visualization"
description: "."
comments: true
published: true
typora-a-copy-images-to: C:\Users\sgi40\OneDrive\WallPaper\github\Blog_Image
---





# 필요 라이브러리 불러오기


```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import math
from datetime import date, timedelta

import warnings
warnings.filterwarnings("ignore")

# 한글 폰트 불러오기
%matplotlib inline
import matplotlib.font_manager as fm
font_path = 'C:/Users/sgi40/NanumFontSetup_TTF_ALL/NanumBarunGothic.ttf'
font_name = fm.FontProperties(fname=font_path).get_name()
plt.rc('font', family=font_name)
font_name

# 한글 문제 대응하기
import matplotlib as mpl
%config InlineBackend.figure_format = 'retina'
font = fm.FontProperties(fname=font_path, size=9)
plt.rc('font', family='Malgun Gothic') 
mpl.font_manager._rebuild()

#Plotly 활용 준비
import plotly
import plotly.graph_objs as go
from plotly import tools
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
import matplotlib.animation as animation
from IPython.display import set_matplotlib_formats
set_matplotlib_formats("retina")
```

# 재택지수 데이터 파악

## 데이터 형태 파악


```python
working_home = pd.read_csv('data/Working_Home_Idx.csv')
working_home.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>age_cd</th>
      <th>sex_cd</th>
      <th>home_sido_nm</th>
      <th>h0d0h0_dur_r</th>
      <th>h0d0h1_dur_r</th>
      <th>h0d1h0_dur_r</th>
      <th>h0d1h1_dur_r</th>
      <th>h1d0h0_dur_r</th>
      <th>h1d0h1_dur_r</th>
      <th>h1d1h0_dur_r</th>
      <th>h1d1h1_dur_r</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20181029</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>경상남도</td>
      <td>0.166836</td>
      <td>0.833054</td>
      <td>0.467698</td>
      <td>0.532190</td>
      <td>0.203548</td>
      <td>0.795637</td>
      <td>0.407189</td>
      <td>0.591995</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20181029</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>전라북도</td>
      <td>0.174534</td>
      <td>0.825149</td>
      <td>0.471113</td>
      <td>0.528571</td>
      <td>0.204232</td>
      <td>0.793462</td>
      <td>0.408986</td>
      <td>0.588710</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20181029</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>전라남도</td>
      <td>0.173301</td>
      <td>0.826440</td>
      <td>0.461816</td>
      <td>0.537923</td>
      <td>0.210939</td>
      <td>0.787248</td>
      <td>0.416458</td>
      <td>0.581742</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20181029</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>충청남도</td>
      <td>0.175661</td>
      <td>0.823899</td>
      <td>0.475354</td>
      <td>0.524204</td>
      <td>0.239265</td>
      <td>0.758896</td>
      <td>0.427152</td>
      <td>0.571001</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20181029</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.169477</td>
      <td>0.830302</td>
      <td>0.484912</td>
      <td>0.514864</td>
      <td>0.197705</td>
      <td>0.801207</td>
      <td>0.388608</td>
      <td>0.610308</td>
    </tr>
  </tbody>
</table>
</div>



## 열 조회 및 열의 자료 수, 자료형, 누락값 여부 확인


```python
working_home.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2772 entries, 0 to 2771
    Data columns (total 12 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   dt            2772 non-null   int64  
     1   age_cd        792 non-null    float64
     2   sex_cd        198 non-null    float64
     3   home_sido_nm  1683 non-null   object 
     4   h0d0h0_dur_r  2772 non-null   float64
     5   h0d0h1_dur_r  2772 non-null   float64
     6   h0d1h0_dur_r  2772 non-null   float64
     7   h0d1h1_dur_r  2772 non-null   float64
     8   h1d0h0_dur_r  2772 non-null   float64
     9   h1d0h1_dur_r  2772 non-null   float64
     10  h1d1h0_dur_r  2772 non-null   float64
     11  h1d1h1_dur_r  2772 non-null   float64
    dtypes: float64(10), int64(1), object(1)
    memory usage: 260.0+ KB


# 주간 평균 값을 담은 행들만으로 데이터프레임 재구성


```python
#연령별, 성별, 시도별 행이 모두 NaN 인 행들만 추출
home_weekly = working_home[(working_home['age_cd'].isnull()) & (working_home['sex_cd'].isnull()) & (working_home['home_sido_nm'].isnull())]
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>age_cd</th>
      <th>sex_cd</th>
      <th>home_sido_nm</th>
      <th>h0d0h0_dur_r</th>
      <th>h0d0h1_dur_r</th>
      <th>h0d1h0_dur_r</th>
      <th>h0d1h1_dur_r</th>
      <th>h1d0h0_dur_r</th>
      <th>h1d0h1_dur_r</th>
      <th>h1d1h0_dur_r</th>
      <th>h1d1h1_dur_r</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>20181029</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.169477</td>
      <td>0.830302</td>
      <td>0.484912</td>
      <td>0.514864</td>
      <td>0.197705</td>
      <td>0.801207</td>
      <td>0.388608</td>
      <td>0.610308</td>
    </tr>
    <tr>
      <th>39</th>
      <td>20181105</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.167393</td>
      <td>0.831570</td>
      <td>0.478517</td>
      <td>0.520444</td>
      <td>0.193548</td>
      <td>0.806157</td>
      <td>0.375405</td>
      <td>0.624302</td>
    </tr>
    <tr>
      <th>80</th>
      <td>20181112</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.168209</td>
      <td>0.831380</td>
      <td>0.479196</td>
      <td>0.520392</td>
      <td>0.196675</td>
      <td>0.803020</td>
      <td>0.378299</td>
      <td>0.621397</td>
    </tr>
    <tr>
      <th>94</th>
      <td>20181119</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.167206</td>
      <td>0.832517</td>
      <td>0.477306</td>
      <td>0.522416</td>
      <td>0.191222</td>
      <td>0.808450</td>
      <td>0.357341</td>
      <td>0.642333</td>
    </tr>
    <tr>
      <th>129</th>
      <td>20181126</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.167841</td>
      <td>0.831863</td>
      <td>0.477450</td>
      <td>0.522253</td>
      <td>0.189658</td>
      <td>0.809217</td>
      <td>0.360841</td>
      <td>0.638037</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2633</th>
      <td>20200817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.164157</td>
      <td>0.835696</td>
      <td>0.428228</td>
      <td>0.571625</td>
      <td>0.164748</td>
      <td>0.835187</td>
      <td>0.304974</td>
      <td>0.694960</td>
    </tr>
    <tr>
      <th>2681</th>
      <td>20200824</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.146649</td>
      <td>0.853237</td>
      <td>0.403955</td>
      <td>0.595931</td>
      <td>0.148492</td>
      <td>0.851268</td>
      <td>0.268423</td>
      <td>0.731337</td>
    </tr>
    <tr>
      <th>2688</th>
      <td>20200831</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.141435</td>
      <td>0.858467</td>
      <td>0.399269</td>
      <td>0.600633</td>
      <td>0.149267</td>
      <td>0.850167</td>
      <td>0.275415</td>
      <td>0.724018</td>
    </tr>
    <tr>
      <th>2733</th>
      <td>20200907</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.143690</td>
      <td>0.855593</td>
      <td>0.404244</td>
      <td>0.595038</td>
      <td>0.158167</td>
      <td>0.841622</td>
      <td>0.296118</td>
      <td>0.703672</td>
    </tr>
    <tr>
      <th>2744</th>
      <td>20200914</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.150234</td>
      <td>0.849636</td>
      <td>0.424249</td>
      <td>0.575621</td>
      <td>0.164736</td>
      <td>0.835069</td>
      <td>0.318531</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 12 columns</p>
</div>



# 필요 열만 추출


```python
home_weekly.columns
```




    Index(['dt', 'age_cd', 'sex_cd', 'home_sido_nm', 'h0d0h0_dur_r',
           'h0d0h1_dur_r', 'h0d1h0_dur_r', 'h0d1h1_dur_r', 'h1d0h0_dur_r',
           'h1d0h1_dur_r', 'h1d1h0_dur_r', 'h1d1h1_dur_r'],
          dtype='object')




```python
home_weekly = home_weekly.drop(['age_cd', 'sex_cd', 'home_sido_nm'], axis = 1)
for idx, col_nm in enumerate(home_weekly.columns):
    if idx%2 == 1:
        home_weekly = home_weekly.drop(col_nm, axis = 1)
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>h0d0h1_dur_r</th>
      <th>h0d1h1_dur_r</th>
      <th>h1d0h1_dur_r</th>
      <th>h1d1h1_dur_r</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>20181029</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
    </tr>
    <tr>
      <th>39</th>
      <td>20181105</td>
      <td>0.831570</td>
      <td>0.520444</td>
      <td>0.806157</td>
      <td>0.624302</td>
    </tr>
    <tr>
      <th>80</th>
      <td>20181112</td>
      <td>0.831380</td>
      <td>0.520392</td>
      <td>0.803020</td>
      <td>0.621397</td>
    </tr>
    <tr>
      <th>94</th>
      <td>20181119</td>
      <td>0.832517</td>
      <td>0.522416</td>
      <td>0.808450</td>
      <td>0.642333</td>
    </tr>
    <tr>
      <th>129</th>
      <td>20181126</td>
      <td>0.831863</td>
      <td>0.522253</td>
      <td>0.809217</td>
      <td>0.638037</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2633</th>
      <td>20200817</td>
      <td>0.835696</td>
      <td>0.571625</td>
      <td>0.835187</td>
      <td>0.694960</td>
    </tr>
    <tr>
      <th>2681</th>
      <td>20200824</td>
      <td>0.853237</td>
      <td>0.595931</td>
      <td>0.851268</td>
      <td>0.731337</td>
    </tr>
    <tr>
      <th>2688</th>
      <td>20200831</td>
      <td>0.858467</td>
      <td>0.600633</td>
      <td>0.850167</td>
      <td>0.724018</td>
    </tr>
    <tr>
      <th>2733</th>
      <td>20200907</td>
      <td>0.855593</td>
      <td>0.595038</td>
      <td>0.841622</td>
      <td>0.703672</td>
    </tr>
    <tr>
      <th>2744</th>
      <td>20200914</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 5 columns</p>
</div>



## 열이름 재지정


```python
home_weekly.columns = ['dt', 'normal_night', 'normal_day', 'end_night', 'end_day']
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>20181029</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
    </tr>
    <tr>
      <th>39</th>
      <td>20181105</td>
      <td>0.831570</td>
      <td>0.520444</td>
      <td>0.806157</td>
      <td>0.624302</td>
    </tr>
    <tr>
      <th>80</th>
      <td>20181112</td>
      <td>0.831380</td>
      <td>0.520392</td>
      <td>0.803020</td>
      <td>0.621397</td>
    </tr>
    <tr>
      <th>94</th>
      <td>20181119</td>
      <td>0.832517</td>
      <td>0.522416</td>
      <td>0.808450</td>
      <td>0.642333</td>
    </tr>
    <tr>
      <th>129</th>
      <td>20181126</td>
      <td>0.831863</td>
      <td>0.522253</td>
      <td>0.809217</td>
      <td>0.638037</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2633</th>
      <td>20200817</td>
      <td>0.835696</td>
      <td>0.571625</td>
      <td>0.835187</td>
      <td>0.694960</td>
    </tr>
    <tr>
      <th>2681</th>
      <td>20200824</td>
      <td>0.853237</td>
      <td>0.595931</td>
      <td>0.851268</td>
      <td>0.731337</td>
    </tr>
    <tr>
      <th>2688</th>
      <td>20200831</td>
      <td>0.858467</td>
      <td>0.600633</td>
      <td>0.850167</td>
      <td>0.724018</td>
    </tr>
    <tr>
      <th>2733</th>
      <td>20200907</td>
      <td>0.855593</td>
      <td>0.595038</td>
      <td>0.841622</td>
      <td>0.703672</td>
    </tr>
    <tr>
      <th>2744</th>
      <td>20200914</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 5 columns</p>
</div>



## dt열 자료형 변경


```python
home_weekly['dt'] = pd.to_datetime(home_weekly['dt'], format = '%Y%m%d')
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>2018-10-29</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
    </tr>
    <tr>
      <th>39</th>
      <td>2018-11-05</td>
      <td>0.831570</td>
      <td>0.520444</td>
      <td>0.806157</td>
      <td>0.624302</td>
    </tr>
    <tr>
      <th>80</th>
      <td>2018-11-12</td>
      <td>0.831380</td>
      <td>0.520392</td>
      <td>0.803020</td>
      <td>0.621397</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2018-11-19</td>
      <td>0.832517</td>
      <td>0.522416</td>
      <td>0.808450</td>
      <td>0.642333</td>
    </tr>
    <tr>
      <th>129</th>
      <td>2018-11-26</td>
      <td>0.831863</td>
      <td>0.522253</td>
      <td>0.809217</td>
      <td>0.638037</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2633</th>
      <td>2020-08-17</td>
      <td>0.835696</td>
      <td>0.571625</td>
      <td>0.835187</td>
      <td>0.694960</td>
    </tr>
    <tr>
      <th>2681</th>
      <td>2020-08-24</td>
      <td>0.853237</td>
      <td>0.595931</td>
      <td>0.851268</td>
      <td>0.731337</td>
    </tr>
    <tr>
      <th>2688</th>
      <td>2020-08-31</td>
      <td>0.858467</td>
      <td>0.600633</td>
      <td>0.850167</td>
      <td>0.724018</td>
    </tr>
    <tr>
      <th>2733</th>
      <td>2020-09-07</td>
      <td>0.855593</td>
      <td>0.595038</td>
      <td>0.841622</td>
      <td>0.703672</td>
    </tr>
    <tr>
      <th>2744</th>
      <td>2020-09-14</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 5 columns</p>
</div>



## index 재설정


```python
home_weekly = home_weekly.sort_values(['dt'], ascending = True)
home_weekly = home_weekly.reset_index()
home_weekly = home_weekly.drop(['index'], axis = 1)
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-10-29</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-11-05</td>
      <td>0.831570</td>
      <td>0.520444</td>
      <td>0.806157</td>
      <td>0.624302</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-11-12</td>
      <td>0.831380</td>
      <td>0.520392</td>
      <td>0.803020</td>
      <td>0.621397</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-11-19</td>
      <td>0.832517</td>
      <td>0.522416</td>
      <td>0.808450</td>
      <td>0.642333</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-11-26</td>
      <td>0.831863</td>
      <td>0.522253</td>
      <td>0.809217</td>
      <td>0.638037</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2020-08-17</td>
      <td>0.835696</td>
      <td>0.571625</td>
      <td>0.835187</td>
      <td>0.694960</td>
    </tr>
    <tr>
      <th>95</th>
      <td>2020-08-24</td>
      <td>0.853237</td>
      <td>0.595931</td>
      <td>0.851268</td>
      <td>0.731337</td>
    </tr>
    <tr>
      <th>96</th>
      <td>2020-08-31</td>
      <td>0.858467</td>
      <td>0.600633</td>
      <td>0.850167</td>
      <td>0.724018</td>
    </tr>
    <tr>
      <th>97</th>
      <td>2020-09-07</td>
      <td>0.855593</td>
      <td>0.595038</td>
      <td>0.841622</td>
      <td>0.703672</td>
    </tr>
    <tr>
      <th>98</th>
      <td>2020-09-14</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 5 columns</p>
</div>



## '주단위 재택지수 평균'열 추가


```python
home_weekly['week_avg'] = home_weekly.mean(axis = 1)
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
      <th>week_avg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-10-29</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
      <td>0.689170</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-11-05</td>
      <td>0.831570</td>
      <td>0.520444</td>
      <td>0.806157</td>
      <td>0.624302</td>
      <td>0.695618</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-11-12</td>
      <td>0.831380</td>
      <td>0.520392</td>
      <td>0.803020</td>
      <td>0.621397</td>
      <td>0.694047</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-11-19</td>
      <td>0.832517</td>
      <td>0.522416</td>
      <td>0.808450</td>
      <td>0.642333</td>
      <td>0.701429</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-11-26</td>
      <td>0.831863</td>
      <td>0.522253</td>
      <td>0.809217</td>
      <td>0.638037</td>
      <td>0.700342</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2020-08-17</td>
      <td>0.835696</td>
      <td>0.571625</td>
      <td>0.835187</td>
      <td>0.694960</td>
      <td>0.734367</td>
    </tr>
    <tr>
      <th>95</th>
      <td>2020-08-24</td>
      <td>0.853237</td>
      <td>0.595931</td>
      <td>0.851268</td>
      <td>0.731337</td>
      <td>0.757943</td>
    </tr>
    <tr>
      <th>96</th>
      <td>2020-08-31</td>
      <td>0.858467</td>
      <td>0.600633</td>
      <td>0.850167</td>
      <td>0.724018</td>
      <td>0.758321</td>
    </tr>
    <tr>
      <th>97</th>
      <td>2020-09-07</td>
      <td>0.855593</td>
      <td>0.595038</td>
      <td>0.841622</td>
      <td>0.703672</td>
      <td>0.748981</td>
    </tr>
    <tr>
      <th>98</th>
      <td>2020-09-14</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
      <td>0.735400</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 6 columns</p>
</div>



## '주'열과 '월'열 추가


```python
home_weekly['week'] = home_weekly['dt'].dt.week
home_weekly['month'] = home_weekly['dt'].dt.month
home_weekly= home_weekly[['dt', 'month','week', 'week_avg','normal_night', 'normal_day', 'end_night', 'end_day']]
home_weekly
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>month</th>
      <th>week</th>
      <th>week_avg</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-10-29</td>
      <td>10</td>
      <td>44</td>
      <td>0.689170</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-11-05</td>
      <td>11</td>
      <td>45</td>
      <td>0.695618</td>
      <td>0.831570</td>
      <td>0.520444</td>
      <td>0.806157</td>
      <td>0.624302</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-11-12</td>
      <td>11</td>
      <td>46</td>
      <td>0.694047</td>
      <td>0.831380</td>
      <td>0.520392</td>
      <td>0.803020</td>
      <td>0.621397</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-11-19</td>
      <td>11</td>
      <td>47</td>
      <td>0.701429</td>
      <td>0.832517</td>
      <td>0.522416</td>
      <td>0.808450</td>
      <td>0.642333</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-11-26</td>
      <td>11</td>
      <td>48</td>
      <td>0.700342</td>
      <td>0.831863</td>
      <td>0.522253</td>
      <td>0.809217</td>
      <td>0.638037</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2020-08-17</td>
      <td>8</td>
      <td>34</td>
      <td>0.734367</td>
      <td>0.835696</td>
      <td>0.571625</td>
      <td>0.835187</td>
      <td>0.694960</td>
    </tr>
    <tr>
      <th>95</th>
      <td>2020-08-24</td>
      <td>8</td>
      <td>35</td>
      <td>0.757943</td>
      <td>0.853237</td>
      <td>0.595931</td>
      <td>0.851268</td>
      <td>0.731337</td>
    </tr>
    <tr>
      <th>96</th>
      <td>2020-08-31</td>
      <td>8</td>
      <td>36</td>
      <td>0.758321</td>
      <td>0.858467</td>
      <td>0.600633</td>
      <td>0.850167</td>
      <td>0.724018</td>
    </tr>
    <tr>
      <th>97</th>
      <td>2020-09-07</td>
      <td>9</td>
      <td>37</td>
      <td>0.748981</td>
      <td>0.855593</td>
      <td>0.595038</td>
      <td>0.841622</td>
      <td>0.703672</td>
    </tr>
    <tr>
      <th>98</th>
      <td>2020-09-14</td>
      <td>9</td>
      <td>38</td>
      <td>0.735400</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
<p>99 rows × 8 columns</p>
</div>



# 연도별 자료

## 연도별로 보기


```python
home_wkly_18 = home_weekly[home_weekly['dt'].dt.year == 2018]
home_wkly_18.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>month</th>
      <th>week</th>
      <th>week_avg</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-10-29</td>
      <td>10</td>
      <td>44</td>
      <td>0.68917</td>
      <td>0.830302</td>
      <td>0.514864</td>
      <td>0.801207</td>
      <td>0.610308</td>
    </tr>
  </tbody>
</table>
</div>




```python
home_wkly_19 = [home_weekly['dt'].dt.year == 2019]
```


```python
home_wkly_20 = home_weekly[home_weekly['dt'].dt.year == 2020]
home_wkly_20.tail(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
    
    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dt</th>
      <th>month</th>
      <th>week</th>
      <th>week_avg</th>
      <th>normal_night</th>
      <th>normal_day</th>
      <th>end_night</th>
      <th>end_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>98</th>
      <td>2020-09-14</td>
      <td>9</td>
      <td>38</td>
      <td>0.7354</td>
      <td>0.849636</td>
      <td>0.575621</td>
      <td>0.835069</td>
      <td>0.681273</td>
    </tr>
  </tbody>
</table>
</div>



# 시각화


```python
import pandas as pd 
pd.options.plotting.backend = 'plotly'

home_weekly[['dt', 'week_avg']].groupby('dt').sum().plot()
```

![2021-03-10-COVID19_Trash](C:\Users\sgi40\OneDrive\WallPaper\github\Blog_Image\2021-03-10-COVID19_Trash.png)