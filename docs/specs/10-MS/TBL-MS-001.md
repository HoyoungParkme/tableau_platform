---
doc_id: TBL-MS-001
type: MS
title: Glovis 완성차 인텔리전스 미니스펙
status: draft
upstream: [TBL-SEQ-001, TBL-API-001, TBL-DOM-001]
---

# MINISPEC

## 0. 이 문서가 다루는 것

시퀀스에 등장한 서비스 함수의 입력·처리·출력·예외를 의사코드 수준으로 적는다. 항목 ID는 `클래스.함수`다. 테이블은 [[TBL-DOM-001]]의 항목명을 그대로 쓴다. 분기는 `if 조건 → 결과 · else → 결과`로 쓴다. 내부 DTO는 [[TBL-API-001]] 4절 스키마와 아래 2.0의 판정 구조체 하나만 둔다.

## 1. 함수 목록

| 패키지 | 함수 | 단계 | 근거 |
|:--|:--|:--|:--|
| ingest | [[#IngestService.preflight]] | 1 | [[TBL-SEQ-001#SEQ-7]] |
| ingest | [[#IngestService.run]] | 1~2 | [[TBL-SEQ-001#SEQ-7]] |
| ingest | [[#IngestService.standardize]] | 2 | [[TBL-SEQ-001#SEQ-7]] |
| ingest | [[#IngestService.regression_check]] | 2 | [[TBL-UC-001#UC-S6]] |
| batch | [[#Pipeline.run]] | 1~7 | [[TBL-SEQ-001#SEQ-3]] |
| event | [[#EventBuilder.build_events]] | 3 | [[TBL-SEQ-001#SEQ-4]] |
| event | [[#EventBuilder.name_event]] | 3 | [[TBL-SEQ-001#SEQ-4]] |
| signal | [[#SignalBuilder.build_facts]] | 4 | [[TBL-SEQ-001#SEQ-5]] |
| signal | [[#SignalBuilder.build_signals]] | 4 | [[TBL-SEQ-001#SEQ-5]] |
| judgment | [[#Judge.judge]] | 5 | [[TBL-SEQ-001#SEQ-5]] |
| judgment | [[#Judge.select_watchlist]] | 5 | [[TBL-SEQ-001#SEQ-5]] |
| briefing | [[#Narrator.narrate]] | 6 | [[TBL-SEQ-001#SEQ-6]] |
| briefing | [[#Narrator.validate]] | 6 | [[TBL-SEQ-001#SEQ-6]] |
| briefing | [[#Narrator.degrade]] | 6 | [[TBL-UC-001#UC-S7]] |
| briefing | [[#Publisher.publish]] | 7 | [[TBL-SEQ-001#SEQ-3]] |
| briefing | [[#Publisher.diff_watchlist]] | 7 | [[TBL-SEQ-001#SEQ-10]] |
| public | [[#BriefingQuery.latest]] | 열람 | [[TBL-SEQ-001#SEQ-1]] |
| public | [[#BriefingQuery.detail]] | 열람 | [[TBL-SEQ-001#SEQ-2]] |
| admin | [[#BatchService.trigger]] | 관리 | [[TBL-SEQ-001#SEQ-8]] |
| admin | [[#BatchService.publish_version]] | 관리 | [[TBL-SEQ-001#SEQ-8]] |
| admin | [[#MasterService.stage]] | 관리 | [[TBL-SEQ-001#SEQ-9]] |
| admin | [[#MasterService.confirm]] | 관리 | [[TBL-SEQ-001#SEQ-9]] |
| admin | [[#ConfigService.save]] | 관리 | [[TBL-UC-001#UC-A3]] |
| infra | [[#HchatClient.complete_json]] | LLM | [[TBL-INFRA-001#C3]] |

## 2. 함수

### 2.0 판정 구조체 (DTO)

LLM 지점 2의 유일한 입력. 2KB 이하.

```json
{
  "iso3": "OMN", "country_name": "오만", "as_of": "2026-09-09",
  "signal": "RED", "exposure": "CBU", "cbu_ratio": 1.0,
  "structured": { "metric": "wholesale", "value": 0, "cmp": 418, "change_pct": -1.0, "method": "YoY" },
  "inventory": { "voyage": 418, "waiting": 0, "ratio": 0.42 },
  "event": { "id": 1207, "title": "호르무즈 해협 봉쇄 위협", "category": "MRT", "severity": 94, "article_count": 8, "state": "open" },
  "entity_tags": [],
  "evidence": [
    { "seq": 1, "kind": "signal", "display": { "label": "판매 YoY", "value": "-100%", "as_of": "2026-09-09" } },
    { "seq": 2, "kind": "article", "display": { "title": "...", "source": "Khaleej Times", "date": "2026-08-12" } },
    { "seq": 3, "kind": "market", "display": { "label": "BDI", "value": 2776, "change_pct": -0.101, "as_of": "2026-08-20" } }
  ],
  "constraints": ["상관관계까지만", "예측 어휘 금지", "각주는 evidence.seq만", "강조 3개 이하"]
}
```

### 2.1 적재

#### IngestService.preflight 적재 사전 검증

**시그니처** `preflight(source: Source, file: UploadFile) -> Preflight`

**입력** source ∈ {prod, sales, news, bbg, mkl, crosswalk}, 파일(xlsx·csv).

**처리**
1. 파일을 스테이징 경로에 저장, stagingId 발급(만료 1시간).
2. 헤더 읽기 → schema_version 표와 대조. `if 일치 버전 있음 → schemaVersion · else → schemaVersion=null, columnMismatch=차이 목록`.
3. 기준일 컬럼으로 periodFrom/To, rowCount.
4. source별 검사: prod → 결측(`-`) 수, 골격 행(측정값 전부 빈 행) 수 · sales → 자연키 중복 수, 일자별 재고 합 급변(전일 대비 1.8배 이상) 일자 · news → result_id 중복, 게시 시각 훼손 수 · bbg → 시트 분류(시계열/재무) · mkl → 기준월 범위.
5. `if 기존 L1에 같은 기준일 존재 → overlapsExisting=true`.

**출력** Preflight.

**예외** | 파일 파싱 불가 → 422 schema-mismatch | 크기 한도 초과 → 413 |

**호출하는 것** [[TBL-DOM-001#schema_version]]

**테스트 관점** 3개월 샘플 3파일이 각각 기대 schema_version으로 판별된다. 뉴스 가공본·원본이 서로 다른 버전으로 판별된다. 컬럼 하나를 바꾼 파일이 columnMismatch에 그 컬럼을 낸다.

근거: [[TBL-API-001#POST/api/admin/intel/ingest/preflight]]

#### IngestService.run 적재 실행

**시그니처** `run(staging_id: str, overwrite: bool, backfill: bool) -> IngestRun`

**처리**
1. 스테이징 조회. `if 만료 → 404`.
2. `if overlapsExisting and not overwrite → 409 ingest-overlap`.
3. 원본 파일을 `/data/raw/{source}/{yyyymm}/`로 이동, ingest_run insert(status=running, backfill).
4. L0 insert (원본 행 텍스트/jsonb, row_no).
5. [[#IngestService.standardize]](run_id).
6. [[#IngestService.regression_check]](run_id).
7. ingest_run 갱신(row_count, null_count, skeleton_count, dup_count, match_rates, regression_flag, status=success).

**출력** IngestRun.

**예외** | 5·6 실패 → status=failed, error 기록, L0는 유지 |

**테스트 관점** 같은 파일 두 번 적재(overwrite) 후 L1 행수가 늘지 않는다. 원본 파일이 볼륨에 존재한다.

근거: [[TBL-SEQ-001#SEQ-7]] · [[TBL-API-001#POST/api/admin/intel/ingest]]

#### IngestService.standardize L0 → L1 표준화

**시그니처** `standardize(run_id: int) -> StdStats`

**처리** (source별)
1. prod: 컬럼 매핑 → production_daily. `if 측정값 전부 빈 → is_skeleton=true`. `if 값 == '-' → null`. iso3 = factory 조인 · 미매핑 → mapping_gap(factory).
2. sales: 컬럼 매핑 → sales_daily. country_cd = agency_cd[:3]. iso3 = country 조인 · 미매핑 → mapping_gap(country). cbu_ckd = model_cbu_ckd 조인 · 없음 → null. 자연키 중복 → dup_flag=true(중복 규칙은 미결, 현재 전부 보존).
3. news: payload → article(upsert on result_id). category_cd = news_category.aliases 정규화. `if published_at 파싱 실패 → published_at=published_date 00:00, precision=date`. llm_country_iso3 split → article_country(is_mapped=country 존재). 미매핑 → mapping_gap(news_country).
4. bbg: `if 시트 컬럼에 DATE·PX_LAST 있음 → market_series upsert · else → 원본 보존만`. identifier ∉ series_map → mapping_gap(series).
5. mkl: → oem_monthly. iso3 = country 영문명 매칭 · 실패 → mapping_gap(oem_country).
6. crosswalk: 여기서 처리하지 않는다 → [[#MasterService.stage]].
7. 기준일 범위 내 기존 L1 행 삭제 후 삽입(overwrite 의미).

**출력** 행수, 결측, 골격, 중복, 매칭률(판매→국가, 생산→국가, 뉴스→국가).

**테스트 관점** 3개월 샘플로 판매→국가 177/189, 생산→국가 39/69, 뉴스→국가 134/148이 재현된다. `FWD`와 `KD`가 `FWD/KD`로 합쳐진다.

근거: [[TBL-PRD-001#R1]] [[TBL-PRD-001#R2]] [[TBL-PRD-001#R3]]

#### IngestService.regression_check 회귀 검사

**시그니처** `regression_check(run_id: int) -> bool` (급변이면 true)

**처리**
1. 같은 source의 직전 성공 ingest_run 조회. `if 없음 → false`.
2. threshold_config.regression_tolerance(row_count ±50%, null_rate +10pt, match_rate -10pt, dup_rate +5pt 기본).
3. 하나라도 초과 → regression_flag=true, ops.batch_state.blocked=true(사유 기록), 관리자 알림 이벤트.

**테스트 관점** 행수가 절반인 파일을 넣으면 blocked가 켜지고 스케줄 배치가 skip된다.

근거: [[TBL-UC-001#UC-S6]]

### 2.2 배치

#### Pipeline.run 7단계 오케스트레이션

**시그니처** `run(as_of: date, start_step: int = 1, trigger: str = 'schedule', backfill: bool = False) -> BatchRun`

**처리**
1. `if batch_state.running → 409`. `if trigger == 'schedule' and batch_state.blocked → skip, 423 기록`.
2. batch_run insert(running). threshold = threshold_config current(effective_from ≤ now). `if trigger == 'regenerate' → threshold = 당시 게시본 judgment.threshold_version`.
3. step ≥ start_step 순으로:
   - 1~2: 대기 스테이징 없음 → skip.
   - 3: [[#EventBuilder.build_events]](as_of, backfill).
   - 4: [[#SignalBuilder.build_facts]](as_of) → [[#SignalBuilder.build_signals]](as_of).
   - 5: [[#Judge.judge]](as_of, threshold) → [[#Judge.select_watchlist]].
   - 6: `if backfill → skip · else → [[#Narrator.narrate]]`.
   - 7: [[#Publisher.publish]](briefing, auto = trigger != 'regenerate') → [[#Publisher.diff_watchlist]].
4. 각 단계 소요·건수 step_log. llm_tokens 합.
5. `if 단계 4·5 예외 → status=failed, failed_step, 중단 · if 단계 3·6 예외 → 강등 처리 후 계속 · else → status = degraded if briefing.degraded else success`.

**예외** | 배치 창 초과(4h) → 경고 기록, 중단하지 않음 |

**테스트 관점** 단계 5에서 예외를 주입하면 이전 게시본이 유지된다. 백필 모드에서 llm_call이 0건이다.

근거: [[TBL-SEQ-001#SEQ-3]]

### 2.3 사건 통합

#### EventBuilder.build_events 기사 묶음과 사건 갱신

**시그니처** `build_events(as_of: date, backfill: bool) -> EventStats`

**입력** article_country where published_date ∈ (as_of - window, as_of], 아직 event_article 없음. window = threshold.event_window_days.

**처리**
1. 해협 귀속: 기사 keywords·title이 strait_country.keywords와 겹치면 (strait_code, category) 묶음으로도 복제.
2. 묶음 키 = (iso3 | strait_code, category_cd).
3. 묶음마다 열린 사건(state=open, last_seen ≥ as_of - window) 조회. `if 있음 → 기사 붙임, last_seen 갱신 · else → event insert(named_by=fallback, title=최고 impact 기사 제목, severity=max impact)`.
4. event_article insert(rank = impact 내림차순). source_count, article_count 재계산.
5. `if not backfill → 신규·변경 사건을 severity 내림차순 정렬 → 토큰 상한 내에서 [[#EventBuilder.name_event]]`.
6. 열린 사건 중 last_seen < as_of - window → state=closed.

**출력** 사건 수(신규·변경·종료), LLM 성공·실패 수.

**테스트 관점** 호르무즈 가공본 460건이 사건 ≤ 5건으로 묶인다. 국가 태그 없는 해협 기사가 strait_country로 OMN·ARE 사건에 들어간다.

근거: [[TBL-SEQ-001#SEQ-4]] · [[TBL-PRD-001#R4]]

#### EventBuilder.name_event 사건 명명 (LLM 지점 1)

**시그니처** `name_event(event_id: int) -> None`

**처리**
1. 상위 5 기사(제목, 요약)와 카테고리로 프롬프트.
2. [[#HchatClient.complete_json]](schema = {title, event_type, event_subtype, severity 0~100}).
3. `if ok → event 갱신(named_by=llm, llm_call_id) · else → 유지(fallback)`.

**테스트 관점** JSON 위반 응답을 주입하면 event가 fallback 값으로 남고 llm_call.outcome=invalid_json.

#### SignalBuilder.build_facts 국가 × 일 팩트

**시그니처** `build_facts(as_of: date) -> int`

**처리**
1. model_cbu_ckd 갱신: production_daily 최근 12개월에서 model_cd(+production_corp)별 cbu_ckd 최빈값과 confidence.
2. sales_daily(date = as_of) → iso3별 플로우 합(shipping, wholesale, retail), 스톡 값(corp, dealer, voyage, waiting; 같은 날 여러 행이면 합), total_inv.
3. production_daily(date = as_of, not skeleton) → iso3별 합.
4. cbu_ratio = 연초부터 as_of까지 sales_daily wholesale 누적 중 cbu_ckd='CBU' 비중. `if 유도 행 비중 < 0.5 → null`.
5. top_models = 누적 wholesale 상위 3.
6. data_quality_flag: `if total_inv > 전일 × 1.8 → 'dup_suspect'`.
7. country_daily_fact upsert (iso3, date). 데이터 없는 국가도 0/null 행.

**테스트 관점** 9/8 재고 급증 일자에 dup_suspect가 붙는다. 179개국 행이 생긴다.

근거: [[TBL-PRD-001#R7]] [[TBL-PRD-001#R9]] [[TBL-PRD-001#R10]]

#### SignalBuilder.build_signals 결합 신호

**시그니처** `build_signals(as_of: date) -> int`

**처리**
1. wholesale_mtd = 당월 1일~as_of wholesale 합.
2. 비교값: `if 전년 동월 같은 일수 데이터 있음 → cmp=그 합, method=YoY · elif 전월 같은 일수 있음 → method=MoM · else → cmp=null, method=none`.
3. wholesale_change: `if cmp null → null · if cmp == 0 → null(표시 "비교 불가") · else → (mtd - cmp)/cmp`.
4. voyage_ratio = voyage_inv/total_inv, waiting_ratio 동일. total 0이면 null.
5. 사건: open_event_count, max_event_severity, top_event_id(심각도 최고), routed_event_count = routing 도메인별 개수.
6. 시장: series_map.in_indicator_bar identifier마다 as_of 이전 최신값(carry_days 이내) → 컬럼/jsonb, indicator_asof.
7. oem_month = oem_monthly 최신 기준월.
8. signal_daily upsert.

**테스트 관점** 3개월 샘플에서 모든 국가가 method=MoM 또는 none이다(YoY 불가). 주말 as_of에 시장지표가 금요일 값으로 이월된다.

근거: [[TBL-PRD-001#R7]] [[TBL-PRD-001#R8]]

### 2.4 판정

#### Judge.judge 판정

**시그니처** `judge(as_of: date, threshold: ThresholdConfig, batch_run_id: int) -> list[Judgment]`

**처리** signal_daily(as_of) 행마다
1. sales_anomaly = `wholesale_change is not null and wholesale_change ≤ threshold.yoy_threshold` (당월 0이고 cmp>0이면 change=-1.0이라 자동 포함).
2. inventory_stay = `voyage_ratio ≥ threshold.inventory_stay_ratio or waiting_ratio ≥ threshold.inventory_stay_ratio`.
3. exposure = `if cbu_ratio null → unknown · elif cbu_ratio ≥ threshold.cbu_dominant_ratio → CBU · else → CKD`.
4. 근거 사건 = open events with severity ≥ threshold.min_evidence_severity and category ∈ routing(판매 or 재고). cross_confirmed = `(sales_anomaly or inventory_stay) and len(근거 사건) ≥ threshold.min_evidence_count`.
5. signal = `if not cross_confirmed → null · elif exposure == CBU → RED · else → YELLOW`.
6. domain_state: 생산 = `if 생산 급감(전월 대비 ≤ yoy_threshold) → YELLOW · else → MONITOR` · 판매 = `if sales_anomaly and 근거 → RED · elif sales_anomaly → YELLOW · else → MONITOR` · 재고 = 동일 논리 + dup_suspect면 note.
7. entity_tags = glovis_entity where iso3. 없으면 [].
8. judgment insert(threshold_version = threshold.version, batch_run_id).

**출력** Judgment 목록.

**테스트 관점** 3개월 샘플 + 뉴스 가공본 + DBA 초안 임계값으로 cross_confirmed 국가 집합에 ARE, OMN, PAK가 포함된다. cbu_ratio null인 국가는 RED가 되지 않는다.

근거: [[TBL-SEQ-001#SEQ-5]] · [[TBL-PRD-001#R11]] [[TBL-PRD-001#R12]]

#### Judge.select_watchlist 워치리스트 선별 (간략형)

**시그니처** `select_watchlist(judgments) -> list[WatchlistDraft]`

**처리** cross_confirmed만 → signal RED 우선, 같은 신호 안에서 max_event_severity 내림차순 → sort_order 부여 → 대표 사건·상위 기사 3·구조체 필드 채움.

**테스트 관점** cross_confirmed=false 국가가 결과에 없다.

### 2.5 서술

#### Narrator.narrate 문장 생성 (LLM 지점 2)

**시그니처** `narrate(as_of: date, batch_run_id: int, judgments, watchlist) -> Briefing`

**처리**
1. briefing insert(published=false, version = 같은 as_of 최대 +1, data_max_dates).
2. 근거 목록 구성: 항목별로 evidence 행 생성(seq 순차, display 복사). signal → judgment 값, event → 대표 사건, article → 상위 3, market → 지표 바 4종.
3. 섹션 순서: headline → domain:production, sales, inventory → watch:{iso3} × N → detail:{iso3} × N.
4. 섹션마다 판정 구조체(2.0) 생성 → [[#HchatClient.complete_json]](schema = {text, footnotes[]}) → [[#Narrator.validate]].
5. `if validate 실패 → 검증 사유를 붙여 재생성 1회 → 재실패 → [[#Narrator.degrade]](섹션)`. `if LLM outcome ≠ ok → degrade(섹션)`.
6. briefing_claim insert. degraded = 강등 섹션 ≥ 1, degrade_reason 목록.
7. headline, headline_badge("상관관계 확인 · 인과관계 미확정"), domain_status jsonb, indicator_bar jsonb 채움.

**출력** Briefing.

**테스트 관점** 워치리스트 0건이면 headline과 domain 3만 생성된다. 입력 구조체 직렬화 크기가 2KB를 넘는 항목이 없다.

근거: [[TBL-SEQ-001#SEQ-6]] · [[TBL-PRD-001#R14]] [[TBL-PRD-001#R15]]

#### Narrator.validate 생성문 검증

**시그니처** `validate(text: str, footnotes: list[int], structure: dict) -> list[str]` (위반 사유 목록, 빈 목록이면 통과)

**처리**
1. 숫자·퍼센트·날짜 토큰 추출 → 전부 structure 값(정규화 비교: 1.0 ↔ 100%, 418 ↔ 418대)에 존재해야 함.
2. 국가명·법인 태그가 structure 안의 것만.
3. footnotes ⊂ structure.evidence.seq. 각주 없는 문장 없음.
4. 금지어 목록(예상됩니다, 전망, 확실, 원인은, 때문에 등 인과·예측 어휘) 없음.
5. 강조 토큰 `[[hl]]…[[/hl]]` 쌍 정합, 문장당 3개 이하, 그 외 마크업 없음.

**테스트 관점** "물동량 20% 감소가 예상됩니다"가 금지어와 근거 없는 수치 두 사유로 실패한다.

근거: [[TBL-PRD-001#R16]] [[TBL-PRD-001#R17]] [[TBL-PRD-001#N4]]

#### Narrator.degrade 템플릿 강등 (간략형)

**시그니처** `degrade(section: str, structure: dict, reason: str) -> Claim`

**처리** 섹션별 템플릿(국가, 정형 신호 값·기준, 재고 신호, 사건 제목·기사 수)에 값 삽입 → 각주는 evidence seq 전부 순서대로 → llm_call.outcome·reason 기록.

**테스트 관점** 강등 문장도 validate를 통과한다.

### 2.6 게시

#### Publisher.publish 게시 (간략형)

**시그니처** `publish(briefing_id: int, auto: bool) -> None`

**처리** `if auto → 같은 as_of의 기존 published=false, 이 briefing published=true · else → 대기(관리자 [[#BatchService.publish_version]])`. watchlist_item insert. 불변식 검사(claim footnotes ⊂ evidence, watchlist ⊂ cross_confirmed) 실패 → 예외(배치 failed).

#### Publisher.diff_watchlist 워치리스트 변화 (간략형)

**시그니처** `diff_watchlist(briefing_id: int) -> list[AlertEvent]`

**처리** 직전 published briefing(as_of 작은 것) watchlist와 비교 → new/up/down/exit → alert_event insert(delivered=false) → watchlist_item.change_badge 갱신 → `if 채널 어댑터 등록 → 발송, delivered=true`.

근거: [[TBL-SEQ-001#SEQ-10]]

### 2.7 열람

#### BriefingQuery.latest 최신 게시 브리핑 (간략형)

**시그니처** `latest() -> BriefingView | None`

**처리** briefing where published order by as_of_date desc limit 1 → watchlist_item, briefing_claim, evidence 조회 → BriefingView 조립(indicator_bar·domain_status는 jsonb 그대로). LLM·mart·std 접근 없음.

**테스트 관점** 쿼리 수가 4 이하, p95 2초.

근거: [[TBL-SEQ-001#SEQ-1]] · [[TBL-API-001#GET/api/intel/briefing/latest]]

#### BriefingQuery.detail 판매 상세 (간략형)

**시그니처** `detail(briefing_id: int, iso3: str) -> DetailView`

**처리** briefing 게시 확인(아니면 404) → signal_daily(iso3, as_of) 표 행 → claims section=detail:{iso3} → evidence → indicator 4종 as-of 값 → DetailView.

근거: [[TBL-SEQ-001#SEQ-2]] · [[TBL-API-001#GET/api/intel/briefing/{briefingId}/detail/{iso3}]]

### 2.8 관리

#### BatchService.trigger 배치 트리거 (간략형)

**시그니처** `trigger(as_of: date, start_step: int, backfill: bool) -> BatchRun`

**처리** `if running → 409 · if blocked and not manual → 423` → trigger = `'regenerate' if as_of < today else 'manual'` → 백그라운드로 [[#Pipeline.run]] → 202 BatchRun.

근거: [[TBL-API-001#POST/api/admin/intel/batch/run]]

#### BatchService.publish_version 재생성본 게시 전환 (간략형)

**시그니처** `publish_version(run_id: int) -> PublishResult`

**처리** run의 briefing 조회 → 같은 as_of 기존 published=false → 이 briefing published=true → [[#Publisher.diff_watchlist]] 재계산 안 함(과거 일자) → PublishResult.

근거: [[TBL-SEQ-001#SEQ-8]]

#### MasterService.stage 크로스워크 차이 계산

**시그니처** `stage(file: UploadFile) -> MasterDiff`

**처리**
1. 시트명 → 마스터 매핑(02→country, 03→factory, 01→news_category, 08→routing, 06→series_map). 그 외 시트 무시.
2. 시트마다 키 기준으로 현재 버전과 비교 → added, changed, removed, removedValues.
3. affectedKeys: removed·changed 키가 최근 30일 L1에 등장한 값.
4. 스테이징 저장(stagingId, 만료 1시간).

**테스트 관점** 기아 공장 30종을 추가한 03 시트가 added=30, removed=0으로 나온다.

근거: [[TBL-SEQ-001#SEQ-9]] · [[TBL-API-001#POST/api/admin/intel/master/upload]]

#### MasterService.confirm 마스터 확정 (간략형)

**시그니처** `confirm(staging_id: str, user: str) -> MasterVersion`

**처리** 스테이징 조회(만료 404) → 마스터별 새 version 행 일괄 insert(트랜잭션) → mapping_gap.resolved_version 갱신(해결된 값) → MasterVersion.

근거: [[TBL-API-001#POST/api/admin/intel/master/confirm/{stagingId}]]

#### ConfigService.save 설정 저장 (간략형)

**시그니처** `save(input: ThresholdConfigInput, user: str) -> ThresholdConfig`

**처리** 범위 검증(yoy -1~0, severity 0~100, count ≥1, cbu 0~1, window 1~30, cap ≥0) 실패 → 422 → threshold_config insert(version+1, effective_from=now, changed_by) → 반환. 즉시 적용 없음.

근거: [[TBL-API-001#POST/api/admin/intel/config]]

### 2.9 LLM 어댑터

#### HchatClient.complete_json JSON 모드 호출

**시그니처** `complete_json(point: str, prompt: str, schema: dict, batch_run_id: int) -> LlmResult`

**처리**
1. 설정(base_url, auth_header 원문 키, model, project_id) 로드. 요청 해시 계산.
2. Chat Completions 호출, response_format=json_object, temperature 0.2, timeout 60s, 재시도 1회(5xx·timeout만).
3. 응답 분기: `if HTTP 오류·timeout → outcome=error · if content_filter_results 차단 또는 finish_reason=content_filter → outcome=filtered · if JSON 파싱·스키마 검증 실패 → outcome=invalid_json · else → outcome=ok, data`.
4. llm_call insert(point, model, 토큰, latency, outcome, request_hash).
5. 일 토큰 합이 threshold.llm_token_daily_cap 초과 → 이후 호출은 outcome=error(reason=cap) 즉시 반환.

**출력** LlmResult(outcome, data | None, call_id).

**예외** 없음(전부 outcome으로 흡수).

**테스트 관점** 필터 차단 응답 fixture로 outcome=filtered. 상한 도달 후 호출이 네트워크를 타지 않는다. 외부 개발 환경에서 OpenAI 엔드포인트로 같은 코드가 돈다.

근거: [[TBL-INFRA-001#C3]] [[TBL-INFRA-001#C12]] · [[TBL-RFQ-001#Q29]] [[TBL-RFQ-001#Q30]]

## 3. 미결사항

- [ ] sales_daily 자연키 중복 규칙(합산 vs 최신). 현재 전부 보존·dup_flag만
- [ ] 생산 도메인 상태 판정 기준(전월 대비 급감). 임계값을 별도로 둘지 yoy_threshold 공유할지
- [ ] 금지어 목록 초안과 현업 검토
- [ ] validate의 수치 정규화 규칙(단위, 반올림 허용 폭)
- [ ] 백필 모드에서 사건 fallback 제목이 화면에 노출돼도 되는지
- [ ] regression_tolerance 기본값의 근거(실데이터 1개월 후 재조정)
- [ ] 판정 구조체 2KB 초과 시 evidence 절삭 규칙. 가정: 기사 3 → 2로 축소
