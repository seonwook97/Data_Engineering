## Delta Lake Lakeflow Connect
- CREATE TABLE
  - CREATE TABLE AS(CTAS)는 기존 테이블 또는 데이터 소스에서 데이터를 선택하여 테이블을 만듦
  - 기본 테이블 형식은 Delta
  ```SQL
  CREATE TABLE mydeltatable
  USING DELTA -- 선택사항
  AS
  <query>
  ```
- UPLOAD UI
  - UPLOAD UI는 파일을 업로드하고 테이블을 생성할 수 있는 포인트 앤 클릭 인터페이스를 제공
  - CSV, TSV, JSON, Avro, Parquet 또는 TEXT 파일을 업로드 할 수 있음
  - 또한 UI를 사용하여 Unity Catalog 볼륨에 직접 파일을 업로드 할 수 있음
- COPY INTO
  - 파일 위치에서 Delta 테이블로 파일을 로드
  - 다양한 파일 형식 및 클라우드 저장 위치 지원
  - 스키마 변경을 자동으로 처리할 수 있음
  - 멱등성 작업 - 원본에 이미 로드된 파일을 건너뜀
  ```SQL
  COPY INTO mydeltatable
  FROM 'your-path'
  FILE_FORMAT = 'format'
  FILE_OPTIONS = ('format-options')
  ```
- Auto Loader
  - Auto Loader 새로운 데이터 파일이 클라우드 스토리지에 도착할 때 증분적으로(스트리밍) 처리 함
  - 스키마를 자동으로 유추하고 스키마 변경 사항을 수용함
  - 스키마를 준수하지 않는 데이터에 대한 복구 데이터 열을 포함함