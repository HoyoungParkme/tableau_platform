---
doc_id: TBL-API-001
type: API
title: Glovis 완성차 인텔리전스 REST API
status: draft
upstream: [TBL-UI-001, TBL-UC-001, TBL-DOM-001, TBL-INFRA-001]
---

# API 명세 REST

## 0. 이 문서가 다루는 것

backend 열람·관리 REST 엔드포인트. 전체 본문은 다음 버전에서 채운다.

## 1. 규칙

경로 접두: 현업 `/api/intel/*`, 관리 `/api/admin/intel/*`.

## 2. 에러

RFC 9457.

## 3. 엔드포인트

### 3.1 현업 열람 (토큰)

#### GET/api/intel/briefing/latest 최신 게시 브리핑

화면 [[TBL-UI-001#UI-1]] · 유스케이스 [[TBL-UC-001#UC-H1]] · 서비스 `BriefingQuery.latest`

응답 1회로 지표 바, 헤드라인, 도메인 상태, 워치리스트 카드(근거 포함)를 전부 준다. LLM 호출 없음.

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

#### GET/api/intel/briefing/{briefingId}/detail/{iso3} 판매 상세

화면 [[TBL-UI-001#UI-2]] · 유스케이스 [[TBL-UC-001#UC-H3]] · 서비스 `BriefingQuery.detail`

```yaml
/api/intel/briefing/{briefingId}/detail/{iso3}:
  get:
    responses:
      '200': { content: { application/json: { schema: { $ref: '#/components/schemas/DetailView' } } } }
```

#### GET/api/intel/indicators/{identifier}/series 시장지표 시계열

화면 [[TBL-UI-001#UI-1]] [[TBL-UI-001#UI-2]] · 서비스 `IndicatorQuery.series`

스파크라인용. `days` 기본 30.

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

## 4. 스키마

다음 버전.

## 5. 미결사항

- [ ] 전체 본문 반영
