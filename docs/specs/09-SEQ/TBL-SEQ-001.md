---
doc_id: TBL-SEQ-001
type: SEQ
title: Glovis 완성차 인텔리전스 시퀀스
status: draft
upstream: [TBL-UC-001, TBL-API-001, TBL-DOM-001, TBL-INFRA-001]
---

# SEQUENCE

## 0. 이 문서가 다루는 것

유스케이스별 객체 간 호출 순서다. 열람 경로가 LLM과 무거운 집계를 거치지 않는 것, 배치의 LLM 지점과 강등 분기, 두 시간축(기준일·배치 ID)이 어디서 찍히는지를 보인다.

이 버전은 끊어진 참조만 해결한 것이다(SEQ-2 판매 상세 제거, 설정 생명선 제거). 새 방향(A 판정 읽기, 후보 검색, 연관 판단을 포함한 8단계와 LLM 세 지점)의 전면 반영은 다음 버전에서 한다. 삭제된 SEQ-2의 ID는 재사용하지 않는다.

### 0.1 생명선

| 생명선 | 약어 | 실체 | 종류 | 정의한 곳 |
|---|---|---|---|---|
| 현업 브라우저 | FE | UI-1 React 앱 | 화면 | [[TBL-UI-001#UI-1]] |
| 관리 브라우저 | ADM | UI-3~5 | 화면 | [[TBL-UI-001#UI-3]] |
| 열람 API | PUB | web/public 라우터 | 라우터 | [[TBL-API-001]] |
| 관리 API | API | web/admin 라우터 | 라우터 | [[TBL-API-001]] |
| 브리핑 조회 | BQ | BriefingQuery | 서비스 | [[TBL-MS-001#BriefingQuery.latest]] |
| 적재 | ING | IngestService | 서비스 | [[TBL-MS-001#IngestService.run]] |
| 스케줄러 | SCH | APScheduler 프로세스 | 워커 | [[TBL-INFRA-001#C8]] |
| 파이프라인 | PIPE | Pipeline(단계 오케스트레이터) | 서비스 | [[TBL-MS-001#Pipeline.run]] |
| 사건 통합 | EVT | EventBuilder | 서비스 | [[TBL-MS-001#EventBuilder.build_events]] |
| 결합·판정 | JDG | SignalBuilder + Judge | 서비스 | [[TBL-MS-001#Judge.judge]] |
| 서술 | NAR | Narrator + Validator | 서비스 | [[TBL-MS-001#Narrator.narrate]] |
| LLM 어댑터 | LLM | HchatClient (OpenAI 호환) | 어댑터 | [[TBL-INFRA-001#C3]] |
| DB | DB | postgres raw/std/master/mart/pub/ops | 저장소 | [[TBL-DOM-001]] |
| 마스터 | MST | MasterService | 서비스 | [[TBL-MS-001#MasterService.stage]] |

## 1. 대응표

| 시퀀스 | 유스케이스 | API | 화면 |
|:--|:--|:--|:--|
| [[#SEQ-1]] | [[TBL-UC-001#UC-H1]] | [[TBL-API-001#GET/api/intel/briefing/latest]] | UI-1 |
| [[#SEQ-3]] | [[TBL-UC-001#UC-S1]] | 없음(스케줄) | UI-4 |
| [[#SEQ-4]] | [[TBL-UC-001#UC-S2]] | 없음 | |
| [[#SEQ-5]] | [[TBL-UC-001#UC-S3]] | 없음 | |
| [[#SEQ-6]] | [[TBL-UC-001#UC-S4]] [[TBL-UC-001#UC-S7]] | 없음 | |
| [[#SEQ-7]] | [[TBL-UC-001#UC-A1]] [[TBL-UC-001#UC-S6]] | [[TBL-API-001#POST/api/admin/intel/ingest/preflight]] [[TBL-API-001#POST/api/admin/intel/ingest]] | UI-3 |
| [[#SEQ-8]] | [[TBL-UC-001#UC-A4]] | [[TBL-API-001#POST/api/admin/intel/batch/run]] [[TBL-API-001#POST/api/admin/intel/batch/runs/{runId}/publish]] | UI-4 |
| [[#SEQ-9]] | [[TBL-UC-001#UC-A2]] | [[TBL-API-001#POST/api/admin/intel/master/upload]] [[TBL-API-001#POST/api/admin/intel/master/confirm/{stagingId}]] | UI-5 |
| [[#SEQ-10]] | [[TBL-UC-001#UC-S5]] | 없음 | UI-1 배지 |

#### SEQ-1 브리핑 열람

근거: [[TBL-UC-001#UC-H1]]. 열람은 pub 스키마 조회 1회다.

```mermaid
sequenceDiagram
  participant FE
  participant PUB
  participant BQ
  participant DB
  FE->>PUB: GET /api/intel/briefing/latest?token
  PUB->>PUB: 토큰 서명·만료 검증 (실패 401)
  PUB->>BQ: latest()
  BQ->>DB: briefing where published order by as_of_date desc limit 1
  alt 없음
    BQ-->>PUB: None
    PUB-->>FE: 503 briefing-unavailable
  else 있음
    BQ->>DB: watchlist_item, briefing_claim, evidence by briefing_id
    BQ->>BQ: BriefingView 조립 (indicator_bar·domain_status는 briefing jsonb)
    BQ-->>PUB: BriefingView
    PUB-->>FE: 200
  end
  FE->>FE: 강조 토큰·각주 렌더, 접힘 상태 복원
  opt 지표 바 팝오버
    FE->>PUB: GET /api/intel/indicators/{identifier}/series?days=30
  end
```

**읽을 때 볼 것**: LLM, mart, std를 전혀 읽지 않는다. evidence.display에 표시값이 복사돼 있어 조인이 없다. degraded면 FE가 안내 문구를 붙인다.

#### SEQ-3 새벽 배치 전체

근거: [[TBL-UC-001#UC-S1]]. 단계 오케스트레이션과 실패 분기.

```mermaid
sequenceDiagram
  participant SCH
  participant PIPE
  participant ING
  participant EVT
  participant JDG
  participant NAR
  participant DB
  SCH->>PIPE: run(as_of=today, start_step=1, trigger=schedule)
  PIPE->>DB: batch_run insert (running)
  PIPE->>DB: batch blocked? (회귀 차단)
  alt blocked
    PIPE-->>SCH: skip (423 기록)
  end
  PIPE->>ING: step1 대기 파일 적재·표준화 (없으면 skip)
  ING-->>PIPE: ingest_run[]
  PIPE->>EVT: step3 build_events(as_of)
  EVT-->>PIPE: event 수, llm 실패 수
  PIPE->>JDG: step4 build_signals(as_of)
  PIPE->>JDG: step5 judge(as_of, threshold current)
  alt step4/5 실패
    PIPE->>DB: batch_run failed(step, error)
    PIPE-->>SCH: 이전 게시본 유지
  end
  PIPE->>NAR: step6 narrate(as_of, batch_run_id)
  NAR-->>PIPE: briefing(degraded?)
  PIPE->>NAR: step7 validate_publish
  PIPE->>DB: briefing published=true, alert_event
  PIPE->>DB: batch_run success/degraded, step_log, llm_tokens
```

**읽을 때 볼 것**: step4·5 실패만 배치를 멈춘다. step3·6 실패는 강등으로 계속된다. batch_run_id가 mart·pub 모든 행에 찍힌다(두 번째 시간축). 다음 버전에서 A 판정 읽기(2단계)·후보 검색(5단계)·연관 판단(7단계)을 넣어 UC-S1의 8단계와 맞춘다.

#### SEQ-4 사건 통합 (LLM 지점 1)

근거: [[TBL-UC-001#UC-S2]].

```mermaid
sequenceDiagram
  participant EVT
  participant DB
  participant LLM
  EVT->>DB: article_country where published_date in (as_of-window, as_of] and not yet grouped
  EVT->>DB: event where state=open
  loop 국가×카테고리 묶음
    EVT->>EVT: 열린 사건에 붙이기 or 새 사건
    EVT->>DB: strait_country keywords 매칭 → 해협 사건 귀속
  end
  loop 신규·변경 사건 (심각도 상위, 토큰 상한 내)
    EVT->>DB: 상위 5 기사 제목·요약
    EVT->>LLM: name_event(JSON 모드)
    alt ok
      LLM-->>EVT: {title, type, subtype, severity}
    else filtered / invalid_json / error
      LLM-->>EVT: outcome
      EVT->>EVT: fallback: 대표 제목, 카테고리, max impact
    end
    EVT->>DB: llm_call insert
  end
  EVT->>DB: event upsert, event_article, 종료 판정(last_seen + window)
```

**읽을 때 볼 것**: 백필 모드면 LLM 루프를 건너뛴다. 토큰 상한에 닿으면 남은 사건은 fallback으로 이름 붙인다. 다음 버전에서 명명 루프를 후보 사건(UC-S11)으로 옮긴다.

#### SEQ-5 결합과 판정

근거: [[TBL-UC-001#UC-S3]].

```mermaid
sequenceDiagram
  participant JDG
  participant DB
  JDG->>DB: sales_daily, production_daily (as_of 월 + 비교 기간)
  JDG->>DB: model_cbu_ckd 갱신 (production_daily 최빈값)
  JDG->>JDG: country_daily_fact 집계 (플로우 합, 스톡 말값, cbu_ratio)
  JDG->>DB: country_daily_fact upsert (batch_run_id)
  JDG->>DB: event open by iso3, market_series as-of (carry_days), oem_monthly 최신월
  JDG->>JDG: signal_daily 조립 179국 전부, cmp_method 결정
  JDG->>DB: signal_daily upsert
  JDG->>DB: threshold_config current
  JDG->>JDG: sales_anomaly, inventory_stay, exposure, cross_confirmed, signal
  JDG->>DB: glovis_entity by iso3 → entity_tags
  JDG->>DB: judgment insert (threshold_version, batch_run_id)
```

**읽을 때 볼 것**: 판정에 LLM이 없다. 임계값은 판정 시점 버전을 함께 저장한다. 데이터 품질 플래그는 country_daily_fact에서 judgment.domain_state로 흘러간다.

#### SEQ-6 서술·검증·강등 (LLM 지점 2)

근거: [[TBL-UC-001#UC-S4]] [[TBL-UC-001#UC-S7]].

```mermaid
sequenceDiagram
  participant NAR
  participant DB
  participant LLM
  NAR->>DB: judgment, signal_daily, event, event_article top3, market as-of
  NAR->>NAR: 항목별 판정 구조체 (≤2KB) + 근거 목록(seq 부여)
  loop headline, domain×3, watch×N
    NAR->>LLM: narrate(structure, evidence list, JSON 모드)
    alt ok
      LLM-->>NAR: {text with [[hl]] and [^n]}
      NAR->>NAR: validate: 숫자·날짜·국가명 ⊂ 입력, 각주 ⊂ 근거, 금지어 없음, 강조 ≤3
      alt validate 실패
        NAR->>LLM: 재생성 1회
        NAR->>NAR: 재실패면 템플릿 강등
      end
    else filtered / invalid_json / error
      NAR->>NAR: 템플릿 강등
    end
    NAR->>DB: llm_call insert
  end
  NAR->>DB: briefing (degraded = 강등 1건 이상), briefing_claim, evidence(display 복사)
```

**읽을 때 볼 것**: 근거 seq는 LLM 호출 전에 코드가 부여한다. LLM은 번호를 고를 뿐 만들지 않는다. evidence.display는 이 시점에 복사돼 열람 경로가 조인을 안 한다.

#### SEQ-7 적재와 회귀 검사

근거: [[TBL-UC-001#UC-A1]] [[TBL-UC-001#UC-S6]].

```mermaid
sequenceDiagram
  participant ADM
  participant API
  participant ING
  participant DB
  ADM->>API: POST ingest/preflight (source, file)
  API->>ING: preflight
  ING->>ING: 원본 임시 저장, schema_version 판별, 컬럼 대조
  ING->>ING: 기간·행수·결측·골격·중복 의심 계산
  ING-->>API: Preflight(stagingId)
  API-->>ADM: 200 (또는 422 schema-mismatch)
  ADM->>API: POST ingest (stagingId, overwrite, backfill)
  API->>ING: run
  ING->>DB: 원본 파일 볼륨 이동, ingest_run insert
  ING->>DB: L0 insert
  ING->>DB: L1 upsert (마스터 조인, 국가 유도, 카테고리 정규화)
  ING->>DB: mapping_gap upsert
  ING->>ING: 회귀 검사 (직전 ingest_run 대비 허용 폭)
  alt 급변
    ING->>DB: ingest_run regression_flag, 배치 차단 플래그
  end
  ING-->>API: IngestRun
  API-->>ADM: 201
```

**읽을 때 볼 것**: preflight와 run이 분리돼 있어 사람이 확인하고 실행한다. 회귀 급변은 적재를 막지 않고 자동 배치만 막는다.

#### SEQ-8 재실행·재생성·게시 전환

근거: [[TBL-UC-001#UC-A4]].

```mermaid
sequenceDiagram
  participant ADM
  participant API
  participant PIPE
  participant DB
  ADM->>API: POST batch/run (asOfDate, startStep)
  API->>PIPE: trigger (running이면 409)
  API-->>ADM: 202 BatchRun
  PIPE->>DB: 당시 threshold_version 조회 (judgment of as_of 최신 게시본)
  PIPE->>PIPE: 지정 단계부터 SEQ-3 흐름 (published=false로 briefing 생성)
  PIPE->>DB: judgment diff (당시 vs 새) + cause 태깅
  ADM->>API: GET batch/runs/{id}
  API-->>ADM: BatchRunDetail(diff)
  ADM->>API: POST batch/runs/{id}/publish
  API->>DB: 기존 published=false, 새 briefing published=true
  API-->>ADM: 200
```

**읽을 때 볼 것**: 재생성은 새 batch_run·새 briefing version이다. 게시 전환은 별도 확인 호출이다. cause 태깅은 마스터 버전·threshold_version·ingest_run 시각 비교로 기계 판정한다.

#### SEQ-9 마스터 보강

근거: [[TBL-UC-001#UC-A2]].

```mermaid
sequenceDiagram
  participant ADM
  participant API
  participant MST
  participant DB
  ADM->>API: GET master/gaps?format=csv
  API->>MST: gaps
  MST->>DB: mapping_gap
  API-->>ADM: csv
  ADM->>API: POST master/upload (엑셀)
  API->>MST: stage
  MST->>MST: 시트별 현재 버전과 diff, 영향 키 계산
  MST-->>API: MasterDiff(stagingId)
  ADM->>API: POST master/confirm/{stagingId}
  API->>MST: confirm
  MST->>DB: 새 version 행 insert, mapping_gap.resolved_version 갱신
  API-->>ADM: 201 MasterVersion
```

**읽을 때 볼 것**: 마스터는 갱신이 아니라 버전 추가다. 과거 판정은 threshold_version처럼 마스터 버전을 참조하지 않으므로(미결: judgment에 master 버전도 저장할지) 재계산은 SEQ-8로만 한다.

#### SEQ-10 워치리스트 변화 이벤트

근거: [[TBL-UC-001#UC-S5]].

```mermaid
sequenceDiagram
  participant PIPE
  participant DB
  PIPE->>DB: 직전 published briefing의 watchlist_item
  PIPE->>PIPE: 국가별 signal 비교 → new/up/down/exit
  PIPE->>DB: alert_event insert (delivered=false)
  PIPE->>DB: watchlist_item.change_badge 갱신
  opt 채널 어댑터 등록
    PIPE->>PIPE: 발송, delivered=true
  end
```

**읽을 때 볼 것**: 채널이 없어도 이벤트와 배지는 남는다.

## 2. 되먹일 것

- DOM: judgment에 사용한 마스터 버전(country, factory, glovis_entity)도 저장할지. SEQ-8의 cause 태깅이 정확해지려면 필요하다. → [[TBL-DOM-001#judgment]] 컬럼 추가 후보.
- DOM: 배치 차단 플래그의 저장 위치. threshold_config가 아니라 ops.batch_state 단일 행 테이블이 낫다. → 테이블 추가 후보. MS는 이미 batch_state를 전제로 썼다([[TBL-MS-001#IngestService.regression_check]]).
- API: preflight의 stagingId 만료 시간(가정 1시간)을 스키마에 명시.
- UI: SEQ-1에서 503일 때 UI-1의 빈 화면 문구는 UI-1 요소 5에 이미 있음. 확인 완료.
- MS: ConfigService.save는 설정 화면·API가 없어졌으므로 다음 버전에서 제거 후보. 설정은 배치가 현재 버전을 읽기만 한다.

## 3. 미결사항

- [ ] judgment에 마스터 버전 3종 저장 여부 (되먹일 것 1)
- [ ] 배치 차단 상태 저장 테이블 (되먹일 것 2)
- [ ] SEQ-4 토큰 상한 도달 시 "심각도 상위"의 기준(impact 합 vs 기사 수). 가정: max impact 내림차순
- [ ] SEQ-6 재생성 1회의 프롬프트 변형 방식(온도 낮춤 vs 검증 실패 사유 첨부). 가정: 사유 첨부
- [ ] SEQ-8 정기 배치와 재실행 동시 발생 시 대기 큐 구현(단일 프로세스 락)
