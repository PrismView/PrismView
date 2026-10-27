# PrismView

<p align="center">
  <img src="docs/images/prismview.png" alt="PrismView" width="360">
</p>

TradingView 유료 플랜 없이 BTC/ETH 차트 위에 나만의 매매 알고리즘 지표와 진입·청산 지점을 표시하고, 같은 로직으로 백테스트하는 개인용 차트 플랫폼입니다.

이 저장소는 **기획·설계 문서 허브**입니다. 코드는 언어별로 나눈 저장소에서 구현합니다.

## 저장소 구성

언어가 Java(Gradle), Python, 프론트엔드로 달라 한 저장소에 모으면 빌드와 CI가 복잡해집니다. 수집기는 독립 저장소로 두면 Java 프로젝트로 바로 보여줄 수 있습니다.

```
PrismView (organization)
 ├─ .github                 조직 소개
 ├─ PrismView               기획·설계 문서 허브 (여기)
 ├─ prismview-collector     Java · OKX 30초봉 수집기
 ├─ prismview-backtester    Python · 백테스트 엔진 (예정)
 └─ prismview-web           React · 차트 프론트엔드 (예정)
```

| 저장소 | 역할 | 상태 |
| --- | --- | --- |
| [PrismView](https://github.com/PrismView/PrismView) | 기획서, ADR, 아키텍처 | 진행 중 |
| [prismview-collector](https://github.com/PrismView/prismview-collector) | 1초봉 수집 → 30초봉 집계, PostgreSQL · Parquet | 1단계 |
| prismview-backtester | 실시간과 같은 전략 코드로 백테스트 | 예정 |
| prismview-web | Lightweight Charts v5 차트, 지표, 드로잉, 대시보드 | 예정 |

## 데이터 흐름

```
OKX REST (1s candles)
        │
        ▼
┌──────────────────────────────────┐
│  prismview-collector (Java)      │
│  - 3개월 백필 · 증분 수집          │
│  - 1초봉 → 30초봉 집계             │
│  - 누락 검사                      │
└──────────────────────────────────┘
        │                     │
        ▼                     ▼
  PostgreSQL              Parquet
  (운영 저장소)            (백테스터 입력)
        │
        ├──────────────────────┐
        ▼                      ▼
 prismview-backtester    prismview-web
 (Python, 예정)          (React, 예정)
```

- 과거·실시간 가격은 모두 OKX에서 받습니다. 거래소가 다르면 백테스트와 실시간이 어긋납니다.
- 원본은 1초봉을 누적 저장하고, 전략·차트용 30초봉은 1초봉을 집계해 같이 둡니다.
- 백테스터는 Parquet 스냅샷을 읽어 "같은 입력 → 같은 결과"를 재현합니다.

## 대상 상품

| 거래소 | 상품 | 수집 원본 | 전략 시간봉 |
| --- | --- | --- | --- |
| OKX | BTC-USDT-SWAP, ETH-USDT-SWAP | 1s | 30s (1초봉 30개를 집계) |

화면 표시는 KST, 저장·집계 경계는 UTC입니다.

## 오늘까지 정한 설계

자세한 이유와 대안 비교는 [docs/adr](docs/adr)에 있습니다.

| ADR | 결정 |
| --- | --- |
| [0001](docs/adr/0001-postgresql.md) | 운영 저장소는 PostgreSQL |
| [0002](docs/adr/0002-30s-candle.md) | 1초봉을 모아 30초봉을 만든다 |
| [0003](docs/adr/0003-cumulative-storage.md) | 원본·집계 봉을 DB에 누적 저장한다 |
| [0004](docs/adr/0004-parquet.md) | 백테스터 입력은 Parquet로 내보낸다 |

## 문서

- [기획 개요](docs/planning/overview.md)
- [의사결정 시트](docs/planning/decision-sheet.md)
- [수집기 설계 결정 정리노트](docs/planning/PrismView_수집기_설계결정_정리노트.pdf) — 선택지 비교와 함정 모음
- [ADR](docs/adr/README.md)

## 로드맵

- [x] 조직·저장소 분리, 수집기 설계 결정
- [ ] 수집기: PostgreSQL, 3개월 1초봉 백필, 30초봉 집계, 누락 검사
- [ ] 수집기: Parquet 내보내기
- [ ] 백테스터: 체결 모델, 요약 지표, 거래 목록
- [ ] 웹: 캔들 차트, 종목·시간봉 전환, 1차 드로잉 툴
- [ ] 실시간: OKX 웹소켓 중계, 끊김 복구
- [ ] 검증: 기존 TradingView 전략과 거래 내역·총 PnL 비교
- [ ] 연동: PrismQuant 시그널 형식 정의

## 라이선스

차트 렌더링에 [TradingView Lightweight Charts](https://github.com/tradingview/lightweight-charts)를 사용하며, 출처 표기를 유지합니다.
