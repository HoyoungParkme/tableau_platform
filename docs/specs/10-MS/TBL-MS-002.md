---
doc_id: TBL-MS-002
type: MS
title: Glovis 완성차 인텔리전스 미니스펙
status: draft
upstream: [TBL-SEQ-002, TBL-DOM-005, TBL-DOM-006, TBL-API-002, TBL-INFRA-002, TBL-UC-002, TBL-PRD-002]
---

# MINISPEC

## 0. 이 문서가 다루는 것

[[TBL-DOM-005]]가 정한 클래스 스물아홉의 공개 메서드 하나하나가 **무엇을 받아 무엇을 돌려주고 어디서 갈라지는가**를 정한다. 호출 순서는 [[TBL-SEQ-002]]가, 테이블과 컬럼은 [[TBL-DOM-006]]이, 응답 스키마는 [[TBL-API-002]] 4장이 정한다. 이 문서는 그 셋 사이의 계약이다.

**뼈대는 한 줄이다. 판정은 배치 1~5단계에서 코드가 끝내고, 6~8단계는 확정된 판정을 읽을 수 있게 만드는 일이다**([[TBL-INFRA-002#C19]]). 그래서 [[#TrafficLightJudge.judge]]와 [[#ProximityCalculator.calculate]]의 시그니처에 LLM 인자가 없고, 그 두 함수가 사는 `judgment/` 아래 어느 파일도 `adapters/`를 import 하지 않는다. LLM 세 역할이 전부 실패해도 변동·후보·근접도·신호등은 그대로 게시된다.

등급·점수·순위를 만드는 함수는 없다. 후보에 붙는 값은 근접도 넷(날짜 차이·기사 수·출처 수·국가 일치 방식)과 정렬 규칙이 낳은 자리 번호 `sort_order` 하나뿐이다([[TBL-PRD-002#R10]]).

### 0.1 항목 ID와 열거 값

항목 ID는 `클래스.함수`이고 [[TBL-DOM-005]] 4장 메서드 표의 공개 메서드와 1:1이다. 사설 헬퍼(`-`로 표시된 것)는 항목이 아니다.

열거 값은 [[TBL-API-002]]를 따른다. API가 바깥 계약이기 때문이다. [[TBL-DOM-005]]와 [[TBL-DOM-006]]도 같은 값을 쓴다. 계획 정본 `operationPlan`·`businessPlan`, 국가 일치 방식 `countryDirect`·`straitAttributed`·`crossDomainSameCountry`·`dateOnly`, 후보 유형 `event`·`article`·`marketMetric`·`crossDomainAnomaly`, 명명 주체 `llm`·`representativeArticle`, 워치리스트 변화 `new`·`raised`·`lowered`·`dropped`, 노출 취득 경로 `columnDirect`·`productionDerived`다. 세 문서에서 값이 갈리면 API가 정본이고 나머지를 고친다.

컬럼 이름은 [[TBL-DOM-006]]을, 응답 필드 이름은 [[TBL-API-002]]를 그대로 쓴다. 둘은 층이 달라 같을 필요가 없다(`plan_alt_diff` 컬럼이 `planAlternativeDiff` 필드로 나간다).

**딜러 구간의 이름도 층위가 넷이다.** 화면 표기는 "유통 체류", 설계 문서의 구간 이름은 "딜러 구간", metric 이름은 `distributionStay`, 컬럼 이름은 `dealer_stage_gap`이다. 넷을 섞어 "딜러 단계 체류"로 적지 않는다. 법인 쪽은 화면 표기가 "법인 단계 체류", 구간 이름이 "법인 구간", metric이 `entityStageStay`, 컬럼이 `entity_stage_gap`이다.

## 1. 함수 목록

함수는 111개이고 이 문서의 항목 수와 같다. 계층 다섯과 단계 여덟은 [[TBL-DOM-005]] 1장을 그대로 따른다.

| 계층 | 클래스 | 함수 | 단계 |
|:--|:--|:--|:--|
| 뼈대 | [[TBL-DOM-005#PipelineRunner]] | `run` `stage_names` | 1~8 |
| 적재 | [[TBL-DOM-005#IngestService]] | `preflight` `commit` `unblock_regression` `list_history` `get_file` | 1 |
| 적재 | [[TBL-DOM-005#FormDetector]] | `detect` `explain_mismatch` | 1 |
| 적재 | [[TBL-DOM-005#FormALedgerParser]] | `parse` `check_columns` `base_date_range` | 1 |
| 적재 | [[TBL-DOM-005#FormBPivotParser]] | `parse` `expand_merged_header` `split_period_and_measure` `split_total_rows` `detect_file_base_date` | 1 |
| 적재 | [[TBL-DOM-005#RecordStandardizer]] | `standardize` `map_country` `build_period_key` `collect_unmapped` | 1 |
| 적재 | [[TBL-DOM-005#RegressionChecker]] | `check` `is_abrupt` `abrupt_reasons` | 1 |
| 판정 | [[TBL-DOM-005#AJudgmentReader]] | `read` `copy_snapshot` `validate_shape` `fallback_to_supplementary` | 2 |
| 판정 | [[TBL-DOM-005#EventClusterer]] | `cluster` `count_sources` `pick_representative` `max_impact` `close_expired` | 3 |
| 판정 | [[TBL-DOM-005#FactJoiner]] | `join` `resolve_exposure` `attach_market_asof` `count_events` | 4 |
| 판정 | [[TBL-DOM-005#StageFlowCalculator]] | `calculate` `entity_stage_gap` `dealer_stage_gap` `gap_rate` `alternative_diff` | 4 |
| 판정 | [[TBL-DOM-005#AnomalyDetector]] | `detect` `from_a_judgment` `from_stage_flow` `from_supplementary` `progress_rate` | 4 |
| 판정 | [[TBL-DOM-005#CandidateSearcher]] | `search` `search_events` `search_market` `search_cross_domain` `resolve_country_match` `apply_limit` | 5 |
| 판정 | [[TBL-DOM-005#ProximityCalculator]] | `calculate` `day_diff` `sort` `sort_rule_text` | 5 |
| 판정 | [[TBL-DOM-005#TrafficLightJudge]] | `judge` `build_domain_status` `build_watch_items` `basis` | 5 |
| 서술 | [[TBL-DOM-005#EventNamer]] | `name_events` `fallback_to_representative` | 6 |
| 서술 | [[TBL-DOM-005#CauseLinkWriter]] | `write` `extract_citations` | 7 |
| 서술 | [[TBL-DOM-005#ClaimWriter]] | `write_headline` `write_domain_status` `write_card` | 8 |
| 서술 | [[TBL-DOM-005#CitationVerifier]] | `verify_cause_link` `verify_claim` `out_of_scope_ids` `has_forbidden_words` | 7~8 |
| 서술 | [[TBL-DOM-005#DegradeHandler]] | `degrade` `fill_template` `degraded_roles` | 6~8 |
| 게시 | [[TBL-DOM-005#ReportPublisher]] | `number_evidence` `publish` `publish_a_report` `build_domain_blocks` `diff_watchlist` `copy_display_values` | 8 |
| 열람 | [[TBL-DOM-005#CReportService]] | `get_latest` `get_by_id` | 없음 |
| 열람 | [[TBL-DOM-005#DomainReportService]] | `get_latest_by_domain` `get_by_id` `list_versions` `build_domain_extra` `list_missing_metrics` `render_pdf` | 없음 |
| 열람 | [[TBL-DOM-005#MarketService]] | `get_series` `as_of` `is_carried_over` | 4에서도 쓴다 |
| 관리 | [[TBL-DOM-005#BatchService]] | `get_status` `list_runs` `get_run` `request_rerun` `publish_version` | 없음 |
| 관리 | [[TBL-DOM-005#MasterService]] | `get_summary` `list_unmapped` `upload_crosswalk` `confirm_crosswalk` | 없음 |
| 관리 | [[TBL-DOM-005#ExposureService]] | `summary` `switch` `verify` | 없음 |
| 어댑터 | [[TBL-DOM-005#HChatClient]] | `complete` `remaining_tokens` | 6~8 |
| 어댑터 | [[TBL-DOM-005#BriefingStoreReader]] | `read_judgments` `read_contributions` `read_report_document` `snapshot_id` `ping` | 2 |

[[TBL-DOM-005]] 4장에서 `-`로 표시된 사설 헬퍼는 이 표에도 항목에도 두지 않는다. `build_prompt` 셋, `store_original` `resolve_idempotency_unit` `null_when_missing` `skip_when_country_null` `detection_unit` `meets_threshold` `is_country_axis` `next_version` `assert_published_or_admin` `assert_not_running` `diff_against_current` `log_call` `run_stage` `on_stage_failed` `drop_future_skeleton_rows` `compare_form` `compare_period_types`와 [[TBL-DOM-005#FormDetector]]의 세 헬퍼다. 호출부 안에서만 쓰이고 다른 클래스가 부르지 않는다.

## 2. 공통 계약

- **내부 타입 이름은 [[TBL-API-002]] 4장을 그대로 쓴다.** `Problem` `PeriodKey` `Anomaly` `Contribution` `CauseCandidate` `TrafficLight` `CauseLink` `StageFlow` `MarketMetric` `Evidence` `FormDetection` `TrackingMetric` `MissingMetric` 열셋이다. 같은 뜻의 타입을 새로 만들지 않는다.
- **시간은 언제나 두 칸이다.** `period_type`(`day` `month` `cumulative` `year`) + `file_base_date`. 단일 기준일 인자를 받는 함수를 만들지 않는다([[TBL-INFRA-002#C15]]).
- **축도 두 칸이다.** `axis_type`(`country` `plant`) + `country_code` 또는 `plant_code`. `axis_type`이 `plant`이면 후보 검색·신호등·국가 카드에서 빠진다([[TBL-INFRA-002#C16]]).
- **재현 세 칸을 산출 행마다 남긴다.** `base_date` + `batch_run_id` + `setting_version`. A 판정에서 온 행은 `a_snapshot_id`도 남긴다([[TBL-INFRA-002#C11]]).
- **금지 필드.** `score` `grade` `relevance` `rank` `confidence` 계열을 인자·반환·컬럼 어디에도 두지 않는다. 순서는 `sort_order` 하나다([[TBL-API-002]] 1.3절).
- **결측은 `None`이고 0이 아니다.** 나눗셈의 분모가 0이거나 `None`이면 결과는 `None`이다. 무한대나 0.0으로 바꾸지 않는다.
- **판정 계층 함수는 [[TBL-DOM-005#HChatClient]]를 인자로도 받지 않는다.** 시그니처에 `llm`이 있는 함수는 서술 계층에만 있다.
- 단위: 비율은 소수(`0.050`이 5.0%), 대수는 정수형 `numeric`.
- **열거 값은 [[TBL-API-002]]의 것을 쓴다**(0.1절). 응답으로 나가는 값과 저장하는 값을 다르게 두지 않는다.

## 3. 적재 (배치 1단계)

#### PipelineRunner.run 배치 실행

**시그니처** `run(base_date: date, trigger: str, start_stage: int = 1, batch_run_id: str | None = None) -> BatchRun`

**입력** `trigger`는 `scheduled` `rerun` `regenerate`(시작 단계 1인 재실행) 중 하나. `start_stage`는 1~8. `batch_run_id`는 [[#BatchService.request_rerun]]이 남긴 대기 행이다. 인스턴스 속성 `llm_enabled` `backfill_mode`가 실행 모드를 정하고, 재실행이면 생성자가 받은 당시 설정과 당시 A 판정 스냅샷 저장소를 쓴다.

**처리**
1. 설정을 한 번 정해 `self.setting`에 고정한다. 재실행이면 대기 행의 `setting_version`(당시 설정), 아니면 [[TBL-DOM-006#threshold_setting]] 최신이다. 실행 도중 다시 읽지 않는다.
2. [[TBL-DOM-006#batch_run]] 한 행 생성(`status=running`, `llm_enabled`, `backfill_mode`, `setting_version`). 대기 행을 받으면 새로 만들지 않고 그 행을 `running`으로 바꾼다.
3. `start_stage`부터 8까지 순서대로 돌리고 단계마다 [[TBL-DOM-006#batch_stage_result]] 한 행을 남긴다. 시작 단계가 3~5면 건너뛴 2단계의 산출물(A 판정 사본·보완 집계·판정 출처)을 시작 단계 안에서 다시 싣는다. 5면 그 기준일에 4단계를 끝낸 가장 최근 실행의 변동을 새 식별자로 옮기고, 그런 실행이 없으면 5단계가 멈춘다. 6 이상이면 그 기준일에 5단계를 끝낸 가장 최근 실행의 변동·후보를 읽는다.
4. 1·3·4·5단계 실패 → `status=failed`, `failed_stage` 기록, 중단. 이전 게시본을 그대로 둔다.
5. 2단계 실패 → 멈추지 않는다. [[#AJudgmentReader.fallback_to_supplementary]]로 내려가고 계속한다.
6. 6~8단계 실패 → [[#DegradeHandler.degrade]]를 부르고 계속한다. 8단계의 게시는 언제나 수행한다.
7. `llm_enabled`가 거짓 → 6~8단계의 LLM 역할을 건너뛰고 해당 `batch_stage_result.status=skipped`로 남기되 게시는 한다(회귀 시험용) · `backfill_mode`가 참 → 6~8단계를 건너뛰고 **게시하지 않는다**([[TBL-UC-002#UC-S1]] 8c).
9. 시작 전에 [[TBL-DOM-006#ingest_file]]의 `auto_run_blocked`가 선 건이 있으면 `/problems/regression-blocked`로 시작을 거부한다. 배치 안 1단계에서 급변을 만나면 멈춘다([[TBL-INFRA-002#C20]]).
8. 4시간 창을 넘기면 진행 중 단계를 마치고 경고를 남긴다.

**출력** `BatchRun`. `duration_sec` `llm_call_count` `token_count` `degraded_roles` `report_id`가 채워진다.

**예외** `start_stage`가 1~8 밖이면 `/problems/stage-out-of-range`. 자동 실행이 막혀 있으면 `/problems/regression-blocked`. 같은 기준일이 돌고 있으면 `/problems/batch-in-progress`.

**호출하는 것** [[#IngestService.commit]] [[#AJudgmentReader.read]] [[#EventClusterer.cluster]] [[#FactJoiner.join]] [[#CandidateSearcher.search]] [[#EventNamer.name_events]] [[#CauseLinkWriter.write]] [[#ClaimWriter.write_headline]] [[#ReportPublisher.publish]]

**테스트 관점** 같은 `base_date`를 `llm_enabled=False`로 한 번, 참으로 한 번 돌려 [[TBL-DOM-006#watch_item]]의 `traffic_light`와 [[TBL-DOM-006#cause_candidate]]의 `sort_order`가 전건 일치하는지. 어느 단계에서 끊겨도 `batch_stage_result`가 여덟 행으로 남는지(미실행은 `pending`).

근거: [[TBL-SEQ-002#SEQ-3]] · [[TBL-INFRA-002#C19]] · [[TBL-INFRA-002#C20]] · [[TBL-PRD-002#N3]]

#### PipelineRunner.stage_names 단계 이름 여덟

**시그니처** `stage_names() -> list[str]`

**처리** 아래 여덟을 이 순서 그대로 돌려준다. 상수이고 설정으로 바뀌지 않는다.

```
적재 / A 판정 읽기 / 사건 묶음 / 결합 / 원인 후보·근접도·신호등 / 사건 명명 / 연관 설명 / 서술·검증·게시
```

**테스트 관점** 길이가 8인지. [[TBL-API-002#GET/api/admin/batch/run/{batchRunId}]] 응답의 단계 이름과 [[TBL-DOM-006#batch_stage_result]]의 `stage_name`이 이 목록과 글자까지 같은지.

근거: [[TBL-INFRA-002#C19]] · [[TBL-SEQ-002#SEQ-12]]

#### IngestService.preflight 적재 사전 검증

**시그니처** `preflight(source_type: str, upload: UploadFile, file_base_date: date | None) -> PreflightResult`

**입력** `source_type`은 `vehicleProduction` `vehicleSales` `news` `bloomberg` `marklines` `crosswalk`. `file_base_date`는 형태 B에서 파일이 기준일을 품고 있지 않을 때만 필수다.

**처리**
1. 원본을 볼륨에 먼저 저장한다. 판별 성공 여부와 무관하다([[TBL-PRD-002#N7]]).
2. [[#FormDetector.detect]].
3. `form=A` → [[#FormALedgerParser.check_columns]]와 [[#FormALedgerParser.base_date_range]]로 미리보기를 만든다 · `form=B` → [[#FormBPivotParser.expand_merged_header]] [[#FormBPivotParser.split_total_rows]] [[#FormBPivotParser.detect_file_base_date]]로 기간 구분 목록·지표 종류 목록·총계 행 수·차원 조합 수를 만든다 · `form=unknown` → [[#FormDetector.explain_mismatch]] 결과를 담아 422로 끝낸다.
4. 멱등 단위를 정한다. 형태 A면 `baseDate`, 형태 B면 `fileBaseDate`.
5. 덮어쓸 대상(기준일자 범위 또는 파일 기준일)을 산출한다.
6. `preflight_id`를 발급한다. **`raw`와 `std`에 한 행도 쓰지 않는다.**

**출력** `PreflightResult(preflightId, formDetection, previewA|previewB, idempotencyUnit, overwriteTarget)`.

**예외** `/problems/form-undetected` 422(`mismatch[]`) · `/problems/file-too-large` 413 · 형태 B인데 기준일을 못 읽고 인자도 없으면 `/problems/validation-failed` 400.

**테스트 관점** 인입 샘플 6개 파일이 전부 `form=B`로 나오고 `headerRowCount`가 2~3, `hasBaseDateColumn=false`, `periodBlockRepeated=true`인지. 422로 끝난 뒤에도 원본 파일이 볼륨에 남아 있는지.

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-API-002#POST/api/admin/ingest/preflight]] · [[TBL-INFRA-002#C6]]

#### IngestService.commit 적재 실행

**시그니처** `commit(preflight_id: str, options: CommitOptions) -> IngestFile`

**입력** `options.confirm_overwrite: bool`, `options.file_base_date: date | None`(사전 검증에서 관리자가 넣은 값), `options.backfill_mode: bool`(기본값 거짓). **계획 정본과 도매 정본을 고르는 인자는 받지 않는다.** 정본 선택은 [[TBL-DOM-006#threshold_setting]]의 새 버전으로만 바뀐다. 적재 한 건이 정본을 바꾸면 과거 판정과 어긋난다.

**처리**
1. `preflight_id`가 만료·소비됐으면 409로 끝낸다.
2. 덮어쓸 대상이 있는데 `confirm_overwrite`가 거짓 → 409로 끝낸다.
3. 형태 A면 [[TBL-DOM-006#raw_ledger_row]], 형태 B면 [[TBL-DOM-006#raw_pivot_cell]]에 원행을 넣는다.
4. 멱등 단위로 기존 행을 지운다. 형태 A는 `(source, 기준일자)`, 형태 B는 `(source, file_base_date)` 통째다.
5. [[#RecordStandardizer.standardize]] → [[TBL-DOM-006#vehicle_measure]].
6. [[#RecordStandardizer.collect_unmapped]] → [[TBL-DOM-006#unmapped_value]].
7. [[#RegressionChecker.check]]. 급변이면 [[TBL-DOM-006#ingest_file]]에 `is_abrupt=true` `abrupt_reasons` `auto_run_blocked=true` `status=blocked`를 남긴다. **수동 적재는 완료한다. 막히는 것은 배치 자동 실행뿐이다.** 배치 1단계가 부른 경우는 차단이 아니라 멈춤이다([[TBL-INFRA-002#C20]]).
8. [[TBL-DOM-006#ingest_file]] 한 행(`row_count` `total_row_count` `null_rate` `match_rate` `duplicate_rate` `period_types` `measure_types` `ingest_seq`).
9. 한 파일에 계획 두 종류(운영계획·사업계획)나 도매 두 종류(실 도매·도매 공식)가 함께 들어왔으면 **그 사실만** `dual_values`에 남긴다. 각 건은 `kind`(`plan` 또는 `wholesale`)와 두 값이다. **어느 쪽이 정본인지는 여기서 정하지 않는다.**
10. `options.backfill_mode`를 [[TBL-DOM-006#batch_run]]의 `backfill_mode`로 그대로 넘긴다. 그 값이 참이면 [[#PipelineRunner.run]]이 6~8단계를 건너뛰고 해당 [[TBL-DOM-006#batch_stage_result]]를 `skipped`로 남기며 게시하지 않는다. **판정 1~5단계는 그대로 돈다.**

**출력** `IngestFile`.

**예외** `/problems/preflight-expired` 409 · `/problems/overwrite-not-confirmed` 409(`idempotencyUnit`, `target`).

**테스트 관점** 같은 파일을 두 번 넣어도 `vehicle_measure` 행수가 같은지(형태 B는 파일 기준일 단위 통째 덮어쓰기). 총계 행이 `is_total_row=true`로 들어가 판정 쿼리에서 빠지되 `total_row_count`로 세어져 있는지. 결측 `-`이 `NULL`이고 0이 아닌지. 요청 본문에 정본 선택 키를 넣어도 받지 않는지. `backfill_mode=true`로 적재한 뒤 도는 배치에서 6~8단계가 `skipped`이고 게시본이 생기지 않으며 신호등과 후보 순서가 정상 배치와 전건 같은지.

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-API-002#POST/api/admin/ingest/commit]] · [[TBL-PRD-002#R3]]

#### IngestService.unblock_regression 회귀 차단 해제

**시그니처** `unblock_regression(ingest_file_id: str, reason: str, actor: str) -> IngestFile`

**처리** `reason`이 비면 400. [[TBL-DOM-006#ingest_file]]에 `unblocked_at` `unblocked_by` `unblock_reason`을 남기고 `auto_run_blocked=false`, `status=unblocked`로 바꾼다. **판정값은 하나도 바꾸지 않는다.** 다음 배치가 자동으로 돌 수 있게만 한다.

**테스트 관점** 해제 후 [[#BatchService.get_status]]의 차단 목록에서 빠지는지. 사유 없이 부르면 400인지.

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-API-002#POST/api/admin/ingest/unblock/{ingestFileId}]]

#### IngestService.list_history 적재 이력 조회

**시그니처** `list_history(filters: dict, cursor: str | None) -> tuple[list[IngestFile], str | None]`

**처리** `source` `data_form` `abrupt` `기간`으로 거르고 `loaded_at` 내림차순으로 커서 페이징한다. 집계하지 않는다. 각 행에 그 파일이 품고 있던 기간 구분 목록 `period_types`(`day` `month` `cumulative` `year`)를 함께 싣는다. 목록에서 바로 "이 파일에 누계가 들어 있었다"를 읽게 하기 위해서다.

**테스트 관점** 같은 커서를 두 번 부르면 같은 쪽이 나오는지. 형태 B 행의 `periodTypes`에 실제로 들어 있던 구분만 담기고 네 값 밖이 나오지 않는지.

근거: [[TBL-API-002#GET/api/admin/ingest/history]]

#### IngestService.get_file 적재 1건 상세

**시그니처** `get_file(ingest_file_id: str) -> IngestFile`

**처리** [[TBL-DOM-006#ingest_file]] 한 행을 회귀 검사 결과(`is_abrupt` `abrupt_reasons` `previous_ingest_file_id`), `form_evidence`, `period_types`, `measure_types`, `total_row_diff`와 함께 돌려준다. 없으면 404.

**테스트 관점** 형태 B 파일에서 `period_types`가 일·월·누계·년 중 실제로 들어 있던 것만 담는지.

근거: [[TBL-API-002#GET/api/admin/ingest/file/{ingestFileId}]]

#### FormDetector.detect 형태 판별

**시그니처** `detect(file_path: str, source_type: str) -> FormDetection`

**처리**
1. 헤더 줄 수, 기준일자 컬럼 유무, 기간 블록 반복 여부, 차원 컬럼 수를 센다.
2. 첫 행이 바로 헤더이고 기준일자 컬럼이 있으면 → `form=A` · 병합 헤더가 2~3줄이고 기간 블록이 반복되면 → `form=B` · 둘 다 아니면 → `form=unknown`.
3. **셋째 형태를 추론하지 않는다.** 새 형태를 받는 것은 개발 작업이다([[TBL-INFRA-002#C6]]).

**출력** `FormDetection(form, formLabel, schemaVersion, evidence, mismatch)`. `evidence`는 위 넷을 그대로 담는다.

**테스트 관점** 자사 인입 샘플(생산 2, 판매 4)이 전부 `B`인지. 기준일자 컬럼만 있고 헤더가 3줄인 섞인 파일이 `unknown`으로 떨어지는지.

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-API-002]] 4.11절

#### FormDetector.explain_mismatch 판별 실패 설명

**시그니처** `explain_mismatch(file_path: str) -> list[dict]`

**처리** `{expected, found}` 쌍 목록을 돌려준다. 화면이 그대로 적는다. 추측한 형태를 담지 않는다.

**테스트 관점** 목록이 비어 있지 않은지. 사람이 읽고 무엇을 고쳐야 할지 알 수 있는 문장인지.

근거: [[TBL-API-002]] 4.11절

#### FormALedgerParser.parse 형태 A 읽기

**시그니처** `parse(file_path: str, source_type: str) -> DataFrame`

**처리** 첫 행을 헤더로 읽고, 값이 전부 빈 미래 일자 골격 행을 센 뒤 뺀다. 행마다 기준일자가 있으므로 기간 구분은 `day`로 고정된다.

**출력** 원장 그대로의 표와 골격 행 수.

**테스트 관점** 실물 원장(현대차 데이터_0910.xlsx)과 IF 레이아웃 정의가 같다. 생산 16열, 판매·재고 22열이고 둘째 줄은 영문 코드 행이라 세어 두고 뺀다. 레이아웃 파일의 `→ 측정값(실적)` 칸은 구분 표시라 열이 아니다(2026-09-30 대조). 실물 앞부분 표본으로 검증한다.

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-INFRA-002#C6]]

#### FormALedgerParser.check_columns 컬럼 대조

**시그니처** `check_columns(raw: DataFrame, schema_version: str) -> list[dict]`

**처리** 기대 컬럼 목록과 실제 컬럼을 대조해 `{expected, found}` 어긋남 목록을 돌려준다. 빈 목록이면 통과다. 어긋나도 예외를 던지지 않고 목록으로 돌려 미리보기가 적는다.

**테스트 관점** 생산 16열, 판매·재고 22열에서 한 열을 지운 파일이 그 열 이름을 정확히 짚는지.

근거: [[TBL-DOM-005#FormALedgerParser]]

#### FormALedgerParser.base_date_range 기준일자 범위

**시그니처** `base_date_range(df: DataFrame) -> tuple[date, date]`

**처리** 파일이 담은 기준일자의 처음과 끝을 돌려준다. 덮어쓸 범위를 사람에게 보여 줄 때 쓴다.

**테스트 관점** 3개월치 파일에서 처음과 끝이 실제 최소·최대와 같은지.

근거: [[TBL-SEQ-002#SEQ-4]]

#### FormBPivotParser.parse 형태 B 읽기

**시그니처** `parse(file_path: str, source_type: str, file_base_date: date) -> DataFrame`

**처리**
1. `header=[0,1,2]`로 읽어 MultiIndex를 얻는다.
2. [[#FormBPivotParser.expand_merged_header]]로 컬럼마다 (기간 구분, 지표 종류) 쌍을 만든다.
3. 왼쪽 차원 컬럼을 식별자로 두고 나머지를 긴 형태로 편다. **원본 한 행이 지표 종류 수만큼의 행으로 늘어난다.**
4. [[#FormBPivotParser.split_total_rows]]로 총계 행을 표시한다.
5. 모든 행에 `file_base_date`를 붙인다.

**출력** `(차원…, period_type, measure_type, measure_value, is_total_row, source_row_no)` 긴 형태.

**예외** 헤더 쌍이 풀리지 않는 칸이 있으면 그 칸 좌표를 담아 422.

**테스트 관점** 판매 파일 한 행이 (기간 4 × 지표 종류)만큼의 행으로 늘어나는지. 펼친 뒤 `measure_value` 합이 원본 셀 합과 같은지(값이 다른 기간·지표에 붙는 사고를 잡는 유일한 검사다).

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-INFRA-002#C6]] · [[TBL-PRD-002#R3]]

#### FormBPivotParser.expand_merged_header 병합 헤더 펼치기

**시그니처** `expand_merged_header(raw: DataFrame) -> list[tuple[str, str]]`

**처리** 병합으로 빈 칸을 왼쪽에서 오른쪽으로 앞 값으로 채운 뒤(forward fill), 컬럼 수만큼의 (기간 구분, 지표 종류) 쌍 목록을 돌려준다. 채우기 방향을 위아래로 잘못 잡으면 값이 통째로 한 칸 밀린다.

**출력** 컬럼 순서 그대로의 쌍 목록. 길이가 원본 컬럼 수와 같아야 한다.

**테스트 관점** 반환 길이 = 원본 컬럼 수. 기간 블록이 넷(일·월·누계·년)이고 판매 지표 계열이 넷이면 쌍 목록에 `(cumulative, retail)` 같은 조합이 빠짐없이 나오는지.

근거: [[TBL-INFRA-002#C6]]

#### FormBPivotParser.split_period_and_measure 헤더 한 칸 풀기

**시그니처** `split_period_and_measure(header_cell: tuple[str, ...]) -> tuple[str, str]`

**처리** 기간 구분은 `일→day` `월→month` `누계→cumulative` `년→year`. 지표 종류는 아홉으로 매핑한다. `운영계획→operationPlan` `사업계획→businessPlan` `실적→actual` `진도율→progressRate` `전년대비→yoyRate` `선적→shipment` `실 도매→actualWholesale` `도매(공식)→officialWholesale` `소매→retail`. 어느 쪽도 못 풀면 그 칸 원문을 담아 예외.

**테스트 관점** 아홉 매핑이 [[TBL-DOM-006#vehicle_measure]]의 `measure_type` 아홉과 글자까지 같은지. 실 도매와 도매(공식)이 서로 다른 값으로 갈리는지(미주 누계 상세 행 기준으로 실 도매 664,269와 도매(공식) 677,201로 갈려 차이가 12,932대다. 둘을 한 값으로 합치면 이 차이가 사라진다).

근거: [[TBL-INFRA-002#C18]] · [[TBL-DOM-006#vehicle_measure]]

#### FormBPivotParser.split_total_rows 총계 행 분리

**시그니처** `split_total_rows(df: DataFrame) -> TotalRowSplit`

**처리** 총계 행을 상세 행과 나눈다. 상세 행은 `is_total_row=false`, 총계 행은 `is_total_row=true`로 표시하고 **지우지 않는다.** 지표 종류마다 상세 행 합과 총계 행 값의 차이를 센다. 판정 대상은 상세 행뿐이다. 지우면 나중에 원본과 대조할 수 없고, 남겨 두고 세지 않으면 이중 계상이 된다. **총계 행은 상세 행과 범위가 다를 수 있다.** 그 파일의 차원 범위를 총계 행이 따른다고 가정하지 않고, 상세 행과 함께 더하지 않으며, 따로 보관해 대조에만 쓴다.

**출력** `TotalRowSplit(detailRows, totalRows, totalRowCount, diffByMeasure)`. `diffByMeasure`는 미리보기의 총계 행 대조 경고([[TBL-API-002#POST/api/admin/ingest/preflight]]의 `totalRowCheck`)와 [[TBL-DOM-006#ingest_file]]의 `total_row_diff`가 된다.

**테스트 관점** 생산 파일에서 `progressRate`에 값이 있는 행이 전부 `is_total_row=true`인지(개별 행은 전부 0이다). 집계가 `is_total_row=false`인 행만 더하는지. 판매 샘플에서 상세 행 합은 선적 679,551인데 같은 파일의 총계 행은 1,710,715이라 총계 행이 상세 행 합의 약 2.5배다. 총계 행을 상세 행과 함께 더하면 값이 두 배 넘게 부푼다. 상세 행 대권역이 전부 미주인데도 총계 행은 더 넓은 범위를 담고 있으므로, 총계 행 값을 그 파일의 차원 범위 값으로 읽지 않는지도 함께 본다. 차이가 경고로만 나가고 적재를 막지 않는지([[TBL-UC-002#UC-A1]] 4a).

근거: [[TBL-DOM-006#vehicle_measure]] · [[TBL-SEQ-002#SEQ-4]] · [[TBL-RFQ-002]] 4.2절 · [[TBL-PRD-002#N8]]

#### FormBPivotParser.detect_file_base_date 파일 기준일 찾기

**시그니처** `detect_file_base_date(raw: DataFrame) -> date | None`

**처리** 헤더 위 제목 줄과 파일명에서 날짜를 찾는다. 찾으면 그 날짜, 못 찾으면 `None`을 돌려주고 관리자 입력을 요구한다. **추정하지 않는다.** 오늘 날짜로 대신 채우지 않는다.

**출력** `date` 또는 `None`. 어느 경로로 얻었는지는 `ingest_file`이 `file_base_date`와 함께 남긴다.

**테스트 관점** 기준일이 없는 샘플에서 `None`이 나오고 사전 검증이 관리자 입력을 요구하는지(파일 안에서 읽을 수 있는지가 미결이다).

근거: [[TBL-INFRA-002#C15]]

#### RecordStandardizer.standardize 표준화

**시그니처** `standardize(rows: DataFrame, ingest_file: IngestFile) -> int`

**처리**
1. 행마다 [[#RecordStandardizer.build_period_key]]로 시간 두 칸을 만든다.
2. 판매·재고 행은 [[#RecordStandardizer.map_country]]로 국가를 붙인다.
3. **생산 행의 `country_code`는 비운다.** 목적지 국가가 데이터에 없으므로 유추하지 않는다([[TBL-INFRA-002#C16]]).
4. 값은 결측 표기(`-`, 빈칸)를 `NULL`로 바꾼다. 0으로 바꾸지 않는다.
5. [[TBL-DOM-006#vehicle_measure]]에 넣는다. 유일 제약은 `(ingest_file_id, source_row_no, measure_type, period_type)`.

**출력** 넣은 행수.

**테스트 관점** 생산 행 전건이 `country_code IS NULL`인지. 형태 A로 넣은 파일과 형태 B로 넣은 같은 내용이 같은 컬럼 구성의 행이 되는지. 계획이 비어 있는 행에서 값이 `NULL`이지 0이 아닌지(0이면 달성률이 무한대가 된다).

근거: [[TBL-SEQ-002#SEQ-4]] · [[TBL-INFRA-002]] 6.1절

#### RecordStandardizer.map_country 국가 매핑

**시그니처** `map_country(dealer_code: str | None) -> str | None`

**처리** 대리점 코드 앞 세 자리로 [[TBL-DOM-006#dealer_prefix_map]]을 찾는다. 못 찾거나 코드가 비면 `None`을 돌려주고 [[#RecordStandardizer.collect_unmapped]]가 줍는다.

**테스트 관점** `B28AB`→미국, `B06AA`→캐나다, `B20AB`→멕시코. 세 자리보다 짧은 코드에서 예외를 던지지 않고 `None`인지.

근거: [[TBL-PRD-002#R4]] · [[TBL-DOM-006#dealer_prefix_map]]

#### RecordStandardizer.build_period_key 기간 키 만들기

**시그니처** `build_period_key(form: str, base_date: date, period_type: str | None) -> PeriodKey`

**처리** `form=A` → `periodType=day`이고 `fileBaseDate`는 그 행의 기준일자 · `form=B` → `periodType`은 헤더에서 온 기간 구분이고 `fileBaseDate`는 파일 기준일. 두 형태가 섞여 들어오면 일자를 기준으로 삼고 형태 B의 값을 그 기준일에 붙인다.

**출력** `PeriodKey(periodType, periodTypeLabel, fileBaseDate, dataForm)`.

**테스트 관점** 형태 A 행에서 `periodType`이 언제나 `day`인지. 같은 국가의 월 행과 누계 행이 같은 `fileBaseDate`를 갖고도 서로 구분되는지.

근거: [[TBL-INFRA-002#C15]] · [[TBL-API-002]] 4.2절

#### RecordStandardizer.collect_unmapped 미매핑 수집

**시그니처** `collect_unmapped(rows: DataFrame) -> list[dict]`

**처리** 국가·차종·법인 중 못 붙인 값을 `(source, kind, value)`로 모아 등장 횟수를 세고 [[TBL-DOM-006#unmapped_value]]에 누적한다. 같은 값이 다시 나오면 `occurrence_count`를 올리고 `last_seen`를 갱신한다.

**테스트 관점** 같은 미매핑 값이 두 파일에 나와도 행이 하나이고 횟수가 2인지.

근거: [[TBL-SEQ-002#SEQ-13]] · [[TBL-PRD-002#R4]]

#### RegressionChecker.check 회귀 검사

**시그니처** `check(ingest_file: IngestFile, previous: IngestFile | None) -> RegressionResult`

**처리** 행수·결측률·매칭률·중복률을 직전 같은 소스 파일과 견준다. `previous`가 `None`(첫 적재)이면 비교 없이 `abrupt=false`로 통과시킨다. 결과를 [[TBL-DOM-006#ingest_file]]에 남긴다.

**출력** `RegressionResult(abrupt, reasons, form_changed, period_types_changed, row_count_delta_rate, tolerance)`.

**테스트 관점** 첫 적재가 급변으로 잡히지 않는지. 적용된 `tolerance`가 행에 남아 나중에 설정이 바뀌어도 그때 기준으로 읽히는지.

근거: [[TBL-INFRA-002#C14]] · [[TBL-SEQ-002#SEQ-4]]

#### RegressionChecker.is_abrupt 급변 판정

**시그니처** `is_abrupt(current: IngestFile, previous: IngestFile) -> bool`

**처리** 아래 중 하나라도 참이면 급변이다. `data_form`이 직전과 다름 · `period_types` 또는 `measure_types` 목록이 직전과 다름 · 행수 변화율이 `regression_tolerance` 초과 · 매칭률이 `tolerance`를 넘어 떨어짐.

**테스트 관점** **수치가 멀쩡해도 형태가 바뀌면 참인지.** 샘플(형태 B)에서 실데이터(형태 A)로 넘어가는 그 한 번이 이 검사가 존재하는 이유다.

근거: [[TBL-INFRA-002#C6]] · [[TBL-DOM-005#RegressionChecker]]

#### RegressionChecker.abrupt_reasons 급변 사유

**시그니처** `abrupt_reasons(current: IngestFile, previous: IngestFile) -> list[str]`

**처리** 사람이 읽을 문장 목록을 돌려준다. 차단 해제 화면에 그대로 나간다. 예: "형태가 B에서 A로 바뀌었습니다", "행수가 12,400에서 3,100으로 75% 줄었습니다".

**테스트 관점** 급변인데 사유가 빈 목록인 경우가 없는지.

근거: [[TBL-API-002#POST/api/admin/ingest/unblock/{ingestFileId}]]

## 4. 판정 (배치 2~5단계)

이 장의 어느 함수도 [[TBL-DOM-005#HChatClient]]를 부르지 않는다. 5단계가 끝나면 화면에 나갈 판정이 전부 확정된다([[TBL-INFRA-002#C19]]).

### 4.1 A 판정 읽기 (2단계)

#### AJudgmentReader.read A 판정 읽기

**시그니처** `read(base_date: date) -> list[DomainJudgment]`

**처리**
1. [[#BriefingStoreReader.ping]]으로 저장소에 붙는지 먼저 본다. 거짓이면 셋 모두 [[#AJudgmentReader.fallback_to_supplementary]]로 내려가고 여기서 끝낸다.
2. [[#BriefingStoreReader.read_judgments]]로 `production` `inventory` `sales` 셋을 차례로 읽는다. 읽는 중에 접속이 끊겨도 같은 자리로 내려간다. 뷰 `tbl_a_judgment`에 그 도메인 행이 없으면(브리핑 갈래가 축 판정을 만들기 전) 미수신으로 보고 보완 집계로 간다(유저 결정 2026-10-01).
3. 읽은 payload마다 [[#AJudgmentReader.validate_shape]]. 어긋나면 그 도메인만 보완 집계로 내려간다.
4. 정상이면 [[#BriefingStoreReader.read_contributions]]로 기여 상위 항목을 함께 읽는다. **A 리포트가 쓴 문장은 읽지 않는다.**
5. [[#AJudgmentReader.copy_snapshot]].
6. 미수신 도메인 목록을 함께 돌려준다. [[#ReportPublisher.publish]]가 이 목록을 리포트 `notices`의 `aJudgmentNotReceived`로 도메인마다 한 건씩 담는다.

**출력** 도메인 셋의 판정 목록과 미수신 목록. **값을 다시 계산하지 않는다.**

**예외** 없다. **이 단계의 실패는 배치를 멈추지 않는 유일한 코드 단계다.**

**테스트 관점** 저장소를 막아 놓고 돌려도 배치가 8단계까지 가는지. 보완 집계로 만든 판정의 기여가 비고 `is_supplementary=true`인지. 읽어 온 값과 [[TBL-DOM-006#a_judgment]]의 `current_value`가 소수점까지 같은지(재집계하면 대시보드와 어긋난다).

근거: [[TBL-SEQ-002#SEQ-5]] · [[TBL-INFRA-002#C5]] · [[TBL-PRD-002#R2]]

#### AJudgmentReader.copy_snapshot 스냅샷 복사

**시그니처** `copy_snapshot(judgments: list, batch_run_id: str) -> str`

**처리** [[TBL-DOM-006#a_judgment]] [[TBL-DOM-006#a_contribution]]에 그대로 넣고 `a_snapshot_id`를 돌려준다. 원본이 나중에 바뀌어도 그날 리포트는 이 사본으로 다시 만들어진다.

**테스트 관점** 원본을 바꾼 뒤 같은 기준일을 재생성했을 때 변동 목록이 그대로인지([[TBL-PRD-002#N3]]).

근거: [[TBL-INFRA-002#C11]] · [[TBL-SEQ-002#SEQ-5]]

#### AJudgmentReader.validate_shape 스냅샷 모양 검사

**시그니처** `validate_shape(payload: dict) -> list[str]`

**처리** 기대한 키와 필드가 있는지 보고 **어긋난 목록**을 돌려준다. 빈 목록이면 통과다. 예외를 던지지 않는다. 대조 목록은 뷰 `tbl_a_judgment`의 칸이다(design/briefing_interface.md). 필수 `judgment_id` `domain` `axis_type` `country_or_plant` `metric` `period_type` `file_base_date` `current_value` `compare_basis` `is_anomaly`, 열거 `axis_type`(country·plant) `period_type`(day·month·cumulative·year) `compare_basis`(plan·yoy·mom), 생산은 `plant` 축만.

**테스트 관점** 키 하나를 지운 payload에서 그 키 이름이 목록에 나오는지. 목록이 비지 않으면 그 도메인만 보완 집계로 내려가고 나머지 둘은 정상 경로로 가는지.

근거: [[TBL-DOM-005#AJudgmentReader]] · [[TBL-INFRA-002]] 7장

#### AJudgmentReader.fallback_to_supplementary 보완 집계

**시그니처** `fallback_to_supplementary(domain: str, base_date: date) -> list[DomainJudgment]`

**처리** [[TBL-DOM-006#vehicle_measure]]를 직접 집계해 판정을 대신 만든다. `source=supplementaryAggregate`로 표시하고 기여 분해는 비워 둔다. 임계값은 [[TBL-DOM-006#threshold_setting]]의 `change_threshold`를 쓴다(C의 자체 임계는 여기에만 적용된다).

**출력** 판정 목록. 전부 `is_supplementary=true`.

**테스트 관점** A1 생산의 보완 집계가 `axis_type=plant`로만 나오는지(국가 보완이 되지 않는다). 화면에 "대시보드 판정 미수신"이 적히는지.

근거: [[TBL-PRD-002#R2]] · [[TBL-SEQ-002#SEQ-5]]

### 4.2 사건 묶음 (3단계)

#### EventClusterer.cluster 사건 묶음

**시그니처** `cluster(articles: list[Article], window_days: int) -> list[Event]`

**처리**
1. 기사를 (국가, 카테고리)로 나눈다. 해협 기사는 [[TBL-DOM-006#strait_country]]로 인접국을 보태 같은 묶음에 넣는다.
2. `last_seen`가 시간창 안인 열린 사건이 있으면 → 그 사건의 `event_id`를 [[TBL-DOM-006#article]]에 채우고 `last_seen`을 갱신한다 · 없으면 → [[TBL-DOM-006#event]]를 새로 만들고 `first_seen`를 기록한다.
3. [[#EventClusterer.count_sources]] [[#EventClusterer.pick_representative]] [[#EventClusterer.max_impact]]를 채운다.
4. `title`은 이 단계에서 이미 대표 기사 제목으로 채우고 `named_by=representativeArticle`로 둔다. **명명이 실패해도 제목이 비지 않는다.** `named_by`는 `llm`과 `representativeArticle` 둘뿐이고 6단계가 이름을 붙이면 `llm`으로 바뀐다.
5. 시간창을 넘긴 사건은 상태를 종료로 바꾼다.
6. 국가 태그가 없는 기사는 국가 없음 묶음에 두고 후보 검색 대상에서 뺀다.

**출력** 사건 목록. **LLM 호출 0회.**

**예외** 실패하면 강등이 아니라 배치 멈춤이다. `article_count`와 `source_count`가 신호등의 입력이기 때문이다.

**테스트 관점** 호르무즈 기사 460건이 사건 소수로 묶이고 오만·아랍에미리트에 귀속되는지. 모든 사건에서 기사 URL로 역추적되는지. 같은 기사 집합을 두 번 돌려 같은 사건 구성이 나오는지.

근거: [[TBL-SEQ-002#SEQ-6]] · [[TBL-PRD-002#R6]]

#### EventClusterer.count_sources 출처 수 세기

**시그니처** `count_sources(articles: list[Article]) -> int`

**처리** `source_name`의 서로 다른 개수를 센다. 기사 수와 따로 세는 이유는 한 매체가 같은 일을 열 번 쓴 것과 열 매체가 한 번씩 쓴 것을 구분하기 위해서다.

**테스트 관점** 같은 매체 기사 10건이면 1인지. 기사 수와 출처 수가 [[TBL-DOM-006#event]]에 컬럼으로 굳혀지는지(조회할 때마다 세면 같은 배치 안에서도 값이 달라진다).

근거: [[TBL-PRD-002#R5]] · [[TBL-DOM-006#event]]

#### EventClusterer.pick_representative 대표 기사 고르기

**시그니처** `pick_representative(articles: list[Article]) -> Article`

**처리** 붙어 온 영향도 내림차순 → 같으면 `published_at` 오름차순 → 그래도 같으면 `article_id` 사전순. **동률을 끝까지 깨서 같은 입력이면 같은 기사가 나오게 한다.**

**테스트 관점** 같은 묶음을 두 번 돌려 같은 `representative_article_id`가 나오는지.

근거: [[TBL-PRD-002#N3]]

#### EventClusterer.max_impact 영향도 최대값

**시그니처** `max_impact(articles: list[Article]) -> float | None`

**처리** 기사에 **이미 붙어 온** 영향도의 최대값을 돌려준다. 전부 비어 있으면 `None`. **우리가 새로 매기지 않는다.**

**테스트 관점** 영향도가 전부 비어 있어도 예외 없이 `None`인지. 이 값이 신호등 규칙의 입력이 아닌지(입력은 기사 수·출처 수·날짜 차이 셋이다).

근거: [[TBL-PRD-002#R6]] · [[TBL-PRD-002#R10]]

#### EventClusterer.close_expired 사건 종료

**시그니처** `close_expired(events: list[Event], today: date) -> int`

**처리** `today − last_seen > setting.event_window_days`인 열린 사건의 `status`를 `closed`로 바꾸고 바꾼 건수를 돌려준다. 신규 기사가 0건인 날은 [[#EventClusterer.cluster]]가 이 함수만 돌리고 끝낸다([[TBL-UC-002#UC-S3]] 2a). 닫힌 사건은 후보 검색의 시간창 안에 들어오지 않는다. 다시 여는 규칙은 없다(미결).

**테스트 관점** 시간창 안 사건이 닫히지 않는지. 닫힌 사건이 [[#CandidateSearcher.search_events]]에 나오지 않는지.

근거: [[TBL-SEQ-002#SEQ-6]] · [[TBL-DOM-006#event]]

### 4.3 결합·단계별 흐름·변동 판정 (4단계)

#### FactJoiner.join 국가·기간 결합

**시그니처** `join(period_key: PeriodKey) -> list[CountryPeriodFact]`

**처리**
1. [[TBL-DOM-006#vehicle_measure]]에서 `is_total_row=false`인 행을 읽는다.
2. **`country_code`가 `NULL`인 행을 뺀다. 생산이 여기서 빠진다.**
3. 국가 × 기간 키로 모아 `shipment` `actual_wholesale` `official_wholesale` `retail`을 채운다. **재고 원천이 없는 동안 `inventory_source`는 `derived`로 고정하고 재고 네 칸은 전부 `NULL`로 둔다.** 0으로 채우지 않는다. 실측 원천이 들어오면 네 칸을 채우고 `inventory_source`를 `measured`로 바꾼다.
4. 설정의 계획 정본(`self.setting.plan_source`, `operationPlan` 또는 `businessPlan`)으로 고른 계획 값을 `plan_value`에 채우고, 고른 이름을 `plan_source`에, 고르지 않은 계획과의 차이를 `plan_alt_diff`에 남긴다. 계획이 한 종류만 들어온 행의 `plan_alt_diff`는 `NULL`이다.
5. 비교 대상을 같은 행에 굳힌다. `plan_value`가 있으면 → `compare_value=plan_value`, `compare_basis=plan` · 없고 전년 동월 값이 있으면 → 그 값과 `yoy` · 둘 다 없으면 → 전월 값과 `mom`. **계획이 있으면 계획 대비를 먼저 본다는 규칙이 보완 집계 경로에서도 서게 하는 칸이다**([[TBL-PRD-002#R13]], [[TBL-PRD-002]] 6.4). [[#AnomalyDetector.from_supplementary]]는 여기서 채운 칸을 읽을 뿐 다시 고르지 않는다. 비교에 쓰는 계열은 도매 정본(`official_wholesale`)이고 계획도 그 계열의 계획이다. 전년 값은 같은 기간 구분에서 1년 전 월말 이하 가장 최근 파일, 전월 값은 같은 기간 구분의 직전 파일에서 읽는다.
6. [[#FactJoiner.resolve_exposure]] [[#FactJoiner.attach_market_asof]] [[#FactJoiner.count_events]]를 붙인다.
7. [[TBL-DOM-006#country_period_fact]]에 넣는다. 유일 제약은 `(batch_run_id, country_code, period_type, file_base_date)`.

**출력** 결합 행 목록.

**테스트 관점** 결과에 생산 도메인 행이 0건인지. 실적이 있는 미주 29개국 전부에 행이 있는지. 같은 국가의 월 행과 누계 행이 따로 서는지. 재고 네 칸이 전건 `NULL`이고 `inventory_source`가 전건 `derived`인지. 계획이 들어 있는 파일에서 `plan_value`가 비지 않고 `compare_basis=plan`인지. 계획 정본 설정을 바꿔 다시 돌리면 `plan_value`와 `plan_source`가 함께 바뀌고 `setting_version`이 그 사실을 행에 남기는지.

근거: [[TBL-SEQ-002#SEQ-7]] · [[TBL-INFRA-002#C16]] · [[TBL-PRD-002#R12]]

#### FactJoiner.resolve_exposure 차종 노출 판정

**시그니처** `resolve_exposure(country: str, period_key: PeriodKey) -> ModelExposure`

**처리** 형태 B → 판매 파일의 `cbu_ckd` 컬럼을 그대로 읽고 `acquired_by=columnDirect` · 형태 A → 생산 모델코드로 유도하고 `acquired_by=productionDerived`, `derived_ratio`를 남긴다 · 유도 실패 → 노출 미확인이고 그 국가는 [[#TrafficLightJudge.judge]]에서 RED까지 올라가지 못한다.

**출력** [[TBL-DOM-006#model_exposure]] 한 행. 유도 근거는 `derivation_note`에 **글로** 적는다. 수치 점수를 두지 않는다. 칸 이름에 `confidence`를 쓰지 않는 이유는 금지 이름 회귀 검사(2장)에 예외를 두지 않기 위해서다.

**테스트 관점** 형태 B 샘플에서 `acquired_by`가 전건 `columnDirect`인지. 유도 실패 국가가 YELLOW에서 멈추는지. 반환과 컬럼 이름에 `confidence`가 한 곳도 없는지(이름으로 찾는 회귀 검사).

근거: [[TBL-PRD-002#R15]] · [[TBL-DOM-006#model_exposure]]

#### FactJoiner.attach_market_asof 시장지표 as-of 붙이기

**시그니처** `attach_market_asof(fact: CountryPeriodFact, period_key: PeriodKey) -> MarketPoint | None`

**처리** [[#MarketService.as_of]]를 그 기간의 **마지막 날**로 부른다. 조인 축이 날짜뿐이라 국가·차종과 잇지 않는다. 이월 한도를 넘으면 `None`을 붙이고 "기준일 지연"만 적으며 **판정에 쓰지 않는다.**

**출력** `market_asof` jsonb 또는 `None`.

**테스트 관점** `periodType=cumulative`일 때 붙는 날짜가 그 기간의 마지막 날인지. 이월 한도 초과 지표가 변동 판정에 끼지 않는지.

근거: [[TBL-SEQ-002#SEQ-7]] · [[TBL-API-002]] 4.9절

#### FactJoiner.count_events 사건 수 세기

**시그니처** `count_events(country: str, period_key: PeriodKey) -> int`

**처리** 그 국가·기간에 걸린 [[TBL-DOM-006#event]] 수를 센다. 해협 귀속으로 걸린 사건도 포함한다. 결합 행의 표시값이고 신호등의 입력이 아니다.

**테스트 관점** 사건이 0건인 국가도 행이 남고 `event_count=0`인지.

근거: [[TBL-PRD-002#R12]]

#### StageFlowCalculator.calculate 단계별 흐름 계산

**시그니처** `calculate(country: str, period_key: PeriodKey) -> SalesStageFlow`

**처리**
1. 도매 정본을 고른다. `self.wholesale_basis`(기본값 `officialWholesale`).
2. [[#StageFlowCalculator.entity_stage_gap]] [[#StageFlowCalculator.dealer_stage_gap]] [[#StageFlowCalculator.gap_rate]] [[#StageFlowCalculator.alternative_diff]]를 채운다.
3. `derivation=derived`와 `wholesale_basis`를 행에 남긴다. **이 값은 재고가 아니라 재고의 대체물이다.**
4. 재고 원천이 들어오면 같은 자리를 `measured`로 갈아 끼우고 유도 행을 지우지 않는다.
5. [[TBL-DOM-006#sales_stage_flow]]에 넣는다.

**출력** `SalesStageFlow` 한 건.

**예외** 없다. **음수를 예외로 처리하지 않는다.**

**테스트 관점** 미주 누계(CDO 인입 샘플 CSV 추출본, 미주 30개국 629행)에서 선적 679,551 · 도매(공식) 677,201 · 소매 643,097로 `entity_stage_gap=2,350`, `dealer_stage_gap=34,104`, `dealer_stage_gap_rate=0.050`이 나오는지. 국가별 딜러 구간 상위가 칠레 0.252, 페루 0.212, 파나마 0.108, 미국 0.066 순인지. 푸에르토리코 -0.137과 콜롬비아 -0.105가 오류가 아니라 값으로 저장되는지. 법인 구간에서 캐나다 0.083이 잡히는지. 저장소 시험은 엑셀 표본(CDO 판매 인입 데이터 샘플)의 미주 부분집합으로 짠다. 그 표본은 선적 681,761 · 도매(공식) 679,411 · 소매 644,519라 법인 구간 2,350은 같고 딜러 구간은 34,892다.

근거: [[TBL-SEQ-002#SEQ-7]] · [[TBL-PRD-002#R9]] · [[TBL-INFRA-002#C17]]

#### StageFlowCalculator.entity_stage_gap 법인 단계 체류

**시그니처** `entity_stage_gap(shipment: float, wholesale: float) -> float`

**처리** `shipment − wholesale`. 앞 단계에서 뒤 단계를 뺀다. 양수면 법인 단계에 남아 있고 음수면 이전에 쌓인 것을 덜어내는 중이다.

**테스트 관점** 679,551 − 677,201 = 2,350.

근거: [[TBL-API-002]] 1.4절

#### StageFlowCalculator.dealer_stage_gap 유통 체류

**시그니처** `dealer_stage_gap(wholesale: float, retail: float) -> float`

**처리** `wholesale − retail`. 도매 정본에서 소매를 뺀다. 재고 신호로 실제로 쓰는 구간이 이쪽이다.

**테스트 관점** 677,201 − 643,097 = 34,104.

근거: [[TBL-PRD-002#R9]] · [[TBL-API-002]] 1.4절

#### StageFlowCalculator.gap_rate 격차율

**시그니처** `gap_rate(gap: float, denominator: float | None) -> float | None`

**처리** `gap ÷ denominator`. 분모는 언제나 **도매 정본**이다. 분모가 `None`이거나 0이면 `None`을 돌려준다.

**테스트 관점** 34,104 ÷ 677,201 = 0.050. 도매가 0인 국가에서 예외가 아니라 `None`인지.

근거: [[TBL-API-002]] 1.4절

#### StageFlowCalculator.alternative_diff 다른 도매 기준과의 차이

**시그니처** `alternative_diff(actual: float | None, official: float | None) -> float | None`

**처리** `official − actual`. 어느 쪽이 정본인지와 무관하게 부호를 고정한다. 한쪽이 없으면 `None`.

**테스트 관점** 미주 누계 상세 행 기준(CDO 인입 샘플 CSV 추출본)으로 실 도매 664,269와 도매(공식) 677,201, 차이 12,932대다. 검증식은 이 상세 행 기준 값으로 짠다. 같은 파일의 총계 행 값(실 도매 1,692,227 대 도매(공식) 1,706,430, 차이 14,203대)은 상세 행과 범위가 달라 상세 행 합과 더하면 안 되고 대조에만 쓴다. 범위가 다른 것은 미주만 남긴 추출본에 전 권역 총계 행이 남은 탓이다. 같은 국가의 체류율이 `wholesale_basis`에 따라 달라질 때 이 값으로 그 차이가 설명되는지.

근거: [[TBL-INFRA-002#C18]] · [[TBL-DOM-006#sales_stage_flow]]

#### AnomalyDetector.detect 변동 판정

**시그니처** `detect(facts: list, judgments: list, flows: list) -> list[Anomaly]`

**처리**
1. 출처 셋에서 각각 만든다. [[#AnomalyDetector.from_a_judgment]] → [[#AnomalyDetector.from_stage_flow]] → [[#AnomalyDetector.from_supplementary]].
2. 같은 `(domain, axis_type, country_code, plant_code, metric, period_type, file_base_date)`가 겹치면 A 판정을 우선하고 보완 집계를 버린다.
3. 감지 단위는 `self.setting.detection_unit`(기본값 `modelGroup`)이다. **세부 차종 단위로 감지하지 않는다.**
4. [[TBL-DOM-006#anomaly]]에 넣고 `setting_version` `a_snapshot_id` `batch_run_id`를 남긴다.
5. **원인은 여기서 보지 않는다.** 후보는 5단계가 붙인다.

**출력** 변동 목록. `axis_type=plant`인 행도 만든다(도메인 상태 한 줄이 읽는다).

**테스트 관점** 2025년 11월 전년 동월 대비에서 전년 판매가 0보다 컸는데 0이 된 세부 차종은 셋이고 그 달에는 이 셋뿐이다. 팰리세이드 LX2 2,282대→0대 · 넥쏘 FE 111대→0대 · 니로 DE PBV 1대→0대가 **모두** 변동으로 잡히지 않는지(모델 교체 오탐 0건). 셋 중 둘만 덮으면 회귀가 성공 지표를 다 받치지 못한다. **니로 DE PBV는 값이 1대라 임계값 아래로 조용히 통과할 수 있고, 그래서 오탐 0건이 모델 교체 판정 덕인지 값이 작아서인지 가리려면 오히려 이 건이 검사에 필요하다.** 차종 그룹 단위 회귀도 같이 둔다. 팰리세이드를 그룹으로 묶으면 2024년 20,967대에서 2025년 55,291대로 늘어나므로, 세부 차종 LX2의 93% 감소가 급락으로 보고되지 않는지. 같은 입력 재실행 시 변동 구성과 건수가 같은지.

근거: [[TBL-SEQ-002#SEQ-7]] · [[TBL-PRD-002#R13]] · [[TBL-PRD-002]] 6.15

#### AnomalyDetector.from_a_judgment A 판정에서 만들기

**시그니처** `from_a_judgment(judgment: DomainJudgment) -> Anomaly | None`

**처리** `judgment.changed`가 참이면 변동 하나를 만든다. 값·비교값·기준 방식·변화율을 **그대로** 옮기고 임계값을 다시 적용하지 않는다. `source=aJudgment`, `domain_judgment_id`를 연결하고 `axis_type`은 판정이 준 것을 그대로 쓴다.

**테스트 관점** C가 만든 변동의 `value`가 대시보드 화면 수치와 일치하는지. A1 생산 판정이 `axis_type=plant`로 남는지.

근거: [[TBL-PRD-002#R2]] · [[TBL-DOM-006#a_judgment]]

#### AnomalyDetector.from_stage_flow 단계 흐름에서 만들기

**시그니처** `from_stage_flow(flow: SalesStageFlow) -> list[Anomaly]`

**처리** 구간 둘을 각각 본다. 이름과 계산이 서로 바뀌지 않게 아래를 계약으로 굳힌다.

```
딜러 구간 = 도매 정본 − 소매 = flow.dealer_stage_gap   → metric = distributionStay   (화면 "유통 체류")
법인 구간 = 선적 − 도매 정본 = flow.entity_stage_gap   → metric = entityStageStay    (화면 "법인 단계 체류")
```

`abs(flow.dealer_stage_gap_rate) ≥ setting.stay_threshold`면 `metric=distributionStay`로 변동을 만든다 · 법인 구간 비율이 임계를 넘으면 `metric=entityStageStay`로 하나 더 만든다 · 둘 다 미달이면 빈 목록. `source=derived`, `stage_flow_id` 연결, `domain=inventory`. **`wholesaleToRetailGap`은 쓰지 않는다.** [[TBL-API-002]] 4.3절 `Anomaly.metric` enum에 그 값이 없다.

**출력** 변동 0~2건. 음수 비율도 절대값으로 임계를 보고 방향은 값의 부호가 말한다.

**테스트 관점** 미주 누계(CSV 추출본. 선적 679,551 · 도매 정본 677,201 · 소매 643,097)에서 `metric=distributionStay` 변동의 값이 **34,104**, `metric=entityStageStay` 변동의 값이 **2,350**인지. 두 지표 이름이 뒤바뀌면 이 두 값이 서로 자리를 바꾸므로 한 번에 걸린다. 칠레 0.252와 페루 0.212가 `distributionStay`로 올라오는지. 푸에르토리코 -0.137이 방향 표시와 함께 올라오는지. 화면에 유도값 표기가 붙는지.

근거: [[TBL-PRD-002#R9]] · [[TBL-PRD-002#R14]] · [[TBL-INFRA-002#C17]]

#### AnomalyDetector.from_supplementary 보완 집계에서 만들기

**시그니처** `from_supplementary(fact: CountryPeriodFact) -> Anomaly | None`

**처리** **A 판정이 없는 지표만** 대상이다. 결합 행에 굳은 `fact.compare_value`와 `fact.compare_basis`를 그대로 읽고(여기서 다시 고르지 않는다. 계획 정본은 설정이 정하고 결합이 채운다) 변화율이 `setting.change_threshold`를 넘으면 변동을 만든다. 비교 기준은 `plan_value`가 있으면 `plan`, 없고 전년 동월이 있으면 `yoy`, 둘 다 없으면 `mom`이다. `source=supplementaryAggregate`, `fact_id` 연결. 기여 분해는 비고 화면이 "보완 집계"를 적는다.

**테스트 관점** A 판정이 있는 지표에서 이 함수가 중복 변동을 만들지 않는지.

근거: [[TBL-PRD-002]] 6.13 · [[TBL-PRD-002#R13]]

#### AnomalyDetector.progress_rate 진도율 직접 계산

**시그니처** `progress_rate(actual: float | None, plan: float | None) -> float | None`

**처리** `actual ÷ plan`. **원본 `progressRate` 컬럼을 읽지 않는다.** 인입 샘플에서 그 컬럼은 총계 행에만 값이 있고 개별 행은 전부 0이다. `plan`이 `None`이거나 0이면 `None`.

**테스트 관점** 생산 누적 실적 1,739,722 ÷ 사업계획 2,020,531 = 0.861. 법인별로 HMGMA 0.459, BHMC 0.726, HMMI 0.738, HMC 0.850이 나오는지. 운영계획으로 바꾸면 분모가 266,381만큼 달라지고 `setting_version`이 그 사실을 행에 남기는지.

근거: [[TBL-PRD-002#R13]] · [[TBL-DOM-006#vehicle_measure]]

### 4.4 후보·근접도·신호등 (5단계)

#### CandidateSearcher.search 원인 후보 검색

**시그니처** `search(anomaly: Anomaly) -> tuple[list[CauseCandidate], int, list[CauseCandidate]]`

**처리**
1. `anomaly.axis_type != 'country'`이면 → 빈 목록 둘과 0을 돌려준다. **생산이 여기서 빠진다.**
2. [[#CandidateSearcher.search_events]] + [[#CandidateSearcher.search_market]] + [[#CandidateSearcher.search_cross_domain]]을 합친다.
3. 후보마다 [[#CandidateSearcher.resolve_country_match]]로 국가 일치 방식을 정한다.
4. [[#ProximityCalculator.calculate]] → [[#ProximityCalculator.sort]].
5. 상한을 적용하기 전 정렬 목록을 사본으로 남기고 [[#CandidateSearcher.apply_limit]]으로 상한을 적용해 잘린 건수를 받는다.
6. [[TBL-DOM-006#cause_candidate]]에 `sort_order`와 근접도 값 넷을 넣는다. 유일 제약은 `(anomaly_id, sort_order)`. 잘린 건수는 변동 행의 `candidate_truncated_count`에 적는다. 6단계부터 다시 돈 재실행의 게시본도 이 값을 읽는다.

**출력** `(후보 목록, 잘린 건수, 상한 전 정렬 목록)`. **후보가 0건인 변동도 목록에 남는다.** 신호등은 셋째 값으로 판정한다([[#TrafficLightJudge.judge]]). 저장과 화면은 상한까지다. 다만 신호등을 정한 사건이 상한 밖이면 5단계가 그 사건 하나를 목록 끝에 더한다([[#TrafficLightJudge.judge]]).

**테스트 관점** 같은 기준일을 두 번 돌려 후보 구성과 순서가 같은지. `axis_type=plant`인 변동의 후보가 전건 0인지. 후보 0건 변동이 "원인 미확인"으로 화면에 남는지. 상한 밖으로 밀린 사건도 신호등 판정에 드는지.

근거: [[TBL-SEQ-002#SEQ-8]] · [[TBL-PRD-002#R10]]

#### CandidateSearcher.search_events 사건·기사 후보

**시그니처** `search_events(anomaly: Anomaly) -> list[CauseCandidate]`

**처리** 그 국가에 직접 걸린 사건과 [[TBL-DOM-006#strait_country]]로 귀속된 사건 중 `abs(event.last_seen − anomaly.file_base_date) ≤ setting.event_window_days`인 것을 찾는다. 그 가운데 변동의 도메인에 배정된 뉴스 카테고리([[TBL-DOM-006#category_route]])의 사건만 남긴다(자사 크로스워크 08 시트 이관본. FVL은 판매·재고 임시 배정, 유저 결정 2026-09-30, #27 재결정). 카테고리가 없는 사건은 남기지 않는다. 사건마다 상위 기사 3건을 함께 담는다(화면이 최소 3건을 요구한다).

**테스트 관점** 호르무즈 사건이 오만·아랍에미리트 변동의 후보로 나오고 `countryMatch=straitAttributed`로 구분되는지. 해협 표가 비어 있으면 중동 변동의 후보가 통째로 비는지(그 사실이 화면에 드러나는지). 재고 변동에 GEO 사건이 붙지 않고 MRT·FWD/KD·FVL 사건은 붙는지. 판매 변동에 FWD/KD 사건이 붙지 않는지. 카테고리가 없는 사건이 어디에도 붙지 않는지.

근거: [[TBL-PRD-002#R10]] · [[TBL-PRD-002#R5]] · [[TBL-DOM-006#strait_country]] · [[TBL-DOM-006#category_route]]

#### CandidateSearcher.search_market 시장지표 후보

**시그니처** `search_market(anomaly: Anomaly) -> list[CauseCandidate]`

**처리** 지표 4종(`GPR` `BRENT` `BDI` `SCFI`)과 카테고리 연결 시리즈 중 같은 기간의 변화율이 임계를 넘은 것을 후보로 만든다. 국가와 이어지지 않으므로 `countryMatch=dateOnly`로 고정한다. **이 사실을 숨기지 않고 값으로 남긴다.**

**테스트 관점** 시장 후보의 `countryMatch`가 전건 `dateOnly`인지. 이월 한도를 넘은 지표가 후보에서 빠지는지.

근거: [[TBL-PRD-002#R10]] · [[TBL-API-002]] 4.9절

#### CandidateSearcher.search_cross_domain 다른 도메인 변동 후보

**시그니처** `search_cross_domain(anomaly: Anomaly) -> list[CauseCandidate]`

**처리** 같은 국가·같은 기간 키의 다른 도메인 변동을 후보로 만든다. 자기 자신과 같은 도메인 변동은 뺀다. `countryMatch=crossDomainSameCountry`.

**테스트 관점** 판매 변동의 후보에 같은 국가 재고(유도) 변동이 올라오는지. 생산 변동이 후보로 올라오지 않는지(국가 축이 없다).

근거: [[TBL-PRD-002#R10]] · [[TBL-INFRA-002#C16]]

#### CandidateSearcher.resolve_country_match 국가 일치 방식

**시그니처** `resolve_country_match(anomaly: Anomaly, target: object) -> str`

**처리** 대상의 국가가 변동의 국가와 같으면 → `countryDirect` · 해협 표로 귀속됐으면 → `straitAttributed` · 같은 국가의 다른 도메인 변동이면 → `crossDomainSameCountry` · 국가로 이어지지 않고 날짜만 겹치면 → `dateOnly`.

**테스트 관점** 네 값 밖의 문자열이 나오지 않는지. 직접 걸린 후보와 해협으로 걸린 후보가 화면에서 구분되는지.

근거: [[TBL-API-002]] 4.5절

#### CandidateSearcher.apply_limit 후보 상한

**시그니처** `apply_limit(candidates: list[CauseCandidate]) -> tuple[list, int]`

**처리** `sort_order` 순으로 `setting.candidate_limit`까지 남기고(기본 사건 5, 기사 10, 지표 6) **잘린 건수를 함께** 돌려준다. 잘린 축의 후보에는 `is_truncated=true`를 표시한다.

**테스트 관점** 잘린 건수가 0보다 크면 화면에 그 수가 적히는지. 상한을 늘렸다 줄여도 남는 후보의 순서가 같은지.

근거: [[TBL-PRD-002#R10]] · [[TBL-SEQ-002#SEQ-8]]

#### ProximityCalculator.calculate 근접도 값 넷

**시그니처** `calculate(anomaly: Anomaly, candidate: CauseCandidate) -> Proximity`

**입력** 변동 한 건과 후보 한 건. **LLM 인자가 없다.**

**처리** 아래 넷을 센다. 이것이 전부다.

```
day_diff       = ProximityCalculator.day_diff(anomaly, candidate)
article_count  = 사건이면 event.article_count · 기사면 1 · 지표·교차도메인이면 0
source_count   = 사건이면 event.source_count · 기사면 1 · 지표·교차도메인이면 0
country_match  = CandidateSearcher.resolve_country_match의 결과
```

**출력** 근접도 넷. **가중합·정규화·종합 점수를 만들지 않는다.** 등급·점수·순위 필드를 반환에 두지 않는다.

**테스트 관점** 반환 필드 이름에 `score` `grade` `relevance` `rank` `confidence`가 없는지(이름으로 찾는 회귀 검사). 같은 입력이면 같은 넷이 나오는지.

근거: [[TBL-PRD-002#R10]] · [[TBL-PRD-002]] 6.14 · [[TBL-INFRA-002#C19]]

#### ProximityCalculator.day_diff 날짜 차이

**시그니처** `day_diff(anomaly: Anomaly, candidate: CauseCandidate) -> int`

**처리** 기준점은 **변동의 `file_base_date`**다. 후보 쪽 관측일은 종류마다 다르다.

```
사건        → event.last_seen        (마지막 관측일)
기사        → article.published_at 의 날짜
시장지표    → market_point.observed_on
교차 도메인 → 그 변동의 file_base_date
day_diff = abs((후보 관측일 − anomaly.file_base_date).days)
```

기간 구분이 넓은 변동에서 값이 크게 나오는 것은 **보정하지 않는다.** 화면이 기간 구분을 함께 적어 읽는 사람이 감안한다.

**테스트 관점** 사건의 `first_seen`가 아니라 `last_seen`를 쓰는지(시작일을 쓰면 오래 이어진 사건이 구조적으로 멀어진다). `periodType=cumulative` 변동에서 값이 커지되 순서가 뒤집히지 않는지.

근거: [[TBL-API-002]] 1.4절 · [[TBL-DOM-006#event]]

#### ProximityCalculator.sort 후보 정렬

**시그니처** `sort(candidates: list[CauseCandidate]) -> list[CauseCandidate]`

**처리** 정렬 열쇠는 아래 한 줄이다.

```
key = (day_diff 오름차순, source_count 내림차순, article_count 내림차순, 동률 열쇠 사전순)
```

**네 열쇠 모두가 계약이다.** 앞의 셋은 화면이 문장으로 밝히는 규칙이고, 마지막 동률 열쇠는 셋이 모두 같을 때 순서를 끝까지 깨는 열쇠다. 동률 열쇠는 `{candidate_type}:{대상}`이다. 대상은 사건·기사·시장지표면 그 식별자이고, 다른 도메인 변동이면 그 변동의 `도메인:지표:기간 구분:비교 기준`이다. `candidate_id`를 쓰지 않는 이유는 그 안의 변동 식별자가 실행마다 새로 매겨져 재생성마다 순서가 흔들리기 때문이다(재생성 대조에서 확인). 이 열쇠가 없으면 같은 입력에서도 `sort_order`가 흔들려 재현이 깨지므로 구현에서 뺄 수 없다([[TBL-PRD-002#N3]]). 정렬 결과의 자리 번호가 `sort_order`가 되며 이것은 관련도 순위가 아니다.

**출력** 정렬된 목록. 화면은 다시 정렬하지 않는다.

**테스트 관점** 같은 목록을 순서만 섞어 두 번 넣어도 결과가 같은지. 날짜 차이가 같고 출처 수가 3과 5이면 5가 앞인지. 잘린 후보를 빼도 남은 것의 상대 순서가 같은지. **앞의 셋이 모두 같은 후보 다섯을 순서만 바꿔 열 번 넣어도 `sort_order`가 매번 같은지**(마지막 열쇠가 빠지면 여기서 흔들린다).

근거: [[TBL-PRD-002#R10]] · [[TBL-DOM-006#cause_candidate]]

#### ProximityCalculator.sort_rule_text 정렬 규칙 문장

**시그니처** `sort_rule_text() -> str`

**처리** 아래 한 문장을 그대로 돌려준다. 응답의 `candidateSortRule`과 화면 글자가 이 값을 쓴다. [[#ProximityCalculator.sort]]의 마지막 열쇠(`candidate_id` 사전순)는 동률을 깨는 장치라 이 문장에 넣지 않는다. 읽는 사람이 판단에 쓰는 규칙은 앞의 셋이다.

```
날짜 차이 오름차순 → 출처 수 내림차순 → 기사 수 내림차순
```

**테스트 관점** 반환 문자열이 [[TBL-DOM-006#c_report]]의 `candidate_sort_rule`에 그대로 복사되는지. [[#ProximityCalculator.sort]]의 실제 열쇠와 어긋나지 않는지(문장과 코드가 갈리면 화면이 거짓말을 한다).

근거: [[TBL-API-002]] 1.3절 · [[TBL-PRD-002#R26]]

#### TrafficLightJudge.judge 신호등 판정

**시그니처** `judge(anomaly: Anomaly, candidates: list[CauseCandidate]) -> TrafficLight`

**입력** 변동 한 건, 상한을 적용하기 전의 정렬된 후보 목록([[#CandidateSearcher.search]]의 셋째 값), 인스턴스의 `setting`. 기준을 채우는 사건이 날짜 차이 때문에 상한 밖으로 밀려도 신호등이 본다. 그 사건이 상한 밖이면 5단계가 그 사건 하나를 저장 목록 끝에 더하고 잘린 건수를 하나 줄인다(상한을 하나 넘을 수 있다). 신호등의 이유가 목록에서 보이게 하기 위해서다. **LLM 클라이언트를 인자로도 속성으로도 받지 않는다. 이 함수가 사는 `judgment/traffic_light.py`는 `adapters/`를 import 하지 않는다.**

**처리**
1. `anomaly.axis_type != 'country'`이면 → `level=notApplicable`, `label='판정 대상 아님'`, `basis=None`. 생산은 여기서 빠진다. **`none`('원인 미확인')으로 적지 않는다.** 후보를 찾지 못한 것과 처음부터 판정 대상이 아닌 것은 다른 사실이다.
2. 후보 중 `candidate_type='event'`인 것을 `sort_order` 오름차순으로 보며 아래 셋을 모두 채우는 첫 후보를 고른다.

```
article_count ≥ setting.min_article_count
source_count  ≥ setting.min_source_count
day_diff      ≤ setting.event_window_days
```

3. 그런 후보가 있고 노출이 확인됐으면 → `level=red`, `label='확인 필요'` · 있으나 노출 미확인이면 → `level=yellow`, `label='주의'` · 하나라도 못 채우면 → `level=none`, `label='원인 미확인'`이고 **목록에는 남는다.**
4. [[#TrafficLightJudge.basis]]로 값과 기준값을 쌍으로 담고 `setting_version`을 남긴다.

**출력** `TrafficLight(level, label, basis, settingVersion)`. `level`은 `red` `yellow` `none` `notApplicable` 넷이고 `label`은 '확인 필요' '주의' '원인 미확인' '판정 대상 아님'과 1대1이다([[TBL-API-002]] 4장 `TrafficLight`).

**예외** 없다. 후보가 비어도 `none`으로 정상 반환한다.

**테스트 관점** **`llm_enabled=False`로 돌린 배치와 신호등이 전건 일치하는지.** 색만 바꿔도 `label`이 함께 바뀌는지(색만으로 구분하지 않는다). 생산 변동이 국가 카드에 오르지 않고 `level=notApplicable`인지. 노출 미확인 국가가 RED로 올라가지 않는지. 임계값을 바꾸면 결과가 바뀌되 그때 `setting_version`이 행에 남는지.

근거: [[TBL-SEQ-002#SEQ-8]] · [[TBL-PRD-002#R16]] · [[TBL-INFRA-002#C19]] · [[TBL-DOM-006#watch_item]]

#### TrafficLightJudge.build_domain_status 도메인 상태 판정

**시그니처** `build_domain_status(domain: str, anomalies: list[Anomaly]) -> DomainStatus`

**입력** 도메인 하나(`production` `inventory` `sales`)와 그 도메인 변동 목록. 변동마다 [[#TrafficLightJudge.judge]]가 매긴 신호등과 A 판정에서 온 기여 분해가 이미 붙어 있다. **LLM 인자가 없고 인스턴스 속성으로도 받지 않는다. 도메인 상태 3카드의 신호등도 5단계 규칙 산출물이다.**

**처리**
1. `domain='production'`이면 → `trafficLight.level=notApplicable`, `label='판정 대상 아님'`. 생산은 국가 축이 없어 원인 후보를 받지 못한다([[TBL-INFRA-002#C16]]). **`none`('원인 미확인')으로 적지 않는다.**
2. 나머지 도메인은 도메인 안 변동 신호등의 **최댓값**을 쓴다. 센 것부터 `red` → `yellow` → `none`이고, 변동이 하나도 없으면 `none`이다.
3. `judgmentSource`를 정한다. [[#AJudgmentReader.read]]가 그 도메인 판정을 정상으로 읽었으면 `aJudgment` · 보완 집계로 내려갔으면 `supplementaryAggregate` · 접속 실패나 모양 검사 실패로 아무것도 못 받았으면 `notReceived`.
4. `topContributions`에 [[TBL-DOM-006#a_contribution]] 사본의 상위 항목을 자리 번호 순으로 담는다. **사본을 그대로 옮기고 여기서 다시 계산하지 않는다.** 보완 집계면 빈 목록이다.
5. `externalCauseAllowed`는 `domain='production'`이면 거짓, 나머지는 참이다.
6. `claim`은 비워 둔다. **문장은 8단계 [[#ClaimWriter.write_domain_status]]가 채운다.**

**출력** `DomainStatus(domain, domainLabel, trafficLight, claim, judgmentSource, externalCauseAllowed, topContributions)`. 도메인 셋이므로 세 건이 나오고 [[#ReportPublisher.publish]]가 [[TBL-DOM-006#c_report]]의 `domain_status`에 넣는다.

**예외** 없다. 변동이 0건이어도 세 건을 다 만든다. 카드가 사라지는 일은 없다.

**테스트 관점** `llm_enabled=False`로 돌린 배치와 세 카드의 신호등·판정 출처·기여 상위가 전건 일치하는지. 생산 카드가 `notApplicable`이고 `none`이 아닌지. 도메인 안에 RED 변동이 하나라도 있으면 카드가 RED인지. 브리핑 저장소를 막아 놓고 돌리면 세 카드의 `judgmentSource`가 `notReceived`이고 리포트 `notices`에 `aJudgmentNotReceived`가 도메인 셋으로 함께 담기는지.

근거: [[TBL-UI-002#UI-1]] · [[TBL-INFRA-002#C19]] · [[TBL-INFRA-002#C16]] · [[TBL-PRD-002#R16]]

#### TrafficLightJudge.build_watch_items 워치리스트 줄 만들기

**시그니처** `build_watch_items(anomalies: list[Anomaly]) -> list[WatchItem]`

**처리** `axis_type='country'`인 변동만 담는다. 정렬은 `신호등 순(red → yellow → none) → 대표 후보의 day_diff 오름차순 → country_code 오름차순`이고 그 자리가 `sort_order`가 된다. [[TBL-DOM-006#watch_item]]에 넣는다. 유일 제약은 `anomaly_id`.

**출력** 워치리스트 줄 목록. **5단계 산출물이다.** [[TBL-DOM-005#ReportPublisher]]로 바로 넘어가고 서술 계층을 거치지 않는다.

**테스트 관점** LLM을 끈 배치와 줄 순서가 같은지. 변동은 있는데 후보가 없는 국가가 "원인 미확인"으로 남는지. 법인 매핑이 비어 있어도 오류 없이 `entity_code=NULL`로 나가는지.

근거: [[TBL-SEQ-002#SEQ-8]] · [[TBL-SEQ-002#SEQ-14]] · [[TBL-PRD-002#R16]]

#### TrafficLightJudge.basis 신호등 근거

**시그니처** `basis(candidate: CauseCandidate | None) -> dict | None`

**처리** 기준을 넘긴 후보가 없으면 `None`. 있으면 값과 **그때 적용된 기준값**을 쌍으로 담는다. `candidate_id` `article_count` `article_count_min` `source_count` `source_count_min` `day_diff` `window_days` `exposure_confirmed`.

**테스트 관점** 나중에 설정을 바꾼 뒤 과거 리포트를 열어도 그때 기준값으로 설명되는지(기준값을 설정 테이블에서 조회하면 지금 기준으로 설명하게 된다). 화면에서 "기사 12건, 기준 5건"처럼 읽히는지.

근거: [[TBL-DOM-006#watch_item]] · [[TBL-PRD-002#R16]]

## 5. 서술 (배치 6~8단계)

이 장의 함수만 [[TBL-DOM-005#HChatClient]]를 부른다. 셋이 전부 실패해도 4장의 산출물은 그대로 게시된다([[TBL-INFRA-002#C13]]).

#### EventNamer.name_events 사건 명명

**시그니처** `name_events(events: list[Event]) -> int`

**입력** **후보로 뽑힌 사건 중 아직 명명되지 않은 것만.** 묶인 사건 전부가 아니다.

**처리**
1. 사건마다 상위 5건의 기사 제목과 출처만 담은 프롬프트를 만든다. 심각도 필드는 스키마에 없다.
2. [[#HChatClient.complete]](`purpose=naming`).
3. 응답이 스키마에 맞으면 → `event.title` `event.event_type`을 채우고 `named_by=llm` · 실패·콘텐츠 필터·JSON 위반·토큰 상한·백필 모드면 → [[#EventNamer.fallback_to_representative]] + [[#DegradeHandler.degrade]](`naming`).
4. **`article_count` `source_count` `max_impact`를 건드리지 않는다.**

**출력** 이름을 붙인 건수.

**테스트 관점** LLM 호출 수가 "후보로 뽑힌 사건 수"를 넘지 않는지. 이 단계를 통째로 막아도 [[#TrafficLightJudge.judge]] 결과와 후보 순서가 그대로인지. 명명 실패 사건의 제목이 비지 않고 대표 기사 제목인지.

근거: [[TBL-SEQ-002#SEQ-9]] · [[TBL-PRD-002#R6]]

#### EventNamer.fallback_to_representative 대표 기사 제목으로 내리기

**시그니처** `fallback_to_representative(event: Event) -> str`

**처리** 대표 기사 제목을 사건 이름으로 쓰고 `named_by=representativeArticle`로 남긴다. 이름이 없어도 사건은 후보로 쓰인다.

**테스트 관점** `named_by`가 화면 후보 상세에 그대로 나가는지(사람이 이름의 출처를 안다).

근거: [[TBL-API-002]] 4.5절

#### CauseLinkWriter.write 연관 설명

**시그니처** `write(anomaly: Anomaly, candidates: list[CauseCandidate], llm: LlmPort) -> CauseLink`

**처리**
1. 프롬프트에 변동 사실과 **후보 목록(식별자와 근접도 값)만** 담는다. 후보 밖의 사실과 원본 테이블을 넣지 않는다. 항목당 4KB 이하.
2. [[#HChatClient.complete]](`purpose=causeLink`). 응답 스키마에 등급·점수·순위 필드가 없다.
3. [[#CauseLinkWriter.extract_citations]] → [[#CitationVerifier.verify_cause_link]].
4. 통과 → [[TBL-DOM-006#cause_link]]에 넣는다. **기본키가 `anomaly_id`라 변동마다 하나다.**
5. 실패 → 검증 사유를 붙여 1회 재요청 · 다시 실패 → [[#DegradeHandler.degrade]](`causeLink`, `citationVerificationFailed`).
6. **신호등을 읽지도 바꾸지도 않는다.**

**출력** `CauseLink(text, citedCandidateIds, footnotes, verified, degraded, degradeReason)`. 강등이면 `text=None`.

**테스트 관점** 출력의 모든 인용이 입력 후보 안에 있는지. 설명에 쓰인 수치가 전부 입력 후보의 근접도 값이나 변동 사실과 같은지. 강등된 변동에서도 후보 목록과 근접도 값 넷이 화면에 남는지. 같은 변동에 설명이 둘 생기지 않는지.

근거: [[TBL-SEQ-002#SEQ-10]] · [[TBL-PRD-002#R11]] · [[TBL-PRD-002#R20]]

#### CauseLinkWriter.extract_citations 인용 뽑기

**시그니처** `extract_citations(text: str) -> list[str]`

**처리** 문단에서 인용한 후보 식별자를 뽑아 중복 없이 돌려준다. 모델이 응답 필드로 준 목록과 문단 본문에서 뽑은 것이 다르면 **둘의 합집합**을 검증에 넘긴다. 좁게 잡으면 검증을 빠져나가는 인용이 생긴다.

**테스트 관점** 본문에만 있고 필드에 없는 인용이 검증에 걸리는지.

근거: [[TBL-DOM-005#CauseLinkWriter]]

#### ClaimWriter.write_headline 헤드라인 생성

**시그니처** `write_headline(context: dict, llm: LlmPort) -> Claim`

**처리** 신호등이 붙은 국가 상위와 그 첫 후보, 그리고 **[[#ReportPublisher.number_evidence]]가 이미 매긴 각주 번호 목록**을 담아 부른다. 모델이 번호를 만들지 않는다. 헤드라인은 두 도메인 이상 또는 도메인과 외부 원인을 함께 언급한다.

**출력** `Claim(text, footnotes, highlights)`.

**테스트 관점** 각주 없는 주장이 0건인지. 헤드라인이 한 도메인만 말하고 끝나지 않는지.

근거: [[TBL-SEQ-002#SEQ-11]] · [[TBL-PRD-002#R21]]

#### ClaimWriter.write_domain_status 도메인 상태 생성

**시그니처** `write_domain_status(domain: str, context: dict, llm: LlmPort) -> Claim`

**처리** 생산·재고·판매 세 장의 **문장만** 쓴다. 신호등·판정 출처·기여 상위는 5단계 [[#TrafficLightJudge.build_domain_status]]가 이미 확정했고, 이 함수는 그 `DomainStatus`를 입력으로 받아 읽을 문장으로 옮길 뿐이다. **셋 중 어느 값도 바꾸지 않고 다시 계산하지도 않는다.** 종합 관점으로 새로 쓰고 A 리포트 문장을 복사하지 않는다. **`domain='production'`이면 프롬프트에 외부 후보를 아예 넣지 않는다.** 검증기로 뒤에서 거르지 않고 입력으로 막는다.

**출력** `Claim` 한 건. [[#ReportPublisher.publish]]가 `DomainStatus`의 `claim` 칸에 끼워 넣는다. 생산 장의 `externalCauseAllowed`는 언제나 거짓이고 그 값도 5단계가 정한 것이다.

**테스트 관점** 생산 문장에 뉴스·시장지표 인용이 0건인지. 도메인 상태 문장이 A 리포트 문장과 동일하지 않은지(표본 대조, 동일 비율 0%). **이 함수를 통째로 막아도 세 카드의 신호등·판정 출처·기여 상위가 그대로 게시되는지**(문장 칸만 템플릿으로 내려간다).

근거: [[TBL-INFRA-002#C16]] · [[TBL-PRD-002#R21]] · [[TBL-API-002]] 1.6절

#### ClaimWriter.write_card 변동 카드 문장 생성

**시그니처** `write_card(anomaly: Anomaly, context: dict, llm: LlmPort) -> Claim`

**처리** 변동 사실, A 판정의 내부 분해, 노출, 후보와 근접도 값, [[#CauseLinkWriter.write]]의 설명을 담아 카드 요약을 쓴다. 인과 단정 어휘와 예측 어휘, **등급을 뜻하는 어휘("관련도 높음", "강한 연관")를 금지 목록에 둔다.** 강조는 토큰으로만 표시하고 렌더는 화면이 한다(문장당 3개 이하).

**출력** `Claim`.

**테스트 관점** 금지 어휘가 나오면 재생성 또는 강등되는지. 카드 문장의 수치가 [[TBL-DOM-006#report_anomaly]]의 값과 같은지.

근거: [[TBL-PRD-002#R22]] · [[TBL-PRD-002#R23]] · [[TBL-PRD-002#R26]]

#### CitationVerifier.verify_cause_link 연관 설명 검증

**시그니처** `verify_cause_link(cause_link: CauseLink, candidates: list[CauseCandidate]) -> bool`

**처리** 인용한 식별자 집합이 그 변동의 후보 식별자 집합 안에 전부 들어 있는지 본다. 하나라도 벗어나면 거짓이고 [[#CitationVerifier.out_of_scope_ids]]가 벗어난 것을 돌려준다. 금지 어휘와 입력에 없는 수치도 함께 본다. **코드가 한다. 모델에게 검사시키지 않는다.**

**테스트 관점** 없는 후보 식별자를 심은 응답이 거짓으로 떨어지는지. 검증 자체가 예외로 죽지 않는지(집합 비교라 단순해야 한다).

근거: [[TBL-SEQ-002#SEQ-10]] · [[TBL-PRD-002#R11]]

#### CitationVerifier.verify_claim 문장 검증

**시그니처** `verify_claim(claim: Claim, evidence: list[Evidence], context: dict) -> bool`

**처리** 각주 번호가 전부 근거 목록의 `number` 안에 있는지 · [[#CitationVerifier.has_forbidden_words]]가 빈 목록인지 · 숫자·날짜·국가명이 `context`에 있는 값인지 · 강조 토큰 외 마크업이 없는지 · A 리포트 문장과 동일하지 않은지 · 생산 문장에 외부 원인 인용이 없는지를 본다. 하나라도 어긋나면 거짓이다.

**테스트 관점** 근거 목록에 없는 각주 번호를 쓴 문장이 걸리는지. A 리포트 문장을 그대로 복사한 도메인 상태가 걸리는지.

근거: [[TBL-SEQ-002#SEQ-11]] · [[TBL-PRD-002#N4]]

#### CitationVerifier.out_of_scope_ids 범위 밖 인용

**시그니처** `out_of_scope_ids(cited: list[str], allowed: list[str]) -> list[str]`

**처리** `cited − allowed` 차집합을 돌려준다. 강등 사유에 그대로 담긴다.

**테스트 관점** 빈 목록이면 검증이 통과인지. 순서에 의존하지 않는지.

근거: [[TBL-DOM-005#CitationVerifier]]

#### CitationVerifier.has_forbidden_words 금지 어휘 검사

**시그니처** `has_forbidden_words(text: str) -> list[str]`

**처리** 문장에서 인과 단정 어휘, 예측 어휘, 등급을 뜻하는 어휘("관련도 높음" "강한 연관" "확실히" 등)를 찾아 걸린 어휘 목록을 돌려준다. 빈 목록이면 통과다. 목록은 코드 상수이고 설정이 아니다. [[#CitationVerifier.verify_cause_link]]와 [[#CitationVerifier.verify_claim]]이 부른다.

**테스트 관점** 금지 목록의 어휘 하나를 심은 문장이 걸리는지. 걸린 어휘가 강등 사유에 그대로 담기는지.

근거: [[TBL-PRD-002#R22]] · [[TBL-UC-002#UC-S7]] 3

#### DegradeHandler.degrade 강등 기록

**시그니처** `degrade(role: str, reason: str, target: str | None) -> None`

**처리** `role`은 `naming` `causeLink` `narration` 셋. `reason`은 `llmFailure` `contentFilter` `jsonViolation` `citationVerificationFailed` `backfillMode` 다섯. [[TBL-DOM-006#batch_run]]의 `degraded_roles`와 해당 산출 행의 강등 칸에 남긴다. **예외를 던지지 않는다.** 던지면 호출부 하나만 빠뜨려도 배치가 멈춘다.

**출력** 없다. 배치는 계속된다.

**테스트 관점** 1~5단계에서 이 함수를 부르는 자리가 코드에 없는지(강등은 6~8단계에만 생긴다). 세 역할이 전부 강등돼도 배치가 `status=degraded`로 끝나고 게시가 되는지.

근거: [[TBL-INFRA-002#C13]] · [[TBL-PRD-002#N2]]

#### DegradeHandler.fill_template 템플릿 문장 채우기

**시그니처** `fill_template(anomaly: Anomaly, candidates: list[CauseCandidate]) -> Claim`

**처리** 서술이 강등됐을 때 변동 사실, 후보 제목, 근접도 값 넷을 고정 템플릿에 채우고 각주를 후보 `sort_order` 순으로 기계 부여한다. LLM을 부르지 않는다. 템플릿 채우기도 실패하면 문장 없이 `body`를 비우고 `is_degraded=true`로 돌려준다([[TBL-UC-002#UC-S11]] 3a). [[#ReportPublisher.publish]]가 결과를 [[TBL-DOM-006#report_claim]]에 넣는다.

**출력** `Claim(body, footnoteNumbers, emphasis, isDegraded=true)`.

**테스트 관점** 강등된 카드의 각주 번호가 후보 순서와 같은지. 템플릿 문장에 등급 어휘가 없는지. LLM 호출이 0회인지.

근거: [[TBL-SEQ-002#SEQ-11]] · [[TBL-UC-002#UC-S11]] · [[TBL-INFRA-002#C13]]

#### DegradeHandler.degraded_roles 강등 역할 목록

**시그니처** `degraded_roles(batch_run_id: str) -> list[str]`

**처리** 그 배치에서 강등된 역할을 중복 없이 돌려준다. [[#ReportPublisher.publish]]가 리포트의 `is_degraded`와 `degraded_roles`에 싣는다.

**테스트 관점** 같은 역할이 열 번 강등돼도 목록에 한 번만 나오는지.

근거: [[TBL-SEQ-002#SEQ-11]]

## 6. 게시 (8단계)

#### ReportPublisher.number_evidence 각주 번호 매기기

**시그니처** `number_evidence(anomalies: list[Anomaly], candidates: list[CauseCandidate]) -> list[Evidence]`

**처리** **8단계 맨 앞에서 [[TBL-DOM-005#ClaimWriter]]보다 먼저 돈다.** 순서는 결정적이다. 변동을 워치리스트 `sort_order` 순으로 돌고, 변동 안에서 후보를 `sort_order` 순으로 돌며 1부터 번호를 매긴다. `kind`는 `candidate` `causeLink` `anomaly` `contribution` `marketMetric` `article` `aJudgment` 일곱.

**출력** [[TBL-DOM-006#report_evidence]] 행 목록. 기본키는 `(report_id, number)`.

**테스트 관점** 같은 입력이면 같은 번호가 나오는지. 문장 생성보다 먼저 도는지(뒤집히면 없는 각주가 생긴다). 번호에 빠진 수가 없는지.

근거: [[TBL-SEQ-002#SEQ-11]] · [[TBL-PRD-002#N4]]

#### ReportPublisher.copy_display_values 표시값 복사

**시그니처** `copy_display_values(evidence: list[Evidence]) -> None`

**처리** 근거 행에 표시용 값 사본(`display_value`)과 기사 원문 `url`을 복사해 둔다. **열람할 때 조인이 없게 하기 위해서다.** `mart`를 참조로 두지 않는다.

**테스트 관점** 열람 응답 생성에 `mart` 스키마 조회가 0회인지. 근거 패널을 펼칠 때 추가 호출이 없는지.

근거: [[TBL-INFRA-002#C10]] · [[TBL-PRD-002#N5]]

#### ReportPublisher.publish 리포트 게시

**시그니처** `publish(batch_run: BatchRun, base_date: date) -> CReport`

**처리**
1. 같은 기준일의 다음 버전 번호를 매긴다.
2. [[TBL-DOM-006#report_anomaly]] [[TBL-DOM-006#report_watch_item]] [[TBL-DOM-006#report_claim]] [[TBL-DOM-006#report_evidence]] [[TBL-DOM-006#report_claim_evidence]]에 **참조가 아니라 복사**로 넣는다. `axis_type='plant'`인 변동은 게시하지 않는다.
3. 서술이 강등됐으면 → [[#DegradeHandler.fill_template]]이 채운 템플릿 문장을 넣는다 · 정상이면 → 검증을 통과한 문장과 각주를 넣는다.
4. [[#TrafficLightJudge.build_domain_status]]가 만든 도메인 셋의 `DomainStatus`에 [[#ClaimWriter.write_domain_status]]의 문장을 끼워 [[TBL-DOM-006#c_report]]의 `domain_status`에 넣는다. **신호등·판정 출처·기여 상위는 5단계 값을 그대로 복사하고 여기서 다시 계산하지 않는다.** 서술이 강등됐으면 문장 칸만 템플릿으로 채우고 나머지 셋은 그대로 간다.
5. 소스별 최신일 셋을 각각 채운다. `vehicle_latest_date`는 완성차 적재 파일 기준일의 최대, `news_latest_date`는 기사 관측일의 최대, `market_latest_date`는 시장지표 `observed_on`의 최대다. `data_latest_date`는 **그 셋 중 가장 늦은 것**이고 따로 세지 않는다. 소스 하나가 비면 그 칸은 `NULL`이고 남은 칸들로 `data_latest_date`를 정한다.
6. `notices`를 모아 채운다. 코드는 다섯이 전부이고 화면 예외 상태와 1대1이다([[TBL-UI-002#UI-1]]).

```
noAnomaly             게시할 변동이 0건일 때           (5단계, 리포트 전체)
generationDegraded    degraded_roles가 비지 않을 때     (6~8단계, 리포트 전체와 카드 머리)
aJudgmentNotReceived  미수신 도메인마다 한 건           (2단계, domain 칸을 채운다)
batchFailed           직전 배치가 1·3·4·5단계에서 멈춰 이전 게시본을 유지했을 때 (lastSuccessDate 칸을 채운다)
marketCarriedOver     is_carried_over가 참인 지표마다 한 건 (시장지표 바)
```

각 건은 `code` `domain` `message` `lastSuccessDate` 네 칸이다. 입력은 [[#AJudgmentReader.read]]의 미수신 목록, [[#DegradeHandler.degraded_roles]], [[#MarketService.is_carried_over]] 결과 셋이다. `batchFailed`는 게시 때 넣지 않는다. 정기 배치가 1·3·4·5단계에서 멈추면 러너가 그때 보이는 게시본의 안내에 `batchFailed` 한 줄을 붙이고(같은 줄은 바꿔 넣는다), 새 게시본이 나오면 그 게시본이 대신 보인다. `domain`은 `aJudgmentNotReceived`에만, `lastSuccessDate`는 `batchFailed`에만 채운다. 변동이 있으면 `noAnomaly`를 넣지 않는다. 첫 배치 전(게시본 없음)은 조회 0건으로 화면이 판단하므로 대응 코드가 없다.
7. `is_degraded` `degraded_roles` `candidate_sort_rule` `setting_version` `a_snapshot_id`를 채운다. `candidate_sort_rule`은 [[#ProximityCalculator.sort_rule_text]]가 준 한 문장이고 **리포트에 한 칸뿐이다.** 변동 카드마다 같은 문장을 되풀이해 싣지 않는다.
8. `market_bar` 네 칸을 채운다. **지표가 이월됐어도 네 칸을 다 내려보내고 칸을 비우지 않는다.** 이월 사실은 `notices`의 `marketCarriedOver`가 말한다.
9. 지표 시계열을 [[TBL-DOM-006#market_series]]에 사본으로 올리고, [[#BriefingStoreReader.read_report_document]]로 도메인 셋의 A 리포트를 읽어 [[#ReportPublisher.publish_a_report]]로 올린다. **열람 경로가 `mart`를 보지 않게 하는 마지막 단계다**([[TBL-INFRA-002#C10]]). 한 도메인을 못 읽어도 배치를 이어 가고 그 도메인 화면은 옛 사본을 계속 보여 준다.
10. [[#ReportPublisher.diff_watchlist]].
11. 정기 배치는 `is_published=true`, 재생성은 `is_published=false`로 만들고 사람이 [[#BatchService.publish_version]]으로 전환한다. **기존 게시본을 덮지 않는다.** 백필 모드면 이 함수를 부르지 않는다.

**출력** `CReport(reportId, versionNo)`.

**테스트 관점** 서술 셋이 전부 강등돼도 게시되는지, 그리고 그때 신호등·도메인 상태 3카드·후보 순서가 5단계 산출물과 전건 같은지. 같은 기준일에 `is_published=true`인 행이 언제나 하나인지(부분 유일 제약). 소스 하나가 비어도 `data_latest_date`가 남은 칸 중 가장 늦은 날인지. `notices`의 `code`가 다섯 밖으로 나가지 않는지. 변동 0건 배치에서 `noAnomaly` 한 건이 담기고 변동이 있는 배치에서는 담기지 않는지. `market_bar`가 비어 있는 게시본이 0건인지.

근거: [[TBL-SEQ-002#SEQ-11]] · [[TBL-SEQ-002#SEQ-3]] · [[TBL-INFRA-002#C13]]

#### ReportPublisher.publish_a_report A 리포트 사본 게시

**시그니처** `publish_a_report(domain: str, document: dict) -> str`

**입력** [[#BriefingStoreReader.read_report_document]]가 뷰 `tbl_a_report`에서 읽어 온 A 리포트 한 건(대시보드 식별자·이름, 스냅샷 식별자, 요약, 해설, 트래킹 지표, 판정 목록, 각주 근거, 버전 번호, 게시 시각, 강등 여부)과 [[#ReportPublisher.build_domain_blocks]]가 만든 분해 블록. 고정 문구·못 만드는 지표·제약 경고·데이터 형태는 이 시스템의 고정표와 적재 이력에서 채운다(유저 결정 2026-10-01). 판정 목록이 비면 빈 배열로 넣는다. 이 칸의 정본은 `mart`의 판정과 기여이고 여기 담기는 것은 A 리포트 화면이 읽는 사본이다. **C 생성은 이 칸을 읽지 않는다.**

**처리**
1. [[TBL-DOM-006#a_report_snapshot]]에 한 행을 넣는다. 유일 제약이 `(domain, base_date, published_version)`이라 같은 버전을 두 번 받아도 한 행이다.
2. 각주 근거를 [[TBL-DOM-006#a_report_evidence]]에 `number` 순으로 넣는다. 근거가 0건이어도 스냅샷은 성립한다.
3. `base_date` `batch_run_id` `copied_at`을 남긴다. 원본이 나중에 바뀌어도 그날 화면은 이 사본을 읽는다.
4. **문장을 새로 쓰거나 고치지 않는다. 받은 글자를 그대로 옮긴다.** 고정 문구는 LLM 없는 고정표에 설정 값을 채운 이 시스템의 글이다.

**출력** `a_report_snapshot_id`. 읽어 온 문서가 없으면 아무것도 넣지 않고 빈 문자열을 돌려준다.

**예외** 없다. 한 도메인이 실패해도 배치를 멈추지 않는다.

**이 사본은 [[#AJudgmentReader.copy_snapshot]]이 만드는 판정 사본과 다른 테이블이다.** 판정 사본([[TBL-DOM-006#a_judgment]] [[TBL-DOM-006#a_contribution]])은 C 리포트를 만들려고 두는 것이라 문장을 담지 않는다. 이 사본은 A 리포트 화면이 그대로 다시 뿌리려고 두는 것이라 문장을 담는다. 두 길은 섞이지 않는다.

**테스트 관점** 사본의 요약·해설 글자가 브리핑 갈래 원본과 한 글자도 다르지 않은지. 분해 블록 값이 같은 기준일 C 리포트의 단계 흐름·달성률과 같은지. 같은 버전을 두 번 올려도 행이 하나인지. [[TBL-DOM-006#a_judgment]]에 문장 컬럼이 0개인지. 사본이 있는 도메인의 화면 조회에 `mart` 접근이 0회인지.

근거: [[TBL-INFRA-002#C10]] · [[TBL-UI-002#UI-2]] · [[TBL-SEQ-002#SEQ-2]]

#### ReportPublisher.build_domain_blocks 도메인 분해 블록

**시그니처** `build_domain_blocks(domain: str, base_date: date) -> dict`

**처리** A 리포트 화면의 표와 막대를 이 시스템 데이터로 만든다(유저 결정 2026-10-01). 브리핑 갈래에는 차원별 값이 없다. 값은 그 기준일을 판정한 실행의 `mart`와 `std`에서 꺼내며 C 리포트가 쓰는 것과 같은 계산이다. LLM을 부르지 않는다.
- `production`: 누적 사업계획·실적·달성률(계획 정본은 설정), 다른 계획과의 차이(`planAlternativeDiff`), 법인별 달성률([[#FactJoiner.join]]의 공장 축 집계와 같은 집계), 완성차·반조립 비중과 내수·수출 비중(표준 행의 구분 칸 합).
- `inventory`: 미주 도매 누계·소매 누계·격차·격차율([[TBL-DOM-006#sales_stage_flow]] 합), `derivationType`(흐름 행의 `derivation`), 국가별 체류(딜러 구간 비율 내림차순), 재고 회전 상태는 원천이 없어 `unstable`, 자리 지표는 고정표.
- `sales`: 단계별 흐름(네 계열과 두 구간의 합, 도매 정본과 대체 차이), 국가별 흐름(`countryFlows`. 열람이 부호로 쌓이는 쪽과 덜어내는 쪽으로 나눈다), 격차가 가장 큰 국가의 차종그룹 분해(표준 행을 차종그룹으로 집계한 도매·소매·격차 상위).
블록 키 이름은 [[TBL-API-002]] 4장 ProductionExtra·InventoryExtra·SalesExtra를 그대로 쓴다. 재료가 없는 블록은 빈 칸으로 두고 예외를 내지 않는다.

**출력** `{blocks, dimensions, measureComposition}`. `dimensions`는 분해 차원과 값 개수(표준 행에서 센다), `measureComposition`은 적재 이력의 형태와 기간 구분으로 만든 글이다.

**예외** 없다. 판정 실행이 없으면 빈 dict다.

**테스트 관점** 같은 기준일 C 리포트의 단계 흐름·법인별 달성률과 블록 값이 소수점까지 같은지. 재료가 없는 도메인에서 예외 없이 빈 블록이 나오는지. 이 함수가 H-chat을 부르지 않는지. 브리핑 갈래 문서가 없을 때는 불리지 않는지(사본이 없으면 블록도 없다).

근거: [[TBL-PRD-002#R1]] · [[TBL-UC-002#UC-H1]] 2 · [[TBL-SEQ-002#SEQ-11]] · [[TBL-API-002]] 4장

#### ReportPublisher.diff_watchlist 워치리스트 변화 비교

**시그니처** `diff_watchlist(current: list[WatchItem], previous: list[WatchItem]) -> list[AlertEvent]`

**처리** 어제 없던 국가가 올라오면 `new` · 신호등이 올라갔으면 `raised` · 내려갔으면 `lowered` · 목록에서 빠졌으면 `dropped`. [[TBL-DOM-006#alert_event]]에 넣는다. 유일 제약 `(report_id, country_code)`에 기대 **upsert로 넣어 같은 리포트에 두 번 돌아도 한 행이 되게 한다.** 8단계 게시와 게시 전환 양쪽에서 돌기 때문이다.

**출력** 변화 목록. 발송 채널이 없어도 실패하지 않는다. 기록과 배지는 남는다.

**테스트 관점** 같은 리포트에 두 번 돌려도 알림이 두 벌 남지 않는지. 첫 리포트(직전 게시본 없음)에서 전건 `new`인지.

근거: [[TBL-SEQ-002#SEQ-14]] · [[TBL-SEQ-002#SEQ-12]] · [[TBL-PRD-002#R24]]

## 7. 열람

열람 경로에 집계·조인·LLM이 없다([[TBL-INFRA-002#C9]] [[TBL-INFRA-002#C10]]).

#### CReportService.get_latest 최신 C 리포트 조회

**시그니처** `get_latest() -> CReport`

**처리** [[TBL-DOM-006#c_report]]의 부분 인덱스로 게시본 1건을 찾고, 그 리포트의 [[TBL-DOM-006#report_anomaly]] [[TBL-DOM-006#report_watch_item]] [[TBL-DOM-006#report_claim]] [[TBL-DOM-006#report_evidence]] [[TBL-DOM-006#alert_event]]를 읽어 한 덩어리로 돌려준다. 표시값이 복사돼 있어 조인이 없다. 후보와 기사가 첫 응답에 이미 들어 있어 근거 펼치기가 서버를 다시 부르지 않는다.

**출력** `marketBar` 4, `headline`, `domainStatus` 3, `anomalies`, `evidence`, `notices`.

**예외** 게시본이 없으면 `/problems/no-published-report` 404(`nextScheduledAt`). **나머지 예외(변동 없음·강등·판정 미수신·지표 이월)는 200으로 내려가 `notices`에 담긴다.**

**테스트 관점** p95 2초 안에 끝나는지. 호출 중 H-chat 호출이 0회인지. `anomalies`에 `axisType='plant'`가 0건인지.

근거: [[TBL-SEQ-002#SEQ-1]] · [[TBL-API-002#GET/api/intel/creport/latest]]

#### CReportService.get_by_id 특정 C 리포트 조회

**시그니처** `get_by_id(report_id: str, viewer: Viewer) -> CReport`

**처리** 현업 서명 주소로 미게시본을 부르면 **404**다. 존재 여부를 알려 주지 않기 위해 403이 아니다. 관리자 세션이면 미게시본도 돌려준다.

**테스트 관점** 현업 서명 주소로 미게시 리포트를 부르면 404인지. 관리자면 200인지.

근거: [[TBL-API-002#GET/api/intel/creport/version/{reportId}]] · [[TBL-SEQ-002#SEQ-1]]

#### DomainReportService.get_latest_by_domain A 리포트 조회

**시그니처** `get_latest_by_domain(domain: str) -> DomainReport`

**처리** `production` `inventory` `sales` 중 하나의 최신 [[TBL-DOM-006#a_report_snapshot]]과 그 [[TBL-DOM-006#a_report_evidence]]를 읽고 [[#DomainReportService.build_domain_extra]] [[#DomainReportService.list_missing_metrics]]를 붙인다. **조회 대상은 게시 스키마 사본뿐이다.** 요약·해설·고정 문구·각주 근거·버전 번호는 [[#ReportPublisher.publish_a_report]]가 옮겨 둔 글자를 그대로 내보내고 **이 시스템이 문장을 새로 쓰지 않는다**([[TBL-INFRA-002#C10]]). **응답 어디에도 뉴스·시장지표 인용이 없고 C 리포트로 가는 링크 필드도 없다.**

**출력** `DomainReport(summary, commentary, evidence, versions, trackingMetrics 3, breakdownDimensions, missingMetrics, domainExtra)`.

**예외** 없다. 그 도메인 사본이 없으면 404가 아니라 `summary`·`commentary`·`evidence`를 비우고 리포트 문장 미수신을 표시해 돌려준다([[TBL-UC-002#UC-H1]] 4a). 그때 판정값과 차트를 `pub` 어디서 읽을지는 미결이다(10장).

**테스트 관점** 응답 JSON에 기사·시장지표 계열 키가 0건인지. A1 응답의 축이 `plant`인지. 응답 생성에 `mart`·`std` 조회가 0회인지. 화면 버전 알약과 사이드 버전 목록의 번호가 사본의 `published_version`과 같은지.

근거: [[TBL-SEQ-002#SEQ-2]] · [[TBL-PRD-002#R1]]

#### DomainReportService.get_by_id A 리포트 특정 버전 조회

**시그니처** `get_by_id(domain_report_id: str) -> DomainReport`

**처리** 버전 하나를 그대로 읽는다. 없으면 404. 재계산하지 않는다.

**테스트 관점** 같은 버전을 두 번 불러 값이 같은지.

근거: [[TBL-API-002#GET/api/intel/areport/version/{domainReportId}]]

#### DomainReportService.list_versions A 리포트 버전 목록

**시그니처** `list_versions(domain: str) -> list[dict]`

**처리** 그 도메인의 [[TBL-DOM-006#a_report_snapshot]]을 `base_date` 내림차순, `published_version` 내림차순으로 읽어 `(domainReportId, baseDate, publishedVersion, publishedAt)` 목록을 돌려준다. 사이드의 버전 목록이 이 값이다. 번호는 브리핑 갈래가 매긴 것 그대로다.

**테스트 관점** 목록 첫 건이 [[#DomainReportService.get_latest_by_domain]]이 돌려준 본과 같은지. `pub` 밖 조회가 0회인지.

근거: [[TBL-UC-002#UC-H1]] 5 · [[TBL-SEQ-002#SEQ-2]]

#### DomainReportService.build_domain_extra 도메인별 덩어리

**시그니처** `build_domain_extra(domain: str, report: DomainReport) -> dict`

**처리** 도메인마다 다른 블록을 만든다. A1은 법인·공장·차종 분해, A2는 단계별 흐름 유도값과 유도 표기, A3는 단계별 흐름과 계획 대비 진도율·전년 동월 대비다. **값은 전부 [[TBL-DOM-006#a_report_snapshot]]의 `breakdowns` 사본에서 꺼낸다. `mart`와 `std`를 한 번도 읽지 않는다**([[TBL-INFRA-002#C10]]). [[#DomainReportService.get_latest_by_domain]]과 같은 계약이다.

단계 흐름 값이 사본에 실려 있어야 하므로 [[#ReportPublisher.build_domain_blocks]]가 게시 때 네 계열(선적·실 도매·도매(공식)·소매)과 두 구간(`entityStageGap` 법인 구간 · `dealerStageGap` 딜러 구간)을 `blocks`에 넣는다. 열람은 그 값을 그대로 옮기고 여기서 다시 빼거나 더하지 않는다(유저 결정 2026-10-01).

**테스트 관점** A2 응답에 유도값임이 표시되는지. A3의 트래킹 지표 셋이 A1과 다른지. 이 함수가 도는 동안 `mart`·`std` 조회가 0회인지. 사본의 `breakdowns`에 네 계열과 두 구간이 다 들어 있는지(하나라도 비면 단계별 흐름 블록이 빈 칸으로 나간다).

근거: [[TBL-PRD-002#R1]] · [[TBL-API-002]] 4.12절

#### DomainReportService.list_missing_metrics 못 만드는 지표

**시그니처** `list_missing_metrics(domain: str) -> list[MissingMetric]`

**처리** 데이터 구조상 만들 수 없는 지표를 이유와 함께 돌려준다. **빈 자리를 추정값으로 채우지 않는다.** 배정은 아래로 고정한다.

```
A1 생산  destinationCountry · powertrainBreakdown                              (둘)
A2 재고  afloat · awaitingShipment · entityVsDealer · countryDailySeries        (넷)
A3 판매  globalCountryByModel                                                   (하나)
```

**출력** `MissingMetric(code, label, reason, willFillWhen)` 목록.

**테스트 관점** A1에 목적지 국가, A2에 항해중·선적대기가 이유와 함께 나오는지. 건수가 A1 둘·A2 넷·A3 하나인지. `countryDailySeries`가 A3가 아니라 **A2**에 있는지(국가 일별 시계열은 재고 쪽에서 요구된 것이다). `code`가 [[TBL-API-002]] 4.13절 enum 밖으로 나가지 않는지.

근거: [[TBL-PRD-002#R1]] · [[TBL-API-002]] 4.13절

#### DomainReportService.render_pdf A 리포트 PDF 지면

**시그니처** `render_pdf(report: DomainReport) -> tuple[bytes, str]`

**처리** [[#DomainReportService.get_by_id]]가 돌려준 한 본을 화면과 같은 순서(머리, 트래킹 지표, 요약, 해설, 도메인별 분해, 제약 경고, 못 만드는 지표, 근거, 고정 문구)의 HTML 한 장으로 옮기고 WeasyPrint로 PDF를 만든다. 막대는 서버가 SVG로 그린다. **사본 글자를 그대로 쓰고 문장을 새로 만들지 않는다.** 각주 표기 `[n]`만 위첨자로 바꾼다. 화면의 색 규칙(법인 달성률 주의선)은 옮기지 않고 값을 적는다. 외부 주소·글꼴·그림을 싣지 않는다. 저장하지 않고 LLM을 부르지 않는다. 같은 발주처 브리핑 갈래의 방식이다(유저 결정 2026-09-30).

**출력** `(PDF 바이트, 파일 이름)`. 파일 이름은 `areport-{domain}-{baseDate}-v{version}.pdf`다.

**예외** 없다. 버전이 없으면 앞선 `get_by_id`가 404를 낸다.

**테스트 관점** 세 도메인의 HTML에 사본의 해설·근거·못 만드는 지표·고정 문구가 글자 그대로 있는지. `[1]`이 남지 않고 외부 주소(`http`)와 스크립트가 없는지. 실제 PDF가 `%PDF`로 시작하고 한글 글꼴(나눔고딕)이 박히는지(WeasyPrint 시스템 라이브러리가 있는 이미지에서 돈다). 내려받기 응답이 `application/pdf` 첨부이고 `Cache-Control: no-store`인지.

근거: [[TBL-API-002#GET/api/intel/areport/pdf/{domainReportId}]] · [[TBL-SEQ-002#SEQ-2]] · [[TBL-UC-002#UC-H1]] 6

#### MarketService.get_series 시장지표 시계열 조회

**시그니처** `get_series(indicator_id: str, days: int = 30) -> list[MarketMetric]`

**처리** 게시 스키마 사본 [[TBL-DOM-006#market_series]]에서 최근 `days`일 값을 날짜 오름차순으로 돌려준다. 이월된 값은 `carriedOver=true`로 표시한다. **열람 경로는 `std`의 [[TBL-DOM-006#market_point]]를 보지 않는다**([[TBL-INFRA-002#C10]]). 사본은 [[#ReportPublisher.publish]]가 8단계에 올린다. 같은 클래스의 [[#MarketService.as_of]]는 4단계 파이프라인 함수라 원본을 읽으며, 이 둘은 조회 대상이 다르다.

**테스트 관점** 지표 4종(`GPR` `BRENT` `BDI` `SCFI`)이 전부 조회되는지. 값이 비어도 `asOfDate`가 내려가는지. 이 함수 실행 중 `mart` 조회가 0회인지.

근거: [[TBL-API-002#GET/api/intel/market/series/{indicatorId}]] · [[TBL-SEQ-002#SEQ-1]]

#### MarketService.as_of 기준일 값 조회

**시그니처** `as_of(indicator_id: str, on_date: date) -> MarketMetric | None`

**처리** 그 날짜 이하의 가장 최근 값을 돌려준다. **파이프라인 함수다.** [[#FactJoiner.attach_market_asof]]가 4단계에서 부른다. 이월 한도(7일, 상수)를 넘으면 `None`.

**테스트 관점** 주말·공휴일 날짜로 불러도 직전 영업일 값이 나오는지. 한도 초과에서 예외가 아니라 `None`인지.

근거: [[TBL-SEQ-002#SEQ-7]] · [[TBL-SEQ-002#SEQ-7]]

#### MarketService.is_carried_over 이월 여부

**시그니처** `is_carried_over(point: MarketMetric, on_date: date) -> bool`

**처리** 지표의 기준일이 조회 날짜보다 이르면 참이다. 참이면 화면이 "값 이월"을, 한도까지 넘으면 "기준일 지연"을 적고 **판정에 쓰지 않는다.**

**테스트 관점** 이월 지표가 [[#CandidateSearcher.search_market]]의 후보에서 빠지는지.

근거: [[TBL-PRD-002#R27]] · [[TBL-API-002]] 4.9절

## 8. 관리

#### BatchService.get_status 배치 상태 조회

**시그니처** `get_status() -> dict`

**처리** 최근 [[TBL-DOM-006#batch_run]]과 **자동 실행을 막고 있는 것**(`auto_run_blocked=true`인 [[TBL-DOM-006#ingest_file]])을 함께 돌려준다. 실행 중 여부와 worker가 남긴 [[#BriefingStoreReader.ping]] 결과([[TBL-DOM-006#briefing_store_check]]. `briefingStore.reachable` `checkedAt` `lastSnapshotId`)도 담는다. web은 브리핑 저장소에 붙지 않는다([[TBL-INFRA-002#C5]]). 판정을 한 건도 읽지 않는 가벼운 확인이라 실패해도 이 함수는 200이다.

**테스트 관점** 회귀 차단이 걸려 있을 때 무엇이 막고 있는지가 파일 단위로 나오는지.

근거: [[TBL-API-002#GET/api/admin/batch/status]] · [[TBL-SEQ-002#SEQ-12]]

#### BatchService.list_runs 배치 이력 목록

**시그니처** `list_runs(filters: dict, cursor: str | None) -> tuple[list[BatchRun], str | None]`

**처리** 기준일·상태·트리거로 거르고 `started_at` 내림차순 커서 페이징.

**테스트 관점** `llm_enabled=false`인 실행이 목록에서 구분되는지(회귀 대조를 그 행으로 찾는다).

근거: [[TBL-API-002#GET/api/admin/batch/history]]

#### BatchService.get_run 배치 상세 조회

**시그니처** `get_run(batch_run_id: str) -> BatchRun`

**처리** [[TBL-DOM-006#batch_stage_result]] **여덟 행**과 [[TBL-DOM-006#llm_call]] 집계를 함께 돌려준다. 단계 이름은 [[#PipelineRunner.stage_names]]가 준 것을 쓴다. 대기 중이라 단계 행이 아직 없으면 시작 단계 앞은 `skipped`, 나머지는 `pending`으로 채운다.

재실행·재생성이면 당시와의 비교 표를 붙인다. **비교 표와 LLM 없는 실행 대조는 worker가 실행을 마칠 때 계산해 [[TBL-DOM-006#batch_run]]의 `comparison` `llm_free_comparison`에 둔 값을 그대로 읽는다.** web은 `mart`를 읽지 않는다([[TBL-INFRA-002#C10]]). 비교 기준은 요청 때 고정한 `baseline_batch_run_id`이고, 그 실행이 판정을 끝내지 못했으면 비교 표를 내지 않는다. 끝나지 못한(`failed`) 실행은 비교하지 않는다. 판정을 직접 만들지 않은 실행(시작 단계 6 이상)은 앞서 5단계를 끝낸 실행의 판정을 쓴 것으로 본다. 실행마다 식별자가 달라 자연 열쇠로 맞춘다. 변동은 `도메인 국가(또는 공장) 지표 기간 비교기준`, 후보는 `종류:대상`이고 다른 도메인 변동 후보의 대상은 그 변동의 열쇠로 바꾼다. 원인은 기준 실행이 끝난 뒤 바뀐 입력으로 정하고 매핑, 설정, 늦게 온 데이터, A 판정 순이다. 문장은 두 실행의 게시 사본(헤드라인, 변동별 연관 설명)만 비교하고 위반이 아니다. LLM 없는 실행 대조는 같은 기준일·같은 설정 버전·같은 A 판정 스냅샷의 반대쪽 실행과만 한다.

**테스트 관점** 단계가 언제나 여덟인지. 6~8단계에만 `degrade_reason`이 붙는지.

근거: [[TBL-API-002#GET/api/admin/batch/run/{batchRunId}]]

#### BatchService.request_rerun 배치 재실행

**시그니처** `request_rerun(base_date: date, start_stage: int, options: dict) -> BatchRun`

**처리**
1. `start_stage`가 1~8 밖이면 → `/problems/stage-out-of-range` 400. 자동 실행이 막혀 있으면 → `/problems/regression-blocked` 409. 같은 기준일 배치가 대기 중이거나 돌고 있으면 → `/problems/batch-in-progress` 409.
2. **실행하지 않고 대기 행을 남긴다.** [[TBL-DOM-006#batch_run]]에 `status=queued` 행을 넣는다. `trigger`는 시작 단계 1이면 `regenerate`, 아니면 `rerun`이다. web에는 LLM 키와 브리핑 저장소 접속이 없고 worker는 한 번에 하나만 돌기 때문이다([[TBL-INFRA-002]] 4장).
3. **당시 입력과 비교 기준을 대기 행에 싣는다.** 비교 기준은 `compareWith`, 없으면 그 기준일 게시본을 만든 실행, 없으면 앞서 판정을 끝낸 가장 최근 실행이고 `baseline_batch_run_id`에 고정한다. 당시 입력(`setting_version` `a_snapshot_id`)은 비교 기준에서, 비교 기준이 없으면 그 기준일의 가장 최근 끝난 실행에서 옮겨 적는다. 둘 다 없으면 최신 설정이고 저장소를 새로 읽는다.
4. worker가 대기 행을 들어온 순서대로 집어 [[#PipelineRunner.run]]을 부른다. 설정은 대기 행의 버전으로 고정하고, A 판정은 그 스냅샷을 복사해 둔 mart 사본을 브리핑 저장소와 같은 모양으로 다시 읽는다. **그 기준일의 데이터 스냅샷, 당시 A 판정 스냅샷, 당시 설정 버전으로 돈다.** 시작을 거부당한 대기 행은 `failed`로 닫는다. 실행을 마치면 비교 표와 LLM 없는 실행 대조를 계산해 그 행에 적는다.
5. `options.backfillMode`가 참이면 5단계까지만 채우고 LLM 세 역할을 생략하며 게시하지 않는다.
6. 결과는 **새 버전이고 `is_published=false`**다.
7. 정기 배치도 같은 대기열을 거친다. 재생성이 돌고 있으면 정기 배치는 그 뒤에서 기다린다([[TBL-UC-002#UC-A3]] 2a).

**출력** `202 queued`와 배치 실행 식별자, 앞에서 기다리는 실행(`queuedBehind`).

**테스트 관점** 시작 단계 1(재생성)과 6(서술만)이 같은 호출로 갈리는지. 재실행 결과의 변동·후보·근접도·신호등이 당시와 전건 같고 설명·문장만 달라지는지. 재실행이 게시본을 덮지 않는지.

근거: [[TBL-SEQ-002#SEQ-12]] · [[TBL-PRD-002#N3]] · [[TBL-API-002#POST/api/admin/batch/rerun]]

#### BatchService.publish_version 게시 전환

**시그니처** `publish_version(report_id: str) -> dict`

**처리** 직전 게시본을 미게시로 내리고 이 버전을 게시본으로 바꾼다. **지우지 않는다.** 이미 게시본이면 `/problems/already-published` 409. 전환 후 [[#ReportPublisher.diff_watchlist]]를 다시 돌려 알림을 재계산한다.

**출력** `{reportId, publishedAt, alertsRecomputed}`.

**테스트 관점** 전환 후 같은 기준일의 게시본이 하나인지. 알림이 두 벌 남지 않는지.

근거: [[TBL-API-002#POST/api/admin/batch/publish/{reportId}]] · [[TBL-SEQ-002#SEQ-12]]

#### MasterService.get_summary 마스터 조회

**시그니처** `get_summary() -> dict`

**처리** 매칭률 셋(판매→국가, 생산→국가, 뉴스→국가)과 [[TBL-DOM-006#crosswalk_version]] 이력을 돌려준다.

**테스트 관점** 생산→국가 매칭률이 목적지 국가가 없는 동안 무엇을 뜻하는지 화면에 적히는지(형태 A에만 해당한다).

근거: [[TBL-API-002#GET/api/admin/master/summary]] · [[TBL-PRD-002#R4]]

#### MasterService.list_unmapped 미매핑 목록 조회

**시그니처** `list_unmapped(filters: dict) -> list[dict]`

**처리** [[TBL-DOM-006#unmapped_value]]를 `occurrence_count` 내림차순으로 돌려준다. `source` `kind`로 거른다.

**테스트 관점** 내려받아 크로스워크 엑셀을 보강할 수 있는 형태인지.

근거: [[TBL-API-002#GET/api/admin/master/unmapped]]

#### MasterService.upload_crosswalk 크로스워크 업로드

**시그니처** `upload_crosswalk(upload: UploadFile) -> dict`

**처리** [[TBL-DOM-006#crosswalk_upload]]에 저장하고 현재 버전과 **차이(추가·변경·삭제)를 미리 계산해** 돌려준다. 삭제로 미매핑이 될 건수를 함께 센다. 아직 적용하지 않는다. 시트 `category_route`(domain·category·is_provisional·note)는 도메인 값이 `production` `inventory` `sales` 밖인 줄을 오류로 돌려준다.

**출력** `{crosswalkUploadId, added, changed, removed, willBecomeUnmapped, affected}`.

**테스트 관점** 한 줄 빠진 파일을 올렸을 때 그 국가가 삭제 목록에 나오는지.

근거: [[TBL-SEQ-002#SEQ-13]] · [[TBL-API-002#POST/api/admin/master/crosswalk]]

#### MasterService.confirm_crosswalk 마스터 확정

**시그니처** `confirm_crosswalk(upload_id: str, confirm_removal: bool) -> dict`

**처리** 삭제되는 매핑이 있는데 `confirm_removal`이 거짓이면 → `/problems/removal-not-confirmed` 409(`willBecomeUnmapped`) · 확정이면 → [[TBL-DOM-006#crosswalk_version]] 새 버전을 만들고 [[TBL-DOM-006#crosswalk_entry]]를 교체한다. 올린 시트에 `category_route`가 있으면 [[TBL-DOM-006#category_route]]도 통째 바꾼다. 버전 이름은 확정 시각을 밀리초까지 쓴다(같은 초에 두 번 확정해도 겹치지 않는다). **다음 배치부터 적용된다. 과거 판정을 바꾸려면 재생성이다.**

**출력** `{version, appliedAt}`.

**테스트 관점** 확정 후 이미 게시된 리포트의 수치가 그대로인지. 글로비스 법인 매핑이 비어 있어도 오류가 아니라 "법인 미매핑"으로 표시되는지.

근거: [[TBL-SEQ-002#SEQ-13]] · [[TBL-PRD-002#R17]]

#### ExposureService.summary 열람 주소 조회

**시그니처** `summary() -> list[dict]`

**처리** [[TBL-DOM-006#screen_exposure]]를 읽어 화면 넷(`creport` `areportProduction` `areportInventory` `areportSales`)을 이 순서로 돌려준다. 행이 없는 화면은 `enabled=false`, `url=null`이다. `url`은 `PUBLIC_BASE_URL`과 화면 경로와 `token`을 이은 완성본이다.

**출력** `ScreenExposure` 넷([[TBL-API-002]] 4.14절).

**테스트 관점** 아무도 켜지 않았을 때 넷 모두 꺼짐이고 주소가 없는지. `PUBLIC_BASE_URL`이 있으면 주소가 그것으로 시작하는지.

근거: [[TBL-API-002#GET/api/admin/exposure/summary]] · [[TBL-SEQ-002#SEQ-15]]

#### ExposureService.switch 열람 주소 전환·회전

**시그니처** `switch(screen: str, enabled: bool, rotate_token: bool, admin_id: str) -> dict`

**처리** 화면 값이 넷 밖이면 → `/problems/validation-failed` 400 · 행이 없거나 `rotate_token`이 참이면 → URL에 실을 수 있는 무작위 32바이트 서명값을 새로 만든다 · 아니면 서명값을 그대로 두고 `enabled`만 바꾼다. 행을 upsert하고 `updated_by` `updated_at`을 남긴다. **회전하면 이전 서명값은 그 자리에서 무효다. 자동 회전은 없다.**

**출력** 그 화면의 `ScreenExposure` 한 줄.

**테스트 관점** 처음 켜면 주소가 생기고 그 서명값이 [[#ExposureService.verify]]를 통과하는지. 회전 뒤 이전 서명값이 막히고 새 값이 열리는지. 끄면 막히고 다시 켜면 같은 주소가 열리는지. web 계정이 켜고 바꿀 수는 있고 지울 수는 없는지.

근거: [[TBL-API-002#POST/api/admin/exposure/switch/{screen}]] · [[TBL-SEQ-002#SEQ-15]] · [[TBL-UC-002#UC-A4]]

#### ExposureService.verify 열람 주소 확인

**시그니처** `verify(token: str) -> str | None`

**처리** 켜져 있는 화면의 서명값과 글자 단위로 같은 것을 찾아 그 화면 값을 돌려준다. 비교는 걸린 시간으로 값이 새지 않는 방식으로 한다. 없으면 `None`. 열람 API의 인증 의존성이 부르고, 토큰이 없으면 `/problems/unauthenticated` 401, `None`이거나 다른 화면이면 `/problems/invalid-token` 403을 낸다. **읽는 표는 `pub`의 [[TBL-DOM-006#screen_exposure]] 하나다**([[TBL-INFRA-002#C10]]).

**테스트 관점** C 리포트 주소로 A 리포트가, A1 주소로 A3가 403인지. 특정 버전과 PDF도 그 버전 도메인의 주소로만 열리는지. 틀린 서명값이 오류 문장에 되풀이되지 않는지. 확인 중 `pub` 밖 조회가 0회인지.

근거: [[TBL-SEQ-002#SEQ-1]] · [[TBL-SEQ-002#SEQ-2]] · [[TBL-API-002]] 1.1절

## 9. 어댑터

#### HChatClient.complete LLM 호출

**시그니처** `complete(prompt: dict, json_schema: dict, purpose: str) -> dict`

**입력** `purpose`는 `naming` `causeLink` `narration` 셋. 항목당 프롬프트 4KB 이하.

**처리**
1. [[#HChatClient.remaining_tokens]]가 0이면 → 부르지 않고 실패로 돌려준다.
2. OpenAI 호환 규격으로 부른다. `base_url`과 `model`은 환경변수이고 **키는 로그에 남기지 않는다.**
3. 응답이 스키마에 맞으면 → 결과 · 콘텐츠 필터 → `contentFilter` · 스키마 위반 → `jsonViolation` · 그 밖의 오류 → `error`.
4. [[TBL-DOM-006#llm_call]]에 지점·모델·토큰·지연·결과를 건마다 남긴다. **프롬프트 원문과 응답 원문은 저장하지 않는다.**

**출력** 응답 본문 또는 실패 사유. 예외를 호출부로 던지지 않는다(호출부가 강등으로 처리한다).

**테스트 관점** 응답 스키마 어디에도 등급·점수·순위 필드가 없는지. 게이트웨이를 막아도 배치가 8단계까지 가는지. 로그와 DB에 키 문자열이 없는지. JSON 모드 지원은 아직 가정이다.

근거: [[TBL-INFRA-002#C2]] · [[TBL-INFRA-002#C3]] · [[TBL-SEQ-002#SEQ-9]]

#### HChatClient.remaining_tokens 남은 토큰

**시그니처** `remaining_tokens() -> int`

**처리** 일 상한에서 오늘 쓴 토큰을 뺀다. 0이면 그날은 더 부르지 않고 이후 호출은 전부 강등으로 간다. **상한에 걸려도 판정은 이미 5단계에서 끝나 있다.**

**테스트 관점** 상한을 0으로 두고 돌렸을 때 리포트가 템플릿 문장으로 게시되고 신호등이 정상 배치와 같은지.

근거: [[TBL-INFRA-002]] 8장 · [[TBL-PRD-002#N2]]

#### BriefingStoreReader.read_judgments A 판정 읽기

**시그니처** `read_judgments(base_date: date, domain: str) -> list[dict]`

**처리** 읽기 전용 계정 `intel_reader`로 뷰 `tbl_a_judgment`에서 그 기준일·도메인의 최신 게시 버전 행을 읽는다. 기여는 행의 `contributions` jsonb에 실려 온다. **쓰기 메서드를 두지 않는다.** 접속 실패면 예외를 올리고 [[#AJudgmentReader.read]]가 보완 집계로 내려간다. 반환 모양은 뷰 칸 그대로다(design/briefing_interface.md). 브리핑 갈래가 축 판정을 만들기 전에는 빈 목록이다.

**테스트 관점** 클래스에 쓰기 계열 메서드가 없는지(코드 검사). 계정 권한으로도 막혀 있는지.

근거: [[TBL-INFRA-002#C5]] · [[TBL-SEQ-002#SEQ-5]]

#### BriefingStoreReader.read_contributions 기여 읽기

**시그니처** `read_contributions(judgment_id: str) -> list[dict]`

**처리** 그 판정 행의 `contributions`(차원, 항목명, 항목 코드, 기여값, 비중, 자리)를 돌려준다. 별도 조회가 아니라 [[#BriefingStoreReader.read_judgments]]가 읽은 행에서 꺼낸다. **A 리포트가 쓴 문장은 읽지 않는다.**

**테스트 관점** 반환에 문장 필드가 섞이지 않는지. 기여 자리 번호가 기여값 내림차순이 낳은 것인지.

근거: [[TBL-PRD-002#R2]] · [[TBL-API-002]] 4.4절

#### BriefingStoreReader.read_report_document A 리포트 통째 읽기

**시그니처** `read_report_document(base_date: date, domain: str) -> dict | None`

**처리** 뷰 `tbl_a_report`에서 그 기준일·도메인의 최신 게시 버전 한 건을 **요약·해설·트래킹 지표·판정·각주 근거·버전 번호·게시 시각까지 통째로** 읽는다. 읽기 전용 계정이고 쓰기 메서드를 두지 않는다. 그 기준일 게시본이 없으면 `None`.

**출력** 반환 키는 뷰 `tbl_a_report`의 칸이다(design/briefing_interface.md). `snapshot_id` `domain` `dashboard_id` `dashboard_name` `base_date` `published_version` `published_at` `is_degraded` `summary` `commentary` `tracking_metrics` `judgments` `evidence`. 판정 목록은 비어도 빈 배열이고, 각주 근거는 `number` `kind` `source_id` `display_value`다. 제목·범위 표기·고정 문구·못 만드는 지표·제약 안내·데이터 형태·분해 블록은 뷰에 없고 [[#ReportPublisher.publish_a_report]]가 이 시스템 쪽에서 채운다(유저 결정 2026-10-01).

**이 함수와 [[#BriefingStoreReader.read_judgments]]는 쓰임이 다르다.** 판정 읽기는 C 리포트를 만들려고 **판정만** 가져오는 길이라 A가 쓴 문장을 읽지 않는다. 이 함수는 A 리포트 화면에 그대로 다시 뿌리려고 사본을 뜨는 길이라 문장을 가져온다. **가져온 문장은 C의 서술에 쓰지 않는다.**

**예외** 접속 실패면 예외를 올린다. [[#ReportPublisher.publish]]는 그 도메인 사본만 건너뛰고 배치를 이어 간다.

**테스트 관점** 반환 키가 뷰 칸과 빠짐없이 맞는지(하나만 어긋나도 사본이 빈 칸으로 게시된다). `dashboard_id` `tracking_metrics` `judgments` `evidence`가 반환에 다 있는지. C 리포트 문장에 이 함수가 가져온 문장이 인용으로 들어가지 않는지(문자열 대조, 동일 비율 0%). 클래스에 쓰기 계열 메서드가 없는지.

근거: [[TBL-INFRA-002#C5]] · [[TBL-INFRA-002#C10]] · [[TBL-SEQ-002#SEQ-2]] · [[TBL-UI-002#UI-2]]

#### BriefingStoreReader.snapshot_id 스냅샷 식별자

**시그니처** `snapshot_id(base_date: date) -> str`

**처리** 그 기준일 도메인 셋의 `snapshot_id`(`{view_luid}:{generation}`)를 도메인 순으로 `|`로 이어 돌려준다. 한 도메인이 없으면 그 자리는 비운다. 재생성 때 같은 입력을 다시 쓰기 위한 열쇠다.

**테스트 관점** 같은 기준일을 두 번 불러 같은 값이 나오는지. 이 값이 [[TBL-DOM-006#anomaly]]와 [[TBL-DOM-006#batch_run]]에 남는지.

근거: [[TBL-INFRA-002#C11]]

#### BriefingStoreReader.ping 저장소 접속 확인

**시그니처** `ping() -> bool`

**처리** 브리핑 갈래 저장소에 가벼운 조회 하나를 던져 붙는지만 본다. 붙으면 참, 못 붙으면 **거짓을 돌려준다. 예외를 던지지 않는다.** 던지면 이 확인 하나 때문에 2단계가 아니라 배치 전체가 멈춘다.

**출력** 참 또는 거짓. 어떤 데이터도 읽지 않고 어떤 행도 쓰지 않는다.

**예외** 없다. 시간 초과도 거짓으로 접는다.

**부르는 곳** [[#AJudgmentReader.read]]가 2단계 맨 앞에서 한 번 부른다. 거짓이면 도메인 셋을 모두 [[#AJudgmentReader.fallback_to_supplementary]]로 내리고 미수신 목록에 셋을 담는다. [[#BatchService.get_status]]도 같은 값을 관리 화면에 싣는다.

**테스트 관점** 저장소를 막아 놓고 불렀을 때 예외가 아니라 거짓인지. 거짓일 때 배치가 8단계까지 가고 리포트 `notices`에 `aJudgmentNotReceived`가 도메인 셋으로 담기는지. 이 함수가 판정을 한 건도 읽지 않는지(쿼리 로그 대조).

근거: [[TBL-INFRA-002#C5]] · [[TBL-SEQ-002#SEQ-5]]

## 10. 미결사항

- [x] [[#TrafficLightJudge.judge]]의 임계값. 현업 검토 전 기본값을 설정 초기 행 v1로 넣었다(유저 결정 2026-09-30). 현업 검토는 발주처 확인 요청에 싣는다
- [ ] [[#TrafficLightJudge.build_watch_items]]의 2차·3차 정렬 열쇠. 이 문서는 `대표 후보 날짜 차이 → 국가 코드`로 두었으나 상위 문서에 확정 문장이 없다
- [ ] [[#ProximityCalculator.day_diff]]를 기간 구분에 따라 보정할지. 누계·년 변동은 후보가 구조적으로 멀어 보인다. 지금은 보정하지 않고 화면에 기간 구분을 적는 것으로 둔다
- [x] [[#BriefingStoreReader.read_judgments]]의 반환 키와 필드. 뷰 `tbl_a_judgment`의 칸으로 정했고 [[#AJudgmentReader.validate_shape]]의 대조 목록을 적었다(유저 결정 2026-10-01)
- [x] [[#FormALedgerParser.check_columns]]가 대조할 실제 컬럼 목록. 실물 원장과 IF 레이아웃 정의가 같다(생산 16열, 판매·재고 22열, 2026-09-30)
- [ ] [[#FormBPivotParser.detect_file_base_date]]가 파일 안에서 기준일을 읽을 수 있는지. 못 읽으면 관리자 입력이 필수가 된다
- [ ] [[#HChatClient.complete]]의 JSON 모드. 게이트웨이가 응답 스키마 지정을 지원한다는 것은 가정이며 실호출 확인 범위가 좁다. 지원되지 않으면 세 역할의 검증 단계가 늘어난다
- [x] [[#DomainReportService.get_latest_by_domain]]에서 사본이 없을 때 판정값과 차트를 어디서 읽는가. 읽지 않는다. 문장 미수신만 보인다. 분해 블록은 게시 때 사본에 함께 넣으므로 사본이 있으면 늘 있다(유저 결정 2026-10-01)
- [ ] 설정 변경 경로. [[TBL-DOM-006#threshold_setting]] 새 버전 행을 SQL로 넣을지 CLI를 만들지. 정해지면 함수가 하나 는다
- [ ] 현업 피드백("맞다·아니다·모르겠다") 저장 함수. 화면에 넣을지가 미결이라 이 문서에도 두지 않았다([[TBL-PRD-002#R19]]). 피드백은 사번이 필요한데 열람은 화면 단위 서명 주소라 개인을 식별하지 않는다(2026-09-30)