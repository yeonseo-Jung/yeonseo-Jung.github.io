---
title: project_finance (1)
date: 2021-09-04
categories: project python 재무제표 dart
---
### 단일회사 5개년치 재무제표 테이블 병합하기   

#### OpenDartReader 라이브러리를 활용한 재무제표 분석  
[OpenDartReader/github](https://github.com/FinanceData/OpenDartReader)  
  
#### project_finance
전체코드 [finstate.py/github](https://github.com/yeonseo-Jung/project_finance/blob/aca4af282fedc2452e5f95f44f3d58ab07d4f09a/finstate.py)

* fstate_all_account: 단일회사의 재무제표를 분기별로 병합한 테이블  
       
    columns_0 = [stock_name, stock_code, corp_code]   
    (리스트의 각 원소 자료형: string)  
        
    columns_1 = [sj_div, account_id, account_nm]   
    (리스트의 각 원소 자료형: string)  
      
    columns_2 = [thstrm_amount_2021_1, thstrm_amount_2021_2, thstrm_amount_2020_1, thstrm_amount_2020_2, ...]   
    (리스트의 각 원소 자료형: float32) ⇒ 최근 4개년치 분기별 금액  
  
    병합(join) 기준 key = ["sj_div", "account_id", "account_nm"]
  
#### code
___
##### 분기별 재무제표를 병합해서 DataFrame으로 return 해주는 함수  
```python
# 분기별 재무제표를 병합해서 DataFrame으로 return 해주는 함수 
def finstate_all_account(stock_name, years, quarters):
    for bsns_year in years:
        quarter = 1
        for reprt_code in quarters:
            # fstate 변수에 할당된 데이터가 없을 때(즉, 해당 연도 분기의 재무제표가 dart에 공시되어 있지 않은 경우) 예외처리
            try:
                fstate = dart.finstate_all(stock_name, bsns_year, reprt_code, fs_div="CFS")
                fstate = finstate_quarter(fstate).rename(columns={"thstrm_amount": f'thstrm_amount_{bsns_year}_{quarter}'})
                # stock_name, stock_code columns 삽입
                if dart.company(stock_name)["corp_cls"] == "Y":
                    stock_code = dart.company(stock_name)["stock_code"] + ".KS"
                elif dart.company(stock_name)["corp_cls"] == "K":
                    stock_code = dart.company(stock_name)["stock_code"] + ".KQ"
                fstate.insert(0, "stock_code", stock_code)
                fstate.insert(1, "stock_name", stock_name)
                # 계정과목 이름이 중복되는 행 삭제
                fstate = fstate.drop_duplicates(["sj_div", "account_id", "account_nm"], keep=False, ignore_index=True)
            except AttributeError:
                quarter += 1
                continue
            # 병합할 이전 분기 재무제표 dataframe이 없는경우 예외처리
            try:
                # 분기별 재무제표 병합
                if quarter == 1:
                    fstate_corp = fstate
                else:
                    fstate_corp = fstate_corp.merge(fstate, how="outer", on=["stock_name", "stock_code", "corp_code", "sj_div", "account_id", "account_nm"], suffixes=("", ""))
            except (ValueError, UnboundLocalError):
                fstate_corp = fstate
            # NULL 데이터를 0으로 바꿈
            fstate_corp = fstate_corp.fillna(0)

            if quarter == 4:
                # 4분기 발생 금액 columns 추가
                i = 0
                for s in fstate_corp["sj_div"]:
                    if s == "BS":
                        fstate_corp.loc[i,  f'thstrm_amount_{bsns_year}_{quarter}'] = fstate_corp.loc[i,  f'thstrm_amount_{bsns_year}_{quarter}']
                    else:
                        fstate_corp.loc[i,  f'thstrm_amount_{bsns_year}_{quarter}'] = fstate_corp.loc[i,  f'thstrm_amount_{bsns_year}_{quarter}'] - fstate_corp.loc[i,  f'thstrm_amount_{bsns_year}_{quarter-1}']
                    i += 1
            quarter += 1
        # 병합할 이전 연도 재무제표 dataframe이 없는경우 예외처리
        try:    
            # 연도별 재무제표 병합
            if bsns_year == years[0]:
                fstate_all_account = fstate_corp
            else:
                fstate_all_account = fstate_all_account.merge(fstate_corp, how="outer", on=["stock_name", "stock_code", "corp_code", "sj_div", "account_id", "account_nm"], suffixes=("", ""))
        except (ValueError, UnboundLocalError):
            fstate_all_account = fstate_corp
    # NULL 데이터를 0으로 바꿈
    fstate_all_account = fstate_all_account.fillna(0)
    # 엑셀파일로 저장
    fstate_all_account.to_excel(f"{stock_code}.xlsx")
    return fstate_all_account
```

```python
stock_name = "삼성전자"
years = ["2021", "2020", "2019", "2018", "2017"]
quarters = ["11013", "11012", "11014", "11011"]
fstate_samsung = finstate_all_account(stock_name, years, quarters)
```



##### 병합 이후 단일회사 5개년치 재무제표 테이블 (fstate_samsung)
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
      <th>sj_div</th>
      <th>account_id</th>
      <th>account_nm</th>
      <th>thstrm_amount_2021_1</th>
      <th>thstrm_amount_2021_2</th>
      <th>thstrm_amount_2020_1</th>
      <th>thstrm_amount_2020_2</th>
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
      <td>00126380</td>
      <td>BS</td>
      <td>ifrs-full_CurrentAssets</td>
      <td>유동자산</td>
      <td>2.091554e+14</td>
      <td>1.911185e+14</td>
      <td>1.867397e+14</td>
      <td>1.861369e+14</td>
      <td>...</td>
      <td>1.860421e+14</td>
      <td>1.813853e+14</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>BS</td>
      <td>ifrs-full_CashAndCashEquivalents</td>
      <td>현금및현금성자산</td>
      <td>4.103959e+13</td>
      <td>3.068379e+13</td>
      <td>2.791668e+13</td>
      <td>3.610961e+13</td>
      <td>...</td>
      <td>2.660499e+13</td>
      <td>2.688600e+13</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>BS</td>
      <td>dart_ShortTermDepositsNotClassifiedAsCashEquiv...</td>
      <td>단기금융상품</td>
      <td>8.715927e+13</td>
      <td>7.777703e+13</td>
      <td>7.863802e+13</td>
      <td>7.512761e+13</td>
      <td>...</td>
      <td>6.947697e+13</td>
      <td>7.625205e+13</td>
      <td>4.602770e+13</td>
      <td>4.871714e+13</td>
      <td>5.868142e+13</td>
      <td>6.589380e+13</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.128067e+13</td>
      <td>4.944769e+13</td>
    </tr>
    <tr>
      <th>3</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>BS</td>
      <td>-표준계정코드 미사용-</td>
      <td>단기상각후원가금융자산</td>
      <td>3.526888e+12</td>
      <td>2.350399e+12</td>
      <td>3.037379e+12</td>
      <td>1.224565e+12</td>
      <td>...</td>
      <td>4.021901e+12</td>
      <td>3.914216e+12</td>
      <td>3.733160e+12</td>
      <td>3.896630e+12</td>
      <td>3.446114e+12</td>
      <td>2.703693e+12</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>BS</td>
      <td>ifrs-full_CurrentFinancialAssetsAtFairValueThr...</td>
      <td>단기당기손익-공정가치금융자산</td>
      <td>5.949500e+10</td>
      <td>4.972000e+10</td>
      <td>1.238759e+12</td>
      <td>5.826410e+11</td>
      <td>...</td>
      <td>1.842611e+12</td>
      <td>1.727436e+12</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
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
      <th>270</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>CF</td>
      <td>dart_PurchaseOfAvailableForSaleFinancialAssets</td>
      <td>매도가능금융자산의 취득</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.447670e+11</td>
      <td>-1.447670e+11</td>
    </tr>
    <tr>
      <th>271</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>CF</td>
      <td>ifrs_IncreaseDecreaseInCashAndCashEquivalentsB...</td>
      <td>환율변동효과 반영전 현금및현금성자산의 순증가(감소)</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.031843e+12</td>
      <td>1.247801e+12</td>
    </tr>
    <tr>
      <th>272</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>CF</td>
      <td>ifrs_IncreaseDecreaseInCashAndCashEquivalents</td>
      <td>현금및현금성자산의 순증가(감소)</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1.323216e+12</td>
      <td>1.323216e+12</td>
    </tr>
    <tr>
      <th>273</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>CF</td>
      <td>dart_CashAndCashEquivalentsAtBeginningOfPeriodCf</td>
      <td>기초 현금및현금성자산</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.211144e+13</td>
      <td>-3.211144e+13</td>
    </tr>
    <tr>
      <th>274</th>
      <td>005930.KS</td>
      <td>삼성전자</td>
      <td>00126380</td>
      <td>CF</td>
      <td>dart_CashAndCashEquivalentsAtEndOfPeriodCf</td>
      <td>기말 현금및현금성자산</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>...</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.078823e+13</td>
      <td>-3.078823e+13</td>
    </tr>
  </tbody>
</table>
<p>275 rows × 24 columns</p>
</div>  


#### 병합 이후 문제점  
각 분기별 재무제표에서 같은 계정과목에 대해서   
("account_id", "account_nm")이 다르게 명시되어있는 경우가 많다.  

예를들면 "지배기업 소유주에 귀속되는 당기순이익" 계정과목에 대해서     
("ifrs_ProfitLossAttributableToOwnersOfParent", "지배기업 소유주 지분")도 존재하고,      
("ifrs-full_ProfitLossAttributableToOwnersOfParent", "지배기업 귀속 당기순이익")도 존재한다.  

즉, 회사별 재무제표 table에서 일반키 지정이 불가능한 상황이 된다. 
  
이를 해결하기 위해 병합한 회사별 재무제표 table 자체를 수정하기 보단   
계정과목의 "account_id"를 기준으로 row데이터를 추출하여  
새로운 계정과목별 table을 만들어보기로 했다.  
