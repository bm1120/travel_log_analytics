# travel_log_analytics
tralvel_log 분석환경 구축 공유용
- 해당 환경을 구축하려면 docker가 설치되어있어야 합니다(https://www.docker.com/)
- AI-hub 데이터(국내여행로그 데이터-수도권, https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100&aihubDataSe=data&dataSetSn=71776) 분석 환경 구축용
  - 해당 주제 관련 데이터(국내여행로그 데이터) 모두 추가 형태로 변경(2024-11-04)
  - 해당 데이터중 validation E 이동수단소비내역 e_e000920 데이터 고유키 안맞음 -> 강제로 PAYMENT_SEQ 변경 필요
  - csv 폴더 구조(각 파일을 해당 구조에 맞게 이동 후 실행 필요)
  ```
  csv_data/
  ├── train/
  └── val/
```
- 해당 데이터의 샘플 AI모델(여행로그 장소 추천 고도화)의 구동을 위해 환경 구축(recsys)
- recsys를 기반으로 파이썬 분석환경 세팅(jupyter)
- 현재 해당 데이터중 사진 데이터와 gps관련데이터를 제외한 나머지 정형 데이터 postgreDB 적재(postgres)
- postgres 데이터 jupyter에서 접근가능하도록 네트워크 세팅(docker-compose)
```
 travel_log_analtics/
  ├── config/
    ├── jupyter/
    ├── recsys/
  ├── csv_data/
  ├── notebooks/
  └── sql/
```
## 분석환경 세팅
### 1. 베이스 이미지(recsys) 생성
 - python 3.9(surprise 종속성 때문에 해당 버전 사용)
 - terminal창에서 config/recsys 폴더로 이동
   ```
   cd .\config\recsys\
   ```
 - 도커 이미지 빌드(약 2~3분이상 소요)
   ```
   docker build . -t recsys:py309 -f .\Dockerfile.dockerfile
   ```
### 2. 분석환경 이미지(jupyter) 생성
 - recsy 기반 jupter lab 환경
 - notebooks 폴더 working directory로 설정
 - 이전 terminal창에서 config/jupyter 폴더로 이동
   ```
   cd ..\jupyter
   ```
 - 도커 이미지 빌드(약 1~2분 이상 소요)
   ```
   docker build . -t jupyter -f .\Dockerfile.dockerfile
   ```
### 3. 분석환경 컨테이너 생성
 - Ai-hub의 해당 데이터 다운로드(train 데이터 기준)(1,2번 진행전에 진행 권장)
   - 현재 TL_csv, Other 데이터만 기준으로 적재
   - 다운로드 후 확장자.zip 이후 부분(.data0) 제거
 - 해당 데이터 csv_data 폴더로 모두 이동
 - postgreDB는 도커볼륨(travel_log_analytics_postgres_data)에 따로 생성됨(down시에도 사라지지 않음)
 - poi master 테이블(크기가 매우 큼) 때문에 postgres_db는 세팅이 오래 걸림(터미널에서 완료 후에도 추가로 3분이상 소요)
 - postgreDB port는 5432
 - jupyter는 http://localhost:8888 로 접속(port 8888)
   ```
   travel_log_analtics/
    ├── csv_data/
      ├── tc_codea_코드A.csv
      ├── tc_codea_코드B.csv
      ...
      └── tn_visit_area_info_방문지정보_E.csv
   ```
 - 이전 terminal창에서 docker-compose.yml이 있는 root로 이동
   ```
   cd ..\..
   ```
 - 도커 컴포즈 빌드
   ```
   docker-compose up -d
   ``` 
 - 해당 도커 컴포즈 제거시
   ```
   docker-compose down
   ``` 
 - 해당 도커 컴포즈 중지시
   ```
   docker-compose stop
   ``` 

### 4. 테스트
 - DB 세팅 확인시 주피터 접속(http://localhost:8888 ) sql_test.ipynb 노트북 내의 코드 실행
 - 샘플 모델 코드 확인시 model_analysis.ipynb 노트북 내의 코드 실행