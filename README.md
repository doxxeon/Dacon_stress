## 📌 데이콘 Basic 스트레스 지수 예측 : 건강 데이터로 마음의 균형을 찾아라!

### 📝 대회 개요
	•	주제: 스트레스 점수 예측 AI 알고리즘 개발
	•	목표: 신체 데이터를 기반으로 스트레스 점수를 예측하는 모델 개발
	•	평가 지표: MAE (Mean Absolute Error)
	•	Public Score: 전체 테스트 데이터 중 50%
	•	Private Score: 전체 테스트 데이터 중 나머지 50%

⸻

### 📂 데이터 설명

✅ Features
	•	범주형 변수
	•	gender: 성별
	•	activity: 생활시 운동 강도
	•	smoke_status: 흡연 상태
	•	medical_history: 만성질환
	•	family_medical_history: 가족력
	•	sleep_pattern: 수면패턴
	•	edu_level: 학력
	•	연속형 변수
	•	age: 연령
	•	height: 키(cm)
	•	weight: 몸무게(kg)
	•	cholesterol: 콜레스테롤 수치
	•	systolic_blood_pressure: 수축기 혈압
	•	diastolic_blood_pressure: 이완기 혈압
	•	glucose: 혈당 수치
	•	bone_density: 골밀도(g/cm²)
	•	mean_working: 1주일당 평균 근로 시간
	•	Target
	•	stress_score: 스트레스 점수 (0~1 사이 연속값)

⸻

### 🔎 EDA

1. 결측치 확인
	•	medical_history, family_medical_history, edu_level, mean_working 변수에서 결측치 존재

2. 히트맵
	•	age ↔ bone_density: 강한 음의 상관 (-0.94)
	•	혈압 관련 변수들(MAP, systolic, diastolic) ↔ age: 양의 상관

3. Boxplot
	•	mean_working, sleep_pattern, edu_level → stress_score와 약간의 패턴
	•	다른 변수들은 구간별 큰 차이 없음

4. Target 분포
	•	0~1 사이 거의 균등분포 → 라벨 불균형 문제 없음

⸻

### 🛠️ 데이터 전처리

1. 결측치 처리
	•	medical_history, family_medical_history: 결측 시 "no_history" 대체 + Missing Flag 추가
	•	edu_level: Iterative Imputer (MICE) 사용
	•	mean_working: KNN Imputer 사용

2. 범주형 변수 인코딩
	•	LabelEncoder 사용 → 모든 범주형 변수를 숫자로 변환

3. 파생변수 생성
	•	BMI = weight / height²  (로그 변환 적용)
	•	MAP = (2 × diastolic + systolic) / 3
	•	Age_BoneDensity_Index = age / bone_density (로그 변환 적용)
	•	Pulse_Pressure = systolic - diastolic
	•	Activity_BMI = BMI × activity
	•	Sleep_Work = sleep_pattern × mean_working

4. 구간화 (Binning)
	•	Age_bin: 연령대별 (10대 이하, 20대 … 70대 이상)
	•	BMI_bin: WHO 기준 (저체중, 정상, 과체중, 비만, 고도비만)
	•	MAP_bin: 혈압 단계 (정상, 전단계, 1단계, 2단계)
	•	Cholesterol_bin: 정상/경계/높음
	•	Glucose_bin: 정상/당뇨 전단계/당뇨

⸻

### 🤖 모델링 과정

✅ Validation & Public Score

모델	CV MAE	Validation MAE	Public Score
LightGBM	-	0.1728	0.1863
XGBoost	-	0.1687	0.1848
CatBoost	-	0.1980	0.2187


⸻

✅ 교차검증 (KFold 5-Fold)

모델	               CV MAE	  Public Score
LightGBM (기본)	    0.1815	0.1894
XGBoost (기본)	      0.1793	0.1860
LightGBM (+파생변수)	0.1821	-
XGBoost (+파생변수)	  0.1792	-


⸻

✅ GridSearchCV (XGBoost + 파생변수)
	•	Best Params

{
    "max_depth": 8,
    "learning_rate": 0.05,
    "subsample": 0.8,
    "colsample_bytree": 0.8
}


	•	CV MAE: 0.1741
	•	Public Score: 0.1714 🎉 (최고 성능)

⸻

### 📊 최종 결론
	•	단순한 LightGBM/XGBoost보다 GridSearchCV로 최적화된 XGBoost 모델이 가장 좋은 성능을 보임
	•	특히, 파생변수 추가 + 하이퍼파라미터 튜닝이 성능 향상에 크게 기여
	•	Public Score 0.1714로 기준 성능 대비 개선됨

⸻

📁 파일 구조

stress-prediction/
│── train.csv
│── test.csv
│── sample_submission.csv
│── baseline.ipynb            # 전체 파이프라인 노트북
│── xgb_gridsearch.csv        # 최종 제출 파일
│── README.md                 # 깃허브 소개 문서

