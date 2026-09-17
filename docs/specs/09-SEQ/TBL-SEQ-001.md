---
doc_id: TBL-SEQ-001
type: SEQ
title: Glovis 완성차 인텔리전스 시퀀스
status: draft
upstream: [TBL-UC-001, TBL-INFRA-001, TBL-DOM-002, TBL-DOM-003, TBL-API-001]
---

# SEQUENCE

## 0. 이 문서가 다루는 것

[[TBL-DOM-002]]가 정한 클래스 스물여덟이 **언제 어떤 순서로 서로를 부르는가**를 정한다. 클래스가 무엇을 할 줄 아는지는 그 문서가, 테이블과 컬럼은 [[TBL-DOM-003]]이, 함수 하나의 입출력 계약은 [[TBL-MS-001]]이 정한다. 이 문서는 그 사이에 있다.

시퀀스는 열넷이다. 열람 둘, 배치 전체 하나, 판정 구간 다섯, 서술 구간 셋, 운영 셋이다.

**이 문서의 뼈대는 한 줄이다. 판정은 배치 1~5단계에서 코드가 끝내고, 6~8단계는 그 결과를 사람이 읽을 수 있게 만드는 일이다**([[TBL-INFRA-001#C19]]). 그래서 [[#SEQ-13]]에 그 경계를 눈에 보이게 그렸고, 서술 셋이 전부 실패해도 변동·후보·근접도·신호등·도메인 상태가 그대로 게시되는 경로를 같은 그림 안에 alt로 두었다. [[#SEQ-18]]에 LLM 생명선이 아예 없는 것이 그 경계의 다른 표현이다.

관련도 등급과 점수는 어느 시퀀스에도 나오지 않는다([[TBL-PRD-001#R26]]). 후보에 붙는 값은 근접도 넷(날짜 차이·기사 수·출처 수·국가 일치 방식)과 정렬 규칙이 낳은 자리 번호뿐이다.

항목 ID는 SEQ-11부터 시작한다. SEQ-1부터 SEQ-10까지는 이전 판(7단계 배치, LLM 두 지점)에서 쓰다 삭제된 ID이므로 재사용하지 않는다.

### 0.1 생명선

| 생명선 | 약어 | 실체 | 종류 | 정의한 곳 |
|:--|:--|:--|:--|:--|
| 현업 사용자 | H | VODA에서 대시보드 3본과 C 리포트 단독 주소를 여는 담당자. 읽기만 한다 | 액터 | [[TBL-UC-001#UC-H1]] |
| 관리자 | ADM | 운영 개발자(적재·배치·재생성)와 자사 DBA(매핑) | 액터 | [[TBL-UC-001#UC-A1]] |
| 배치 스케줄러 | SCHED | 매일 02시에 배치를 깨운다. worker 컨테이너에서 한 번에 하나만 돈다 | 인프라 | [[TBL-INFRA-001#C8]] |
| 열람·관리 API | API | FastAPI 라우터. 판단하지 않고 서비스로 넘긴다 | 인프라 | [[TBL-API-001]] 3장 |
| 파일 볼륨 | VOL | 받은 파일 원본을 그대로 보관한다 | 인프라 | [[TBL-INFRA-001#C7]] |
| raw 스키마 | DB_RAW | 받은 파일을 행 그대로. 형태 A는 원장 행, 형태 B는 펼치기 전 셀 | 저장소 | [[TBL-DOM-003#raw_pivot_cell]] |
| std 스키마 | DB_STD | 표준화한 완성차 긴 형태, 기사, 시장지표 | 저장소 | [[TBL-DOM-003#vehicle_measure]] |
| master 스키마 | DB_MASTER | 크로스워크 이관본과 버전, 국가·해협·법인 | 저장소 | [[TBL-DOM-003#crosswalk_version]] |
| mart 스키마 | DB_MART | 판정이 사는 자리. 1~5단계가 여기까지 채운다 | 저장소 | [[TBL-DOM-003#anomaly]] |
| pub 스키마 | DB_PUB | 게시본. 열람 계정이 읽는 유일한 스키마 | 저장소 | [[TBL-DOM-003#c_report]] |
| ops 스키마 | DB_OPS | 적재·배치 이력, LLM 호출, 설정 버전, 미매핑 | 저장소 | [[TBL-DOM-003#batch_run]] |
| 브리핑 갈래 저장소 | BRIEF | A 판정과 A 리포트 원본. 읽기 전용 계정으로만 붙는다 | 외부 | [[TBL-INFRA-001#C5]] |
| H-chat 게이트웨이 | HCHAT | 사내 LLM. 배치 6~8단계만 부른다 | 외부 | [[TBL-INFRA-001#C2]] |
| PipelineRunner | RUN | 여덟 단계를 순서대로 돌리고 어느 실패에서 멈출지 정한다 | 클래스 | [[TBL-DOM-002#PipelineRunner]] |
| IngestService | ING | 적재 진입점. 관리 API와 배치 1단계가 같이 부른다 | 클래스 | [[TBL-DOM-002#IngestService]] |
| FormDetector | FD | 완성차 파일이 형태 A인지 B인지 가른다 | 클래스 | [[TBL-DOM-002#FormDetector]] |
| FormALedgerParser | PA | IF 원장을 읽는다. 행마다 기준일자가 있다 | 클래스 | [[TBL-DOM-002#FormALedgerParser]] |
| FormBPivotParser | PB | 피벗 리포트의 병합 헤더를 편다 | 클래스 | [[TBL-DOM-002#FormBPivotParser]] |
| RecordStandardizer | STZ | 두 형태를 같은 긴 형태 한 테이블로 만든다 | 클래스 | [[TBL-DOM-002#RecordStandardizer]] |
| RegressionChecker | RC | 이번 적재를 직전과 견줘 급변인지 본다 | 클래스 | [[TBL-DOM-002#RegressionChecker]] |
| AJudgmentReader | AJR | A 판정과 내부 분해를 읽어 복사한다. 쓰지 않는다 | 클래스 | [[TBL-DOM-002#AJudgmentReader]] |
| EventClusterer | EVC | 기사를 사건으로 묶고 기사 수와 출처 수를 센다 | 클래스 | [[TBL-DOM-002#EventClusterer]] |
| FactJoiner | FJ | 완성차 표준 행을 국가와 기간으로 모은다 | 클래스 | [[TBL-DOM-002#FactJoiner]] |
| StageFlowCalculator | SFC | 판매 네 계열의 단계 차이를 센다. 재고 신호가 나오는 자리 | 클래스 | [[TBL-DOM-002#StageFlowCalculator]] |
| AnomalyDetector | AD | 변동을 확정한다. 원인은 여기서 보지 않는다 | 클래스 | [[TBL-DOM-002#AnomalyDetector]] |
| CandidateSearcher | CS | 변동 하나에 원인 후보를 붙인다 | 클래스 | [[TBL-DOM-002#CandidateSearcher]] |
| ProximityCalculator | PX | 후보마다 셀 수 있는 값 넷을 세고 정렬한다. 상태가 없다 | 클래스 | [[TBL-DOM-002#ProximityCalculator]] |
| TrafficLightJudge | TLJ | 신호등을 규칙으로 정하고 워치리스트 줄과 도메인 상태 3건을 만든다 | 클래스 | [[TBL-DOM-002#TrafficLightJudge]] |
| EventNamer | EN | 이미 묶인 사건에 이름만 붙인다 | 클래스 | [[TBL-DOM-002#EventNamer]] |
| CauseLinkWriter | CLW | 변동과 후보가 어떻게 이어지는지 문단 하나를 쓴다 | 클래스 | [[TBL-DOM-002#CauseLinkWriter]] |
| ClaimWriter | CW | 헤드라인, 도메인 상태 3, 국가 카드 문장을 쓴다 | 클래스 | [[TBL-DOM-002#ClaimWriter]] |
| CitationVerifier | CV | 모델이 쓴 인용이 후보·근거 밖으로 나갔는지 대조한다. 코드다 | 클래스 | [[TBL-DOM-002#CitationVerifier]] |
| DegradeHandler | DH | 실패한 역할을 기록하고 배치를 계속 진행시킨다 | 클래스 | [[TBL-DOM-002#DegradeHandler]] |
| ReportPublisher | RP | 근거에 번호를 매기고 리포트 한 본을 게시한다. A 리포트 사본도 게시 스키마에 올린다 | 클래스 | [[TBL-DOM-002#ReportPublisher]] |
| CReportService | CRS | 게시된 C 리포트를 읽어 준다. 다시 계산하지 않는다 | 클래스 | [[TBL-DOM-002#CReportService]] |
| DomainReportService | DRS | A1·A2·A3 중 한 도메인의 게시 사본을 읽어 준다 | 클래스 | [[TBL-DOM-002#DomainReportService]] |
| MarketService | MKT | 지표 하나의 시계열과 as-of 값을 읽어 준다 | 클래스 | [[TBL-DOM-002#MarketService]] |
| BatchService | BAT | 배치 상태와 이력, 재실행, 게시 전환 | 클래스 | [[TBL-DOM-002#BatchService]] |
| MasterService | MST | 매칭률, 미매핑, 크로스워크 업로드와 확정 | 클래스 | [[TBL-DOM-002#MasterService]] |
| HChatClient | HCC | 사내 게이트웨이 호출. 서술 계층만 쓴다 | 클래스 | [[TBL-DOM-002#HChatClient]] |
| BriefingStoreReader | BSR | A 판정과 A 리포트 문서를 읽는다. 쓰기 메서드가 없다 | 클래스 | [[TBL-DOM-002#BriefingStoreReader]] |

### 0.2 이 문서가 그리지 않는 것

A 리포트를 만드는 브리핑 갈래의 내부 순서는 그리지 않는다. 이 시스템에서 보이는 것은 [[#SEQ-15]]의 읽기 둘뿐이다. 판정 읽기와 리포트 문서 읽기다. 태블로 대시보드의 렌더링, 뉴스 API가 기사에 분류와 영향도를 붙이는 과정도 밖에 있다.

A 리포트에서 C 리포트로 가는 화면 흐름은 없다. 그래서 [[#SEQ-11]]과 [[#SEQ-12]]는 서로를 부르지 않고 각각 독립된 그림이다([[TBL-UC-001#UC-H4]]).

## 1. 열람 시퀀스

#### SEQ-11 C 리포트 열람

근거는 [[TBL-UC-001#UC-H1]]과 [[TBL-UC-001#UC-H2]]다. 게시 스키마 조회로 끝나고 LLM 호출이 0회다([[TBL-INFRA-001#C9]] [[TBL-INFRA-001#C10]]).

```mermaid
sequenceDiagram
  autonumber
  actor H as 현업 사용자
  participant API as 열람 API
  participant CRS as CReportService
  participant MKT as MarketService
  participant DB_PUB as pub 스키마
  H->>API: GET /api/intel/creport/latest (VODA 서명 토큰)
  API->>CRS: get_latest()
  CRS->>DB_PUB: c_report 부분 인덱스로 최신 게시본 1건
  DB_PUB-->>CRS: reportId, baseDate, 소스별 최신일 셋과 dataLatestDate, settingVersion, degraded
  CRS->>DB_PUB: report_anomaly, report_watch_item, report_claim, report_evidence, report_claim_evidence, report_alert_event
  DB_PUB-->>CRS: 표시값이 복사된 행. 조인 없음
  CRS-->>API: CReport 한 덩어리 (marketBar 4, headline, domainStatus 3, anomalies, evidence, notices)
  API-->>H: 화면 한 장
  Note over H,DB_PUB: 근거 펼치기는 추가 호출이 없다. 후보와 기사가 이미 응답 안에 있다
  opt 게시본이 아직 없다
    CRS-->>API: 404 problems/no-published-report
    API-->>H: 다음 배치 예정 시각 안내
  end
  opt 최신본이 강등본이다
    CRS-->>API: degradedRoles 포함
    API-->>H: 템플릿 문장 표시. 신호등과 후보 순서는 그대로
  end
  opt 지표 하나의 시계열을 연다
    H->>API: GET /api/intel/market/series/{indicatorId}
    API->>MKT: get_series(indicatorId, days)
    MKT->>DB_PUB: market_series 게시 사본. indicator_id와 period_key로
    DB_PUB-->>MKT: 값, 변화율, 이월 여부, 원천 관측일
    MKT-->>API: MarketMetric 목록
    API-->>H: 시계열
  end
  opt 과거 버전을 연다
    H->>API: GET /api/intel/creport/version/{reportId}
    API->>CRS: get_by_id(reportId, viewer)
    CRS->>CRS: assert_published_or_admin (get_by_id 처리문). 현업 토큰으로 미게시본을 부르면 404
  end
```

**읽을 때 볼 것.** 화살표가 [[TBL-DOM-002#CReportService]]에서 `pub` 스키마로만 간다. `mart`로 가는 화살표가 하나도 없는 것이 [[TBL-INFRA-001#C10]]이고, 조회 결과에 조인이 없는 것이 [[TBL-PRD-001#N5]]의 2초다. 후보 목록과 기사가 첫 응답에 이미 들어 있어 근거 펼치기가 서버를 다시 부르지 않는다. 게시본 없음만 404이고 나머지 예외(변동 없음·강등·판정 미수신·지표 이월)는 200으로 내려가 `notices`에 담긴다. 시장지표 시계열도 [[TBL-DOM-003#market_series]]라는 게시 사본을 읽으므로 이 경로가 `pub` 밖으로 나가는 자리가 없다. 데이터 최신일은 완성차·뉴스·시장 셋을 각각 내려보내고, 한 줄로 줄일 때만 셋 중 가장 늦은 `dataLatestDate`를 쓴다.

#### SEQ-12 A 리포트 열람

근거는 [[TBL-UC-001#UC-H4]]다. 브리핑 갈래가 쓴 문장을 게시 사본 그대로 내주고, 응답 어디에도 외부요인 인용이 없다.

```mermaid
sequenceDiagram
  autonumber
  actor H as 현업 사용자
  participant API as 열람 API
  participant DRS as DomainReportService
  participant DB_PUB as pub 스키마
  H->>API: GET /api/intel/areport/domain/{domain}
  API->>DRS: get_latest_by_domain(domain)
  DRS->>DB_PUB: a_report_snapshot 최신 1건 (domain, report_base_date, version_no)
  DB_PUB-->>DRS: 요약, 해설, 고정 문구, 트래킹 지표, 분해, 못 만드는 지표, published_at
  DRS->>DB_PUB: a_report_evidence
  DB_PUB-->>DRS: 문장 각주와 근거. footnote, kind, source_id, display_value
  DRS->>DRS: build_domain_extra(domain, report)
  DRS->>DRS: list_missing_metrics(domain). 사본의 못 만드는 지표를 그대로 내보낸다
  DRS-->>API: DomainReport (summary, commentary, evidence, versions, trackingMetrics 3, breakdownDimensions, missingMetrics, domainExtra)
  API-->>H: A1 생산 또는 A2 재고 또는 A3 판매 한 장
  Note over DRS,DB_PUB: 문장을 새로 쓰지 않는다. 브리핑 갈래가 쓴 문장을 사본 그대로 내보낸다
  Note over DRS,DB_PUB: 응답에 뉴스와 시장지표 인용이 없다. C 리포트로 가는 링크 필드도 없다
  Note over DRS,DB_PUB: 못 만드는 지표는 A2 넷(afloat, awaitingShipment, entityVsDealer, countryDailySeries)과 A3 하나(globalCountryByModel)다
  opt 그 도메인 사본이 아직 없다
    DRS-->>API: 404
  end
  opt 과거 버전을 연다
    H->>API: GET /api/intel/areport/version/{domainReportId}
    API->>DRS: get_by_id(domainReportId). 사본의 version_no로 고른다
  end
```

**읽을 때 볼 것.** 이 그림에 [[TBL-DOM-002#CReportService]]도 [[TBL-DOM-002#CandidateSearcher]]도 없다. A 리포트는 자기 대시보드 데이터 안에서만 말하고, 외부 원인은 C 리포트의 일이다([[TBL-PRD-001#R29]]). 화살표가 [[TBL-DOM-003#a_report_snapshot]]과 [[TBL-DOM-003#a_report_evidence]]로만 가는 것이 [[TBL-INFRA-001#C10]]이다. 열람 경로는 게시 스키마 밖으로 나가지 않고, 이 시스템은 A 리포트 문장을 새로 쓰지 않는다. 버전 알약과 사이드 버전 목록의 번호는 사본의 `version_no`, 곧 브리핑 갈래가 매긴 번호 그대로다. 빈 자리를 추정값으로 채우지 않고 `missingMetrics`에 이유를 적는 것이 이 화면의 인수 기준이다.

## 2. 배치 전체

#### SEQ-13 새벽 배치 8단계와 실패 분기

근거는 [[TBL-UC-001#UC-S1]]과 [[TBL-INFRA-001#C19]]다. 단계 이름 여덟은 `PipelineRunner.stage_names()`가 주는 것을 그대로 쓴다.

```mermaid
sequenceDiagram
  autonumber
  participant SCHED as 배치 스케줄러
  participant RUN as PipelineRunner
  participant ING as IngestService
  participant AJR as AJudgmentReader
  participant EVC as EventClusterer
  participant FJ as FactJoiner
  participant AD as AnomalyDetector
  participant CS as CandidateSearcher
  participant TLJ as TrafficLightJudge
  participant EN as EventNamer
  participant CLW as CauseLinkWriter
  participant CW as ClaimWriter
  participant DH as DegradeHandler
  participant RP as ReportPublisher
  participant DB_OPS as ops 스키마
  participant DB_PUB as pub 스키마

  SCHED->>RUN: run(baseDate, trigger=schedule, startStage=1)
  RUN->>DB_OPS: batch_run 생성 (baseDate, settingVersion, llmEnabled, backfillMode)

  rect rgb(222, 235, 255)
    Note over RUN,TLJ: 판정 구간 1~5단계. 코드만 쓴다. 여기서 화면에 나갈 값이 전부 확정된다
    RUN->>ING: 1단계 적재. commit(preflightId, options)
    ING-->>RUN: ingestFileId 목록, 회귀 급변 여부, options.backfillMode
    RUN->>DB_OPS: batch_run.backfill_mode에 그 값을 남긴다
    opt 1단계 실패 또는 회귀 급변
      RUN->>DB_OPS: batch_stage_result(1, blocked)
      RUN-->>SCHED: 중단. 이전 게시본 유지. 관리자가 사유를 적고 푼다
    end
    RUN->>AJR: 2단계 A 판정 읽기
    alt 스냅샷 정상
      AJR-->>RUN: domain_judgment, contribution 복사. aJudgmentSnapshotId
      AJR->>RP: publish_a_report(domain, document). A 리포트 문장과 각주 근거와 버전을 게시 사본으로
      RP->>DB_PUB: a_report_snapshot, a_report_evidence
    else 스냅샷 없음 또는 모양 어긋남
      AJR-->>RUN: 보완 집계로 대체. notices 판정 미수신
      Note over AJR: 읽기 실패가 배치를 멈추지 않는 유일한 코드 단계다
    end
    RUN->>EVC: 3단계 사건 묶음
    EVC-->>RUN: event, event_article. 기사 수와 출처 수
    opt 3단계 실패
      RUN->>DB_OPS: batch_stage_result(3, failed)
      RUN-->>SCHED: 중단. 기사 수와 출처 수가 신호등의 입력이다
    end
    RUN->>FJ: 4단계 결합
    FJ->>AD: country_period_fact, sales_stage_flow
    AD-->>RUN: anomaly 목록 확정
    RUN->>CS: 5단계 원인 후보
    CS->>TLJ: cause_candidate와 근접도 값 넷
    TLJ-->>RUN: traffic_light, watch_item
    TLJ->>TLJ: build_domain_status(domain, anomalies). 도메인 상태 3카드의 신호등과 판정 출처
    TLJ->>RP: 워치리스트와 도메인 상태를 바로 넘긴다
    opt 4단계 또는 5단계 실패
      RUN->>DB_OPS: batch_stage_result(stage, failed)
      RUN-->>SCHED: 중단. 판정이 없으면 리포트가 성립하지 않는다
    end
  end

  alt backfillMode가 참
    RUN->>DB_OPS: batch_stage_result(6, 7, 8을 skipped로)
    Note over RUN,DH: 백필은 서술을 만들지 않는다. 판정은 5단계에서 이미 끝났다
  else 평시
    rect rgb(255, 238, 222)
      Note over RUN,DH: 서술 구간 6~8단계. LLM 셋. 확정된 판정을 읽을 수 있게 만드는 일이다
      RUN->>EN: 6단계 사건 명명 (LLM 역할 1)
      opt 실패, 콘텐츠 필터, JSON 위반, 토큰 상한
        EN->>DH: degrade(naming, reason)
      end
      RUN->>CLW: 7단계 연관 설명 (LLM 역할 2)
      opt 실패 또는 인용 검증 2회 실패
        CLW->>DH: degrade(causeLink, reason)
      end
      RUN->>CW: 8단계 문장 생성 (LLM 역할 3)
      opt 실패 또는 검증 2회 실패
        CW->>DH: degrade(narration, reason)
      end
    end
  end

  RUN->>RP: 8단계 게시
  alt 서술 셋이 전부 강등됐거나 백필로 건너뛰었다
    DH-->>RP: degradedRoles 셋 전부
    RP->>RP: 템플릿에 변동, 후보 제목, 근접도 값을 채우고 각주를 후보 순서대로 기계 부여
    RP->>DB_PUB: c_report (degraded 표시, notices)
    Note over RP,DB_PUB: 변동, 후보, 근접도, 신호등, 도메인 상태는 5단계 산출물 그대로 게시된다
  else 서술이 정상이다
    CW-->>RP: 검증을 통과한 문장과 각주
    RP->>DB_PUB: c_report (published, notices)
  end
  RP-->>RUN: reportId, version
  RUN->>DB_OPS: 단계별 소요, 처리 건수, LLM 호출 수와 토큰
  opt 배치 창 4시간 초과
    RUN->>DB_OPS: 진행 중 단계를 마치고 경고를 남긴다
  end
  RUN-->>SCHED: 종료
```

**읽을 때 볼 것.** 색이 다른 두 상자가 이 문서의 전부다. 파란 상자 안에 [[TBL-DOM-002#HChatClient]] 생명선이 없고, 주황 상자 안에 `mart`로 값을 되쓰는 화살표가 없다. 한 방향으로만 흐른다. 마지막 alt가 그 경계의 증명이다. 서술 셋이 전부 강등돼도 [[TBL-DOM-002#ReportPublisher]]로 들어오는 입력 중 판정 산출물은 바뀌지 않아 같은 신호등, 같은 후보 순서, 같은 도메인 상태가 게시된다([[TBL-INFRA-001#C13]]).

실패 처리가 단계마다 다른 것도 같이 본다. 1·3·4·5단계는 멈춤, 2단계는 보완 집계로 내려가고 계속, 6~8단계는 강등하고 계속이다. 이 갈림은 [[TBL-INFRA-001#C20]] 그대로이며, 멈춘 배치는 이전 게시본을 유지하고 실패한 단계부터 다시 돈다.

주황 상자를 감싼 alt가 백필이다. 관리 API가 받은 `backfillMode`는 [[TBL-DOM-002#IngestService]]의 `commit`을 거쳐 `batch_run.backfill_mode`로 남고, 참이면 6~8단계를 건너뛴 판정만의 게시본이 나간다. 2단계 안의 `publish_a_report` 화살표는 A 리포트 열람 경로의 입구다. C 생성은 A의 판정만 쓰고, 문장 사본은 [[#SEQ-12]]만 읽는다.

## 3. 판정 구간 (배치 1~5단계)

#### SEQ-14 형태 판별과 적재

근거는 [[TBL-UC-001#UC-A1]]과 [[TBL-INFRA-001#C6]]다. 관리 화면의 수동 적재와 배치 1단계가 같은 코드를 부른다.

```mermaid
sequenceDiagram
  autonumber
  actor ADM as 관리자
  participant API as 관리 API
  participant ING as IngestService
  participant VOL as 파일 볼륨
  participant FD as FormDetector
  participant PA as FormALedgerParser
  participant PB as FormBPivotParser
  participant STZ as RecordStandardizer
  participant RC as RegressionChecker
  participant DB_RAW as raw 스키마
  participant DB_STD as std 스키마
  participant DB_OPS as ops 스키마

  ADM->>API: POST /api/admin/ingest/preflight (sourceType, file, fileBaseDate)
  API->>ING: preflight(sourceType, upload, fileBaseDate)
  ING->>VOL: store_original (preflight 안). 판별 전에 먼저 보존한다
  ING->>FD: detect(filePath, sourceType)
  alt 형태 A. IF 원장
    FD-->>ING: form=A. 첫 행이 헤더이고 행마다 기준일자가 있다
    ING->>PA: parse, check_columns, base_date_range
    PA-->>ING: 컬럼 어긋남 목록, 기준일자 처음과 끝, 미래 골격 행 수
    ING->>ING: resolve_idempotency_unit (preflight 안). 기준일자 단위 멱등 재적재
  else 형태 B. 피벗 리포트
    FD-->>ING: form=B. 병합 헤더 2~3줄이고 행에 날짜가 없다
    ING->>PB: expand_merged_header, split_period_and_measure, drop_total_rows
    PB-->>ING: 기간 블록 목록, 지표 종류 목록, 총계 행 수, 차원 조합 수
    ING->>PB: detect_file_base_date(raw)
    opt 파일 안에서 기준일을 못 읽는다
      PB-->>ING: 없음
      ING-->>ADM: 관리자 입력 요구. fileBaseDateSource=manual
    end
    ING->>ING: resolve_idempotency_unit (preflight 안). 파일 기준일 단위 덮어쓰기
  else 어느 쪽도 아니다
    FD->>FD: explain_mismatch(filePath)
    FD-->>ING: form=unknown
    ING-->>ADM: 422 problems/form-undetected. 어느 대목이 다른지 목록. 적재 차단
  end
  ING-->>API: PreflightResult (preflightId, formDetection, previewA 또는 previewB, overwrite 대상)
  API-->>ADM: 미리보기. 무엇이 지워지는지 보고 누른다

  ADM->>API: POST /api/admin/ingest/commit (preflightId, options)
  API->>ING: commit(preflightId, options). options.backfillMode는 batch_run.backfill_mode로 간다
  ING->>DB_RAW: raw_ledger_row 또는 raw_pivot_cell
  ING->>STZ: standardize(rows, ingestFile)
  STZ->>STZ: build_period_key(form, baseDate, periodType). 시간은 언제나 두 칸이다
  STZ->>STZ: map_country(dealerCode). 앞 세 자리가 국가 코드다
  STZ->>STZ: null_when_missing (standardize 안). 결측을 0으로 바꾸지 않는다
  STZ->>DB_STD: vehicle_measure 긴 형태 행. 생산 행의 country_code는 비운다
  STZ->>DB_OPS: collect_unmapped 결과를 unmapped_value로
  ING->>RC: check(ingestFile, previous)
  alt 급변
    RC-->>ING: is_abrupt=true, abrupt_reasons
    ING->>DB_OPS: ingest_regression 기록. 배치 자동 실행 차단
    Note over ING,DB_OPS: 적재는 완료된다. 막히는 것은 배치 자동 실행뿐이다
  else 정상
    RC-->>ING: 행수, 결측률, 매칭률, 중복률
  end
  ING->>DB_OPS: ingest_file 이력
  ING-->>ADM: 형태, 기간 단위, 행수, 결측률, 매칭률
  opt 차단을 푼다
    ADM->>API: POST /api/admin/ingest/unblock/{ingestFileId} (reason 필수)
    API->>ING: unblock_regression(ingestFileId, reason, actor)
  end
```

**읽을 때 볼 것.** 첫 화살표가 판별이 아니라 원본 보존이다. 판별이 실패해도 원본은 볼륨에 남는다([[TBL-PRD-001#N7]]). alt 세 갈래가 [[TBL-INFRA-001#C6]]의 두 형태와 판별 불가이고, 셋째 형태를 자동 추론하는 갈래는 일부러 없다. 두 갈래가 서로 다른 것은 위쪽뿐이고 [[TBL-DOM-002#RecordStandardizer]] 아래로는 같은 화살표다. 같은 파일이 어느 경로로 들어와도 같은 표준 행이 나와야 하기 때문이다. `preflight`과 `commit`을 둘로 나눈 이유는 형태 B가 파일 기준일 단위 통째 덮어쓰기라서 사람이 무엇이 지워지는지 보고 눌러야 해서다. 급변일 때 적재는 끝나고 배치 자동 실행만 막히는 갈림이 오른쪽 아래에 있다. `store_original` `resolve_idempotency_unit` `null_when_missing`은 계약으로 실린 함수가 아니라 소속 함수 안의 사설 헬퍼이며, 그래서 라벨 옆에 소속을 적었다.

#### SEQ-15 A 판정 읽기

근거는 [[TBL-UC-001#UC-S10]]과 [[TBL-INFRA-001#C5]]다. 배치 2단계이고 값을 다시 계산하지 않는다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant AJR as AJudgmentReader
  participant BSR as BriefingStoreReader
  participant BRIEF as 브리핑 갈래 저장소
  participant RP as ReportPublisher
  participant DB_STD as std 스키마
  participant DB_MART as mart 스키마
  participant DB_PUB as pub 스키마
  RUN->>AJR: 2단계 read(baseDate)
  AJR->>BSR: ping()
  alt 접속 정상
    BSR-->>AJR: true
    loop domain in production, inventory, sales
      AJR->>BSR: read_judgments(baseDate, domain)
      BSR->>BRIEF: 읽기 전용 조회
      BRIEF-->>BSR: 스냅샷 판정 payload
      BSR-->>AJR: 지표, 값, 비교값, 기준 방식, 변동 여부
      AJR->>AJR: validate_shape(payload)
      alt 기대한 키와 필드가 있다
        AJR->>BSR: read_contributions(judgmentId)
        BSR-->>AJR: 기여 상위 항목. A 리포트가 쓴 문장은 읽지 않는다
        AJR->>BSR: read_report_document(baseDate, domain)
        BSR->>BRIEF: 읽기 전용 조회
        BRIEF-->>BSR: A 리포트 문서
        BSR-->>AJR: 요약, 해설, 고정 문구, 트래킹 지표, 분해, 못 만드는 지표, 각주 근거, version_no, published_at
        AJR->>RP: publish_a_report(domain, document)
        RP->>DB_PUB: a_report_snapshot, a_report_evidence 사본
      else 어긋난 목록이 나온다
        AJR->>AJR: fallback_to_supplementary(domain, baseDate)
        AJR->>DB_STD: vehicle_measure 직접 집계
        Note over AJR: 보완 집계로 만든 변동은 기여 분해가 빈다. A 리포트 사본도 없다
      end
    end
    AJR->>BSR: snapshot_id(baseDate)
    BSR-->>AJR: aJudgmentSnapshotId
    AJR->>DB_MART: copy_snapshot. domain_report_snapshot, domain_judgment, contribution
  else 접속 실패
    BSR-->>AJR: false
    AJR->>AJR: fallback_to_supplementary(domain, baseDate) 셋 전부
    AJR->>DB_STD: vehicle_measure 직접 집계
  end
  AJR-->>RUN: 도메인 3의 판정과 미수신 목록. notices에 담긴다
  Note over AJR,DB_PUB: mart 사본은 판정만 담는다. 문장은 pub 사본에만 있고 C 리포트의 서술에 쓰지 않는다
  Note over AJR,DB_MART: 기간 키는 데이터 형태를 따른다. 형태 A면 periodType=day, 형태 B면 헤더 기간 구분과 파일 기준일
  Note over AJR,DB_MART: A1은 목적지 국가가 없어 axis_type이 plant로 남는다. 국가 보완이 되지 않는다
```

**읽을 때 볼 것.** [[TBL-DOM-002#BriefingStoreReader]] 쪽으로 가는 화살표에 쓰기가 하나도 없다. 클래스에 쓰기 메서드를 두지 않았고 계정도 읽기 전용이다. 읽어 오는 것이 둘로 갈리는 것이 이 그림의 핵심이다. `read_judgments`와 `read_contributions`가 가져온 판정은 `copy_snapshot`으로 `mart`에 앉고 C 리포트가 그것을 쓴다. `read_report_document`가 가져온 문장·각주 근거·버전은 `publish_a_report`를 타고 `pub` 사본으로 앉고 [[#SEQ-12]]만 그것을 읽는다. 원본이 나중에 바뀌어도 그날 리포트는 `aJudgmentSnapshotId`로 같은 입력을 다시 쓴다([[TBL-INFRA-001#C11]]). 실패 갈래가 둘 다 `fallback_to_supplementary`로 내려가고 배치는 계속된다. 이 단계가 배치를 멈추지 않는 유일한 코드 단계다. 마지막 Note가 [[TBL-INFRA-001#C16]]이 여기서 처음 나타나는 자리이며, A1의 판정이 `plant` 축으로 남는 순간 이후 [[#SEQ-18]]에서 신호등 대상에서 빠진다.

#### SEQ-16 사건 묶음

근거는 [[TBL-UC-001#UC-S2]]다. 배치 3단계이고 LLM 호출이 0회다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant EVC as EventClusterer
  participant DB_STD as std 스키마
  participant DB_MASTER as master 스키마
  participant DB_MART as mart 스키마
  RUN->>EVC: 3단계 cluster(articles, windowDays)
  EVC->>DB_STD: 당일 신규 article, article_country
  DB_STD-->>EVC: 기사와 국가 태그, 카테고리, 기사에 붙어 온 영향도
  EVC->>DB_MASTER: strait, strait_country
  DB_MASTER-->>EVC: 해협에 인접한 국가 목록
  EVC->>EVC: 국가와 카테고리로 나눈다. 해협 이슈는 인접국을 보탠다
  EVC->>DB_MART: 열린 사건 조회. last_seen_date가 시간창 이내
  DB_MART-->>EVC: 열린 event 목록
  alt 열린 사건에 붙는다
    EVC->>DB_MART: event_article 추가. last_seen_date 갱신
  else 붙을 사건이 없다
    EVC->>DB_MART: event 새로 생성. first_seen_date 기록
  end
  EVC->>EVC: count_sources(articles). 서로 다른 매체 이름의 개수
  EVC->>EVC: pick_representative(articles)
  EVC->>EVC: max_impact(articles). 붙어 온 영향도의 최대값. 새로 매기지 않는다
  EVC->>DB_MART: article_count, source_count, title은 대표 기사 제목, named_by=representativeArticle
  opt 시간창을 넘긴 사건
    EVC->>DB_MART: 상태를 종료로 바꾼다
  end
  opt 국가 태그가 없는 기사
    EVC->>EVC: 국가 없음 묶음에 둔다. 후보 검색 대상에서 뺀다
  end
  EVC-->>RUN: event 목록. LLM 호출 0회
  Note over EVC,DB_MART: 기사 수와 출처 수가 신호등의 입력이다. 그래서 실패가 강등이 아니라 멈춤이다
```

**읽을 때 볼 것.** [[TBL-DOM-002#HChatClient]] 생명선이 이 그림에 없다. 묶음은 규칙이고 이름만 LLM이며 이름은 [[#SEQ-19]]에서 따로 붙는다. 이 분리가 [[TBL-DOM-002]]가 지시받은 계층 배치에서 바꾼 한 가지다. 기사 수로만 세면 많이 보도된 나라가 자동으로 커지므로 묶어서 세고, 한 사건이 몇 개 매체에서 나왔는지를 따로 센다. `title`이 이 단계에서 이미 채워지는 것도 눈여겨본다. 명명이 실패해도 사건 제목이 비지 않고, 그때 `named_by`는 `representativeArticle`이다. 이 칸이 가질 수 있는 값은 `llm`과 `representativeArticle` 둘뿐이다. 해협에서 인접국을 펴는 화살표가 [[TBL-DOM-003#strait_country]]로 가는데, 이 값이 없으면 중동 변동의 후보가 통째로 빈다.

#### SEQ-17 결합과 단계별 흐름 계산

근거는 [[TBL-UC-001#UC-S3]]과 [[TBL-INFRA-001#C17]]이다. 배치 4단계에서 변동 목록이 확정된다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant FJ as FactJoiner
  participant SFC as StageFlowCalculator
  participant AD as AnomalyDetector
  participant MKT as MarketService
  participant DB_STD as std 스키마
  participant DB_MART as mart 스키마
  RUN->>FJ: 4단계 join(periodKey)
  FJ->>DB_STD: vehicle_measure. is_total_row 제외
  DB_STD-->>FJ: 국가, 기간, 지표 종류별 값
  FJ->>FJ: skip_when_country_null (join 안). 생산 행이 여기서 빠진다
  FJ->>FJ: resolve_exposure(country, periodKey). 형태 B는 CBU 컬럼, 형태 A는 모델코드 유도
  FJ->>MKT: as_of(indicatorId, 기간의 마지막 날)
  MKT-->>FJ: MarketPoint 또는 이월 한도 초과
  FJ->>FJ: count_events(country, periodKey)
  FJ->>DB_MART: country_period_fact, model_exposure
  FJ->>SFC: calculate(country, periodKey)
  SFC->>SFC: entity_stage_gap. 선적에서 도매정본을 뺀다
  SFC->>SFC: dealer_stage_gap. 도매정본에서 소매를 뺀다
  SFC->>SFC: gap_rate(gap, 도매정본), alternative_diff(실 도매, 도매(공식))
  SFC->>DB_MART: sales_stage_flow. wholesale_basis와 derivation_type=derived를 행에 남긴다
  SFC-->>AD: 체류 변동 재료
  RUN->>AD: detect(facts, judgments, flows)
  AD->>AD: from_a_judgment(judgment). A 판정이 있는 지표는 그 값을 쓴다
  AD->>AD: from_stage_flow(flow)
  AD->>AD: from_supplementary(fact). A 판정이 없는 지표만
  AD->>AD: pick_compare_basis(fact). 계획, 없으면 전년 동월, 그다음 전월
  AD->>AD: progress_rate(actual, plan). 진도율 컬럼을 믿지 않고 직접 계산한다
  AD->>DB_MART: anomaly. axis_type, period_type, file_base_date, setting_version, a_judgment_snapshot_id
  AD-->>RUN: 변동 목록 확정
  Note over AD,DB_MART: 감지 단위는 차종 그룹이다. 세부 차종은 contribution 분해에만 쓴다
  Note over SFC,DB_MART: 단계 차이의 음수는 예외가 아니다. 앞 단계에 쌓인 것을 덜어내는 중이라는 값이다
  opt 시장지표 이월 한도 초과
    MKT-->>FJ: 값 없음
    FJ->>DB_MART: 지표 바에 기준일 지연으로 표시. 판정에 쓰지 않는다
  end
  opt 재고 원천이 들어왔다
    SFC->>DB_MART: derivation_type을 measured로 갈아 끼운다. 유도 행은 지우지 않는다
  end
  opt 중복 적재 의심
    FJ->>DB_MART: 플래그. 도메인 상태에 데이터 확인 필요를 넣는다
  end
```

**읽을 때 볼 것.** `skip_when_country_null` 한 줄이 생산을 국가 축에서 떨어뜨린다. 목적지 국가가 데이터에 없어 유추하지 않는다는 결정이 코드 한 줄로 지켜지는 자리다([[TBL-INFRA-001#C16]]). [[TBL-DOM-002#StageFlowCalculator]]로 가는 화살표가 재고 신호가 나오는 유일한 경로다. 재고 원천이 없어 판매 네 계열의 단계 차이로 만들고, 행에 `derivation_type`을 남겨 원천이 들어왔을 때 어느 기간이 유도값이었는지 되짚을 수 있게 한다([[TBL-INFRA-001#C17]]). 실측으로는 미주 누계에서 선적 679,551, 도매 677,201, 소매 643,097이고 법인 구간이 2,350대, 딜러 구간이 34,104대(5.0%)다. `progress_rate`를 직접 계산하는 화살표도 실측 때문이다. 인입 샘플에서 진도율 컬럼은 총계 행에만 값이 있고 개별 행은 전부 0이었다.

#### SEQ-18 후보 검색, 근접도, 신호등

근거는 [[TBL-UC-001#UC-S8]]과 [[TBL-INFRA-001#C19]]다. 배치 5단계이고 여기서 판정이 끝난다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant CS as CandidateSearcher
  participant PX as ProximityCalculator
  participant TLJ as TrafficLightJudge
  participant DB_MART as mart 스키마
  participant RP as ReportPublisher
  RUN->>CS: 5단계 search(anomaly). 변동마다
  CS->>CS: axis_type이 plant면 대상에서 뺀다. 생산은 여기서 빠진다
  CS->>DB_MART: search_events(anomaly). 같은 국가와 해협 귀속, 시간창 안
  DB_MART-->>CS: 사건과 상위 기사 3건
  CS->>DB_MART: search_market(anomaly). 지표 4종의 변화율이 임계 초과
  CS->>DB_MART: search_cross_domain(anomaly). 같은 국가의 다른 도메인 변동
  CS->>CS: resolve_country_match. 국가 직접, 해협 귀속, 같은 국가 다른 도메인, 날짜만 일치
  CS->>PX: 후보 목록
  PX->>PX: day_diff. 변동의 file_base_date가 기준점, 사건은 last_seen_date
  PX->>PX: calculate. 날짜 차이, 기사 수, 출처 수, 국가 일치 방식 넷
  PX->>PX: sort. 날짜 차이 오름차순, 같으면 출처 수 내림차순, 그다음 기사 수 내림차순
  PX->>PX: sort_rule_text(). 이 규칙을 한 문장으로 돌려준다
  PX-->>CS: 정렬된 목록과 자리 번호
  CS->>CS: apply_limit(candidates). 상한을 적용하고 잘린 건수를 함께 돌려준다
  CS->>DB_MART: cause_candidate. sort_order, 근접도 값 넷, 잘린 건수
  PX->>TLJ: 후보와 근접도
  TLJ->>TLJ: meets_threshold (judge 안의 조건). 기사 수 최소 이상, 출처 수 최소 이상, 날짜 차이 시간창 이하
  alt 셋을 채우고 완성차 노출이 확인됐다
    TLJ->>DB_MART: traffic_light=red. basis에 값과 기준값을 쌍으로
  else 셋을 채웠으나 노출 미확인
    TLJ->>DB_MART: traffic_light=yellow
  else 하나라도 못 채웠다
    TLJ->>DB_MART: traffic_light=none. 목록에는 남는다
  end
  TLJ->>TLJ: build_watch_items(anomalies). 신호등 순 정렬
  TLJ->>DB_MART: watch_item
  TLJ->>TLJ: build_domain_status(domain, anomalies). 도메인 안 변동 신호등의 최댓값
  Note over TLJ: judgmentSource는 aJudgment, supplementaryAggregate, notReceived 셋 중 하나다. 생산은 notApplicable 판정 대상 아님이다
  TLJ->>RP: 워치리스트와 도메인 상태 3건을 바로 넘긴다. 서술 계층을 거치지 않는다
  TLJ-->>RUN: 5단계 완료. 화면에 나갈 판정이 전부 확정됐다
  Note over CS,TLJ: 이 그림에 LLM 생명선이 없다. 등급, 점수, 순위를 만드는 호출도 없다
  Note over PX,DB_MART: sortOrder는 관련도 순위가 아니라 정렬 규칙이 낳은 자리 번호다
```

**읽을 때 볼 것.** 생명선 목록에 [[TBL-DOM-002#HChatClient]]가 없는 것이 이 시퀀스의 요지다. LLM을 끄고 같은 기준일을 돌려도 같은 신호등과 같은 후보 순서가 나와야 하며, 그 대조를 회귀 검사에 둔다([[TBL-INFRA-001#C19]]). 첫 화살표가 `plant` 축을 걸러내고, 마지막에서 두 번째 화살표가 [[TBL-DOM-002#TrafficLightJudge]]에서 [[TBL-DOM-002#ReportPublisher]]로 바로 간다.

워치리스트와 함께 도메인 상태 3건도 여기서 만들어진다. 도메인 상태 카드의 신호등은 그 도메인 안 변동 신호등의 최댓값이고, 판정 출처는 A 판정을 받았는지 보완 집계로 내려갔는지 아예 못 받았는지를 적는다. 기여 상위는 2단계에서 복사한 분해의 사본이다. 8단계 LLM은 이 셋 위에 문장만 얹는다([[#SEQ-21]]). 그래서 서술이 전부 실패해도 3카드의 신호등이 그대로 게시된다. 생산 카드는 국가 축이 없어 신호등 대상이 아니며 `notApplicable` "판정 대상 아님"으로 적는다. 후보가 없어 `none` "원인 미확인"이 된 국가와 뜻이 다르므로 같은 말로 적지 않는다.

워치리스트를 게시기가 만들면 8단계 산출물이 되고 8단계에는 LLM이 섞여 있어 "LLM을 꺼도 워치리스트가 같다"를 보장할 수 없다. alt 세 갈래가 신호등 규칙 전부이며 모델이 끼어드는 자리가 없다. 잘린 건수를 따로 돌려주는 것은 화면에 적기 위해서다. 후보가 상한에 걸려 사라진 것을 모르면 "후보가 이것뿐"이라고 읽힌다.

## 4. 서술 구간 (배치 6~8단계)

#### SEQ-19 사건 명명 (LLM 역할 1)

근거는 [[TBL-UC-001#UC-S11]]이다. 배치 6단계이고 손대는 칸은 사건 이름 하나다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant EN as EventNamer
  participant HCC as HChatClient
  participant HCHAT as H-chat 게이트웨이
  participant DH as DegradeHandler
  participant DB_MART as mart 스키마
  participant DB_OPS as ops 스키마
  RUN->>EN: 6단계 name_events(events)
  EN->>DB_MART: 후보로 뽑힌 사건 중 아직 명명되지 않은 것만
  DB_MART-->>EN: 사건 목록
  loop 사건마다
    EN->>EN: build_prompt (name_events 안). 상위 5건의 제목과 출처만 담는다
    EN->>HCC: complete(prompt, jsonSchema, purpose=naming)
    HCC->>HCC: remaining_tokens()
    alt 남은 토큰이 있고 응답이 스키마에 맞다
      HCC->>HCHAT: 요청. 심각도 필드는 스키마에 없다
      HCHAT-->>HCC: title, type
      HCC->>DB_OPS: log_call (complete 안). naming, model, tokens, latency, ok
      HCC-->>EN: 제목과 유형
      EN->>DB_MART: event.title, event.type, named_by=llm
    else 실패, 콘텐츠 필터, JSON 위반, 토큰 상한, 백필 모드
      HCC->>DB_OPS: log_call (complete 안). naming, tokens, latency, filtered 또는 jsonViolation 또는 error
      EN->>EN: fallback_to_representative(event)
      EN->>DH: degrade(naming, reason, eventId)
      DH->>DB_OPS: 강등 역할과 사유 기록
      EN->>DB_MART: event.title은 대표 기사 제목, named_by=representativeArticle
    end
  end
  EN-->>RUN: 명명 건수
  Note over EN,DB_MART: 기사 수, 출처 수, 영향도는 3단계 값 그대로다. 명명은 신호등과 후보 순서를 바꾸지 않는다
```

**읽을 때 볼 것.** [[TBL-DOM-002#EventClusterer]]가 이 그림에 없다. 묶음은 3단계에 끝났고 이 단계는 이미 만들어진 사건의 이름 칸만 채운다. `article_count`나 `source_count`로 가는 화살표가 없는 것이 그 증거이며, 그래서 이 단계가 통째로 실패해도 [[#SEQ-18]]의 신호등이 흔들리지 않는다. `named_by`가 갈리는 두 자리를 나란히 본다. 모델이 붙이면 `llm`, 실패해 대표 기사 제목으로 돌아가면 `representativeArticle`이다. 후보로 뽑힌 사건만 고르는 첫 화살표는 호출 수를 줄이려는 것이다. 묶인 사건 전부에 이름을 붙이면 화면에 나오지 않을 것까지 부른다.

#### SEQ-20 연관 설명 (LLM 역할 2)

근거는 [[TBL-UC-001#UC-S9]]다. 배치 7단계이고 설명은 변동마다 하나다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant CLW as CauseLinkWriter
  participant HCC as HChatClient
  participant HCHAT as H-chat 게이트웨이
  participant CV as CitationVerifier
  participant DH as DegradeHandler
  participant DB_MART as mart 스키마
  RUN->>CLW: 7단계 write(anomaly, candidates, llm). 변동마다
  CLW->>CLW: build_prompt (write 안). 후보 밖의 사실은 넣지 않는다
  CLW->>HCC: complete(prompt, jsonSchema, purpose=causeLink)
  HCC->>HCHAT: 요청. 스키마에 등급, 점수, 순위를 뜻하는 필드가 없다
  HCHAT-->>HCC: explanationText, citedCandidateIds
  HCC-->>CLW: 설명 문단과 인용 목록
  CLW->>CLW: extract_citations(text)
  CLW->>CV: verify_cause_link(causeLink, candidates)
  alt 인용이 전부 후보 안이고 수치가 입력과 같다
    CV-->>CLW: true
    CLW->>DB_MART: cause_link. 기본키가 anomaly_id이고 cited_candidate_ids를 담는다
  else 후보 밖 인용, 인과 단정, 등급 어휘
    CV->>CV: out_of_scope_ids(cited, allowed)
    CV-->>CLW: false
    CLW->>HCC: 검증 사유를 붙여 1회 재요청
    alt 재요청이 통과했다
      CLW->>DB_MART: cause_link
    else 다시 실패했다
      CLW->>DH: degrade(causeLink, citationFailed, anomalyId)
    end
  end
  opt H-chat 실패, 콘텐츠 필터, JSON 위반
    CLW->>DH: degrade(causeLink, reason, anomalyId)
  end
  CLW-->>RUN: 설명 건수
  Note over CLW,DB_MART: 신호등은 여기서 건드리지 않는다. 5단계에서 이미 정해졌다
  Note over CV,DH: 쓴 쪽과 검사하는 쪽을 나눴다. 모델이 쓴 것을 모델에게 검사시키지 않는다
```

**읽을 때 볼 것.** [[TBL-DOM-002#TrafficLightJudge]]로 가는 화살표가 없다. 설명은 신호등을 읽지도 바꾸지도 않는다. [[TBL-DOM-002#CitationVerifier]]가 별도 생명선으로 서 있는 것이 계층 규칙이며, 검사를 코드가 해야 검사가 성립한다. 첫 화살표가 LLM 포트를 인자로 받는 것도 같은 규칙이다. 쓰는 쪽이 어댑터를 속성으로 들고 있지 않아야 LLM을 끈 실행이 성립한다. 프롬프트에 후보 목록만 넣는 것도 같은 이유다. 후보 밖의 값을 넣으면 검증기가 잡을 수 없는 문장이 나온다. 강등으로 끝난 변동은 설명 자리만 비고 후보 목록과 근접도 값 넷은 화면에 그대로 남는다. `cause_link`의 기본키가 `anomaly_id`라는 것이 후보마다 등급을 매기던 구조가 사라진 자리다.

#### SEQ-21 문장 생성, 검증, 강등 (LLM 역할 3)

근거는 [[TBL-UC-001#UC-S4]]와 [[TBL-UC-001#UC-S7]]이다. 배치 8단계이고 각주 번호는 코드가 먼저 매긴다.

```mermaid
sequenceDiagram
  autonumber
  participant RUN as PipelineRunner
  participant RP as ReportPublisher
  participant CW as ClaimWriter
  participant HCC as HChatClient
  participant CV as CitationVerifier
  participant DH as DegradeHandler
  participant DB_MART as mart 스키마
  participant DB_PUB as pub 스키마
  RUN->>RP: 8단계 시작
  RP->>DB_MART: anomaly, cause_candidate, cause_link, watch_item, 도메인 상태 3건
  RP->>RP: number_evidence(anomalies, candidates). 각주 번호를 코드가 먼저 매긴다
  RP->>DB_PUB: report_evidence. footnote, kind, source_id, display_value, url
  RP->>CW: 번호가 매겨진 근거 목록을 넘긴다
  CW->>CW: build_prompt (write_headline 안). 신호등이 붙은 국가 상위와 그 첫 후보
  CW->>HCC: complete 헤드라인
  CW->>CW: build_prompt (write_domain_status 안)
  Note over CW: 생산 구역 프롬프트에는 외부 후보를 아예 넣지 않는다
  Note over CW: 도메인 상태의 신호등, 판정 출처, 기여 상위는 5단계 산출물이다. 여기서는 문장만 쓴다
  CW->>HCC: complete 도메인 상태 3장
  CW->>CW: build_prompt (write_card 안). 변동과 설명
  CW->>HCC: complete 국가 카드
  HCC-->>CW: text, footnotes, highlights
  CW->>CV: verify_claim(claim, evidence)
  alt 검증 통과
    CV-->>CW: true
    CW->>DB_PUB: report_claim, report_claim_evidence
  else 각주가 근거 밖, 금지어, 입력에 없는 수치, A 리포트 문장과 동일
    CV-->>CW: false
    CW->>HCC: 1회 재생성
    alt 재생성이 통과했다
      CW->>DB_PUB: report_claim
    else 다시 실패했다
      CW->>DH: degrade(narration, verificationFailed)
    end
  end
  DH-->>RP: degraded_roles(batchRunId)
  opt 서술이 강등됐거나 백필로 건너뛰었다
    RP->>RP: 템플릿에 변동, 후보 제목, 근접도 값을 채우고 각주를 후보 순서대로 기계 부여
    RP->>DB_PUB: report_claim에 강등 표시
  end
  RP->>RP: copy_display_values(evidence)
  RP->>DB_PUB: report_anomaly, report_watch_item. mart에서 참조가 아니라 복사
  RP->>DB_PUB: c_report.domain_status. 5단계가 만든 신호등과 판정 출처에 8단계 문장만 얹는다
  RP->>DB_PUB: c_report.notices. noAnomaly, generationDegraded, aJudgmentNotReceived, batchFailed, marketCarriedOver
  RP->>DB_PUB: c_report.market_bar 네 칸. 지표가 이월이어도 빈 값으로 내려간다
  RP->>DB_PUB: c_report의 소스별 최신일 셋. vehicle_latest_date, news_latest_date, market_latest_date
  RP->>RP: next_version (publish 1단계)
  RP->>DB_PUB: c_report 한 행. data_latest_date는 소스별 최신일 셋 중 가장 늦은 날이다
  RP-->>RUN: reportId, version
  Note over RP,DB_PUB: 서술이 비어 있어도 게시한다. 변동, 후보, 근접도, 신호등, 도메인 상태는 5단계 산출물 그대로다
```

**읽을 때 볼 것.** `number_evidence`가 첫 화살표이고 `complete`가 그 뒤다. 순서가 뒤집히면 모델이 번호를 만들게 되고 없는 각주가 생긴다([[TBL-PRD-001#N4]]). 생산 구역의 Note가 [[TBL-INFRA-001#C16]]을 코드로 지키는 방법이다. 프롬프트에 외부 후보를 넣지 않는 것으로 지키지 검증기로 뒤에서 거르지 않는다.

도메인 상태를 쓰는 화살표가 둘로 갈리는 것도 같이 본다. [[TBL-DOM-002#ClaimWriter]]는 문장만 만들고, 신호등·판정 출처·기여 상위는 5단계가 넘긴 값을 [[TBL-DOM-002#ReportPublisher]]가 그대로 적는다. `notices`도 게시기가 채운다. 다섯 코드가 화면의 예외 상태와 1대1이라 E1 변동 없음, E3 생성 실패 강등, E4 판정 미수신, E5 배치 실패, E6 지표 바 지연이 코드 하나씩을 가져간다. `domain`은 E4에서 어느 카드에 붙일지에, `lastSuccessDate`는 E5의 마지막 성공일에 쓰인다.

`mart`에서 `pub`으로 가는 화살표에 "복사"라고 적힌 것이 [[TBL-INFRA-001#C10]]이다. 참조로 두면 열람이 `mart`를 읽어야 한다. 마지막 두 화살표가 강등 여부와 무관하게 언제나 그려지는 것이 이 그림의 결론이다.

## 5. 운영 시퀀스

#### SEQ-22 재실행, 재생성, 게시 전환

근거는 [[TBL-UC-001#UC-A4]]와 [[TBL-PRD-001#N3]]이다. 기존 게시본을 덮어쓰지 않는다.

```mermaid
sequenceDiagram
  autonumber
  actor ADM as 관리자
  participant API as 관리 API
  participant BAT as BatchService
  participant RUN as PipelineRunner
  participant RP as ReportPublisher
  participant DB_OPS as ops 스키마
  participant DB_PUB as pub 스키마
  ADM->>API: GET /api/admin/batch/status
  API->>BAT: get_status()
  BAT->>DB_OPS: batch_run 최근, abrupt 플래그가 선 ingest_file
  BAT-->>ADM: 실행 중 여부, 자동 실행을 막고 있는 것
  ADM->>API: GET /api/admin/batch/run/{batchRunId}
  API->>BAT: get_run(batchRunId)
  BAT->>DB_OPS: batch_stage_result 여덟 행, llm_call
  BAT-->>ADM: 단계 이름 여덟과 상태. 이름은 stage_names()가 준다
  ADM->>API: POST /api/admin/batch/rerun (baseDate, startStage, backfillMode, compareWith)
  API->>BAT: request_rerun(baseDate, startStage, options)
  BAT->>BAT: assert_not_running (request_rerun 1단계)
  alt 같은 기준일 배치가 돌고 있다
    BAT-->>ADM: 409 problems/batch-in-progress
  else 실행 가능
    BAT->>RUN: run(baseDate, trigger=rerun, startStage)
    Note over RUN: 그 기준일의 데이터 스냅샷, 당시 A 판정 스냅샷, 당시 설정 버전으로 돈다
    opt backfillMode가 참
      RUN->>RUN: 5단계까지만 채우고 6~8단계를 건너뛴다
      Note over RUN: backfillMode는 API가 받는 값이고 llmEnabled는 재현성 시험용 내부 스위치다. 둘 다 남는다
    end
    RUN->>DB_PUB: 새 버전 생성. published는 false
    RUN-->>BAT: batchRunId, 새 reportId
    BAT-->>ADM: 202 queued 또는 running
  end
  ADM->>ADM: 변동, 후보, 근접도, 신호등, 도메인 상태를 당시와 대조한다. 같은 입력이면 같아야 한다
  Note over ADM: 설명과 문장은 새로 생성되므로 달라질 수 있다. 화면이 그것을 적는다
  ADM->>API: POST /api/admin/batch/publish/{reportId}
  API->>BAT: publish_version(reportId)
  BAT->>DB_PUB: 직전 게시본을 미게시로 내리고 이 버전을 게시본으로. 지우지 않는다
  BAT->>RP: diff_watchlist(current, previous)
  RP->>DB_PUB: report_alert_event 다시 계산
  BAT-->>ADM: alertsRecomputed 건수
  opt 재생성 중 정기 배치 시각이 왔다
    BAT->>BAT: 정기 배치를 대기시킨다. worker는 한 번에 하나만 돈다
  end
```

**읽을 때 볼 것.** 지정 단계부터 다시 돌리는 것과 특정 일자를 재생성하는 것이 같은 화살표다. 시작 단계가 1이면 재생성이고 6이면 서술만 다시 도는 것이다. 실행과 게시 전환이 두 호출로 나뉜 것이 핵심이다. 자동 전환이면 재생성이 게시본을 덮는다. 가운데 자기 호출 둘이 재현성의 검사 지점이다. 판정 넷과 도메인 상태는 같아야 하고 설명과 문장은 달라도 된다([[TBL-PRD-001#N3]]).

스위치 둘의 쓰임이 갈린다. `backfillMode`는 관리 API가 요청 본문으로 받아 배치 실행에 남는 값이고 6~8단계를 건너뛰게 한다. `llmEnabled`는 요청 본문에 없는 내부 스위치이며, LLM을 끈 실행과 켠 실행의 신호등·후보 순서·도메인 상태가 같은지 대조하는 회귀 시험에만 쓴다([[TBL-INFRA-001#C19]]). 둘을 하나로 합치지 않는다. `diff_watchlist`가 [[#SEQ-24]]와 같은 함수인데 여기서 다시 돌아 같은 기준일의 알림이 두 번 계산될 수 있다. 되먹일 것 F5다.

#### SEQ-23 마스터 보강

근거는 [[TBL-UC-001#UC-A2]]다. 과거 판정은 바뀌지 않고 다음 배치부터 적용된다.

```mermaid
sequenceDiagram
  autonumber
  actor ADM as 관리자
  participant API as 관리 API
  participant MST as MasterService
  participant DB_OPS as ops 스키마
  participant DB_MASTER as master 스키마
  ADM->>API: GET /api/admin/master/summary
  API->>MST: get_summary()
  MST->>DB_MASTER: crosswalk_version 이력
  MST->>DB_OPS: 매칭률 셋. 판매에서 국가, 생산에서 국가, 뉴스에서 국가
  MST-->>ADM: 매칭률과 버전 이력
  ADM->>API: GET /api/admin/master/unmapped
  API->>MST: list_unmapped(filters)
  MST->>DB_OPS: unmapped_value. source, kind, value, occurrence_count, first_seen_date
  MST-->>ADM: 미매핑 목록. 내려받아 크로스워크 엑셀을 보강한다
  ADM->>API: POST /api/admin/master/crosswalk (엑셀)
  API->>MST: upload_crosswalk(upload)
  MST->>DB_MASTER: crosswalk_upload 저장
  MST->>MST: diff_against_current (upload_crosswalk 안). 추가, 변경, 삭제
  MST-->>ADM: 차이와 영향 받는 국가와 공장, 삭제로 미매핑이 될 건수
  ADM->>API: POST /api/admin/master/confirm/{crosswalkUploadId} (confirmRemoval)
  API->>MST: confirm_crosswalk(uploadId, confirmRemoval)
  alt 삭제되는 매핑이 있는데 확인이 없다
    MST-->>ADM: 409 problems/removal-not-confirmed
  else 확정
    MST->>DB_MASTER: crosswalk_version 새 버전, crosswalk_entry 교체
    MST-->>ADM: 새 버전 번호와 반영 시점
  end
  Note over MST,DB_MASTER: 다음 배치부터 적용된다. 과거 판정을 바꾸려면 재생성이다
  Note over MST,DB_MASTER: 글로비스 법인 매핑은 아직 비어 있다. 빈 동안은 법인 미매핑으로 표시하고 오류로 보지 않는다
```

**읽을 때 볼 것.** 확정 전에 `diff_against_current`가 반드시 한 번 돈다. 크로스워크는 통째 교체라서 한 줄 빠진 파일을 올리면 그 국가의 과거 매핑이 조용히 사라진다. 409 갈래가 그것을 막는다. `mart`나 `pub`으로 가는 화살표가 없는 것도 본다. 마스터를 고쳐도 이미 게시된 리포트는 그대로다. 반영하려면 [[#SEQ-22]]의 재생성을 거쳐야 하고, 어디까지 다시 돌릴지는 아직 미결이다.

#### SEQ-24 워치리스트 변화 이벤트

근거는 [[TBL-UC-001#UC-S5]]다. 8단계 게시 직후에 돌고 채널 어댑터 없이도 실패하지 않는다.

```mermaid
sequenceDiagram
  autonumber
  participant TLJ as TrafficLightJudge
  participant RP as ReportPublisher
  participant DB_MART as mart 스키마
  participant DB_PUB as pub 스키마
  participant API as 열람 API
  actor H as 현업 사용자
  TLJ->>DB_MART: watch_item. 5단계 산출물이고 국가별 신호등과 sort_order를 갖는다
  RP->>DB_MART: 오늘 watch_item
  RP->>DB_PUB: 직전 게시본의 report_watch_item
  RP->>RP: diff_watchlist(current, previous)
  alt 어제 없던 국가가 올라왔다
    RP->>DB_PUB: report_alert_event 신규
  else 신호등이 올라갔다
    RP->>DB_PUB: report_alert_event 상승
  else 내려갔다
    RP->>DB_PUB: report_alert_event 하강
  else 목록에서 빠졌다
    RP->>DB_PUB: report_alert_event 이탈
  end
  RP->>DB_PUB: report_watch_item 복사. traffic_light, basis, sort_order
  opt 채널 어댑터가 등록돼 있다
    RP->>RP: 발송. 1차에는 어댑터가 없다
  end
  H->>API: GET /api/intel/creport/latest
  API->>DB_PUB: report_alert_event를 함께 읽는다
  DB_PUB-->>API: 신규와 상승 목록
  API-->>H: 카드의 신규, 상승 배지
  Note over TLJ,RP: 워치리스트 줄은 5단계가 만든다. 게시기는 비교만 한다. LLM을 꺼도 같은 줄이 나온다
```

**읽을 때 볼 것.** 첫 화살표가 [[TBL-DOM-002#TrafficLightJudge]]에서 나온다. 워치리스트를 만드는 것과 어제와 비교하는 것을 서로 다른 단계에 둔 것이 이 그림의 계약이다. 만들기가 5단계에 있어야 LLM 상태와 무관하고, 비교는 게시 시점에만 뜻이 있어 8단계에 있다. 채널 발송이 `opt`인 것은 1차에 어댑터가 없기 때문이며, 없다고 이 경로가 실패하지 않는다. 마지막 세 화살표는 [[#SEQ-11]]의 한 조각이다. 배지는 별도 호출이 아니라 리포트 응답에 실려 온다.

## 6. 대응표

| 시퀀스 | 배치 단계 | 유스케이스 | API 또는 화면 | 주 클래스 |
|:--|:--|:--|:--|:--|
| [[#SEQ-11]] | 없음 | [[TBL-UC-001#UC-H1]] [[TBL-UC-001#UC-H2]] | [[TBL-API-001#GET/api/intel/creport/latest]] [[TBL-API-001#GET/api/intel/creport/version/{reportId}]] [[TBL-API-001#GET/api/intel/market/series/{indicatorId}]] | [[TBL-DOM-002#CReportService]] [[TBL-DOM-002#MarketService]] |
| [[#SEQ-12]] | 없음 | [[TBL-UC-001#UC-H4]] | [[TBL-API-001#GET/api/intel/areport/domain/{domain}]] [[TBL-API-001#GET/api/intel/areport/version/{domainReportId}]] | [[TBL-DOM-002#DomainReportService]] |
| [[#SEQ-13]] | 1~8 전부 | [[TBL-UC-001#UC-S1]] | [[TBL-API-001#GET/api/admin/batch/status]] | [[TBL-DOM-002#PipelineRunner]] |
| [[#SEQ-14]] | 1 | [[TBL-UC-001#UC-A1]] [[TBL-UC-001#UC-S6]] | [[TBL-API-001#POST/api/admin/ingest/preflight]] [[TBL-API-001#POST/api/admin/ingest/commit]] [[TBL-API-001#POST/api/admin/ingest/unblock/{ingestFileId}]] | [[TBL-DOM-002#IngestService]] [[TBL-DOM-002#FormDetector]] [[TBL-DOM-002#FormALedgerParser]] [[TBL-DOM-002#FormBPivotParser]] [[TBL-DOM-002#RecordStandardizer]] [[TBL-DOM-002#RegressionChecker]] |
| [[#SEQ-15]] | 2 | [[TBL-UC-001#UC-S10]] | 없음 | [[TBL-DOM-002#AJudgmentReader]] [[TBL-DOM-002#BriefingStoreReader]] [[TBL-DOM-002#ReportPublisher]] |
| [[#SEQ-16]] | 3 | [[TBL-UC-001#UC-S2]] | 없음 | [[TBL-DOM-002#EventClusterer]] |
| [[#SEQ-17]] | 4 | [[TBL-UC-001#UC-S3]] | 없음 | [[TBL-DOM-002#FactJoiner]] [[TBL-DOM-002#StageFlowCalculator]] [[TBL-DOM-002#AnomalyDetector]] |
| [[#SEQ-18]] | 5 | [[TBL-UC-001#UC-S8]] | 없음 | [[TBL-DOM-002#CandidateSearcher]] [[TBL-DOM-002#ProximityCalculator]] [[TBL-DOM-002#TrafficLightJudge]] |
| [[#SEQ-19]] | 6 | [[TBL-UC-001#UC-S11]] [[TBL-UC-001#UC-S7]] | 없음 | [[TBL-DOM-002#EventNamer]] [[TBL-DOM-002#HChatClient]] [[TBL-DOM-002#DegradeHandler]] |
| [[#SEQ-20]] | 7 | [[TBL-UC-001#UC-S9]] [[TBL-UC-001#UC-S7]] | 없음 | [[TBL-DOM-002#CauseLinkWriter]] [[TBL-DOM-002#CitationVerifier]] [[TBL-DOM-002#DegradeHandler]] |
| [[#SEQ-21]] | 8 | [[TBL-UC-001#UC-S4]] [[TBL-UC-001#UC-S7]] | 없음 | [[TBL-DOM-002#ClaimWriter]] [[TBL-DOM-002#CitationVerifier]] [[TBL-DOM-002#ReportPublisher]] |
| [[#SEQ-22]] | 1~8 재실행 | [[TBL-UC-001#UC-A4]] | [[TBL-API-001#POST/api/admin/batch/rerun]] [[TBL-API-001#POST/api/admin/batch/publish/{reportId}]] [[TBL-API-001#GET/api/admin/batch/history]] [[TBL-API-001#GET/api/admin/batch/run/{batchRunId}]] | [[TBL-DOM-002#BatchService]] [[TBL-DOM-002#PipelineRunner]] |
| [[#SEQ-23]] | 없음 | [[TBL-UC-001#UC-A2]] | [[TBL-API-001#GET/api/admin/master/summary]] [[TBL-API-001#GET/api/admin/master/unmapped]] [[TBL-API-001#POST/api/admin/master/crosswalk]] [[TBL-API-001#POST/api/admin/master/confirm/{crosswalkUploadId}]] | [[TBL-DOM-002#MasterService]] |
| [[#SEQ-24]] | 8 게시 직후 | [[TBL-UC-001#UC-S5]] | 없음 | [[TBL-DOM-002#TrafficLightJudge]] [[TBL-DOM-002#ReportPublisher]] |

시퀀스 열넷에서 [[TBL-DOM-002#HChatClient]] 생명선이 서는 것은 [[#SEQ-19]] [[#SEQ-20]] [[#SEQ-21]] 셋뿐이고, 그 셋이 모두 [[TBL-DOM-002#DegradeHandler]]로 가는 화살표를 갖는다. 판정 다섯([[#SEQ-14]]~[[#SEQ-18]])에는 그 생명선이 없다. 이 대조가 [[TBL-INFRA-001#C19]]가 시퀀스에서 보이는 모습이다.

[[TBL-API-001#GET/api/admin/ingest/history]]와 [[TBL-API-001#GET/api/admin/ingest/file/{ingestFileId}]]는 [[#SEQ-14]]의 결과를 되읽는 단순 조회라 따로 그리지 않았다. `IngestService.list_history`와 `get_file` 호출 한 번으로 끝난다.

## 7. 되먹일 것

| # | 어느 문서 | 무엇이 어긋났나 | 이 문서가 임시로 택한 것 |
|:--|:--|:--|:--|
| F2 | [[TBL-UC-001#UC-S4]] | 4단계의 저장 대상이 `briefing`, `briefing_claim`, `evidence`로 적혀 있다. ERD 확정 이름은 [[TBL-DOM-003#c_report]] [[TBL-DOM-003#report_claim]] [[TBL-DOM-003#report_evidence]]다 | [[#SEQ-21]]은 ERD 이름을 썼다 |
| F3 | [[TBL-UC-001#UC-S1]] · [[TBL-DOM-002#IngestService]] | 회귀 급변의 처리가 갈린다. UC는 1a에서 "배치를 멈추고 A에게 알린다", 클래스 명세는 "적재는 완료하고 배치 자동 실행만 막는다"다. 수동 적재와 배치 1단계 중 어느 쪽 이야기인지 문장으로 갈리지 않는다 | 수동 적재는 완료하고 차단만, 배치 안에서는 멈춤으로 그렸다([[#SEQ-13]] [[#SEQ-14]]) |
| F5 | [[TBL-DOM-002#ReportPublisher]] · [[TBL-API-001#POST/api/admin/batch/publish/{reportId}]] | `diff_watchlist`가 8단계 게시와 게시 전환 양쪽에서 돈다. [[TBL-DOM-003#report_alert_event]]에 같은 기준일 재계산을 막을 멱등 키가 없어 알림이 두 벌 남을 수 있다 | [[#SEQ-22]]와 [[#SEQ-24]]에 둘 다 그리고 중복 가능성을 적었다 |
| F6 | [[TBL-DOM-002#MarketService]] | `as_of`와 `is_carried_over`는 [[TBL-DOM-002#FactJoiner]]가 쓰는 파이프라인 함수인데 API 바인딩 목록에는 `get_series`만 있다. [[TBL-MS-001]]이 파이프라인 계약으로 실어야 한다 | [[#SEQ-17]]에 `as_of` 호출을 그렸다 |
| F8 | [[TBL-DOM-002]] | 생명선으로 세울 수 없는 타입 넷이 있다. `SchemaRegistry`, `CrosswalkTable`, `BriefingStorePort`, `LlmPort`다. 앞의 둘은 클래스 스물여덟 밖의 실체이고 뒤의 둘은 포트다 | 포트 둘은 구현체([[TBL-DOM-002#BriefingStoreReader]] [[TBL-DOM-002#HChatClient]])로 그렸고 앞의 둘은 그리지 않았다 |

F1·F4·F7은 닫혔다. F1은 게시 스키마에 A 리포트 사본([[TBL-DOM-003#a_report_snapshot]] [[TBL-DOM-003#a_report_evidence]])과 지표 시계열 사본([[TBL-DOM-003#market_series]])을 두는 쪽으로 정해져 열람 경로가 [[TBL-INFRA-001#C10]]과 어긋나지 않는다. F4는 `backfillMode`와 `llmEnabled`를 둘 다 남기고 쓰임을 가르는 쪽으로, F7은 1단계와 3단계 실패를 멈춤으로 정해졌다([[TBL-INFRA-001#C20]]). 닫힌 번호는 다시 쓰지 않는다.

## 8. 미결사항

- [ ] [[#SEQ-15]]의 `read_judgments`와 `read_report_document` 반환 모양. 브리핑 갈래가 무엇을 어떤 키로 내주는지 정해져야 `validate_shape`의 대조 목록과 loop 안 화살표, 그리고 사본 컬럼과의 1대1 대응이 확정된다
- [ ] [[#SEQ-14]]에서 형태 A 파일을 아직 본 적이 없다. 실물이 들어오면 `check_columns` 화살표의 대조 목록이 바뀔 수 있다
- [ ] [[#SEQ-18]]의 임계값 실제 수치. 최소 기사 수, 최소 출처 수, 시간창, CBU 비중 기준이 전부 현업 검토 대기이며 이 값이 정해져야 신호등 alt 세 갈래의 비율을 가늠할 수 있다
- [ ] [[#SEQ-19]]부터 [[#SEQ-21]]까지의 H-chat JSON 모드. 게이트웨이가 응답 스키마 지정을 지원한다는 것은 가정이다. 지원되지 않으면 세 시퀀스의 검증 화살표가 전부 늘어난다
- [ ] [[#SEQ-22]]의 설정 변경 경로. 화면과 API가 없어 [[TBL-DOM-003#threshold_setting]] 새 버전 행을 SQL로 넣을지 CLI를 만들지가 정해지지 않았다. 정해지면 시퀀스가 하나 는다
- [ ] [[#SEQ-23]] 확정 후 재계산 범위. 과거 기준일을 어디까지 다시 돌릴지가 정해지면 [[#SEQ-22]]와 잇는 화살표가 생긴다
- [ ] [[#SEQ-24]]의 채널 발송. 1차에 어댑터가 없고 무엇을 붙일지도 미결이다
