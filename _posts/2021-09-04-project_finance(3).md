---
title: project_finance (3)
date: 2021-09-04
categories: project finance python 재무제표 dart
---
### 다중회사 5개년치 재무제표 주요계정 테이블  

#### OpenDartReader 라이브러리를 활용한 재무제표 분석    
[OpenDartReader/github](https://github.com/FinanceData/OpenDartReader)  
  
#### project_finance
전체코드 [finstate.py/github](https://github.com/yeonseo-Jung/project_finance/blob/aca4af282fedc2452e5f95f44f3d58ab07d4f09a/finstate.py)

#### code
___
##### 테스트 회사 리스트
```python
stock_names = ["삼성전자", "SK하이닉스", "현대자동차", "현대모비스", "엔씨소프트", "원익IPS", "휴젤"]
```
##### columns
```python
columns_infos = ["stock_code", "stock_name", "corp_code"]
columns_quarters = [
 'thstrm_amount_2021_1',
 'thstrm_amount_2021_2',
 'thstrm_amount_2020_1',
 'thstrm_amount_2020_2',
 'thstrm_amount_2020_3',
 'thstrm_amount_2020_4',
 'thstrm_amount_2019_1',
 'thstrm_amount_2019_2',
 'thstrm_amount_2019_3',
 'thstrm_amount_2019_4',
 'thstrm_amount_2018_1',
 'thstrm_amount_2018_2',
 'thstrm_amount_2018_3',
 'thstrm_amount_2018_4',
 'thstrm_amount_2017_1',
 'thstrm_amount_2017_2',
 'thstrm_amount_2017_3',
 'thstrm_amount_2017_4',
]
```  

##### accounts 딕셔너리
##### key: 계정과목 id 
##### values: 계정과목 이름
```python
accounts = {}
accounts["equity"] = ["ifrs_EquityAttributableToOwnersOfParent", "ifrs-full_EquityAttributableToOwnersOfParent"]
accounts["profit"] = ["ifrs_ProfitLossAttributableToOwnersOfParent", "ifrs-full_ProfitLossAttributableToOwnersOfParent"]
```

##### 핵심함수  

* 재무제표 테이블에서 원하는 계정과목의 금액을 추출해서 리스트로 반환해주는 함수
```python
# 재무제표 테이블에서 원하는 계정과목의 금액을 추출해서 리스트로 반환해주는 함수
def find_amounts(finstate, account, accounts):
    columns = list(finstate.columns)
    df = pd.DataFrame(columns=columns)
    i = 0
    for s in finstate["account_id"]:
        # 찾고자 하는 계정과목id 이면 df에 추가 
        if s in accounts[account]:
            df = df.append(finstate.loc[i], ignore_index=True)
        i += 1
    if len(df) == 0:
        return [None] * len(finstate.columns)
    else:
        # df의 각 row값을 벡터 합 시키기
        for i in range(len(df)):
            if i == 0:
                df_tot = df.loc[i]
            else:
                df_tot += df.loc[i]
        return list(df_tot)
```  
  
  
* 재무제표 테이블에서 특정 계정과목의 금액 추출하여 다중회사의 특정 계정과목 테이블에 데이터를 추가하는 함수
```python
# 재무제표 테이블에서 특정 계정과목의 금액 추출하여 다중회사의 특정 계정과목 테이블에 데이터를 추가하는 함수
def append_amounts(account_df, stock_infos, stock_name, account, accounts): 
    stock_code = find_stock_code(stock_infos, stock_name)
    fstate_all_account = read_xlsx(stock_code)
    # columns
    columns = list(account_df.columns)
    columns_infos = columns[:3]
    columns_quarters = columns[3:]
    infos = list(fstate_all_account.loc[0, columns_infos])
    # 계정과목 금액 찾기
    amounts = find_amounts(fstate_all_account, account, accounts)[6:]
    # infos list와 amounts list 합친 후 account_df에 추가하기 
    data = tuple(infos + amounts)
    data_df = pd.DataFrame([data], columns=columns)
    account_df = account_df.append(data_df, ignore_index=True)
    
    return account_df
```
  
  
* 계정과목 금액 테이블에서 결측 데이터 찾는 함수  
```python
# account table에서 결측 데이터 찾기 
def find_zero_null(df, columns_quarters):
    zero_row_columns = []
    null_row_columns = []
    for quarter in columns_quarters:
        i = 0
        for amount in df[quarter]:
            if amount == None:
                null_row_columns.append([i, quarter])
            elif abs(amount) == 0:
                zero_row_columns.append([i, quarter])
            i += 1
    return zero_row_columns, null_row_columns
```
  
  
#### 결과
___  
  
  
* profit_df 테이블
```python
read_xlsx("profit_df_test")
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
      <th>thstrm_amount_2021_1</th>
      <th>thstrm_amount_2021_2</th>
      <th>thstrm_amount_2020_1</th>
      <th>thstrm_amount_2020_2</th>
      <th>thstrm_amount_2020_3</th>
      <th>thstrm_amount_2020_4</th>
      <th>thstrm_amount_2019_1</th>
      <th>...</th>
      <th>thstrm_amount_2019_3</th>
      <th>thstrm_amount_2019_4</th>
      <th>thstrm_amount_2018_1</th>
      <th>thstrm_amount_2018_2</th>
      <th>thstrm_amount_2018_3</th>
      <th>thstrm_amount_2018_4</th>
      <th>thstrm_amount_2017_1</th>
      <th>thstrm_amount_2017_2</th>
      <th>thstrm_amount_2017_3</th>
      <th>thstrm_amount_2017_4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>126380</td>
      <td>7.092786e+12</td>
      <td>9.450676e+12</td>
      <td>4.889599e+12</td>
      <td>5.488964e+12</td>
      <td>9.266815e+12</td>
      <td>1.682403e+13</td>
      <td>5.107490e+12</td>
      <td>...</td>
      <td>6.105039e+12</td>
      <td>1.540002e+13</td>
      <td>1.161183e+13</td>
      <td>1.098155e+13</td>
      <td>1.296743e+13</td>
      <td>3.092345e+13</td>
      <td>7.488532e+12</td>
      <td>1.079994e+13</td>
      <td>1.103977e+13</td>
      <td>3.030480e+13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>000660.KS</td>
      <td>SK하이닉스</td>
      <td>164779</td>
      <td>9.904460e+11</td>
      <td>1.984515e+12</td>
      <td>6.481540e+11</td>
      <td>1.262890e+12</td>
      <td>1.077262e+12</td>
      <td>3.677840e+12</td>
      <td>1.102753e+12</td>
      <td>...</td>
      <td>4.932010e+11</td>
      <td>1.520087e+12</td>
      <td>3.120254e+12</td>
      <td>4.329947e+12</td>
      <td>4.693620e+12</td>
      <td>1.084649e+13</td>
      <td>1.897969e+12</td>
      <td>2.468544e+12</td>
      <td>3.054248e+12</td>
      <td>7.587264e+12</td>
    </tr>
    <tr>
      <th>2</th>
      <td>005380.KS</td>
      <td>현대자동차</td>
      <td>164742</td>
      <td>1.327250e+12</td>
      <td>1.761887e+12</td>
      <td>4.633140e+11</td>
      <td>2.273860e+11</td>
      <td>-3.360610e+11</td>
      <td>1.760497e+12</td>
      <td>8.294770e+11</td>
      <td>...</td>
      <td>4.269110e+11</td>
      <td>2.553138e+12</td>
      <td>6.680140e+11</td>
      <td>7.005990e+11</td>
      <td>2.692450e+11</td>
      <td>1.238839e+12</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>8.523710e+11</td>
      <td>3.180453e+12</td>
    </tr>
    <tr>
      <th>3</th>
      <td>012330.KS</td>
      <td>현대모비스</td>
      <td>164788</td>
      <td>5.997990e+11</td>
      <td>6.665550e+11</td>
      <td>3.484440e+11</td>
      <td>2.342050e+11</td>
      <td>3.897160e+11</td>
      <td>1.139430e+12</td>
      <td>4.829500e+11</td>
      <td>...</td>
      <td>5.771670e+11</td>
      <td>1.713503e+12</td>
      <td>4.665010e+11</td>
      <td>5.530600e+11</td>
      <td>4.488280e+11</td>
      <td>1.439976e+12</td>
      <td>7.611780e+11</td>
      <td>4.812400e+11</td>
      <td>4.821620e+11</td>
      <td>1.085990e+12</td>
    </tr>
    <tr>
      <th>4</th>
      <td>036570.KS</td>
      <td>엔씨소프트</td>
      <td>261443</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>240810.KQ</td>
      <td>원익IPS</td>
      <td>1135941</td>
      <td>2.564883e+10</td>
      <td>7.599476e+10</td>
      <td>1.376603e+10</td>
      <td>3.542319e+10</td>
      <td>8.727212e+10</td>
      <td>1.054670e+10</td>
      <td>4.977547e+09</td>
      <td>...</td>
      <td>1.501040e+09</td>
      <td>4.136163e+10</td>
      <td>2.459296e+10</td>
      <td>3.156078e+10</td>
      <td>3.701010e+10</td>
      <td>4.985029e+10</td>
      <td>2.488083e+10</td>
      <td>3.673446e+10</td>
      <td>2.166335e+10</td>
      <td>7.370123e+10</td>
    </tr>
    <tr>
      <th>6</th>
      <td>145020.KQ</td>
      <td>휴젤</td>
      <td>888347</td>
      <td>1.862436e+10</td>
      <td>1.469514e+10</td>
      <td>6.001495e+09</td>
      <td>1.437236e+10</td>
      <td>9.823475e+09</td>
      <td>3.216862e+10</td>
      <td>1.379860e+10</td>
      <td>...</td>
      <td>1.000559e+10</td>
      <td>3.460651e+10</td>
      <td>1.943144e+10</td>
      <td>1.461857e+10</td>
      <td>2.277522e+10</td>
      <td>4.696118e+10</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>1.720762e+10</td>
      <td>5.561952e+10</td>
    </tr>
  </tbody>
</table>
<p>7 rows × 21 columns</p>
</div>  
  
* equity_df 테이블

```python
read_xlsx("equity_df_test")
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
      <th>thstrm_amount_2021_1</th>
      <th>thstrm_amount_2021_2</th>
      <th>thstrm_amount_2020_1</th>
      <th>thstrm_amount_2020_2</th>
      <th>thstrm_amount_2020_3</th>
      <th>thstrm_amount_2020_4</th>
      <th>thstrm_amount_2019_1</th>
      <th>...</th>
      <th>thstrm_amount_2019_3</th>
      <th>thstrm_amount_2019_4</th>
      <th>thstrm_amount_2018_1</th>
      <th>thstrm_amount_2018_2</th>
      <th>thstrm_amount_2018_3</th>
      <th>thstrm_amount_2018_4</th>
      <th>thstrm_amount_2017_1</th>
      <th>thstrm_amount_2017_2</th>
      <th>thstrm_amount_2017_3</th>
      <th>thstrm_amount_2017_4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>126380</td>
      <td>265767257899008</td>
      <td>274160529965056</td>
      <td>258481768628224</td>
      <td>261745373347840</td>
      <td>267942138740736</td>
      <td>267670331064320</td>
      <td>245499810545664</td>
      <td>...</td>
      <td>255403451482112</td>
      <td>254915469377536</td>
      <td>215884501090304</td>
      <td>225671422935040</td>
      <td>234476374327296</td>
      <td>240068992172032</td>
      <td>183119604875264</td>
      <td>193654455009280</td>
      <td>203504408854528</td>
      <td>207213415104512</td>
    </tr>
    <tr>
      <th>1</th>
      <td>000660.KS</td>
      <td>SK하이닉스</td>
      <td>164779</td>
      <td>52354921529344</td>
      <td>54850368831488</td>
      <td>48231010533376</td>
      <td>49382862880768</td>
      <td>50479073591296</td>
      <td>51888540090368</td>
      <td>47159835623424</td>
      <td>...</td>
      <td>48253965959168</td>
      <td>47928416665600</td>
      <td>36371829882880</td>
      <td>40771700916224</td>
      <td>43484018900992</td>
      <td>46845720002560</td>
      <td>25162328047616</td>
      <td>27757134217216</td>
      <td>30896684007424</td>
      <td>33815279960064</td>
    </tr>
    <tr>
      <th>2</th>
      <td>005380.KS</td>
      <td>현대자동차</td>
      <td>164742</td>
      <td>70601578381312</td>
      <td>72723250282496</td>
      <td>69438925701120</td>
      <td>69654412263424</td>
      <td>69053146202112</td>
      <td>69480633860096</td>
      <td>68185340510208</td>
      <td>...</td>
      <td>69840668721152</td>
      <td>70065802182656</td>
      <td>68905540255744</td>
      <td>69261296926720</td>
      <td>68918907502592</td>
      <td>67973968560128</td>
      <td>66602066247680</td>
      <td>68356673634304</td>
      <td>69140429668352</td>
      <td>69103482044416</td>
    </tr>
    <tr>
      <th>3</th>
      <td>012330.KS</td>
      <td>현대모비스</td>
      <td>164788</td>
      <td>33496269586432</td>
      <td>34134032384000</td>
      <td>32547620782080</td>
      <td>32781597933568</td>
      <td>33127330217984</td>
      <td>33252668604416</td>
      <td>30983988445184</td>
      <td>...</td>
      <td>32363413241856</td>
      <td>32330018193408</td>
      <td>29412735057920</td>
      <td>30095848767488</td>
      <td>30340605280256</td>
      <td>30630494601216</td>
      <td>28456159019008</td>
      <td>29261131939840</td>
      <td>29868234375168</td>
      <td>29295405694976</td>
    </tr>
    <tr>
      <th>4</th>
      <td>036570.KS</td>
      <td>엔씨소프트</td>
      <td>261443</td>
      <td>3038004903936</td>
      <td>3134575869952</td>
      <td>2593551286272</td>
      <td>2775932993536</td>
      <td>3210856628224</td>
      <td>3141584289792</td>
      <td>2376113848320</td>
      <td>...</td>
      <td>2462692933632</td>
      <td>2499168174080</td>
      <td>2532326506496</td>
      <td>2664293466112</td>
      <td>2482215845888</td>
      <td>2367717113856</td>
      <td>1814119186432</td>
      <td>2188696354816</td>
      <td>2447746269184</td>
      <td>2721242677248</td>
    </tr>
    <tr>
      <th>5</th>
      <td>240810.KQ</td>
      <td>원익IPS</td>
      <td>1135941</td>
      <td>678936313856</td>
      <td>755790053376</td>
      <td>579267723264</td>
      <td>614530088960</td>
      <td>702146543616</td>
      <td>663027843072</td>
      <td>526018707456</td>
      <td>...</td>
      <td>552337408000</td>
      <td>566554132480</td>
      <td>289167081472</td>
      <td>338846351360</td>
      <td>375512694784</td>
      <td>369216487424</td>
      <td>243022430208</td>
      <td>279915921408</td>
      <td>301857701888</td>
      <td>313468387328</td>
    </tr>
    <tr>
      <th>6</th>
      <td>145020.KQ</td>
      <td>휴젤</td>
      <td>888347</td>
      <td>760155209728</td>
      <td>776661172224</td>
      <td>708633755648</td>
      <td>726264578048</td>
      <td>737391935488</td>
      <td>746673537024</td>
      <td>735421661184</td>
      <td>...</td>
      <td>709576491008</td>
      <td>699495546880</td>
      <td>721645338624</td>
      <td>735678627840</td>
      <td>744941879296</td>
      <td>727719936000</td>
      <td>272907010048</td>
      <td>296767848448</td>
      <td>689262559232</td>
      <td>695710384128</td>
    </tr>
  </tbody>
</table>
<p>7 rows × 21 columns</p>
</div>

#### 문제점
___
##### 데이터 누락

* accounts(dict)에 존재하지 않는 account_id를 사용함  

* 찾고자 하는 계정과목에 대응되는 account_id가 재무제표 자체에서 누락됨




