---
title: project_finstate 계획서 
date: 2021-08-08
categories: project python 
---
### OpenDartReader 라이브러리를 활용한 재무제표 분석  
[OpenDartReader/github](https://github.com/FinanceData/OpenDartReader)  
  
### project_finstate
전체코드 [github.com/yeonseo-Jung/project_finance](https://github.com/yeonseo-Jung/project_finance/blob/aca4af282fedc2452e5f95f44f3d58ab07d4f09a/finstate.py)  

### 재무제표 분석을 통한 기업 분석 및 기업가치 평가 & 포트폴리오 구성 프로그램

### 목표
* when? 언제 
* what? 무슨 종목을  
* how? 어떤 전략으로 매도, 매수할 지를 제시하기  
**가장 중요한 것은 정량적 데이터를 통해 why?에 대한 대답**

### 필요한 기능

* 수정과 병합이 용이한 단일회사에 대한 분기별 재무제표
* 단일 회사에 대한 사업연도별, 분기별로 원하는 계정과목의 정확한 금액 호출
* 단일 회사에 대한 지배구조, 주주 현황, 임직원 현황,  자기주식 및 유통 주식 수 파악

### tables 
  
* corp_info (DataFrame) : 회사의 기본정보 table  

    columns = [stock_name, stock_code, corp_code] (리스트의 각 원소 자료형: string)

    key = stock_name

* fstate_all_account (DataFrame) : 단일회사 사업연도별, 분기별 전체 재무제표(BS, IS, CIS, CF)를 병합한 table  

    columns_0 = [stock_name, stock_code, corp_code] (리스트의 각 원소 자료형: string)

    columns_1 = [sj_div, account_id, account_nm] (리스트의 각 원소 자료형: string)

    columns_2 = [thstrm_amount_2021_1, thstrm_amount_2021_2, thstrm_amount_2020_1, thstrm_amount_2020_2, ...] (리스트의 각 원소 자료형: float32) ⇒ 최근 4개년치 분기별 금액

    병합(join) 기준 key = ["sj_div", "account_id", "account_nm"]
  
* fstate_core_account (DataFrame) : 다중회사의 특정 계정과목에 대한 분기별 금액 table  

    columns_0 = [stock_name, stock_code, corp_code, account] (리스트의 각 원소 자료형: string)

    columns_1 = [thstrm_amount_2021_1, thstrm_amount_2021_2, thstrm_amount_2020_1, thstrm_amount_2020_2, ...] (리스트의 각 원소 자료형: float32) ⇒ 최근 4개년치 분기별 금액  
      
* accounts (dictionary) : 계정과목 이름(key), 계정과목 id(value) 구조의 딕셔너리, 회사별, 분기별로 다른 계정과목 이름과 id 때문에 필요함  

    key = account_nm (string)  

    value = [account_id, ...] (리스트의 각 원소 자료형: string)
