# 월간 데이콘 항공편 지연 예측 AI 경진대회
- 일부 레이블만 주어진 학습 데이터셋을 이용한 항공편 지연 여부 예측
- 레이블 없는 데이터와 함께 항공편 지연 여부 예측하는 AI 모델 개발
<br>

**데이터** <br>
- https://dacon.io/competitions/official/236094/overview/rules
- 외부 데이터 사용 금지
<br>

**평가** <br>
![image](https://github.com/ssyeon2/Flight-Delay-Prediction/assets/105052724/f2283b00-adfd-438a-80f4-81e8ab6b0c10)
- 심사 기준: LogLoss
- p는 target(Delayed)에 대한 확률값, 1-p는 Not_Delayed에 대한 확률값
- LogLoss
  - 분류 문제의 대표젓인 평가지표로 교차 엔트로피(cross-entropy)라고도 함
  - 분류 모델 자체의 잘못 분류된 수치적인 손실값(loss)을 계산
  - logloss는 낮을수록 좋은 지표
  - 로지스틱 회귀모델에서의 손실 함수
<br>

**사용 툴**
- Google colab
<br>

**최종 결과 및 성과**
- LogLoss 최종 0.67236
- 상위 3% 전체 14등
<br>
<br>

## 해결방안
### 1. EDA <br>
**대응 변수 확인하기**
1) 1:1대응 <br>
   (1) Origin_Airport, Origin_Airport_ID <br>
   (2) Destination_Airport, Destination_Airport_ID <br>
   (3) Airline, Carrier_ID(DOT) <br>
   -> 둘 중 하나의 변수는 drop <br>
2) 1:N대응 <br>
   (1) Origin_Airport, Origin_State <br>
   (2) Destination_Airport, Destination_State <br>
3) 대응 관계 없음 <br>
   (1) Airline, Tail_Number <br>
   (2) Airline, Carrier_Code(IATA) <br>

### 2. 결측치 처리 <br>
1) Estimated_Departure_Time, Estimated_Arrival_Time 변수
   -> KNNImputer 사용
2) _Airport_ID → _State 변수
   -> 1:1 대응으로 결측치 대체
3) Carrier_ID → Airline 변수
   -> 1:1 대응으로 결측치 대체
4) 대응으로 채울 수 없는 값
   - Label을 제외한 결측치가 존재하는 변수 → 최빈값

### 3. 파생변수 생성
1) Total time : 도착시간 - 출발 시간
2) Departure_Hour, Departure_Minute, Arrival_Hour, Arrival_Minute
3) Departure_Time : Departure Hour(0-2,3-5,6-8,9-11,12-14,15-17,18-20,21-24)
4) Arrival_Time : Arrival Hour(0-2,3-5,6-8,9-11,12-14,15-17,18-20,21-24)

### 4. 사용 변수 추출
- 'Origin_Airport'
- 'Origin_State'
- 'Destination_Airport'
- 'Destination_State'
- 'Distance'
- 'Airline'
- 'Carrier_ID(DOT)'
- 'Total_time'
- 'Month'
- 'Departure_Hour'
- 'Arrival_Hour'
- 'Delay'
  
### 5. 스케일링
1. 범주화
   - 범주 컬럼이 너무 많은 관계로 get_dummies 사용시 Ram 부족 현상 발생
   - **빈도 인코딩**(Frequency Encoding)
       - 각 라벨의 횟수 혹은 출현 빈도로 범주형 변수를 대체하는 방법
       - 학습 데이터와 테스트 데이터를 결합하여 변환
       - 각 라벨의 출현 빈빈도와 목적변수 간에 관련성이 있을 때 유효
    - 원핫인코딩(One-hot-encoding)
        - 출현 빈도가 적은 변수에 한해 One-hot-encoding 진행
2. Min-max scaling
   - 수치형 데이터에 한해 min-max scaling 진행

### 6. 비지도 학습
selftraining <br>
   - randomforest 정확도 0.818<br>
   - **catboost 정확도 0.827**<br>
   - decisiontree 정확도 0.713<br>

### 7. Voting
<img width="300" alt="스크린샷 2023-09-08 오전 4 42 57" src="https://github.com/kwonkw/Flight_Delay_Prediction/assets/131172214/b259330a-4d84-48c1-b41a-317dbb74d9de">

  - 세 가지 모델을 Hard Voting /Soft Voting 으로 앙상블하여 정확도를 확인해 본 결과 0.822

### 7. 불균형 데이터 
![다운로드 (1)](https://github.com/ssyeon2/Flight-Delay-Prediction/assets/105052724/963fc5ef-6662-4df5-8b36-148d985ac57a)
- SMOTE 사용
- Oversamgpling하여 불균형 데이터 처리

### 8. 모델 학습
- LogLoss 지표에 성능이 좋은 LogisticRegresion 사용
- GridSearchCV 사용
- 최종 0.67236
- 상위 3%

