# 의사결정 시트

직접 정한 항목과 아직 기본값인 항목을 구분한다. 수집기 쪽 상세 이유는 [ADR](../adr/README.md)과 [정리노트](PrismView_수집기_설계결정_정리노트.pdf)를 본다.

## 제품

| 항목 | 결정 | 근거 |
| --- | --- | --- |
| 상품 | USDT 마진 무기한 선물 BTC-USDT-SWAP, ETH-USDT-SWAP | 확정 |
| 전략 시간봉 | 30초봉. OKX 1초봉 30개를 집계 | [ADR-0002](../adr/0002-30s-candle.md) |
| 시그널 | 마감된 봉으로 계산한 레벨에 도달하면 진입. 리페인팅 없음 | 확정 |
| 주문 | 지정가. 레벨에 일정 거리 이내면 미리 걸어둠 | 확정 |
| 수수료 | OKX 기본 등급 메이커 0.02% | 확정 |
| 슬리피지 | 0 (지정가) | 주문 방식에서 도출 |
| 한 봉 안 손절·목표 동시 도달 | 더 짧은 봉으로 순서 판정, 모르면 손절 우선 | 확정 |
| 일봉/주봉 기준 | UTC | 기본값 |
| 화면 시간대 | KST | 기본값 |
| 사용 환경 | 개인 단독, 로컬, 인증 없음 | 기본값 |

## 수집기 (2026-10-27)

| 항목 | 결정 | 근거 |
| --- | --- | --- |
| DBMS | PostgreSQL 16 | [ADR-0001](../adr/0001-postgresql.md) |
| 원본 | 1초봉, 확정 봉만 | [ADR-0002](../adr/0002-30s-candle.md) |
| 저장 | 1초봉 + 30초봉 누적. PK `(inst_id, timeframe, ts)` | [ADR-0003](../adr/0003-cumulative-storage.md) |
| 백테스터 입력 | Parquet export | [ADR-0004](../adr/0004-parquet.md) |
| DB 접근 | JdbcTemplate 배치 upsert | 대량 insert · 복합키 |
| 백필 | 최근 3개월, `history-candles` + `after` 커서 | 1단계 범위 |
| 실행 | Spring `@Scheduled(fixedDelay)`, 마지막 저장 ts 이후 증분 | 노트북을 꺼 둬도 따라잡음 |
| 정합성 | `LAG`로 공백 탐지 → 재요청 → `known_gaps` | 백테스트 왜곡 방지 |
| 기반 | Spring Boot, Gradle, Java 21, Flyway, Testcontainers | 정리노트 9장 |

## 이후 기본값 (아직 구현 전)

| 항목 | 기본값 |
| --- | --- |
| 지표 계산 | 핵심 지표 직접 구현, RSI는 Wilder RMA |
| 백테스트 엔진 | 자체 이벤트 루프. 실시간과 같은 전략 코드 |
| 전략 정의 | 파이썬 파일 + 파라미터 스키마 |
| PrismQuant | 시그널 JSON 인터페이스만 정의 |
