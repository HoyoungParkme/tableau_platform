---
doc_id: TBL-API-001
type: API
title: Glovis 완성차 인텔리전스 REST API
status: draft
upstream: [TBL-UI-001, TBL-UC-001, TBL-DOM-001, TBL-INFRA-001]
---

# API 명세 REST

## 0. 이 문서가 다루는 것

backend `web/public`(현업 열람, 토큰)과 `web/admin`(관리, 세션)의 REST 엔드포인트다. OpenAPI 3.1 조각을 엔드포인트마다 둔다. 열람 API는 pub 스키마만 읽는다([[TBL-INFRA-001#C10]]). 배치 파이프라인 자체는 API가 아니라 스케줄러 프로세스이며 관리 API가 트리거만 한다.

이 버전은 끊어진 참조만 해결한 것이다(판매 상세, 설정 조회·저장 제거). 새 방향(A1·A2·A3·C 용어, 변동 목록·원인 후보·관련도 응답)의 전면 반영은 다음 버전에서 한다. 설정(임계값·시간창·후보 상한)은 배치가 설정 테이블의 현재 버전을 읽으며 API로 바꾸지 않는다.

## 1. 규칙

- 경로 접두: 현업 `/api/intel/*`, 관리 `/api/admin/intel/*`.
- 인증: 현업은 쿼리 `token`(서명, 사번, 만료) 또는 헤더 `X-Intel-Token`. 관리는 세션 쿠키 + 관리자 권한.
- 시각은 ISO 8601, 날짜는 `YYYY-MM-DD`. 수치는 정수 대수, 비율은 소수(0.153).
- 목록은 `limit`(기본 50, 최대 500)과 `cursor`.
- 쓰기 응답은 202(배치 트리거)와 201(생성)을 구분한다.
- 문장 텍스트는 강조 토큰(hl 태그)과 각주 번호를 포함한 원문이다. 렌더는 화면 몫.

## 2. 에러

RFC 9457 `application/problem+json`. `type`은 `urn:tbl:intel:` 뒤에 코드.

| status | code | 언제 |
|:--|:--|:--|
| 401 | token-invalid | 토큰 서명·만료 실패 |
| 403 | forbidden | 관리자 권한 없음 |
| 404 | not-found | 브리핑·국가·배치 없음 |
| 409 | batch-running | 배치 실행 중 중복 트리거 |
| 409 | ingest-overlap | 같은 기준일 적재 중 |
| 422 | schema-mismatch | 컬럼 대조 실패. detail에 다른 컬럼 목록 |
| 423 | batch-blocked | 회귀 검사 차단 상태에서 자동 실행 요청 |
| 503 | briefing-unavailable | 게시 브리핑이 아직 없음 |

## 3. 엔드포인트

### 3.1 현업 열람 (토큰)

#### GET/api/intel/briefing/latest 최신 게시 브리핑

화면 [[TBL-UI-001#UI-1]] · 유스케이스 [[TBL-UC-001#UC-H1]] · 서비스 `BriefingQuery.latest`

응답 1회로 지표 바, 헤드라인, 도메인 상태, 변동 카드(근거 포함)를 전부 준다. LLM 호출 없음.

```yaml
/api/intel/briefing/latest:
  get:
    summary: 최신 게시 브리핑
    security: [{ intelToken: [] }]
    responses:
      '200':
        content:
          application/json:
            schema: { $ref: '#/components/schemas/BriefingView' }
      '503': { $ref: '#/components/responses/Problem' }
```

#### GET/api/intel/briefing/{briefingId} 특정 브리핑

화면 [[TBL-UI-001#UI-1]] · 유스케이스 [[TBL-UC-001#UC-H1]] · 서비스 `BriefingQuery.get`

게시본만 조회한다. 미게시 버전은 404.

```yaml
/api/intel/briefing/{briefingId}:
  get:
    parameters: [{ name: briefingId, in: path, schema: { type: integer } }]
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/BriefingView' } } } }
```

#### GET/api/intel/indicators/{identifier}/series 시장지표 시계열

화면 [[TBL-UI-001#UI-1]] · 서비스 `IndicatorQuery.series`

지표 바 팝오버 스파크라인용. `days` 기본 30.

```yaml
/api/intel/indicators/{identifier}/series:
  get:
    parameters:
      - { name: identifier, in: path, schema: { type: string } }
      - { name: days, in: query, schema: { type: integer, default: 30, maximum: 365 } }
    responses:
      '200':
        content:
          application/json:
            schema:
              type: object
              properties:
                identifier: { type: string }
                label: { type: string }
                points: { type: array, items: { type: object, properties: { date: { type: string, format: date }, value: { type: number } } } }
```

### 3.2 관리: 적재 (세션)

#### POST/api/admin/intel/ingest/preflight 적재 사전 검증

화면 [[TBL-UI-001#UI-3]] · 유스케이스 [[TBL-UC-001#UC-A1]] · 서비스 `IngestService.preflight`

파일을 임시 저장하고 스키마 판별, 컬럼 대조, 기간, 행수, 결측, 골격 행, 중복 의심을 돌려준다. 적재하지 않는다.

```yaml
/api/admin/intel/ingest/preflight:
  post:
    requestBody:
      content:
        multipart/form-data:
          schema:
            type: object
            required: [source, file]
            properties:
              source: { type: string, enum: [prod, sales, news, bbg, mkl, crosswalk] }
              file: { type: string, format: binary }
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/Preflight' } } } }
      '422': { $ref: '#/components/responses/Problem' }
```

#### POST/api/admin/intel/ingest 적재 실행

화면 [[TBL-UI-001#UI-3]] · 유스케이스 [[TBL-UC-001#UC-A1]] · 서비스 `IngestService.run`

preflight가 준 `stagingId`로 실행. L0·L1 적재, 미매핑 수집, 회귀 검사까지. 완료 후 ingest_run을 돌려준다.

```yaml
/api/admin/intel/ingest:
  post:
    requestBody:
      content:
        application/json:
          schema:
            type: object
            required: [stagingId]
            properties:
              stagingId: { type: string }
              backfill: { type: boolean, default: false }
              overwrite: { type: boolean, default: false }
    responses:
      '201': { content: { application/json: { schema: { $ref: '#/components/schemas/IngestRun' } } } }
      '409': { $ref: '#/components/responses/Problem' }
```

#### GET/api/admin/intel/ingest/runs 적재 이력

화면 [[TBL-UI-001#UI-3]] · 서비스 `IngestService.list`

```yaml
/api/admin/intel/ingest/runs:
  get:
    parameters:
      - { name: source, in: query, schema: { type: string } }
      - { name: limit, in: query, schema: { type: integer, default: 50 } }
      - { name: cursor, in: query, schema: { type: string } }
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/IngestRunList' } } } }
```

### 3.3 관리: 배치

#### GET/api/admin/intel/batch/status 배치 상태 요약

화면 [[TBL-UI-001#UI-4]] · 서비스 `BatchService.status`

```yaml
/api/admin/intel/batch/status:
  get:
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/BatchStatus' } } } }
```

#### GET/api/admin/intel/batch/runs 배치 이력

화면 [[TBL-UI-001#UI-4]] · 서비스 `BatchService.list`

```yaml
/api/admin/intel/batch/runs:
  get:
    parameters:
      - { name: limit, in: query, schema: { type: integer, default: 50 } }
      - { name: cursor, in: query, schema: { type: string } }
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/BatchRunList' } } } }
```

#### GET/api/admin/intel/batch/runs/{runId} 배치 상세

화면 [[TBL-UI-001#UI-4]] · 서비스 `BatchService.get`

단계별 로그와, 재생성 배치면 당시 판정과의 차이를 포함한다.

```yaml
/api/admin/intel/batch/runs/{runId}:
  get:
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/BatchRunDetail' } } } }
```

#### POST/api/admin/intel/batch/run 배치 실행·재생성

화면 [[TBL-UI-001#UI-4]] · 유스케이스 [[TBL-UC-001#UC-A4]] · 서비스 `BatchService.trigger`

기준일과 시작 단계를 받아 비동기로 돌린다. 이미 실행 중이면 409.

```yaml
/api/admin/intel/batch/run:
  post:
    requestBody:
      content:
        application/json:
          schema:
            type: object
            required: [asOfDate]
            properties:
              asOfDate: { type: string, format: date }
              startStep: { type: integer, minimum: 1, maximum: 8, default: 1 }
              backfill: { type: boolean, default: false }
    responses:
      '202': { content: { application/json: { schema: { $ref: '#/components/schemas/BatchRun' } } } }
      '409': { $ref: '#/components/responses/Problem' }
      '423': { $ref: '#/components/responses/Problem' }
```

#### POST/api/admin/intel/batch/runs/{runId}/publish 재생성본 게시 전환

화면 [[TBL-UI-001#UI-4]] · 유스케이스 [[TBL-UC-001#UC-A4]] · 서비스 `BatchService.publish`

해당 배치가 만든 briefing을 그 기준일의 게시본으로 바꾼다. 이전 게시본은 published=false로 남는다.

```yaml
/api/admin/intel/batch/runs/{runId}/publish:
  post:
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/PublishResult' } } } }
```

#### POST/api/admin/intel/batch/unblock 회귀 차단 해제

화면 [[TBL-UI-001#UI-4]] · 유스케이스 [[TBL-UC-001#UC-S6]] · 서비스 `BatchService.unblock`

```yaml
/api/admin/intel/batch/unblock:
  post:
    requestBody:
      content:
        application/json:
          schema: { type: object, required: [reason], properties: { reason: { type: string } } }
    responses:
      '200': { description: 해제됨 }
```

### 3.4 관리: 마스터

#### GET/api/admin/intel/master/{masterName} 마스터 조회

화면 [[TBL-UI-001#UI-5]] · 서비스 `MasterService.get`

`masterName`은 country, factory, news_category, routing, series_map, strait_country, glovis_entity, model_cbu_ckd.

```yaml
/api/admin/intel/master/{masterName}:
  get:
    parameters: [{ name: masterName, in: path, schema: { type: string } }]
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/MasterView' } } } }
```

#### GET/api/admin/intel/master/gaps 미매핑 목록

화면 [[TBL-UI-001#UI-5]] · 유스케이스 [[TBL-UC-001#UC-A2]] · 서비스 `MasterService.gaps`

`format=csv`면 파일로.

```yaml
/api/admin/intel/master/gaps:
  get:
    parameters:
      - { name: master, in: query, schema: { type: string } }
      - { name: format, in: query, schema: { type: string, enum: [json, csv], default: json } }
    responses:
      '200': { content: { application/json: { schema: { type: array, items: { $ref: '#/components/schemas/MappingGap' } } } } }
```

#### POST/api/admin/intel/master/upload 크로스워크 업로드(차이 계산)

화면 [[TBL-UI-001#UI-5]] · 유스케이스 [[TBL-UC-001#UC-A2]] · 서비스 `MasterService.stage`

엑셀을 받아 시트별 차이와 영향 범위를 돌려준다. 반영하지 않는다.

```yaml
/api/admin/intel/master/upload:
  post:
    requestBody:
      content:
        multipart/form-data:
          schema: { type: object, required: [file], properties: { file: { type: string, format: binary } } }
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/MasterDiff' } } } }
```

#### POST/api/admin/intel/master/confirm/{stagingId} 마스터 확정

화면 [[TBL-UI-001#UI-5]] · 유스케이스 [[TBL-UC-001#UC-A2]] · 서비스 `MasterService.confirm`

upload가 준 stagingId의 차이를 새 마스터 버전으로 반영한다. 다음 배치부터 적용된다.

```yaml
/api/admin/intel/master/confirm/{stagingId}:
  post:
    parameters: [{ name: stagingId, in: path, schema: { type: string } }]
    responses:
      '201': { content: { application/json: { schema: { $ref: '#/components/schemas/MasterVersion' } } } }
```

## 4. 스키마

```yaml
components:
  securitySchemes:
    intelToken: { type: apiKey, in: query, name: token }
  responses:
    Problem:
      content:
        application/problem+json:
          schema:
            type: object
            properties: { type: { type: string }, title: { type: string }, status: { type: integer }, detail: { type: string }, instance: { type: string } }
  schemas:
    Claim:
      type: object
      properties: { seq: { type: integer }, text: { type: string }, footnotes: { type: array, items: { type: integer } } }
    Evidence:
      type: object
      properties:
        seq: { type: integer }
        kind: { type: string, enum: [event, article, market, signal] }
        display: { type: object }
    Indicator:
      type: object
      properties: { identifier: { type: string }, label: { type: string }, value: { type: number, nullable: true }, changePct: { type: number, nullable: true }, asOf: { type: string, format: date }, stale: { type: boolean } }
    DomainStatus:
      type: object
      properties:
        domain: { type: string, enum: [production, sales, inventory] }
        signal: { type: string, enum: [RED, YELLOW, MONITOR] }
        claims: { type: array, items: { $ref: '#/components/schemas/Claim' } }
        dataQualityNote: { type: string, nullable: true }
    WatchlistCard:
      type: object
      properties:
        iso3: { type: string }
        countryName: { type: string }
        signal: { type: string, enum: [RED, YELLOW] }
        changeBadge: { type: string, nullable: true, enum: [new, up, down] }
        cbuRatio: { type: number, nullable: true }
        exposure: { type: string, enum: [CBU, CKD, unknown] }
        entityTags: { type: array, items: { type: string } }
        structuredSignal:
          type: object
          properties: { metric: { type: string }, value: { type: number, nullable: true }, changePct: { type: number, nullable: true }, method: { type: string, enum: [YoY, MoM, none] } }
        inventorySignal:
          type: object
          nullable: true
          properties: { voyage: { type: integer }, waiting: { type: integer }, ratio: { type: number } }
        event:
          type: object
          properties: { eventId: { type: integer }, title: { type: string }, category: { type: string }, severity: { type: number }, articleCount: { type: integer } }
        summary: { $ref: '#/components/schemas/Claim' }
        evidenceSeqs: { type: array, items: { type: integer } }
    BriefingView:
      type: object
      required: [briefingId, asOfDate, dataMaxDates, degraded, indicatorBar, headline, domainStatus, watchlist]
      properties:
        briefingId: { type: integer }
        asOfDate: { type: string, format: date }
        version: { type: integer }
        dataMaxDates: { type: object, additionalProperties: { type: string, format: date } }
        degraded: { type: boolean }
        degradeReason: { type: string, nullable: true }
        indicatorBar: { type: array, items: { $ref: '#/components/schemas/Indicator' } }
        headline: { $ref: '#/components/schemas/Claim' }
        headlineBadge: { type: string }
        domainStatus: { type: array, items: { $ref: '#/components/schemas/DomainStatus' } }
        watchlist: { type: array, items: { $ref: '#/components/schemas/WatchlistCard' } }
        evidence: { type: array, items: { $ref: '#/components/schemas/Evidence' } }
    Preflight:
      type: object
      properties:
        stagingId: { type: string }
        source: { type: string }
        schemaVersion: { type: string, nullable: true }
        columnMismatch: { type: array, items: { type: string } }
        periodFrom: { type: string, format: date, nullable: true }
        periodTo: { type: string, format: date, nullable: true }
        rowCount: { type: integer }
        nullCount: { type: integer }
        skeletonCount: { type: integer }
        dupSuspectDates: { type: array, items: { type: string, format: date } }
        overlapsExisting: { type: boolean }
    IngestRun:
      type: object
      properties:
        runId: { type: integer }
        source: { type: string }
        schemaVersion: { type: string }
        periodFrom: { type: string, format: date }
        periodTo: { type: string, format: date }
        rowCount: { type: integer }
        nullRate: { type: number }
        matchRates: { type: object, additionalProperties: { type: number } }
        dupRate: { type: number }
        regressionFlag: { type: boolean }
        backfill: { type: boolean }
        status: { type: string }
        filePath: { type: string }
        createdAt: { type: string, format: date-time }
    IngestRunList:
      type: object
      properties: { items: { type: array, items: { $ref: '#/components/schemas/IngestRun' } }, nextCursor: { type: string, nullable: true } }
    BatchStatus:
      type: object
      properties:
        lastSuccessAt: { type: string, format: date-time, nullable: true }
        nextScheduledAt: { type: string, format: date-time }
        blocked: { type: boolean }
        blockedReason: { type: string, nullable: true }
        running: { type: boolean }
        todayTokens: { type: integer }
        tokenCap: { type: integer }
    BatchRun:
      type: object
      properties:
        runId: { type: integer }
        asOfDate: { type: string, format: date }
        trigger: { type: string, enum: [schedule, manual, regenerate] }
        status: { type: string, enum: [running, success, failed, degraded] }
        failedStep: { type: integer, nullable: true }
        durationSec: { type: integer, nullable: true }
        llmTokens: { type: integer }
        briefingId: { type: integer, nullable: true }
    BatchRunList:
      type: object
      properties: { items: { type: array, items: { $ref: '#/components/schemas/BatchRun' } }, nextCursor: { type: string, nullable: true } }
    BatchStep:
      type: object
      properties: { step: { type: integer }, stepName: { type: string }, status: { type: string }, durationSec: { type: integer }, counts: { type: object }, error: { type: string, nullable: true } }
    JudgmentDiff:
      type: object
      properties: { iso3: { type: string }, field: { type: string }, before: { type: string }, after: { type: string }, cause: { type: string, enum: [mapping, threshold, late_data, upstream_judgment, unknown] } }
    BatchRunDetail:
      allOf:
        - { $ref: '#/components/schemas/BatchRun' }
        - type: object
          properties:
            steps: { type: array, items: { $ref: '#/components/schemas/BatchStep' } }
            diff: { type: array, nullable: true, items: { $ref: '#/components/schemas/JudgmentDiff' } }
    PublishResult:
      type: object
      properties: { briefingId: { type: integer }, asOfDate: { type: string, format: date }, version: { type: integer } }
    MasterVersion:
      type: object
      properties: { master: { type: string }, version: { type: integer }, changedBy: { type: string }, createdAt: { type: string, format: date-time }, note: { type: string } }
    MasterView:
      type: object
      properties:
        version: { type: integer }
        rows: { type: array, items: { type: object } }
        history: { type: array, items: { $ref: '#/components/schemas/MasterVersion' } }
    MappingGap:
      type: object
      properties: { master: { type: string }, rawValue: { type: string }, occurrences: { type: integer }, firstSeen: { type: string, format: date }, lastSeen: { type: string, format: date } }
    MasterSheetDiff:
      type: object
      properties: { master: { type: string }, added: { type: integer }, changed: { type: integer }, removed: { type: integer }, removedValues: { type: array, items: { type: string } }, affectedKeys: { type: array, items: { type: string } } }
    MasterDiff:
      type: object
      properties:
        stagingId: { type: string }
        sheets: { type: array, items: { $ref: '#/components/schemas/MasterSheetDiff' } }
```

## 5. 미결사항

- [ ] 토큰 전달 방식: 쿼리 token만 쓸지 헤더도 허용할지. 가정: 둘 다
- [ ] 뉴스 원본 대용량 업로드의 청크 프로토콜(단일 multipart 한도)
- [ ] 알림 채널 어댑터 API(발송 상태 조회)는 채널 확정 후 추가
- [ ] 관리 API의 발주자 데이터서비스팀 권한 등급
- [ ] BriefingView의 evidence를 전체로 줄지 카드별 지연 로딩할지. 가정: 전체(수백 건 이내)
- [ ] 설정 변경 수단. API 없음. 가정: 개발자가 설정 테이블에 새 버전 행 추가
- [ ] 다음 버전: WatchlistCard를 변동·원인 후보(관련도·인용)·내부 분해 구조로 교체, DomainStatus에 A 판정 출처 표시
