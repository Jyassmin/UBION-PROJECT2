# RETURNUP **"한계기업 예측 모형을 통한 회생기업 투자 전략 검증 및 수익률 비교"**
<br><br>

🏆 2022 UBION Project2

![슬라이드1](https://user-images.githubusercontent.com/88031549/210496313-5ffc49fd-79cd-4f29-a5dc-cfd0fbe2a4eb.png)

![슬라이드6](https://user-images.githubusercontent.com/88031549/210496374-9d83bf4f-32b9-41f6-8f87-6e365a3c7b28.png)

![슬라이드18](https://user-images.githubusercontent.com/88031549/210496578-a88968ac-9e0c-4c55-9ab8-03a9ddf5d0f3.png)

![슬라이드75](https://user-images.githubusercontent.com/88031549/210510808-f6a218f5-8478-4e79-846e-28823181f3fc.png)

---
# **프로젝트 소개**

본 프로젝트는 카드사 고객을 대상으로 한국 신용카드 연체기준(NICE)에 근거한 연체기간을  예측하고, 연체고객에게 영향을 많이 미치는 주요 요인들을 중점으로 장단기연체자별 고객의 경향을 분석한다. 이를 통해 채무상환능력이 악회될 고객을 사전에 파악하고 연체고객의 경향에 따라 고객을 관리하여 카드사의 수익성을 높이는데 기여한다.

[ 배경 ]

최근 금리 인상 및 경기하락 추세가 이어지며 이자율, 물가가 상승해 국민들의 경제력이 하락하고 있는 실정이다. 이에 따라 카드사 고객은 낮아진 경제력으로 인해 카드 상환에 어려움을 겪으며 카드사 입장에서는 악성채권이 발생하고 있다. 2022년 상반기 말 기준 카드사 고정이하여신 중 추정손실 채권 평균 비중이 36.2%를 기록하며 채무상환능력이 악화된 고객들로 인해 카드사 수익성에 부정적인 영향을 미치고 있다.

[ 연체기간 예측 ]

 NICE에서 정의한 한국 신용카드 연체기간에 따라 고객을 4개의 클래스(0:비연체자, 1:단기연체자, 2:장기연체자, 3:초장기연체자)분류하고 고객의 부채, 상환, 신용, 개인 정보를 바탕으로 연체기간을 예측하는 연체기간예측모형 개발한다. 이에 따라 사전에 연체를 예측함으로써 채무상환능력이 악화될 고객들을 파악 및 관리하여 수익성에 기여할 수 있다.

[ 연체고객 경향분석 ]

 NICE에서 제시하는 국내 CB사 신용평가 요소(상환이력, 부채수준, 신용거래기간, 신용형태)에 근거하여 연체고객에게 영향을 많이 미치는 주요 피쳐(체불횟수, 체불액, 신용거래기간, 계좌수, 최소금액 지불여부 등)를 선정한다. 직관적인 분석을 위해 한국 신용카드 연체 기준에서 신용하락을 기준으로 고객을 단기연체자(class0), 장기연체자(class1)로 구분하였고, 위에서 선정한 주요 피쳐를 바탕으로 장단기 연체자별 경향을 분석하였다.(*ppt참고)

---
# **구조**


## **0. 데이터 수집**
- 출처 : KRX & TS200
- 수집기간 : 2011 ~ 2019(IFRS도입 이후 ~ 코로나 이전)
- 대상종목 : KOSPI & KOSDAQ 상장기업
- 제거 종목 : 금융 및 보험업, 스팩주, 상장폐지, 외국계기업
- 선정 피쳐 : 총 80개(재무재표데이터 + 파생변수 + 주가데이터)

## **1. Preprocessing**
- 거래소코드가 2개인 기업(26개) 중복 제거
- 회사명이 2개인 거래소코드 중복 확인
- 데이터가 하나 밖에 없는 기업 데이터 삭제
- 연결제무재표데이터 별도제무재표데이터로 대체
- 999999 (inf, -inf) 제거(이자보상배율 제외)
- 이자보상배율의 inf 값은 winsorizing 처리
- 양극단치 2.5%씩 winsorizing(Object 피쳐 제외)
- 파생변수(증가율) 이상치 처리
  - 전년도, 올해 값이 존재할 경우 증가율 컬럼 생성 
  - 전년도(0) & 올해(0)  :  증가율 =0
  - 전년도(0) & 올해(0이상) : drop
  - inf가 많이 생성되는 컬럼은 drop
- 잠재한계기업(2년 연속 이자보상배율이 1미만인 기업)으로 필터링
  - 기업당 데이터는 1개(최초 잠재한계 데이터만 사용)

## 2. Labeling
- Label1 : 한계기업((t+1)년도에 이자보상배율이 1미만인 기업.(3년 연속))
- Label0 : 회생기업((t+1)년도에 이자보상배율이 1이상인 기업. 즉 잠재한계기업 중 정상화된 기업)

## 3. Data split & Scaling
- Train set : 2012 ~ 2016
- Test set : 2017, 2018
- Scaling : Train set, test set 각각 Standard scaling(Data Leakage 방지)

## 4. Feature Selection
- Train set으로만 Feature Selection
- 검정
  - 정규성검정(Shapiro, Anderson, KS test, Jarque_bera test, Normal test, Dist plot, qq plot)
  - 독립성검정(Durbin-Watson)
  - 등분산검정(Bartlett test)
- 아래 4개 모델 중 3개 이상 중복 피쳐를 최종 선택
  - T-test(통계분석)
  - Lasso(Wrapper Method)
  - Stepwise(Embbeded Method)
  - Kbest(Filter Method)

## 3. Modeling

[단일분류]
- Logistic
- KNN
- Decision Tree
- Naive Bayes
- SVM

[Ensamble]
- Random Forest
**- XGB**
- LGBM
- Voting
- Stacking

[Deeplearning]
- TabNet

**-> 최종모델 : XGBClaasifier**


## 4. Evaluation(Test set)
- ACC : 0,73
- Precision : 0.73
- Recall : 0.97
- F1-score : 0.83
- AUC : 0.59

[Recall이 상대적으로 높은 성능을 보이는 모델]
- 이를 통해 한계기업으로 많이 예측하여 회생기업이라고 잘못 예측했을 때의 리스크를 줄임
- 회생기업(0)의 관점에서는 확실한 회생기업만 회생이라고 예측하기에 투자에 활용하기 좋은 모델 


## 5. 투자
- 백테스팅 기간 : 2013.04~2019.12(회계년도기준 (t+1)년 4월부터 (t+2)년 3월까지)
- 투자유니버스 : 2012~2018 중 예상회생기업
- 벤치마크 : 투자유니버스의 전종목 동등비중/Buy&Hold 수익률


[백테스팅]
- 연도마다 총 15(3x5)가지의 투자
 - 종목별 투자비중(3) : 동등비중 / 샤프지수 최대화 / 리스크 최소화
 - 매매전략 : Buy&Hold / 추세추종1/ 추세추종2 / 추세추종3 / 역추세

- 평가
  - 벤치마크와 CAGR/ Sharpe ratio로 비교평가(매년)


**-> 동등비중 x 역추세 전략이 최적**


(*아래 ppt예시)

![슬라이드7](https://user-images.githubusercontent.com/88031549/210512497-3c48b935-b056-4a3a-a8e1-40c8eab07cb3.png)

![슬라이드59](https://user-images.githubusercontent.com/88031549/210512614-d70995e6-d509-4e08-8c57-507585cbee29.png)

![슬라이드62](https://user-images.githubusercontent.com/88031549/210512677-5efb4d45-ddb2-49ba-aca2-6d7f0b6de407.png)

![슬라이드67](https://user-images.githubusercontent.com/88031549/210512752-ab9a91e8-3e91-49b8-b133-24e2e4d6a382.png)

![슬라이드87](https://user-images.githubusercontent.com/88031549/210512974-2d5083ff-95ca-43ef-b341-e878fdb70cba.png)

![슬라이드89](https://user-images.githubusercontent.com/88031549/210513071-15ace45c-d6c0-4a28-8728-0666d58ab5c7.png)