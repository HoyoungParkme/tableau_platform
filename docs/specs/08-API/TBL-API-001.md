---
doc_id: TBL-API-001
type: API
title: Glovis 완성차 인텔리전스 REST API
status: draft
upstream: [TBL-UI-001, TBL-DOM-001, TBL-INFRA-001, TBL-UC-001, TBL-PRD-001]
---

# API 명세 REST

## 0. 이 문서가 다루는 것

화면 일곱([[TBL-UI-001#UI-10]] [[TBL-UI-001#UI-7]] [[TBL-UI-001#UI-8]] [[TBL-UI-001#UI-9]] [[TBL-UI-001#UI-11]] [[TBL-UI-001#UI-12]] [[TBL-UI-001#UI-13]])이 부르는 REST 엔드포인트와 그 응답 모양을 정한다. 엔드포인트 19개, 공용 스키마 13개다.

두 묶음이다. **현업 열람**은 `/api/intel` 아래 다섯 개이며 서명 토큰으로 연다. **관리**는 `/api/admin` 아래 열네 개이며 세션으로 연다. 열람 쪽은 읽기만 하고 LLM을 부르지 않는다([[TBL-PRD-001#N5]]).

이 문서의 핵심 제약은 하나다. **등급·점수·순위를 뜻하는 필드를 응답 어디에도 두지 않는다.** 원인 후보가 들고 있는 것은 코드가 센 값 넷뿐이며(날짜 차이, 기사 수, 출처 수, 국가 일치 방식) 그 값이 정렬과 신호등의 유일한 입력이다([[TBL-PRD-001#R26]] [[TBL-DOM-001#CauseCandidate]], PRD 6.14). 금지 필드명은 1.3절에 목록으로 적었다.

여기서 정하지 않는 것. 서비스 함수의 내부 흐름은 [[TBL-SEQ-001]]이, 클래스와 함수 시그니처는 [[TBL-MS-001]]이 정한다. 테이블과 컬럼은 DOM 클래스 명세와 ERD가 정하며 이 문서보다 뒤에 나온다.

## 1. 규칙

### 1.1 인증

| 묶음 | 방식 | 헤더 | 근거 |
|:--|:--|:--|:--|
| 현업 열람 `/api/intel` | VODA가 발급한 서명 토큰 | `Authorization: Bearer <token>` | [[TBL-PRD-001#R23]] 단독 주소, 추가 로그인 없음 |
| 관리 `/api/admin` | 세션 쿠키 | `Cookie: sid=...` + 쓰기 메서드에 `X-CSRF-Token` | [[TBL-UC-001#UC-A1]] 관리 화면 로그인 |

- 토큰 검증은 폐쇄망 안에서 끝낸다. 서명 공개키를 컨테이너에 두고 외부 조회를 하지 않는다([[TBL-PRD-001#N1]] [[TBL-INFRA-001#C10]]).
- 세션 쿠키는 `HttpOnly`, `SameSite=Strict`로 내린다.
- 현업 토큰으로 `/api/admin`을 부르면 403이다. 반대 방향은 허용하지 않는다.
- 토큰 발급 주체와 만료 시간의 세부는 [확인 필요]다. VODA 포털 팀과 맞춰야 한다.

### 1.2 공통 응답 규칙

- 성공 본문은 `application/json`, 에러 본문은 `application/problem+json`이다(2장).
- 날짜는 `YYYY-MM-DD`, 시각은 RFC 3339 UTC 오프셋 포함이다.
- 비율은 소수다. `0.252`는 25.2%를 뜻한다. 화면이 백분율로 그린다.
- 열람 응답은 게시된 결과의 조회다. 서버가 그 자리에서 다시 계산하거나 H-chat을 부르지 않는다([[TBL-PRD-001#N5]], p95 2초).
- 값이 없는 것과 만들 수 없는 것을 구분한다. 값이 없으면 `null`, 데이터 구조상 만들 수 없으면 해당 필드를 아예 내리지 않고 `missingMetrics`에 이유와 함께 넣는다(4.13절).
- 열람 응답은 기사 원문 URL 외에 외부 주소를 담지 않는다. 이미지·글꼴·스크립트 주소를 응답에 넣지 않는다.

### 1.3 두지 않는 필드

아래 이름과 그 변형을 요청·응답 스키마 어디에도 두지 않는다. 검토와 회귀 검사에서 이름으로 찾는다.

| 금지 | 왜 |
|:--|:--|
| `score`, `relevanceScore`, `weight` | 가중치의 근거가 임의가 된다(PRD 6.14 C안 보류) |
| `grade`, `relevance`, `relevanceLevel` | 모델이 매기면 재현되지 않고 근거를 댈 수 없다 |
| `rank`, `ranking`, `priority` | 관련도 순위로 읽힌다 |
| `confidence`, `strength`, `certainty` | 판정의 세기를 모델이 정하는 것으로 읽힌다 |

허용하는 순서 필드는 `sortOrder` 하나다. 이것은 관련도가 아니라 **정렬 규칙이 낳은 자리 번호**다. 규칙은 날짜 차이 오름차순, 같으면 출처 수 내림차순, 그다음 기사 수 내림차순이다([[TBL-DOM-001#CauseCandidate]]). 같은 입력이면 같은 번호가 나온다. 응답의 `candidateSortRule`에 이 문장을 그대로 실어 화면이 글자로 적는다([[TBL-UI-001#UI-10]] 규칙).

4.4절의 `sortOrder`도 같다. 기여값 내림차순이 낳은 자리이지 중요도가 아니다.

### 1.4 기간 키와 부호

**시간 축은 칸 두 개다.** 단일 기준일을 쓰지 않는다([[TBL-INFRA-001#C15]] [[TBL-DOM-001#Anomaly]]). 모든 판정·결합·유도 행이 `periodKey`를 가진다(4.2절).

- `periodType`은 `day` `month` `cumulative` `year` 넷이다. 각각 일·월·누계·년이다.
- `fileBaseDate`는 그 값이 온 파일의 기준일이다. 형태 A(IF 원장)면 행의 기준일자와 같고, 형태 B(피벗 리포트)면 파일이 붙여 준 날이다([[TBL-INFRA-001#C6]]).
- 후보의 `dayDiff`는 변동의 `fileBaseDate`를 기준점으로 센다. 기간 구분이 넓을수록 후보가 멀어 보이는 것은 값이 그렇게 나오는 것이 맞다. 화면이 기간 구분을 함께 적는다.

**단계 차이의 부호는 앞 단계에서 뒤 단계를 뺀 값이다.**

| 필드 | 식 | 미주 누계 실측 |
|:--|:--|--:|
| `entityStageGap` | 선적 − 도매정본 | 679,551 − 677,201 = 2,350 |
| `dealerStageGap` | 도매정본 − 소매 | 677,201 − 643,097 = 34,104 |
| `dealerStageGapRate` | `dealerStageGap` ÷ 도매정본 | 0.050 |

양수는 그 단계에 물량이 남아 있다는 뜻이다(칠레 0.252, 페루 0.212). 음수는 뒤 단계가 더 크다는 뜻이며 이전에 쌓인 것을 덜어내는 중이다(푸에르토리코 -0.137, 콜롬비아 -0.105). **음수를 예외로 처리하지 않는다**([[TBL-PRD-001#R30]]). [[TBL-UI-001#UI-9]] 단계별 흐름이 `-34,104`로 그리는 것은 감소 방향을 나타낸 렌더링이며 API 값은 양수 34,104다.

### 1.5 경로 짓는 법

- 경로 파라미터는 **마지막 세그먼트에만** 둔다. `POST/api/admin/ingest/upload/{id}/confirm` 형태를 쓰지 않는다.
- 같은 메서드에서 접두가 겹치면 파라미터 바로 앞 세그먼트를 서로 다른 리터럴로 가른다. `.../creport/latest`와 `.../creport/version/{reportId}`처럼 `latest`와 `version`이 갈라 준다.
- 목록은 복수 리터럴이 아니라 뜻이 담긴 리터럴로 쓴다. `history`, `summary`, `unmapped`.

### 1.6 생산은 국가 축에 오르지 않는다

`domain=production`인 변동은 축 종류가 `plant`이며 [[#GET/api/intel/creport/latest]] 응답의 `anomalies` 배열에 들어가지 않는다. 생산이 들어가는 자리는 `domainStatus` 한 장뿐이고 그 장의 `externalCauseAllowed`는 항상 `false`다([[TBL-INFRA-001#C16]] [[TBL-PRD-001#R11]] [[TBL-PRD-001#R15]]).

## 2. 에러

RFC 9457 `application/problem+json`이다. 공통 필드는 `type` `title` `status` `detail` `instance`이고 아래 표의 확장 필드가 더 붙는다. 스키마는 4.1절.

| type | status | 언제 | 확장 필드 |
|:--|--:|:--|:--|
| `/problems/unauthenticated` | 401 | 토큰 없음·만료, 세션 없음 | |
| `/problems/forbidden` | 403 | 현업 토큰으로 관리 API 호출 | |
| `/problems/not-found` | 404 | 리포트·배치·파일 ID가 없음 | `resource` |
| `/problems/no-published-report` | 404 | 첫 배치 전([[TBL-UI-001#UI-10]] E2) | `nextScheduledAt` |
| `/problems/validation-failed` | 400 | 요청 본문·쿼리 검증 실패 | `errors[]` |
| `/problems/stage-out-of-range` | 400 | 시작 단계가 1~8 밖 | `startStage` |
| `/problems/form-undetected` | 422 | 형태 A·B 어느 쪽으로도 판별 안 됨 | `mismatch[]` |
| `/problems/preflight-expired` | 409 | `preflightId` 만료 또는 소비됨 | `preflightId` |
| `/problems/overwrite-not-confirmed` | 409 | 덮어쓰기 확인 없이 적재 확정 | `idempotencyUnit`, `target` |
| `/problems/regression-blocked` | 409 | 회귀 급변으로 배치 자동 실행 차단 중 | `ingestFileId`, `abruptReasons[]` |
| `/problems/batch-in-progress` | 409 | 같은 기준일 배치가 실행 중 | `batchRunId` |
| `/problems/already-published` | 409 | 이미 게시본인 리포트를 다시 게시 | `reportId` |
| `/problems/removal-not-confirmed` | 409 | 삭제되는 매핑 확인 없이 확정 | `willBecomeUnmapped` |
| `/problems/file-too-large` | 413 | 업로드 한도 초과 | `limitBytes` |

- `type`은 폐쇄망 안에서 해석되는 상대 경로다. 외부 URL을 두지 않는다.
- 400·422는 화면이 그 자리에 사유를 적고 버튼을 막는다. 409는 사람이 확인하면 풀리는 것이라 확인 절차를 함께 안내한다.
- **E2를 뺀 열람 예외는 에러가 아니다.** 변동 없음·강등·판정 미수신·배치 실패·지표 이월은 200으로 내려가고 `notices`에 담긴다([[TBL-UI-001#UI-10]] 예외 상태).

## 3. 엔드포인트

### 3.1 현업 열람 (서명 토큰)

#### GET/api/intel/creport/latest 최신 C 리포트 조회

게시된 최신 C 리포트 한 본을 한 번에 돌려준다. [[TBL-UI-001#UI-10]]이 이 호출 하나로 화면 전부를 그린다.

화면 [[TBL-UI-001#UI-10]] · 유스케이스 [[TBL-UC-001#UC-H1]] [[TBL-UC-001#UC-H2]] · 서비스 `CReportService.get_latest`

응답에 담기는 덩어리는 아홉이다. 기준일과 소스별 최신일, 시장지표 바 4종, 헤드라인과 인과 배지, 도메인 상태 3장, 변동 목록, 변동별 후보 목록, 후보별 근거, 연관 설명, 강등과 배치 번호. 근거 패널까지 이 응답에 들어 있어 펼칠 때 추가 호출이 없다([[TBL-UC-001#UC-H2]]).

`anomalies`는 신호등 순(`red` → `yellow` → `none`)으로 이미 정렬돼 온다. 각 변동의 `candidates`도 1.3절의 정렬 규칙으로 이미 정렬돼 온다. 화면은 순서를 다시 계산하지 않는다.

```yaml
/api/intel/creport/latest:
  get:
    summary: 게시된 최신 C 리포트 전체
    operationId: getLatestCReport
    security: [{ bearerToken: [] }]
    responses:
      '200':
        description: 게시본
        content:
          application/json:
            schema: { $ref: '#/components/schemas/CReport' }
      '404':
        description: 게시된 리포트가 아직 없음 (첫 배치 전)
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
            example:
              type: /problems/no-published-report
              title: 게시된 리포트가 없습니다
              status: 404
              nextScheduledAt: '2026-08-21T02:00:00+09:00'
components:
  schemas:
    CReport:
      type: object
      required: [reportId, baseDate, version, batchRunId, dataFreshness, marketBar,
                 headline, causalBadge, domainStatus, anomalies, evidence, notices]
      properties:
        reportId: { type: string }
        baseDate: { type: string, format: date }
        version: { type: integer }
        published: { type: boolean }
        batchRunId: { type: string }
        publishedAt: { type: string, format: date-time }
        settingVersion: { type: string, description: 판정에 쓴 설정 버전 }
        dataFreshness:
          type: object
          description: 소스별 데이터 최신일. 리포트 기준일과 다를 수 있다
          required: [vehicle, news, market]
          properties:
            vehicle: { type: string, format: date, nullable: true }
            news: { type: string, format: date, nullable: true }
            market: { type: string, format: date, nullable: true }
        marketBar:
          type: array
          minItems: 4
          maxItems: 4
          items: { $ref: '#/components/schemas/MarketMetric' }
        headline: { $ref: '#/components/schemas/Claim' }
        causalBadge:
          type: string
          enum: ['상관관계 확인 · 인과관계 미확정']
        domainStatus:
          type: array
          minItems: 3
          maxItems: 3
          items: { $ref: '#/components/schemas/DomainStatus' }
        anomalies:
          type: array
          description: 신호등 순 정렬. 축 종류가 plant인 변동은 들어오지 않는다
          items: { $ref: '#/components/schemas/Anomaly' }
        evidence:
          type: array
          items: { $ref: '#/components/schemas/Evidence' }
        alerts:
          type: array
          items: { $ref: '#/components/schemas/AlertEvent' }
        degraded:
          type: object
          nullable: true
          properties:
            roles:
              type: array
              items: { type: string, enum: [naming, causeLink, narration] }
            reasons: { type: array, items: { type: string } }
        notices:
          type: array
          description: 200으로 내려가는 예외 상태
          items:
            type: object
            required: [code, message]
            properties:
              code:
                type: string
                enum: [noAnomaly, generationDegraded, aJudgmentNotReceived,
                       batchFailed, marketCarriedOver]
              domain: { type: string, enum: [production, inventory, sales], nullable: true }
              message: { type: string }
              lastSuccessDate: { type: string, format: date, nullable: true }
    DomainStatus:
      type: object
      required: [domain, domainLabel, trafficLight, claim, judgmentSource, externalCauseAllowed]
      properties:
        domain: { type: string, enum: [production, inventory, sales] }
        domainLabel: { type: string, example: 생산 }
        trafficLight: { $ref: '#/components/schemas/TrafficLight' }
        statusText: { type: string, description: 신호등 옆에 병기하는 텍스트 }
        claim: { $ref: '#/components/schemas/Claim' }
        judgmentSource:
          type: string
          enum: [aJudgment, supplementaryAggregate, notReceived]
        topContributions:
          type: array
          items: { $ref: '#/components/schemas/Contribution' }
        externalCauseAllowed:
          type: boolean
          description: production은 항상 false. 국가 축이 없어 외부와 이을 자리가 없다
    Claim:
      type: object
      required: [text, footnotes]
      properties:
        text: { type: string }
        emphasis:
          type: array
          description: 강조 토큰이 적용될 구간. 문장당 3개 이하
          items:
            type: object
            properties:
              start: { type: integer }
              end: { type: integer }
        footnotes: { type: array, items: { type: integer } }
        degraded: { type: boolean, description: true면 템플릿 문장 }
    AlertEvent:
      type: object
      properties:
        countryCode: { type: string }
        countryName: { type: string }
        changeType: { type: string, enum: [new, raised, lowered, dropped] }
        previousLight: { type: string, enum: [red, yellow, none], nullable: true }
        currentLight: { type: string, enum: [red, yellow, none] }
```

#### GET/api/intel/creport/version/{reportId} 특정 C 리포트 조회

리포트 ID로 한 본을 조회한다. 응답 모양은 [[#GET/api/intel/creport/latest]]와 같다. 재생성본을 게시 전에 확인할 때와 과거 기준일을 다시 열 때 쓴다.

화면 [[TBL-UI-001#UI-10]] · 유스케이스 [[TBL-UC-001#UC-A4]] · 서비스 `CReportService.get_by_id`

게시되지 않은 버전은 관리 세션으로만 열린다. 현업 토큰으로 미게시본을 부르면 404다. 존재 여부를 알려 주지 않기 위함이다.

```yaml
/api/intel/creport/version/{reportId}:
  get:
    summary: 리포트 ID로 한 본 조회
    operationId: getCReportById
    security: [{ bearerToken: [] }, { adminSession: [] }]
    parameters:
      - name: reportId
        in: path
        required: true
        schema: { type: string }
    responses:
      '200':
        description: 리포트 한 본
        content:
          application/json:
            schema: { $ref: '#/components/schemas/CReport' }
      '404':
        description: 없거나 현업 토큰으로 부른 미게시본
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### GET/api/intel/market/series/{indicatorId} 시장지표 시계열 조회

지표 하나의 최근 값 목록을 돌려준다. [[TBL-UI-001#UI-10]] 근거 패널의 30일 스파크라인이 쓴다.

화면 [[TBL-UI-001#UI-10]] · 유스케이스 [[TBL-UC-001#UC-H2]] · 서비스 `MarketService.get_series`

시계열 자체는 [[#GET/api/intel/creport/latest]]의 후보 상세에도 들어 있다. 이 엔드포인트는 기간을 늘려 볼 때와 지표 바에서 바로 열 때만 쓴다. 지표는 국가·차종과 이어지지 않아 조인 축이 날짜뿐이다([[TBL-DOM-001#MarketPoint]]).

```yaml
/api/intel/market/series/{indicatorId}:
  get:
    summary: 지표 하나의 시계열
    operationId: getMarketSeries
    security: [{ bearerToken: [] }]
    parameters:
      - name: indicatorId
        in: path
        required: true
        schema: { type: string, example: BDI }
      - name: days
        in: query
        schema: { type: integer, default: 30, minimum: 7, maximum: 365 }
      - name: asOf
        in: query
        description: 이 날짜 기준으로 자른다. 비우면 리포트 기준일
        schema: { type: string, format: date }
    responses:
      '200':
        description: 시계열
        content:
          application/json:
            schema:
              type: object
              required: [indicatorId, name, points, latest]
              properties:
                indicatorId: { type: string }
                name: { type: string, example: 건화물운임 BDI }
                unit: { type: string, nullable: true }
                latest: { $ref: '#/components/schemas/MarketMetric' }
                points:
                  type: array
                  items:
                    type: object
                    properties:
                      date: { type: string, format: date }
                      value: { type: number }
      '404':
        description: 없는 지표
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### GET/api/intel/areport/domain/{domain} A 리포트 조회

A1 생산·A2 재고·A3 판매 중 한 도메인의 최신 게시 리포트를 돌려준다. 도메인 파라미터 하나로 세 화면을 모두 받는다.

화면 [[TBL-UI-001#UI-7]] [[TBL-UI-001#UI-8]] [[TBL-UI-001#UI-9]] · 유스케이스 [[TBL-UC-001#UC-H4]] · 서비스 `DomainReportService.get_latest_by_domain`

공통 덩어리는 여덟이다. 출처(대시보드명·스냅샷 일시), 트래킹 지표 3, 분해 차원 목록과 값 개수, 제약 경고, 버전 목록, 요약과 해설, 고정 문구, 내려받기. 도메인마다 다른 것은 `domainExtra`에 담고 셋의 모양이 다르다.

**응답 어디에도 뉴스·시장지표 인용이 없다.** 이것이 인수 기준이다([[TBL-PRD-001#R29]]). C 리포트로 가는 링크 필드도 두지 않는다([[TBL-UI-001#UI-10]] 0.3절 넷째).

`domainExtra`가 도메인별로 담는 것은 아래와 같다.

| domain | domainExtra |
|:--|:--|
| `production` | 누적 계획·실적·달성률, 계획 정본과 대안 기준 차이, 법인별 달성률 행(실적/계획), 완성차·반조립 구성비, 내수·수출 구성비 |
| `inventory` | 원천 부재 플래그와 배너 문구, 도매·소매 누계와 차이·비율, 국가별 체류 비중 행, MOS 상태, 원천을 받으면 채워질 자리 4 |
| `sales` | 단계별 흐름 4계열 누계와 구간 차이 2(4.8절), 국가별 격차 행을 쌓이는 쪽·덜어내는 쪽으로 나눈 두 묶음, 한 국가 차종별 도매·소매·차이 |

```yaml
/api/intel/areport/domain/{domain}:
  get:
    summary: 도메인 하나의 최신 A 리포트
    operationId: getDomainReport
    security: [{ bearerToken: [] }]
    parameters:
      - name: domain
        in: path
        required: true
        schema: { type: string, enum: [production, inventory, sales] }
    responses:
      '200':
        description: A 리포트 한 본
        content:
          application/json:
            schema: { $ref: '#/components/schemas/DomainReport' }
      '404':
        description: 그 도메인의 게시본이 아직 없음
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
components:
  schemas:
    DomainReport:
      type: object
      required: [domainReportId, domain, domainLabel, version, source,
                 trackingMetrics, breakdownDimensions, missingMetrics, versions,
                 summary, commentary, fixedNote, judgments, domainExtra]
      properties:
        domainReportId: { type: string }
        domain: { type: string, enum: [production, inventory, sales] }
        domainLabel: { type: string, example: A1 생산 리포트 }
        title: { type: string, example: 생산 실적 일일 인사이트 }
        version: { type: integer }
        publishedAt: { type: string, format: date-time }
        scopeNote: { type: string, example: 28개 법인 · 누적 기준 }
        source:
          type: object
          required: [dashboardName, snapshotAt, dataForm]
          properties:
            dashboardName: { type: string, example: 완성차 생산 대시보드 }
            dashboardId: { type: string }
            measureComposition: { type: string, example: 운영계획·사업계획·실적 × 일·월·누계·년 }
            snapshotAt: { type: string, format: date-time }
            snapshotId: { type: string }
            dataForm: { type: string, enum: [A, B], description: A는 IF 원장, B는 피벗 리포트 }
            absent:
              type: object
              nullable: true
              description: A2 전용. 재고 원천이 없다는 사실
              properties:
                absent: { type: boolean }
                message: { type: string }
        trackingMetrics:
          type: array
          minItems: 3
          maxItems: 3
          items: { $ref: '#/components/schemas/TrackingMetric' }
        breakdownDimensions:
          type: array
          items:
            type: object
            properties:
              name: { type: string, example: 생산법인 }
              valueCount: { type: integer, nullable: true, example: 28 }
        constraintWarnings:
          type: array
          items:
            type: object
            properties:
              code: { type: string, example: noDestinationCountry }
              message: { type: string }
        missingMetrics:
          type: array
          items: { $ref: '#/components/schemas/MissingMetric' }
        versions:
          type: array
          items:
            type: object
            properties:
              domainReportId: { type: string }
              version: { type: integer }
              publishedAt: { type: string, format: date-time }
              current: { type: boolean }
        summary: { $ref: '#/components/schemas/Claim' }
        commentary: { $ref: '#/components/schemas/Claim' }
        evidence:
          type: array
          items: { $ref: '#/components/schemas/Evidence' }
        fixedNote: { type: string, description: 화면 하단 고정 문구 }
        download:
          type: object
          nullable: true
          description: 파일 형식은 [확인 필요]
          properties:
            url: { type: string }
            format: { type: string }
        judgments:
          type: array
          description: 트래킹 지표별 변동 판정과 내부 분해
          items:
            type: object
            properties:
              judgmentId: { type: string }
              metric: { type: string }
              metricLabel: { type: string }
              periodKey: { $ref: '#/components/schemas/PeriodKey' }
              axisType: { type: string, enum: [country, plant] }
              value: { type: number }
              compareValue: { type: number, nullable: true }
              compareBasis: { type: string, enum: [plan, yoy, mom] }
              changeRate: { type: number, nullable: true }
              detected: { type: boolean }
              detectionUnit: { type: string, enum: [modelGroup, modelDetail] }
              contributionAvailable: { type: boolean }
              contributions:
                type: array
                items: { $ref: '#/components/schemas/Contribution' }
        domainExtra:
          oneOf:
            - $ref: '#/components/schemas/ProductionExtra'
            - $ref: '#/components/schemas/InventoryExtra'
            - $ref: '#/components/schemas/SalesExtra'
    ProductionExtra:
      type: object
      properties:
        cumulativePlan: { type: number, example: 2020531 }
        cumulativeActual: { type: number, example: 1739722 }
        achievementRate: { type: number, example: 0.861 }
        planBasis: { type: string, enum: [businessPlan, operationPlan] }
        planAlternativeDiff:
          type: number
          description: 다른 계획 기준과의 누적 차이. 고정 문구에 적는다
          example: 266381
        achievementComputed:
          type: boolean
          description: 원본 진도율 컬럼은 개별 행이 0이라 쓰지 않고 직접 계산한다. 항상 true
        entityAchievements:
          type: array
          items:
            type: object
            properties:
              entityCode: { type: string }
              entityName: { type: string, example: HMGMA }
              countryName: { type: string, nullable: true }
              achievementRate: { type: number, example: 0.459 }
              actual: { type: number }
              plan: { type: number }
        compositionCbu:
          type: object
          properties:
            cbu: { type: number, example: 0.78 }
            ckd: { type: number, example: 0.22 }
        compositionTrade:
          type: object
          properties:
            domestic: { type: number, example: 0.63 }
            export: { type: number, example: 0.37 }
    InventoryExtra:
      type: object
      properties:
        sourceAbsent: { type: boolean, description: 항상 true. 재고 원천이 없다 }
        bannerText: { type: string }
        wholesaleCumulative: { type: number, example: 677201 }
        retailCumulative: { type: number, example: 643097 }
        gap: { type: number, example: 34104 }
        gapRate: { type: number, example: 0.050 }
        derivationType: { type: string, enum: [derived, measured] }
        mosStatus:
          type: string
          enum: [unstable, usable]
          description: 음수와 0이 섞여 있으면 unstable. 값 대신 상태를 보여 준다
        countryStay:
          type: array
          items:
            type: object
            properties:
              countryCode: { type: string, example: B07 }
              countryName: { type: string, example: 칠레 }
              gapRate: { type: number, example: 0.252 }
              gapUnits: { type: number, example: 3064 }
        placeholderMetrics:
          type: array
          minItems: 4
          maxItems: 4
          description: 원천을 받으면 채워질 자리. 값을 넣지 않는다
          items:
            type: object
            properties:
              code: { type: string, enum: [afloat, awaitingShipment, entityVsDealer, countryDailySeries] }
              label: { type: string }
    SalesExtra:
      type: object
      properties:
        stageFlow: { $ref: '#/components/schemas/StageFlow' }
        accumulating:
          type: array
          description: 쌓이는 쪽. dealerStageGapRate 양수
          items: { $ref: '#/components/schemas/StageFlow' }
        releasing:
          type: array
          description: 덜어내는 쪽. dealerStageGapRate 음수
          items: { $ref: '#/components/schemas/StageFlow' }
        modelBreakdown:
          type: object
          description: 한 국가 안의 차종별. 감지는 그룹 단위, 세부 차종은 분해에만
          properties:
            countryCode: { type: string }
            countryName: { type: string }
            totalGap: { type: number }
            rows:
              type: array
              items:
                type: object
                properties:
                  modelGroup: { type: string }
                  wholesale: { type: number }
                  retail: { type: number }
                  gap: { type: number }
                  sortOrder: { type: integer }
```

#### GET/api/intel/areport/version/{domainReportId} A 리포트 특정 버전 조회

버전 목록에서 고른 과거 지면을 연다. 응답 모양은 [[#GET/api/intel/areport/domain/{domain}]]과 같다.

화면 [[TBL-UI-001#UI-7]] [[TBL-UI-001#UI-8]] [[TBL-UI-001#UI-9]] · 유스케이스 [[TBL-UC-001#UC-H4]] · 서비스 `DomainReportService.get_by_id`

```yaml
/api/intel/areport/version/{domainReportId}:
  get:
    summary: A 리포트 한 버전
    operationId: getDomainReportById
    security: [{ bearerToken: [] }]
    parameters:
      - name: domainReportId
        in: path
        required: true
        schema: { type: string }
    responses:
      '200':
        description: A 리포트 한 본
        content:
          application/json:
            schema: { $ref: '#/components/schemas/DomainReport' }
      '404':
        description: 없는 버전
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

### 3.2 관리 — 적재 (세션)

#### POST/api/admin/ingest/preflight 적재 사전 검증

파일을 올려 원본을 보존하고 형태를 판별해 미리보기를 돌려준다. **아직 표준 테이블에 넣지 않는다.**

화면 [[TBL-UI-001#UI-11]] · 유스케이스 [[TBL-UC-001#UC-A1]] 2~4단계 · 서비스 `IngestService.preflight`

원본 파일은 판별 결과와 무관하게 먼저 보존한다([[TBL-PRD-001#N7]]). 판별이 실패해도 원본은 남고 응답은 422다.

형태 A면 `previewA`, 형태 B면 `previewB`가 차고 나머지는 `null`이다. 형태 B는 병합 헤더를 펼쳐 기간 구분 목록과 지표 종류 목록을 돌려준다([[TBL-INFRA-001#C15]] [[TBL-INFRA-001#C18]]). 계획 두 종류와 도매 두 기준이 함께 들어온 것도 여기서 알린다([[TBL-DOM-001#ThresholdSetting]]).

응답의 `preflightId`를 [[#POST/api/admin/ingest/commit]]에 넘긴다. 이 값은 일회용이며 만료되면 409다.

```yaml
/api/admin/ingest/preflight:
  post:
    summary: 파일 업로드와 형태 판별, 미리보기
    operationId: preflightIngest
    security: [{ adminSession: [] }]
    requestBody:
      required: true
      content:
        multipart/form-data:
          schema:
            type: object
            required: [sourceType, file]
            properties:
              sourceType:
                type: string
                enum: [vehicleProduction, vehicleSales, news, bloomberg, marklines, crosswalk]
              file: { type: string, format: binary }
              fileBaseDate:
                type: string
                format: date
                description: 형태 B에서 판별이 안 되면 관리자가 넣는다
    responses:
      '200':
        description: 판별과 미리보기 결과
        content:
          application/json:
            schema:
              type: object
              required: [preflightId, sourceType, originalStored, formDetection,
                         idempotencyUnit, regression, overwrite]
              properties:
                preflightId: { type: string }
                sourceType: { type: string }
                originalStored: { type: boolean }
                originalPath: { type: string }
                formDetection: { $ref: '#/components/schemas/FormDetection' }
                previewA:
                  type: object
                  nullable: true
                  properties:
                    columnCheck:
                      type: array
                      items:
                        type: object
                        properties:
                          column: { type: string }
                          expected: { type: string }
                          found: { type: string, nullable: true }
                    baseDateFrom: { type: string, format: date }
                    baseDateTo: { type: string, format: date }
                    rowCount: { type: integer }
                    nullCount: { type: integer }
                    futureSkeletonRowCount: { type: integer }
                previewB:
                  type: object
                  nullable: true
                  properties:
                    periodTypes:
                      type: array
                      items: { type: string, enum: [day, month, cumulative, year] }
                    periodTypeLabels:
                      type: array
                      items: { type: string }
                      example: ['일', '월', '누계', '년']
                    measureTypes:
                      type: array
                      description: 지표 종류. 인프라 확정 아홉 값
                      items:
                        type: string
                        enum: [operationPlan, businessPlan, actual, progressRate, yoyRate,
                               shipment, actualWholesale, officialWholesale, retail]
                    dimensionCombinationCount: { type: integer }
                    totalRowCount: { type: integer, description: 총계 행. 세되 판정에서 뺀다 }
                    nullCount: { type: integer }
                fileBaseDate: { type: string, format: date, nullable: true }
                fileBaseDateSource: { type: string, enum: [detected, manual] }
                dualValues:
                  type: array
                  description: 둘씩 들어온 값. 둘 다 저장하고 정본은 설정에서 고른다
                  items:
                    type: object
                    properties:
                      kind: { type: string, enum: [plan, wholesale] }
                      options: { type: array, items: { type: string } }
                      defaultOption: { type: string }
                idempotencyUnit:
                  type: string
                  enum: [baseDate, fileBaseDate]
                  description: 형태 A는 baseDate, 형태 B는 fileBaseDate
                regression:
                  type: object
                  properties:
                    rowCount: { type: integer }
                    missingRate: { type: number }
                    matchRate: { type: number }
                    duplicateRate: { type: number }
                    previous:
                      type: object
                      nullable: true
                      properties:
                        rowCount: { type: integer }
                        missingRate: { type: number }
                        matchRate: { type: number }
                        duplicateRate: { type: number }
                        dataForm: { type: string, enum: [A, B] }
                    abrupt: { type: boolean }
                    abruptReasons:
                      type: array
                      items: { type: string, enum: [rowCount, missingRate, matchRate, duplicateRate, formChanged] }
                overwrite:
                  type: object
                  properties:
                    willOverwrite: { type: boolean }
                    target: { type: string, description: 겹치는 기준일 또는 기준일자 범위 }
                    message: { type: string }
      '413':
        description: 파일 한도 초과
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
      '422':
        description: 형태 판별 실패. 적재 버튼을 막는다
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
            example:
              type: /problems/form-undetected
              title: 형태를 판별하지 못했습니다
              status: 422
              detail: 병합 헤더가 2~3줄이 아니고 기준일자 컬럼도 없습니다
              mismatch:
                - { expected: 기준일자 컬럼, found: 없음 }
                - { expected: 병합 헤더 2~3줄, found: 5줄 }
```

#### POST/api/admin/ingest/commit 적재 실행

사전 검증 결과를 확정해 표준 테이블에 넣는다. 표준화, 미매핑 수집, 회귀 검사를 함께 돌린다.

화면 [[TBL-UI-001#UI-11]] · 유스케이스 [[TBL-UC-001#UC-A1]] 5~6단계 · 포함 [[TBL-UC-001#UC-S6]] · 서비스 `IngestService.commit`

멱등 단위는 형태가 정한다. 형태 A는 기준일자 단위, 형태 B는 파일 기준일 단위로 통째 덮어쓴다([[TBL-INFRA-001#C6]] [[TBL-DOM-001#IngestFile]]). 겹치는데 `confirmOverwrite`가 없으면 409다.

`backfillMode`가 참이면 이 파일로 도는 배치가 후보·근접도·신호등까지만 채우고 LLM 세 역할을 생략한다([[TBL-UC-001#UC-A1]] 6a).

회귀 검사가 급변으로 판정하면 **적재는 완료하되** `autoBatchBlocked`가 참으로 돌아온다. 배치 자동 실행만 막힌다([[TBL-UC-001#UC-S6]]).

```yaml
/api/admin/ingest/commit:
  post:
    summary: 사전 검증 결과를 확정해 적재
    operationId: commitIngest
    security: [{ adminSession: [] }]
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [preflightId]
            properties:
              preflightId: { type: string }
              fileBaseDate: { type: string, format: date }
              planSource: { type: string, enum: [businessPlan, operationPlan], default: businessPlan }
              wholesaleSource: { type: string, enum: [officialWholesale, actualWholesale], default: officialWholesale }
              backfillMode: { type: boolean, default: false }
              confirmOverwrite: { type: boolean, default: false }
    responses:
      '201':
        description: 적재 완료
        content:
          application/json:
            schema:
              type: object
              required: [ingestFileId, dataForm, idempotencyUnit, rowCount, autoBatchBlocked]
              properties:
                ingestFileId: { type: string }
                sourceType: { type: string }
                dataForm: { type: string, enum: [A, B] }
                schemaVersion: { type: string }
                fileBaseDate: { type: string, format: date, nullable: true }
                idempotencyUnit: { type: string, enum: [baseDate, fileBaseDate] }
                periodTypes:
                  type: array
                  items: { type: string, enum: [day, month, cumulative, year] }
                rowCount: { type: integer }
                missingRate: { type: number }
                matchRate: { type: number }
                duplicateRate: { type: number }
                unmappedCollected: { type: integer }
                regressionAbrupt: { type: boolean }
                autoBatchBlocked: { type: boolean }
                backfillMode: { type: boolean }
                ingestSequence: { type: integer, description: 같은 기준일 파일의 몇 번째인가 }
                ingestedAt: { type: string, format: date-time }
      '409':
        description: preflightId 만료 또는 덮어쓰기 미확인
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### POST/api/admin/ingest/unblock/{ingestFileId} 회귀 차단 해제

회귀 급변으로 막힌 배치 자동 실행을 관리자가 확인하고 푼다.

화면 [[TBL-UI-001#UI-11]] · 유스케이스 [[TBL-UC-001#UC-S6]] 4단계 · 서비스 `IngestService.unblock_regression`

사유를 반드시 받는다. 왜 급변을 정상으로 봤는지가 남지 않으면 다음 사람이 같은 판단을 다시 할 수 없다. 해제해도 회귀 검사 수치는 그대로 이력에 남는다.

```yaml
/api/admin/ingest/unblock/{ingestFileId}:
  post:
    summary: 회귀 급변으로 막힌 배치 자동 실행을 해제
    operationId: unblockRegression
    security: [{ adminSession: [] }]
    parameters:
      - name: ingestFileId
        in: path
        required: true
        schema: { type: string }
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [reason]
            properties:
              reason: { type: string, minLength: 1 }
    responses:
      '200':
        description: 해제됨
        content:
          application/json:
            schema:
              type: object
              properties:
                ingestFileId: { type: string }
                autoBatchBlocked: { type: boolean, enum: [false] }
                unblockedBy: { type: string }
                unblockedAt: { type: string, format: date-time }
                reason: { type: string }
      '404':
        description: 없는 적재 건
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### GET/api/admin/ingest/history 적재 이력 조회

최근 적재 목록이다. 일시, 소스, 형태, 기간 단위, 행수, 결측률, 매칭률, 결과를 준다.

화면 [[TBL-UI-001#UI-11]] · 유스케이스 [[TBL-UC-001#UC-A1]] 6단계 · 서비스 `IngestService.list_history`

```yaml
/api/admin/ingest/history:
  get:
    summary: 적재 이력 목록
    operationId: listIngestHistory
    security: [{ adminSession: [] }]
    parameters:
      - name: sourceType
        in: query
        schema:
          type: string
          enum: [vehicleProduction, vehicleSales, news, bloomberg, marklines, crosswalk]
      - name: from
        in: query
        schema: { type: string, format: date }
      - name: to
        in: query
        schema: { type: string, format: date }
      - name: limit
        in: query
        schema: { type: integer, default: 50, maximum: 200 }
      - name: cursor
        in: query
        schema: { type: string }
    responses:
      '200':
        description: 목록
        content:
          application/json:
            schema:
              type: object
              properties:
                items:
                  type: array
                  items:
                    type: object
                    properties:
                      ingestFileId: { type: string }
                      ingestedAt: { type: string, format: date-time }
                      sourceType: { type: string }
                      dataForm: { type: string, enum: [A, B] }
                      idempotencyUnit: { type: string, enum: [baseDate, fileBaseDate] }
                      fileBaseDate: { type: string, format: date, nullable: true }
                      rowCount: { type: integer }
                      missingRate: { type: number }
                      matchRate: { type: number }
                      duplicateRate: { type: number }
                      regressionAbrupt: { type: boolean }
                      autoBatchBlocked: { type: boolean }
                      result: { type: string, enum: [success, failed] }
                      failureReason: { type: string, nullable: true }
                nextCursor: { type: string, nullable: true }
```

#### GET/api/admin/ingest/file/{ingestFileId} 적재 1건 상세

이력에서 행을 눌렀을 때 그 파일의 판별 근거와 회귀 수치를 펼친다.

화면 [[TBL-UI-001#UI-11]] · 유스케이스 [[TBL-UC-001#UC-A1]] · 서비스 `IngestService.get_file`

```yaml
/api/admin/ingest/file/{ingestFileId}:
  get:
    summary: 적재 1건 상세
    operationId: getIngestFile
    security: [{ adminSession: [] }]
    parameters:
      - name: ingestFileId
        in: path
        required: true
        schema: { type: string }
    responses:
      '200':
        description: 상세
        content:
          application/json:
            schema:
              type: object
              properties:
                ingestFileId: { type: string }
                sourceType: { type: string }
                formDetection: { $ref: '#/components/schemas/FormDetection' }
                fileBaseDate: { type: string, format: date, nullable: true }
                periodTypes:
                  type: array
                  items: { type: string, enum: [day, month, cumulative, year] }
                measureTypes: { type: array, items: { type: string } }
                rowCount: { type: integer }
                missingRate: { type: number }
                matchRate: { type: number }
                duplicateRate: { type: number }
                regressionAbrupt: { type: boolean }
                abruptReasons: { type: array, items: { type: string } }
                autoBatchBlocked: { type: boolean }
                unblock:
                  type: object
                  nullable: true
                  properties:
                    reason: { type: string }
                    unblockedBy: { type: string }
                    unblockedAt: { type: string, format: date-time }
                originalPath: { type: string }
                ingestSequence: { type: integer }
                ingestedAt: { type: string, format: date-time }
      '404':
        description: 없는 적재 건
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

### 3.3 관리 — 배치 (세션)

#### GET/api/admin/batch/status 배치 상태 조회

지금 도는 배치가 있는지, 자동 실행이 막혀 있는지, 다음 예정 시각이 언제인지를 준다.

화면 [[TBL-UI-001#UI-12]] · 유스케이스 [[TBL-UC-001#UC-S1]] · 서비스 `BatchService.get_status`

`autoRunBlocked`가 참이면 회귀 급변 때문이며 `blockedBy`가 어느 적재 건인지 가리킨다. 푸는 것은 [[#POST/api/admin/ingest/unblock/{ingestFileId}]]다.

```yaml
/api/admin/batch/status:
  get:
    summary: 현재 배치 상태
    operationId: getBatchStatus
    security: [{ adminSession: [] }]
    responses:
      '200':
        description: 상태
        content:
          application/json:
            schema:
              type: object
              required: [autoRunBlocked]
              properties:
                running:
                  type: object
                  nullable: true
                  properties:
                    batchRunId: { type: string }
                    baseDate: { type: string, format: date }
                    currentStage: { type: integer, minimum: 1, maximum: 8 }
                    currentStageName: { type: string }
                    startedAt: { type: string, format: date-time }
                    elapsedSec: { type: integer }
                lastSuccess:
                  type: object
                  nullable: true
                  properties:
                    batchRunId: { type: string }
                    baseDate: { type: string, format: date }
                    reportId: { type: string }
                    finishedAt: { type: string, format: date-time }
                autoRunBlocked: { type: boolean }
                blockedBy:
                  type: object
                  nullable: true
                  properties:
                    ingestFileId: { type: string }
                    abruptReasons: { type: array, items: { type: string } }
                nextScheduledAt: { type: string, format: date-time, nullable: true }
                queued:
                  type: array
                  description: 재생성 때문에 대기 중인 정기 배치
                  items:
                    type: object
                    properties:
                      batchRunId: { type: string }
                      baseDate: { type: string, format: date }
                      waitingFor: { type: string }
```

#### GET/api/admin/batch/history 배치 이력 목록

기준일, 시작 시각, 소요, 상태, 실패 단계, 게시 여부를 목록으로 준다.

화면 [[TBL-UI-001#UI-12]] · 유스케이스 [[TBL-UC-001#UC-A4]] · 서비스 `BatchService.list_runs`

```yaml
/api/admin/batch/history:
  get:
    summary: 배치 목록
    operationId: listBatchRuns
    security: [{ adminSession: [] }]
    parameters:
      - name: from
        in: query
        schema: { type: string, format: date }
      - name: to
        in: query
        schema: { type: string, format: date }
      - name: status
        in: query
        schema: { type: string, enum: [success, failed, degraded, running, queued] }
      - name: limit
        in: query
        schema: { type: integer, default: 50, maximum: 200 }
      - name: cursor
        in: query
        schema: { type: string }
    responses:
      '200':
        description: 목록
        content:
          application/json:
            schema:
              type: object
              properties:
                items:
                  type: array
                  items:
                    type: object
                    properties:
                      batchRunId: { type: string }
                      baseDate: { type: string, format: date }
                      trigger: { type: string, enum: [schedule, rerun] }
                      startedAt: { type: string, format: date-time }
                      durationSec: { type: integer, nullable: true }
                      status: { type: string, enum: [success, failed, degraded, running, queued] }
                      failedStage: { type: integer, nullable: true, minimum: 1, maximum: 8 }
                      backfillMode: { type: boolean }
                      published: { type: boolean }
                      reportId: { type: string, nullable: true }
                nextCursor: { type: string, nullable: true }
```

#### GET/api/admin/batch/run/{batchRunId} 배치 상세 조회

8단계 결과와, 재생성이면 당시와의 비교 표를 준다.

화면 [[TBL-UI-001#UI-12]] · 유스케이스 [[TBL-UC-001#UC-A4]] 3단계 · 서비스 `BatchService.get_run`

**단계 이름은 인프라 확정 이름 그대로 내려간다**([[TBL-INFRA-001#C19]]). 적재 / A 판정 읽기 / 사건 묶음 / 결합 / 원인 후보·근접도·신호등 / 사건 명명 / 연관 설명 / 서술·검증·게시. `executor`가 1~5는 `code`, 6~8은 `llm`이며 강등은 6~8에만 생긴다.

비교 표의 `reproducibilityViolation`은 변동·후보·근접도·신호등이 달라졌을 때만 참이다. 참이면 그 자체가 재현성 위반이며 `cause`를 반드시 채운다([[TBL-PRD-001#N3]]). 설명과 문장이 달라진 것은 위반이 아니다.

```yaml
/api/admin/batch/run/{batchRunId}:
  get:
    summary: 배치 한 건의 8단계 결과와 비교 표
    operationId: getBatchRun
    security: [{ adminSession: [] }]
    parameters:
      - name: batchRunId
        in: path
        required: true
        schema: { type: string }
    responses:
      '200':
        description: 상세
        content:
          application/json:
            schema:
              type: object
              required: [batchRunId, baseDate, status, stages]
              properties:
                batchRunId: { type: string }
                baseDate: { type: string, format: date }
                trigger: { type: string, enum: [schedule, rerun] }
                startStage: { type: integer, minimum: 1, maximum: 8 }
                backfillMode: { type: boolean }
                status: { type: string, enum: [success, failed, degraded, running, queued] }
                failedStage: { type: integer, nullable: true }
                startedAt: { type: string, format: date-time }
                durationSec: { type: integer, nullable: true }
                published: { type: boolean }
                reportId: { type: string, nullable: true }
                settingVersion: { type: string }
                stages:
                  type: array
                  minItems: 8
                  maxItems: 8
                  items:
                    type: object
                    required: [stageNo, name, executor, status]
                    properties:
                      stageNo: { type: integer, minimum: 1, maximum: 8 }
                      name:
                        type: string
                        enum: ['적재', 'A 판정 읽기', '사건 묶음', '결합',
                               '원인 후보·근접도·신호등', '사건 명명', '연관 설명', '서술·검증·게시']
                      executor: { type: string, enum: [code, llm] }
                      status: { type: string, enum: [ok, degraded, failed, skipped, running, pending] }
                      durationSec: { type: integer, nullable: true }
                      processedCount: { type: integer, nullable: true }
                      countLabel: { type: string, nullable: true, example: 행 }
                      llmCallCount: { type: integer, default: 0 }
                      tokenCount: { type: integer, default: 0 }
                      degradeReason: { type: string, nullable: true }
                comparison:
                  type: object
                  nullable: true
                  description: 재생성일 때만
                  properties:
                    baselineBatchRunId: { type: string }
                    note:
                      type: string
                      example: 변동·후보·근접도·신호등은 같은 입력이면 같아야 합니다. 설명과 문장은 새로 생성되므로 달라질 수 있습니다.
                    rows:
                      type: array
                      items:
                        type: object
                        properties:
                          field: { type: string }
                          before: { type: string, nullable: true }
                          after: { type: string, nullable: true }
                          cause:
                            type: string
                            enum: [mapping, setting, lateData, aJudgment, llmRegeneration]
                          reproducibilityViolation: { type: boolean }
                llmFreeComparison:
                  type: object
                  nullable: true
                  description: LLM 없는 실행과의 대조. 신호등과 후보 순서가 같아야 한다
                  properties:
                    comparedBatchRunId: { type: string }
                    trafficLightMatch: { type: boolean }
                    candidateOrderMatch: { type: boolean }
                    mismatchCount: { type: integer }
      '404':
        description: 없는 배치
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### POST/api/admin/batch/rerun 배치 실행·재생성

기준일과 시작 단계를 정해 배치를 돌린다. 지정 단계부터 다시 돌리는 것과 특정 일자를 재생성하는 것이 같은 호출이다.

화면 [[TBL-UI-001#UI-12]] · 유스케이스 [[TBL-UC-001#UC-A4]] 1~2단계 · 서비스 `BatchService.request_rerun`

그 기준일의 데이터 스냅샷, 당시 A 판정 스냅샷, 당시 설정 버전으로 돈다. **기존 게시본을 덮어쓰지 않는다.** 결과는 항상 새 버전이고 게시 전환은 [[#POST/api/admin/batch/publish/{reportId}]]가 따로 한다.

`backfillMode`가 참이면 LLM 세 역할을 생략하고 5단계까지만 채운다. 신호등과 후보 순서가 정상 실행과 같은지 대조하는 용도다([[TBL-UI-001#UI-12]] S-2).

같은 기준일 배치가 돌고 있으면 409다. 재생성 중 정기 배치 시각이 오면 정기 배치가 대기한다([[TBL-UC-001#UC-A4]] 2a).

```yaml
/api/admin/batch/rerun:
  post:
    summary: 지정 기준일을 지정 단계부터 실행
    operationId: rerunBatch
    security: [{ adminSession: [] }]
    requestBody:
      required: true
      content:
        application/json:
          schema:
            type: object
            required: [baseDate, startStage]
            properties:
              baseDate: { type: string, format: date }
              startStage: { type: integer, minimum: 1, maximum: 8 }
              backfillMode: { type: boolean, default: false }
              compareWith:
                type: string
                nullable: true
                description: 비교 표의 기준이 될 배치. 비우면 그 기준일의 게시본
    responses:
      '202':
        description: 실행 요청됨
        content:
          application/json:
            schema:
              type: object
              properties:
                batchRunId: { type: string }
                baseDate: { type: string, format: date }
                startStage: { type: integer }
                status: { type: string, enum: [queued, running] }
                queuedBehind: { type: string, nullable: true }
      '400':
        description: 시작 단계가 1~8 밖
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
      '409':
        description: 같은 기준일 배치가 실행 중이거나 회귀 차단 중
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### POST/api/admin/batch/publish/{reportId} 재생성본 게시 전환

재생성으로 만든 버전을 게시본으로 바꾼다. 직전 게시본은 미게시 상태로 내려가고 지워지지 않는다.

화면 [[TBL-UI-001#UI-12]] · 유스케이스 [[TBL-UC-001#UC-A4]] 4단계 · 서비스 `BatchService.publish_version`

게시 전환이 끝나면 워치리스트 변화 이벤트를 다시 계산한다([[TBL-UC-001#UC-S5]]). 직전 게시본과 국가별 신호등을 비교한 결과가 `alertsRecomputed`에 건수로 온다.

```yaml
/api/admin/batch/publish/{reportId}:
  post:
    summary: 리포트 한 버전을 게시본으로 전환
    operationId: publishReportVersion
    security: [{ adminSession: [] }]
    parameters:
      - name: reportId
        in: path
        required: true
        schema: { type: string }
    responses:
      '200':
        description: 게시됨
        content:
          application/json:
            schema:
              type: object
              properties:
                reportId: { type: string }
                baseDate: { type: string, format: date }
                version: { type: integer }
                published: { type: boolean, enum: [true] }
                previousPublishedReportId: { type: string, nullable: true }
                alertsRecomputed: { type: integer }
                publishedAt: { type: string, format: date-time }
      '409':
        description: 이미 게시본
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

### 3.4 관리 — 마스터 (세션)

#### GET/api/admin/master/summary 마스터 조회

국가 매칭률 셋과 마스터 버전 이력을 준다.

화면 [[TBL-UI-001#UI-13]] · 유스케이스 [[TBL-UC-001#UC-A2]] · 서비스 `MasterService.get_summary`

매칭률 셋은 판매→국가, 생산→국가, 뉴스→국가다([[TBL-PRD-001#R2]]). 판매는 대리점 코드 앞 세 자리로 붙는다(미국 B28AB, 캐나다 B06AA).

```yaml
/api/admin/master/summary:
  get:
    summary: 매칭률과 마스터 버전 이력
    operationId: getMasterSummary
    security: [{ adminSession: [] }]
    responses:
      '200':
        description: 요약
        content:
          application/json:
            schema:
              type: object
              properties:
                matchRates:
                  type: array
                  minItems: 3
                  maxItems: 3
                  items:
                    type: object
                    properties:
                      kind:
                        type: string
                        enum: [salesToCountry, productionToCountry, newsToCountry]
                      label: { type: string, example: 판매 → 국가 }
                      rate: { type: number }
                      previousRate: { type: number, nullable: true }
                      delta: { type: number, nullable: true }
                unmappedTotal: { type: integer }
                masterVersions:
                  type: array
                  items:
                    type: object
                    properties:
                      version: { type: string }
                      appliedAt: { type: string, format: date-time }
                      changeCount: { type: integer }
                      appliedBy: { type: string }
```

#### GET/api/admin/master/unmapped 미매핑 목록 조회

소스, 값, 등장 횟수, 첫 등장일을 준다. 엑셀 내려받기 주소도 함께 준다.

화면 [[TBL-UI-001#UI-13]] · 유스케이스 [[TBL-UC-001#UC-A2]] 1단계 · 서비스 `MasterService.list_unmapped`

`format=xlsx`로 부르면 파일을 바로 내려받는다. `json`이 기본이다.

```yaml
/api/admin/master/unmapped:
  get:
    summary: 미매핑 값 목록
    operationId: listUnmapped
    security: [{ adminSession: [] }]
    parameters:
      - name: source
        in: query
        schema: { type: string, enum: [sales, production, news] }
      - name: format
        in: query
        schema: { type: string, enum: [json, xlsx], default: json }
      - name: limit
        in: query
        schema: { type: integer, default: 200, maximum: 2000 }
      - name: cursor
        in: query
        schema: { type: string }
    responses:
      '200':
        description: 목록 또는 엑셀 파일
        content:
          application/json:
            schema:
              type: object
              properties:
                items:
                  type: array
                  items:
                    type: object
                    properties:
                      source: { type: string, enum: [sales, production, news] }
                      value: { type: string }
                      occurrenceCount: { type: integer }
                      firstSeenDate: { type: string, format: date }
                total: { type: integer }
                nextCursor: { type: string, nullable: true }
                downloadUrl: { type: string }
          application/vnd.openxmlformats-officedocument.spreadsheetml.sheet:
            schema: { type: string, format: binary }
```

#### POST/api/admin/master/crosswalk 크로스워크 업로드

보강한 크로스워크 시트를 올려 매핑 차이를 미리 본다. **아직 마스터를 바꾸지 않는다.**

화면 [[TBL-UI-001#UI-13]] · 유스케이스 [[TBL-UC-001#UC-A2]] 2~3단계 · 서비스 `MasterService.upload_crosswalk`

추가·변경·삭제와 영향 받는 국가·공장을 돌려준다. 삭제되는 매핑이 있으면 `willBecomeUnmapped`에 그로 인해 미매핑이 될 값의 건수가 온다([[TBL-UC-001#UC-A2]] 3a). 응답의 `crosswalkUploadId`를 [[#POST/api/admin/master/confirm/{crosswalkUploadId}]]에 넘긴다.

```yaml
/api/admin/master/crosswalk:
  post:
    summary: 크로스워크 시트 업로드와 차이 미리보기
    operationId: uploadCrosswalk
    security: [{ adminSession: [] }]
    requestBody:
      required: true
      content:
        multipart/form-data:
          schema:
            type: object
            required: [file]
            properties:
              file: { type: string, format: binary }
    responses:
      '200':
        description: 차이 미리보기
        content:
          application/json:
            schema:
              type: object
              required: [crosswalkUploadId, counts]
              properties:
                crosswalkUploadId: { type: string }
                sheets: { type: array, items: { type: string } }
                counts:
                  type: object
                  properties:
                    added: { type: integer }
                    changed: { type: integer }
                    removed: { type: integer }
                diff:
                  type: object
                  properties:
                    added:
                      type: array
                      items:
                        type: object
                        properties:
                          sheet: { type: string }
                          key: { type: string }
                          value: { type: string }
                    changed:
                      type: array
                      items:
                        type: object
                        properties:
                          sheet: { type: string }
                          key: { type: string }
                          before: { type: string }
                          after: { type: string }
                    removed:
                      type: array
                      items:
                        type: object
                        properties:
                          sheet: { type: string }
                          key: { type: string }
                          before: { type: string }
                affectedCountries: { type: array, items: { type: string } }
                affectedPlants: { type: array, items: { type: string } }
                willBecomeUnmapped: { type: integer }
      '413':
        description: 파일 한도 초과
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
      '422':
        description: 시트 구조가 크로스워크가 아님
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

#### POST/api/admin/master/confirm/{crosswalkUploadId} 마스터 확정

미리 본 차이를 확정해 마스터를 새 버전으로 갱신한다.

화면 [[TBL-UI-001#UI-13]] · 유스케이스 [[TBL-UC-001#UC-A2]] 4단계 · 서비스 `MasterService.confirm_crosswalk`

**다음 배치부터 적용된다. 과거 판정을 소급해 바꾸지 않는다.** 과거를 다시 계산하려면 [[#POST/api/admin/batch/rerun]]을 쓴다.

삭제되는 매핑이 있는데 `confirmRemoval`이 없으면 409다.

```yaml
/api/admin/master/confirm/{crosswalkUploadId}:
  post:
    summary: 매핑 차이를 확정해 마스터 새 버전 생성
    operationId: confirmCrosswalk
    security: [{ adminSession: [] }]
    parameters:
      - name: crosswalkUploadId
        in: path
        required: true
        schema: { type: string }
    requestBody:
      content:
        application/json:
          schema:
            type: object
            properties:
              confirmRemoval: { type: boolean, default: false }
    responses:
      '201':
        description: 갱신됨
        content:
          application/json:
            schema:
              type: object
              properties:
                masterVersion: { type: string }
                appliedAt: { type: string, format: date-time }
                appliedBy: { type: string }
                changeCount: { type: integer }
                appliesFromNextBatch: { type: boolean, enum: [true] }
                pastJudgmentsUnchanged: { type: boolean, enum: [true] }
      '409':
        description: 삭제되는 매핑 확인 없음 또는 업로드 만료
        content:
          application/problem+json:
            schema: { $ref: '#/components/schemas/Problem' }
```

## 4. 스키마

엔드포인트가 공유하는 조각들이다. OpenAPI의 `components.schemas` 아래에 들어간다. REST 문서의 항목은 엔드포인트뿐이라 아래 열셋은 절로 두고 번호로 가리킨다.

### 4.1 problem 에러 본문

RFC 9457 `application/problem+json`. 확장 필드는 2장 표를 따른다.

```yaml
Problem:
  type: object
  required: [type, title, status]
  properties:
    type: { type: string, description: 폐쇄망 내부 상대 경로. 외부 URL을 두지 않는다 }
    title: { type: string }
    status: { type: integer }
    detail: { type: string }
    instance: { type: string }
    errors:
      type: array
      items:
        type: object
        properties:
          field: { type: string }
          message: { type: string }
    resource: { type: string }
    nextScheduledAt: { type: string, format: date-time }
    mismatch:
      type: array
      items:
        type: object
        properties:
          expected: { type: string }
          found: { type: string }
    preflightId: { type: string }
    idempotencyUnit: { type: string, enum: [baseDate, fileBaseDate] }
    target: { type: string }
    ingestFileId: { type: string }
    abruptReasons: { type: array, items: { type: string } }
    batchRunId: { type: string }
    reportId: { type: string }
    startStage: { type: integer }
    willBecomeUnmapped: { type: integer }
    limitBytes: { type: integer }
```

### 4.2 period_key 기간 키

시간 축 두 칸이다. 단일 기준일을 쓰지 않는다([[TBL-INFRA-001#C15]]).

```yaml
PeriodKey:
  type: object
  required: [periodType, fileBaseDate]
  properties:
    periodType:
      type: string
      enum: [day, month, cumulative, year]
      description: 일·월·누계·년
    periodTypeLabel: { type: string, example: 누계 }
    fileBaseDate:
      type: string
      format: date
      description: 형태 A는 행의 기준일자와 같고 형태 B는 파일이 붙여 준 날
    dataForm: { type: string, enum: [A, B] }
```

### 4.3 anomaly 변동

C 리포트가 다루는 변동 한 건. [[TBL-DOM-001#Anomaly]]의 속성을 그대로 옮긴다.

`axisType`이 `plant`인 변동은 이 배열에 들어오지 않는다(1.6절). `source`가 `supplementaryAggregate`면 기여 분해가 비고 화면이 "보완 집계"를 적는다. `derived`면 4.8절에서 유도된 재고 신호이며 화면이 유도 표기를 붙인다.

```yaml
Anomaly:
  type: object
  required: [anomalyId, domain, axisType, metric, periodKey, value, compareBasis,
             source, trafficLight, candidates, candidateSortRule, settingVersion]
  properties:
    anomalyId: { type: string }
    domain: { type: string, enum: [production, inventory, sales] }
    axisType: { type: string, enum: [country, plant] }
    countryCode: { type: string, nullable: true, example: B07 }
    countryName: { type: string, nullable: true, example: 칠레 }
    plantCode: { type: string, nullable: true }
    plantName: { type: string, nullable: true }
    metric:
      type: string
      enum: [planAchievementRate, cbuShare, exportShare,
             distributionStay, inventoryTurnMos, localInventorySnapshot,
             wholesaleToRetailGap, planProgressRate, yoyChange]
    metricLabel: { type: string, example: 도매 대비 소매 격차 }
    periodKey: { $ref: '#/components/schemas/PeriodKey' }
    comparePeriod: { type: string, nullable: true, example: 2025 누계 }
    value: { type: number }
    compareValue: { type: number, nullable: true }
    compareBasis:
      type: string
      enum: [plan, yoy, mom]
      description: 계획 대비, 전년 동월, 전월 대비. 계획이 있으면 계획을 먼저 본다
    changeRate: { type: number, nullable: true }
    detectionUnit:
      type: string
      enum: [modelGroup, modelDetail]
      description: 기본값 modelGroup. 세부 차종 감지는 모델 교체를 -100%로 잡는다
    source:
      type: string
      enum: [aJudgment, supplementaryAggregate, derived]
    derived: { type: boolean, description: source가 derived면 참. 화면이 유도 표기 }
    supplementary: { type: boolean, description: source가 supplementaryAggregate면 참 }
    contributionAvailable: { type: boolean }
    contributionNote: { type: string, nullable: true, example: 분해 불가 }
    contributionOrigin: { type: string, nullable: true, example: A3 판매 리포트가 낸 값 }
    contributions:
      type: array
      items: { $ref: '#/components/schemas/Contribution' }
    stageFlow:
      $ref: '#/components/schemas/StageFlow'
      nullable: true
    exposure:
      type: object
      nullable: true
      properties:
        status: { type: string, enum: [confirmed, unconfirmed] }
        cbuShare: { type: number, nullable: true }
        acquisitionPath: { type: string, enum: [columnDirect, productionDerived] }
        derivationRatio: { type: number, nullable: true }
    entityTag:
      type: object
      nullable: true
      properties:
        mapped: { type: boolean }
        entityCode: { type: string, nullable: true }
        entityName: { type: string, nullable: true }
        label: { type: string, example: 법인 미매핑 }
    trafficLight: { $ref: '#/components/schemas/TrafficLight' }
    changeBadge:
      type: string
      nullable: true
      enum: [new, raised, null]
    candidates:
      type: array
      description: 정렬 규칙이 적용된 상태로 온다. 화면이 다시 정렬하지 않는다
      items: { $ref: '#/components/schemas/CauseCandidate' }
    candidateTruncatedCount:
      type: integer
      default: 0
      description: 상한에 걸려 잘린 건수. 0보다 크면 화면에 적는다
    candidateSortRule:
      type: string
      example: 날짜 차이 오름차순 → 출처 수 내림차순 → 기사 수 내림차순
    causeLink:
      $ref: '#/components/schemas/CauseLink'
      nullable: true
    settingVersion: { type: string }
    batchRunId: { type: string }
```

### 4.4 contribution 내부 분해 기여

변동을 만든 하위 항목과 그 몫. A 판정이 낸 값을 C가 그대로 실어 나른다([[TBL-DOM-001#Contribution]]).

`sortOrder`는 기여값 내림차순이 낳은 자리 번호다. 중요도가 아니다.

```yaml
Contribution:
  type: object
  required: [dimension, itemName, contributionValue, sortOrder]
  properties:
    dimension:
      type: string
      enum: [country, modelGroup, modelDetail, plant, segment, dealer]
      description: 파워트레인은 44~53%가 미분류라 차원에서 뺐다
    dimensionLabel: { type: string, example: 국가 }
    itemName: { type: string, example: 칠레 }
    itemCode: { type: string, nullable: true }
    contributionValue: { type: number, description: 절대 대수 }
    contributionShare: { type: number, nullable: true, description: 비중. 소수 }
    sortOrder: { type: integer, description: 기여값 내림차순이 낳은 자리 }
```

### 4.5 cause_candidate 원인 후보

변동에 붙은 후보 한 건과 근접도 값 넷. **이 스키마에 등급·점수 필드가 없다는 것이 이 문서의 핵심 제약이다**([[TBL-PRD-001#R26]], PRD 6.14).

`proximity` 네 값이 정렬과 신호등의 유일한 입력이다. 모델은 이 값을 바꾸거나 새로 만들 수 없다([[TBL-PRD-001#R27]]).

```yaml
CauseCandidate:
  type: object
  required: [candidateId, candidateType, targetId, title, proximity, sortOrder, truncated]
  properties:
    candidateId: { type: string }
    candidateType:
      type: string
      enum: [event, article, marketMetric, crossDomainAnomaly]
    targetId: { type: string, description: 사건·기사·지표·다른 변동의 ID }
    title: { type: string }
    proximity:
      type: object
      required: [dayDiff, articleCount, sourceCount, countryMatch]
      properties:
        dayDiff:
          type: integer
          description: 변동의 fileBaseDate와의 날짜 차이. 사건은 마지막 관측일 기준
        articleCount: { type: integer }
        sourceCount: { type: integer, description: 서로 다른 매체 이름의 개수 }
        countryMatch:
          type: string
          enum: [countryDirect, straitAttributed, crossDomainSameCountry, dateOnly]
          description: 국가 직접 · 해협 귀속 · 같은 국가 다른 도메인 · 날짜만 일치
    sortOrder: { type: integer, description: 정렬 규칙이 낳은 자리. 관련도 순위가 아니다 }
    truncated: { type: boolean }
    usedInExplanation: { type: boolean, description: 거짓이면 접힌 채로 보인다 }
    footnote: { type: integer, nullable: true }
    eventDetail:
      type: object
      nullable: true
      properties:
        eventType: { type: string, nullable: true }
        category: { type: string }
        namedBy: { type: string, enum: [llm, representativeArticle] }
        maxImpact: { type: number, nullable: true, description: 기사에 붙어 온 영향도의 최대값 }
        firstSeenDate: { type: string, format: date }
        lastSeenDate: { type: string, format: date }
        articleTotal: { type: integer }
        articles:
          type: array
          minItems: 3
          items:
            type: object
            properties:
              title: { type: string }
              sourceName: { type: string }
              publishedAt: { type: string, format: date-time }
              timePrecision: { type: string, enum: [exact, day] }
              url: { type: string }
    marketDetail:
      type: object
      nullable: true
      properties:
        indicatorId: { type: string }
        name: { type: string }
        value: { type: number }
        changeRate: { type: number, nullable: true }
        asOfDate: { type: string, format: date }
        carriedOver: { type: boolean }
        series:
          type: array
          description: 30일 스파크라인
          items:
            type: object
            properties:
              date: { type: string, format: date }
              value: { type: number }
    crossDomainDetail:
      type: object
      nullable: true
      properties:
        anomalyId: { type: string }
        domain: { type: string, enum: [production, inventory, sales] }
        metricLabel: { type: string }
        value: { type: number }
        compareValue: { type: number, nullable: true }
        changeRate: { type: number, nullable: true }
        periodKey: { $ref: '#/components/schemas/PeriodKey' }
```

### 4.6 traffic_light 신호등과 근거

규칙 산출물이다. LLM을 꺼도 같은 값이 나온다([[TBL-PRD-001#R11]] [[TBL-UC-001#UC-S8]] 7단계).

`basis`는 어느 후보의 어떤 값이 기준을 넘었는지를 담는다. 화면이 이 값으로 왜 그 색인지 보여 준다. 기준에 못 미치면 `level`이 `none`이고 `basis`는 `null`이며 목록에는 남는다.

```yaml
TrafficLight:
  type: object
  required: [level, label]
  properties:
    level: { type: string, enum: [red, yellow, none] }
    label:
      type: string
      enum: ['확인 필요', '주의', '원인 미확인']
      description: 색만으로 구분하지 않는다. 항상 텍스트를 병기한다
    basis:
      type: object
      nullable: true
      description: 기준을 넘긴 후보와 그 값. 넘긴 후보가 없으면 null
      properties:
        candidateId: { type: string }
        articleCount: { type: integer }
        articleCountMin: { type: integer }
        sourceCount: { type: integer }
        sourceCountMin: { type: integer }
        dayDiff: { type: integer }
        windowDays: { type: integer }
        exposureConfirmed:
          type: boolean
          description: 거짓이면 RED로 올라가지 못하고 YELLOW에서 멈춘다
    settingVersion: { type: string }
```

### 4.7 cause_link 연관 설명

후보와 변동을 잇는 문단 하나. H-chat이 쓰고 검증기가 인용 ID를 대조한다([[TBL-DOM-001#CauseLink]] [[TBL-UC-001#UC-S9]]).

**출력에 등급·점수·순위 필드를 두지 않는다.** 설명이 없어도 후보와 근접도 값은 그대로 남는다.

```yaml
CauseLink:
  type: object
  required: [text, citedCandidateIds, verified, degraded]
  properties:
    text: { type: string, nullable: true }
    emphasis:
      type: array
      items:
        type: object
        properties:
          start: { type: integer }
          end: { type: integer }
    citedCandidateIds:
      type: array
      description: 전부 그 변동의 candidates 안에 있어야 한다
      items: { type: string }
    footnotes: { type: array, items: { type: integer } }
    verified: { type: boolean }
    degraded: { type: boolean }
    degradeReason:
      type: string
      nullable: true
      enum: [llmFailure, contentFilter, jsonViolation, citationVerificationFailed, backfillMode, null]
```

### 4.8 stage_flow 단계별 흐름

판매 네 계열의 값과 단계 사이의 차이. 재고 원천이 없는 동안 재고 신호가 나오는 자리다([[TBL-PRD-001#R30]] [[TBL-DOM-001#SalesStageFlow]]).

부호 규약은 1.4절에 적었다. 음수를 예외로 두지 않는다.

```yaml
StageFlow:
  type: object
  required: [periodKey, shipment, officialWholesale, retail,
             dealerStageGap, dealerStageGapRate, wholesaleBasis, derivationType]
  properties:
    countryCode: { type: string, nullable: true, example: B07 }
    countryName: { type: string, nullable: true, example: 칠레 }
    periodKey: { $ref: '#/components/schemas/PeriodKey' }
    shipment: { type: number, example: 679551 }
    actualWholesale: { type: number, nullable: true, example: 677201 }
    officialWholesale: { type: number, example: 677201 }
    retail: { type: number, example: 643097 }
    entityStageGap:
      type: number
      description: 선적 − 도매정본. 법인 단계에 남은 물량
      example: 2350
    entityStageGapRate: { type: number, nullable: true }
    dealerStageGap:
      type: number
      description: 도매정본 − 소매. 딜러 단계에 남은 물량. 음수는 덜어내는 중
      example: 34104
    dealerStageGapRate: { type: number, example: 0.050 }
    wholesaleBasis:
      type: string
      enum: [officialWholesale, actualWholesale]
      description: 어느 기준으로 계산했는지. 설정에 따라 체류율이 조용히 달라지는 것을 막는다
    wholesaleAlternativeDiff:
      type: number
      nullable: true
      description: 다른 도매 기준과의 차이. 미주 누계 실측 27,135
    derivationType:
      type: string
      enum: [derived, measured]
      description: 재고 원천이 들어오면 measured로 바뀐다
    sortOrder: { type: integer, nullable: true }
```

### 4.9 market_metric 시장지표 값

지표 바 한 칸과 후보 상세가 함께 쓴다([[TBL-DOM-001#MarketPoint]]).

`carriedOver`가 참이면 화면이 "값 이월"을 적고, `carryOverLimitExceeded`가 참이면 값을 비우고 "기준일 지연"만 적으며 판정에 쓰지 않는다([[TBL-UI-001#UI-10]] E6).

```yaml
MarketMetric:
  type: object
  required: [indicatorId, name, asOfDate, carriedOver]
  properties:
    indicatorId: { type: string, enum: [GPR, BRENT, BDI, SCFI] }
    name: { type: string, example: 건화물운임 BDI }
    value: { type: number, nullable: true }
    changeRate: { type: number, nullable: true }
    asOfDate: { type: string, format: date, nullable: true }
    carriedOver: { type: boolean }
    carryOverLimitExceeded: { type: boolean, default: false }
    displayInBar: { type: boolean, default: true }
```

### 4.10 evidence 근거와 각주

문장이 가리키는 근거 한 줄. 번호는 코드가 먼저 매기고 모델이 아니다([[TBL-DOM-001#Evidence]]).

표시값을 복사해 두어 열람할 때 조인이 없다. 각주 없는 주장을 두지 않는다([[TBL-PRD-001#N4]]).

```yaml
Evidence:
  type: object
  required: [footnote, kind, sourceId]
  properties:
    footnote: { type: integer }
    kind:
      type: string
      enum: [candidate, causeLink, anomaly, contribution, marketMetric, article, aJudgment]
    sourceId: { type: string }
    displayValue: { type: string, description: 표시용 값 사본 }
    url: { type: string, nullable: true, description: 기사 원문. 새 창으로만 연다 }
```

### 4.11 form_detection 형태 판별 결과

완성차 데이터의 형태 A·B 판별과 그 근거([[TBL-INFRA-001#C6]] [[TBL-DOM-001#IngestFile]]).

첫 행이 바로 헤더이고 기준일자 컬럼이 있으면 형태 A, 병합 헤더가 2~3줄이고 기간 블록이 반복되면 형태 B다. 어느 쪽도 아니면 `unknown`이고 적재를 막는다. 새 형태 등록은 개발 작업이다.

```yaml
FormDetection:
  type: object
  required: [form, formLabel, evidence]
  properties:
    form: { type: string, enum: [A, B, unknown] }
    formLabel: { type: string, example: 형태 B · 피벗 리포트 }
    schemaVersion: { type: string, nullable: true }
    evidence:
      type: object
      properties:
        headerRowCount: { type: integer }
        hasBaseDateColumn: { type: boolean }
        periodBlockRepeated: { type: boolean }
        detectedDimensionColumns: { type: integer }
    mismatch:
      type: array
      description: form이 unknown일 때만 채운다
      items:
        type: object
        properties:
          expected: { type: string }
          found: { type: string }
```

### 4.12 tracking_metric A 리포트 트래킹 지표

A 리포트 사이드의 지표 셋. 현업 인터뷰 전까지 임시 지정임을 표시한다([[TBL-UC-001#UC-H4]] 1a).

```yaml
TrackingMetric:
  type: object
  required: [metric, label, provisional]
  properties:
    metric: { type: string }
    label: { type: string, example: 사업계획 달성 }
    value: { type: number, nullable: true }
    displayValue: { type: string, nullable: true, example: 86.1% }
    status: { type: string, enum: [normal, warning, unstable], description: 색만으로 구분하지 않는다 }
    active: { type: boolean, description: 본문의 기준이 되는 지표 }
    provisional: { type: boolean, description: 참이면 화면에 임시 지정 표시 }
    note: { type: string, nullable: true }
```

### 4.13 missing_metric 못 만드는 지표

데이터 구조상 만들 수 없는 지표. 빈 자리를 추정값으로 채우지 않고 이유와 함께 목록으로 보여 준다([[TBL-PRD-001#R29]] 넷째 인수 기준).

```yaml
MissingMetric:
  type: object
  required: [code, label, reason]
  properties:
    code:
      type: string
      enum: [destinationCountry, powertrainBreakdown, afloat, awaitingShipment,
             entityVsDealer, countryDailySeries, globalCountryByModel]
    label: { type: string, example: 항해중 재고 }
    reason: { type: string, example: 인입 샘플 어디에도 원천이 없습니다 }
    willFillWhen: { type: string, nullable: true, example: 재고 원천을 받으면 }
```

## 5. 미결사항

- [ ] 현업 서명 토큰의 발급 주체, 만료 시간, 갱신 방식. VODA 포털 팀과 맞춰야 한다(1.1절)
- [ ] A 리포트 내려받기 파일 형식. `download.format` 값이 정해지지 않았다([[TBL-UI-001#UI-7]])
- [ ] A 판정 스냅샷을 브리핑 갈래에서 어떤 키와 모양으로 받는지. 이것이 정해져야 `judgments`와 4.4절의 필드가 확정된다([[TBL-DOM-001#DomainJudgment]])
- [ ] 후보 상한 기본값(사건 5, 기사 10, 지표 6)과 시간창 기본값. 지금은 설정값이라는 것만 정했다([[TBL-PRD-001#R26]])
- [ ] 신호등 임계값(최소 기사 수, 최소 출처 수, 시간창, CBU 비중 기준)을 응답에 내릴지. 지금은 4.6절의 `basis`에만 담고 화면에 기준선을 그리지 않는다
- [ ] [[#GET/api/intel/creport/latest]] 응답 크기. 후보와 근거를 한 번에 담으면 국가가 늘었을 때 커진다. 근거 패널을 별도 호출로 나눌지 [확인 필요]
- [ ] 미매핑 엑셀 내려받기를 `format=xlsx`로 같은 엔드포인트에서 받을지 별도 주소를 둘지
- [ ] 관리 API의 감사 로그를 어디에 남길지. 지금은 해제·확정 응답에 행위자만 담았다
- [ ] 현업 피드백("맞다·아니다·모르겠다") 엔드포인트. 화면에 넣을지가 아직 미결이라 여기도 두지 않았다([[TBL-PRD-001#R28]])
- [ ] 2본째 상세 화면을 만들면 그 조회 엔드포인트가 더 필요하다([[TBL-PRD-001#R22]])
- [ ] 알림 채널이 정해지면 발송 상태 조회가 필요한지([[TBL-PRD-001#R18]])
