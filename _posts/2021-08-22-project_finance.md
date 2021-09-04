---
title: project_finance (0)
date: 2021-08-22
categories: project python dart
---

### OpenDartReader 라이브러리를 활용한 재무제표 분석  
[OpenDartReader/github](https://github.com/FinanceData/OpenDartReader)  
  
### project_finance
전체코드 [finstate.py/github](https://github.com/yeonseo-Jung/project_finance/blob/aca4af282fedc2452e5f95f44f3d58ab07d4f09a/finstate.py)  

### 코스피, 코스닥에 상장된 기업들의 기업개황 table 만들기
* stock_infos: 코스피, 코스닥에 상장된 기업들의 기업개황 table
  columns = [stock_code, stock_name, corp_code]   
  (리스트의 각 원소 자료형: string)  
  key = stock_code  
    
#### code  
```python
import requests
import zipfile
import io
import os
import json
import pandas as pd
import xml.etree.ElementTree as ET
from datetime import datetime, timedelta

try:
    from pandas import json_normalize
except ImportError:
    from pandas.io.json import json_normalize
```


```python
import OpenDartReader
api_key = "ef3149d745caee09f48df5004b905ec4ef3f5d7e"
dart = OpenDartReader(api_key)
```

```python
def corp_codes(api_key):
        url = 'https://opendart.fss.or.kr/api/corpCode.xml'
        params = { 'crtfc_key': api_key, }

        r = requests.get(url, params=params)
        try:
            tree = ET.XML(r.content)
            status = tree.find('status').text
            message = tree.find('message').text
            if status != '000':
                raise ValueError({'status': status, 'message': message})
        except ET.ParseError as e:
            pass

        zf = zipfile.ZipFile(io.BytesIO(r.content))
        xml_data = zf.read('CORPCODE.xml')

        # XML to DataFrame
        tree = ET.XML(xml_data)
        all_records = []

        element = tree.findall('list')
        for i, child in enumerate(element):
            record = {}
            for i, subchild in enumerate(child):
                record[subchild.tag] = subchild.text
            all_records.append(record)
        return pd.DataFrame(all_records)
```


```python
corps = corp_codes(api_key)
```


```python
# dart에서 제공하는 회사들의 기업개황 table
corps
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
      <th>corp_code</th>
      <th>corp_name</th>
      <th>stock_code</th>
      <th>modify_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00434003</td>
      <td>다코</td>
      <td></td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00434456</td>
      <td>일산약품</td>
      <td></td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>2</th>
      <td>00430964</td>
      <td>굿앤엘에스</td>
      <td></td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>3</th>
      <td>00432403</td>
      <td>한라판지</td>
      <td></td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00388953</td>
      <td>크레디피아제이십오차유동화전문회사</td>
      <td></td>
      <td>20170630</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>87912</th>
      <td>01560457</td>
      <td>삼원개발</td>
      <td></td>
      <td>20210506</td>
    </tr>
    <tr>
      <th>87913</th>
      <td>01556533</td>
      <td>에이더블유파트너스</td>
      <td></td>
      <td>20210506</td>
    </tr>
    <tr>
      <th>87914</th>
      <td>01546299</td>
      <td>동영와이케이</td>
      <td></td>
      <td>20210506</td>
    </tr>
    <tr>
      <th>87915</th>
      <td>00694605</td>
      <td>미디어윌</td>
      <td></td>
      <td>20210506</td>
    </tr>
    <tr>
      <th>87916</th>
      <td>01368761</td>
      <td>엔브이에이치원방테크</td>
      <td></td>
      <td>20210506</td>
    </tr>
  </tbody>
</table>
<p>87917 rows × 4 columns</p>
</div>




```python
# corps에서 stock_code가 존재하는 회사만 stock_codes table에 할당
stock_codes = pd.DataFrame()
i = 0
for s in corps["stock_code"]:
    if len(s) == 6:
        stock_codes = stock_codes.append(corps.loc[i], ignore_index=True)
    i += 1
```


```python
# 종목코드가 존재하는 회사들의 기업개황 table
stock_codes
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
      <th>corp_code</th>
      <th>corp_name</th>
      <th>modify_date</th>
      <th>stock_code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>00260985</td>
      <td>한빛네트</td>
      <td>20170630</td>
      <td>036720</td>
    </tr>
    <tr>
      <th>1</th>
      <td>00264529</td>
      <td>엔플렉스</td>
      <td>20170630</td>
      <td>040130</td>
    </tr>
    <tr>
      <th>2</th>
      <td>00358545</td>
      <td>동서정보기술</td>
      <td>20170630</td>
      <td>055000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>00231567</td>
      <td>애드모바일</td>
      <td>20170630</td>
      <td>032600</td>
    </tr>
    <tr>
      <th>4</th>
      <td>00247939</td>
      <td>씨모스</td>
      <td>20170630</td>
      <td>037600</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3349</th>
      <td>00925295</td>
      <td>에프엔씨엔터</td>
      <td>20210901</td>
      <td>173940</td>
    </tr>
    <tr>
      <th>3350</th>
      <td>00521390</td>
      <td>뉴파워프라즈마</td>
      <td>20210602</td>
      <td>144960</td>
    </tr>
    <tr>
      <th>3351</th>
      <td>00838500</td>
      <td>엘브이엠씨</td>
      <td>20210611</td>
      <td>900140</td>
    </tr>
    <tr>
      <th>3352</th>
      <td>00160232</td>
      <td>KSS해운</td>
      <td>20210628</td>
      <td>044450</td>
    </tr>
    <tr>
      <th>3353</th>
      <td>01412822</td>
      <td>솔루스첨단소재</td>
      <td>20210824</td>
      <td>336370</td>
    </tr>
  </tbody>
</table>
<p>3354 rows × 4 columns</p>
</div>




```python
# stock_codes에서 코스피, 코스닥 시장에 상장된 회사들만 stock_infos table에 할당
columns_infos = ["stock_code", "stock_name", "corp_code"]
stock_infos = pd.DataFrame(columns=columns_infos)
i = 0
for s in stock_codes["corp_name"]:
    market = dart.company(s)["corp_cls"]
    if market == "Y":
        stock_code = stock_codes.loc[i, "stock_code"] + ".KS"
        stock_name = stock_codes.loc[i, "corp_name"]
        corp_code = stock_codes.loc[i, "corp_code"] + ".KS"
        infos = pd.DataFrame([(stock_code, stock_name, corp_code)], columns=columns_infos)
        stock_infos = stock_infos.append(infos, ignore_index=True)
    elif market == "K":
        stock_code = stock_codes.loc[i, "stock_code"] + ".KQ"
        stock_name = stock_codes.loc[i, "corp_name"]
        corp_code = stock_codes.loc[i, "corp_code"] + ".KQ"
        infos = pd.DataFrame([(stock_code, stock_name, corp_code)], columns=columns_infos)
        stock_infos = stock_infos.append(infos, ignore_index=True)  
    i += 1
```


```python
# 코스피, 코스닥 시장에 상장된 회사들의 기업개황 table
stock_infos
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
      <th>stock_code</th>
      <th>stock_name</th>
      <th>corp_code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>950130.KQ</td>
      <td>엑세스바이오</td>
      <td>00956028.KQ</td>
    </tr>
    <tr>
      <th>1</th>
      <td>152550.KS</td>
      <td>한국ANKOR유전</td>
      <td>00907013.KS</td>
    </tr>
    <tr>
      <th>2</th>
      <td>900070.KQ</td>
      <td>글로벌에스엠</td>
      <td>00783246.KQ</td>
    </tr>
    <tr>
      <th>3</th>
      <td>900120.KQ</td>
      <td>씨케이에이치</td>
      <td>00800084.KQ</td>
    </tr>
    <tr>
      <th>4</th>
      <td>096300.KS</td>
      <td>베트남개발1</td>
      <td>00626710.KS</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>173940.KQ</td>
      <td>에프엔씨엔터</td>
      <td>00925295.KQ</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>144960.KQ</td>
      <td>뉴파워프라즈마</td>
      <td>00521390.KQ</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>900140.KS</td>
      <td>엘브이엠씨</td>
      <td>00838500.KS</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>044450.KS</td>
      <td>KSS해운</td>
      <td>00160232.KS</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>336370.KS</td>
      <td>솔루스첨단소재</td>
      <td>01412822.KS</td>
    </tr>
  </tbody>
</table>
<p>2020 rows × 3 columns</p>
</div>

stock_infos 테이블은 앞으로 많은 테이블의 부모 테이블이 될 예정이다. 
