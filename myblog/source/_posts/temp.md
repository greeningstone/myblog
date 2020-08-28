---
title: temp crawling
---

```python
# matplotlib 한글 출력 가능하도록 만들기
from matplotlib import font_manager, rc
font_name = font_manager.FontProperties(fname="c:/Windows/Fonts/malgun.ttf").get_name()
rc('font', family=font_name)
```


```python
# 데이터 크롤링 모듈
from selenium import webdriver
from bs4 import BeautifulSoup
import re
```


```python
# 데이터 분석 모듈
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import time
from datetime import datetime
```


```python
submission = pd.read_csv("C:/Users/asia_21/Desktop/kbo/submission.csv")
reg = pd.read_csv("C:/Users/asia_21/Desktop/kbo/Regular_Season_Batter.csv")
```


```python
driver= webdriver.Chrome()
```


```python
# 크롤링
for i in range(86):
    
    # 1982년 부터 2018년 까지 statiz에 기록된 선수들 필터링 (총 8558명)
    url = 'http://www.statiz.co.kr/stat.php?mid=stat&re=0&ys=1982&ye=2018&sn=100&pa={}'.format(i*100)
    
    driver.get(url)
    driver.implicitly_wait(5)
    
    html = driver.find_element_by_xpath('//*[@id="mytable"]/tbody').get_attribute("innerHTML") #기록 table을 str형태로 저장
    soup = BeautifulSoup(html, 'html.parser') #str 객체를 BeautifulSoup 객체로 변경
    
    temp = [i.text.strip() for i in soup.findAll("tr")] #tr 태그에서, text만 저장하기
    temp = pd.Series(temp) #list 객체에서 series 객체로 변경
    
    #'순'이나 'W'로 시작하는 row 제거
    # 즉, 선수별 기록만 남기고, index를 reset 해주기
    temp = temp[~temp.str.match("[순W]")].reset_index(drop=True) 
    
    temp = temp.apply(lambda x: pd.Series(x.split(' '))) #띄어쓰기 기준으로 나눠서 dataframe으로 변경
    
    #선수 팀 정보 이후 첫번째 기록과는 space 하나로 구분, 그 이후로는 space 두개로 구분이 되어 있음 
    #그래서 space 하나로 구분을 시키면, 빈 column들이 존재 하는데, 해당 column들 제거 
    temp = temp.replace('', np.nan).dropna(axis=1) 
    
    #WAR 정보가 들어간 column이 2개 있다. (index가 1인 column과, 제일 마지막 column)
    #그 중에서 index가 1인 columm 제거 
    temp = temp.drop(1, axis=1)
    
    #선수 이름 앞의 숫자 제거
    temp[0] = temp[0].str.replace("^\d+", '')

    # 선수들의 생일 정보가 담긴 tag들 가지고 오기
    birth = [i.find("a") for i in soup.findAll('tr') if 'birth' in i.find('a').attrs['href']]
    
    # tag내에서, 생일 날짜만 추출하기 
    p = re.compile("\d{4}\-\d{2}\-\d{2}")
    birth = [p.findall(i.attrs['href'])[0] for i in birth]
    
    # 생일 column 추가
    temp['생일'] = birth
    
    # page별 완성된 dataframe을 계속해서 result에 추가 시켜주기 
    if i == 0:
        result = temp
    else:
        result = result.append(temp)
        result = result.reset_index(drop=True)
        
    print(i, "완료")
        
#column 명 정보 저장        
columns = ['선수'] + [i.text for i in soup.findAll("tr")[0].findAll("th")][4:-3] + ['타율', '출루', '장타', 'OPS', 'wOBA', 'wRC+', 'WAR+', '생일']

#column 명 추가
result.columns = columns

#webdriver 종료
driver.close()
```

    0 완료
    


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-52-37600f2ae032> in <module>
         26     #WAR 정보가 들어간 column이 2개 있다. (index가 1인 column과, 제일 마지막 column)
         27     #그 중에서 index가 1인 columm 제거
    ---> 28     temp = temp.drop(1, axis=1)
         29 
         30     #선수 이름 앞의 숫자 제거
    

    C:\anaconda3\lib\site-packages\pandas\core\frame.py in drop(self, labels, axis, index, columns, level, inplace, errors)
       3988                 weight  1.0     0.8
       3989         """
    -> 3990         return super().drop(
       3991             labels=labels,
       3992             axis=axis,
    

    C:\anaconda3\lib\site-packages\pandas\core\generic.py in drop(self, labels, axis, index, columns, level, inplace, errors)
       3934         for axis, labels in axes.items():
       3935             if labels is not None:
    -> 3936                 obj = obj._drop_axis(labels, axis, level=level, errors=errors)
       3937 
       3938         if inplace:
    

    C:\anaconda3\lib\site-packages\pandas\core\generic.py in _drop_axis(self, labels, axis, level, errors)
       3968                 new_axis = axis.drop(labels, level=level, errors=errors)
       3969             else:
    -> 3970                 new_axis = axis.drop(labels, errors=errors)
       3971             result = self.reindex(**{axis_name: new_axis})
       3972 
    

    C:\anaconda3\lib\site-packages\pandas\core\indexes\base.py in drop(self, labels, errors)
       5016         if mask.any():
       5017             if errors != "ignore":
    -> 5018                 raise KeyError(f"{labels[mask]} not found in axis")
       5019             indexer = indexer[~mask]
       5020         return self.delete(indexer)
    

    KeyError: '[1] not found in axis'



```python
result.shape
```


```python
result.to_csv("C:/Users/asia_21/Desktop/machine_ learning/statiz_origin.csv")
```
