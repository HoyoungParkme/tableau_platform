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

#### GET/api/intel/briefing/latest 최신 게시 브리핑

화면 [[TBL-UI-001#UI-1]] · 유스케이스 [[TBL-UC-001#UC-H1]] · 서비스 `BriefingQuery.latest`

```yaml
/api/intel/briefing/latest:
  get:
    summary: 최신 게시 브리핑
```

#### GET/api/intel/briefing/{briefingId} 특정 브리핑

화면 [[TBL-UI-001#UI-1]] · 서비스 `BriefingQuery.get`

```yaml
/api/intel/briefing/{briefingId}:
  get:
    parameters: [{ name: briefingId, in: path, schema: { type: integer } }]
```

## 4. 스키마

다음 버전.

## 5. 미결사항

- [ ] 전체 본문 반영
