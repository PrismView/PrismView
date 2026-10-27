# ADR-0001 · 운영 저장소로 PostgreSQL 사용

- 상태: 채택
- 날짜: 2026-10-27
- 저장소: [prismview-collector](https://github.com/PrismView/prismview-collector)

## 배경

수집기는 OKX USDT 무기한 선물 BTC·ETH의 1초봉을 받아 30초봉으로 집계한다. 3개월 백필이면 1초봉이 종목당 약 780만 행, 두 종목이면 약 1,560만 행이다. 쓰기는 거의 추가만 있고(append-mostly), 읽기는 종목+시간 범위 조회와 시간봉 집계가 중심이다.

정리노트 기본안은 이미 아는 MariaDB였다. 규모만 보면 어떤 RDB든 충분하다. 기준은 성능이 아니라 정합성 도구, 이후 Python 백테스터와의 연결, 확장 경로였다.

## 결정

운영 저장소는 PostgreSQL 16이다. 스키마는 Flyway로 관리하고, 대량 적재는 JdbcTemplate 배치 upsert(`ON CONFLICT DO UPDATE`)를 쓴다.

## 검토한 대안

| 대안 | 쓰지 않은 이유 |
| --- | --- |
| MariaDB | upsert·윈도 함수는 충분하다. 다만 이후 TimescaleDB 확장, Python 분석 생태계, 표준 SQL 쪽은 PostgreSQL이 더 자연스럽다. |
| TimescaleDB | 하이퍼테이블·`time_bucket`은 매력적이지만 1,500만 행 규모에는 과하다. PostgreSQL을 그대로 두면 나중에 확장으로 붙일 수 있다. |
| ClickHouse | 집계는 빠르지만 upsert가 즉시 완료되지 않아, 재시도가 일상인 수집기와 맞지 않는다. |
| SQLite / DuckDB | 임베디드라 운영·동시 접근·포트폴리오 설득력이 약하다. DuckDB는 Parquet를 읽는 분석 쪽에 가깝다. |

## 이유

1. **규모에서 성능은 갈림이 아니다.** 1,500만 행 범위 조회는 노트북 PostgreSQL로 충분하다. "시계열이라서 시계열 DB"는 근거가 되지 않는다.
2. **정합성이 수집기의 핵심이다.** 자연키 `(inst_id, timeframe, ts)` + `ON CONFLICT`면 재시도·백필 겹침이 멱등하다. `LAG` 윈도 함수로 1초 공백을 찾는다.
3. **다음 단계와 맞다.** 백테스터는 Python, 웹은 같은 DB를 조회한다. PostgreSQL은 양쪽에서 자료가 많고, 나중에 TimescaleDB로 가는 길이 열려 있다.

## 결과 · 대가

- Docker Compose로 PostgreSQL만 띄우고 앱은 IDE에서 실행한다.
- 시간 버킷·연속 집계는 DB가 해주지 않으므로 집계 SQL·Java 코드를 직접 작성한다.
- 테스트는 H2가 아니라 Testcontainers PostgreSQL을 쓴다. upsert와 윈도 함수가 방언에 묶여 있기 때문이다.

## 다시 검토할 조건

종목이 수십 개를 넘거나 틱 단위까지 내려가 행 수가 억 단위가 되면 TimescaleDB(또는 열 저장)를 다시 본다.
