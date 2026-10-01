---
doc_id: TBL-UI-002
type: UI
title: Glovis 완성차 인텔리전스 화면 설계
status: draft
upstream: [TBL-UC-002, TBL-PRD-002, TBL-INFRA-002, TBL-DOM-004]
---

# 화면 설계

## 0. 이 문서가 다루는 것

화면 여덟을 다룬다. 열람 화면 넷과 관리 화면 넷이다.

| 화면 | 무엇 | 누가 본다 |
|:--|:--|:--|
| [[#UI-1]] | C 리포트 메인. 과업지시의 "인텔리전스 서비스 개요" | 현업 사용자 |
| [[#UI-2]] | A1 생산 리포트 | 현업 사용자 |
| [[#UI-3]] | A2 재고 리포트 | 현업 사용자 |
| [[#UI-4]] | A3 판매 리포트 | 현업 사용자 |
| [[#UI-5]] | 데이터 적재 | 관리자 |
| [[#UI-6]] | 배치 이력과 재실행 | 관리자 |
| [[#UI-7]] | 마스터와 미매핑 | 관리자 |
| [[#UI-8]] | 열람 주소 | 관리자 |

### 0.1 시안이 정본이다

화면 시안이 확정돼 있다. 이 문서는 그 시안의 구조를 요소와 규칙으로 옮겨 적은 것이다. 시안과 이 문서가 어긋나면 시안을 따르고 이 문서를 고친다.

시안 링크: https://claude.ai/artifact/5RhDRfeGjwpf5sQYFYtEBq

| 시안 파일 | 대응 화면 |
|:--|:--|
| `docs/glovis/intelligence/design/Main.dc.html` | [[#UI-1]] 기본 상태 |
| `docs/glovis/intelligence/design/Evidence.dc.html` | [[#UI-1]] 근거 펼친 상태 |
| `docs/glovis/intelligence/design/States.dc.html` | [[#UI-1]] 예외 상태 6가지 |
| `docs/glovis/intelligence/design/A1Production.dc.html` | [[#UI-2]] |
| `docs/glovis/intelligence/design/A2Inventory.dc.html` | [[#UI-3]] |
| `docs/glovis/intelligence/design/A3Sales.dc.html` | [[#UI-4]] |

열람 화면 넷([[#UI-1]] [[#UI-2]] [[#UI-3]] [[#UI-4]])의 배치 블록은 위 시안 파일의 내용(스타일과 본문)을 그대로 옮긴 것이다. 시안 html의 해당 요소에 요소 표 번호(data-el)를 붙여 배치의 배지와 요소 표를 이었다. 시안 파일 자체에는 번호가 없고 이 문서에 옮길 때 붙인 것이다. 관리 화면 넷([[#UI-5]] [[#UI-6]] [[#UI-7]] [[#UI-8]])에는 시안이 없다. 유스케이스 흐름에서 유도한 최소 구성을 적었고 그 사실을 각 항목에 표시했다.

### 0.2 용어

A1 생산 리포트, A2 재고 리포트, A3 판매 리포트를 통칭해 A 리포트라 한다. 대시보드에 붙으며 브리핑 갈래가 만든다. C 리포트는 종합 리포트이고 이 명세 사슬이 만드는 것이다. "A 판정"은 A 리포트가 만든 스냅샷 변동 판정값과 내부 분해 결과다. C는 이것만 읽고 A 리포트가 쓴 문장은 읽지 않는다([[TBL-UC-002#UC-S2]]). A 리포트 화면이 보여 주는 요약·해설 문장과 트래킹 지표 판정은 브리핑 갈래가 게시한 것을 사본으로 받은 것이고, 표와 막대(분해 블록)는 이 시스템 배치가 자기 데이터로 만든 것이다([[TBL-DOM-004#DomainReport]]). A 리포트 화면은 이 문서의 [[#UI-2]] [[#UI-3]] [[#UI-4]]뿐이다. 브리핑 갈래는 세 대시보드의 브리핑을 만들되 자기 화면을 띄우지 않는다(유저 결정 2026-10-01).

액터코드 A(관리자)와 리포트 A는 다른 것이다.

### 0.3 이 문서 전체에 걸리는 다섯 가지

첫째, 관련도 등급 칩이 화면에 없다. 모델이 높음과 낮음을 매기지 않는다(PRD 6.14, [[TBL-PRD-002#R10]]). 대신 코드가 센 근접도 값을 칩으로 적는다. 시점 차이, 기사 수, 출처 수, 국가 일치 방식 넷이다. 후보 정렬 기준도 화면에 글자로 적어 왜 이 순서인지 읽는 사람이 확인하게 한다([[TBL-PRD-002#R26]]).

둘째, 신호등은 색만으로 구분하지 않는다. 점이나 칩에 반드시 텍스트를 병기한다. red는 "확인 필요", yellow는 "주의", none은 "원인 미확인", notApplicable은 "판정 대상 아님"이다. 색각 이상과 흑백 인쇄에서 같은 정보가 읽혀야 한다.

셋째, 화면 구성은 고정이 아니다. 발주자가 개요 시트를 디자인 방향으로 설명했다([[TBL-RFQ-002#Q15]], PRD 6.12 확정). 여기 적은 배치는 지금 데이터가 받쳐 주는 만큼의 구성이며 데이터가 바뀌면 구성도 바뀐다([[TBL-PRD-002#R25]]).

넷째, A 리포트에서 C 리포트로 가는 화면 흐름이 없다. C는 VODA 단독 주소다([[TBL-PRD-002#R29]], PRD 6.17). A 리포트 어디에도 C로 가는 버튼이나 링크를 두지 않는다.

다섯째, 판정은 LLM과 무관하게 화면에 나온다. 변동, 후보, 근접도, 신호등, 도메인 상태 신호등은 배치 1~5단계에서 코드가 끝낸다([[TBL-INFRA-002#C19]]). LLM 세 역할이 전부 실패해도 이것들은 그대로 표시되고 문장 자리만 강등된다([[TBL-PRD-002#N2]]).

## 1. 유스케이스 대응

| 유스케이스 | 화면 |
|:--|:--|
| [[TBL-UC-002#UC-H2]] | [[#UI-1]]. 확장점 5 근거 확인은 [[TBL-UC-002#UC-H3]]이며 같은 화면 안에서 펼친다 |
| [[TBL-UC-002#UC-H3]] | [[#UI-1]] 근거 패널 |
| [[TBL-UC-002#UC-H1]] | [[#UI-2]] [[#UI-3]] [[#UI-4]] |
| [[TBL-UC-002#UC-A1]] | [[#UI-5]]. 포함 [[TBL-UC-002#UC-S10]] 회귀 검사. 확장점 6 백필 모드 |
| [[TBL-UC-002#UC-A3]] | [[#UI-6]]. 포함 [[TBL-UC-002#UC-S1]] |
| [[TBL-UC-002#UC-A2]] | [[#UI-7]]. 확장점 4 과거 재계산은 [[#UI-6]]으로 |
| [[TBL-UC-002#UC-A4]] | [[#UI-8]] |

[[TBL-UC-002#UC-H3]]는 별도 화면이 아니다. [[#UI-1]] 변동 카드 안에서 펼치는 패널이다. 시스템 유스케이스([[TBL-UC-002#UC-S1]] 외)는 화면이 없고 [[#UI-6]]의 이력으로만 보인다.

설정을 바꾸는 화면은 이번 범위에 없다. 정본 선택과 임계값은 설정 테이블에 있고 배치가 버전으로 읽는다([[TBL-UC-002#UC-S4]] 9단계, [[TBL-INFRA-002#C11]]).

## 2. 화면 목록

| 화면 | 경로 | 한 줄 목적 |
|:--|:--|:--|
| [[#UI-1]] C 리포트 메인 | `/intel/creport` | 세 도메인의 변동과 외부 원인 후보를 한 화면에 |
| [[#UI-2]] A1 생산 리포트 | `/intel/areport/production` | 사업계획 대비 달성률을 공장과 차종까지 분해 |
| [[#UI-3]] A2 재고 리포트 | `/intel/areport/inventory` | 원천 부재를 밝히고 유도 지표로 체류를 보여 줌 |
| [[#UI-4]] A3 판매 리포트 | `/intel/areport/sales` | 선적·도매·소매 단계별 흐름과 막히는 곳 |
| [[#UI-5]] 데이터 적재 | `/admin/ingest` | 형태 판별과 미리보기를 거쳐 적재 확정 |
| [[#UI-6]] 배치 이력과 재실행 | `/admin/batch` | 8단계 결과를 보고 재실행·재생성 |
| [[#UI-7]] 마스터와 미매핑 | `/admin/master` | 미매핑을 내려받고 크로스워크를 올려 갱신 |
| [[#UI-8]] 열람 주소 | `/admin/exposure` | 화면 넷의 서명 주소를 켜고 바꾼다 |

### 2.1 C 리포트

#### UI-1 C 리포트 메인

| 항목 | 내용 |
|---|---|
| 경로 | `/intel/creport` |
| 주 유스케이스 | [[TBL-UC-002#UC-H2]] · 확장 [[TBL-UC-002#UC-H3]] |
| 진입 / 이탈 | VODA에 건 C 리포트 서명 주소, 추가 로그인 없음([[TBL-PRD-002#R29]] [[#UI-8]]) / 기사 제목을 누르면 새 창으로 원문. 그 외에 다른 화면으로 나가는 링크 없음 |

페이지. 세 도메인의 변동과 외부 원인 후보를 한 화면에 보여 준다. 근거 펼치기와 예외 상태 여섯을 이 화면이 함께 갖는다.

##### 배치

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;600;700&display=swap">
<style>.var{margin:32px 0 8px;font-size:11.5px;color:#A6A49C;border-left:3px solid #DAD6CE;padding-left:10px;font-family:system-ui,sans-serif}</style>

<style>
body { margin: 0; background: #F4F2EE; color: #1A1B19;
      font-family: 'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif;
      -webkit-font-smoothing: antialiased; }
    a { color: #0E6B57; text-decoration: none; }
    a:hover { color: #0B5847; text-decoration: underline; }
    .num { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-variant-numeric: tabular-nums; }
    .lbl { font-size: 10.5px; font-weight: 500; text-transform: uppercase; letter-spacing: .09em; color: #9A9890; }
    .hl { font-weight: 600; background: rgba(181,115,12,.14); border-radius: 3px; padding: 0 2px; }
    .fn { font-size: 10px; vertical-align: super; color: #0E6B57; font-weight: 600; }
</style>
<div data-el="1" style="width: 1280px; padding: 28px 32px 40px; box-sizing: border-box; display: flex; flex-direction: column; gap: 20px;">

  <div style="display: flex; align-items: baseline; justify-content: space-between; border-bottom: 1px solid #E2DFD8; padding-bottom: 14px;">
    <div style="display: flex; align-items: baseline; gap: 10px;">
      <div style="font-size: 17px; font-weight: 600; letter-spacing: -.01em;">완성차 인텔리전스</div>
      <div style="font-size: 11.5px; color: #6B6A65;">종합 리포트</div>
    </div>
    <div data-el="2" style="display: flex; align-items: baseline; gap: 16px; font-size: 11.5px; color: #6B6A65;">
      <div>기준일 <span class="num" style="color:#1A1B19;">2026-08-20</span></div>
      <div>완성차 <span class="num">08-20</span> · 뉴스 <span class="num">08-20</span> · 시장 <span class="num">08-20</span></div>
    </div>
  </div>

  <div data-el="3" style="display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 1px; background: #E2DFD8; border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
    <div style="background: #FFFFFF; padding: 12px 16px; display: flex; flex-direction: column; gap: 4px;">
      <div style="font-size: 11.5px; color: #6B6A65;">지정학 위험 GPR</div>
      <div style="display: flex; align-items: baseline; gap: 8px;">
        <div class="num" style="font-size: 19px; font-weight: 600;">182.4</div>
        <div class="num" style="font-size: 12px; font-weight: 600; color: #B3261E;">+14.2%</div>
      </div>
      <div style="font-size: 10.5px; color: #9A9890;">08-20 기준</div>
    </div>
    <div style="background: #FFFFFF; padding: 12px 16px; display: flex; flex-direction: column; gap: 4px;">
      <div style="font-size: 11.5px; color: #6B6A65;">브렌트유 Brent</div>
      <div style="display: flex; align-items: baseline; gap: 8px;">
        <div class="num" style="font-size: 19px; font-weight: 600;">78.32</div>
        <div class="num" style="font-size: 12px; font-weight: 600; color: #B3261E;">+3.1%</div>
      </div>
      <div style="font-size: 10.5px; color: #9A9890;">08-20 기준</div>
    </div>
    <div style="background: #FFFFFF; padding: 12px 16px; display: flex; flex-direction: column; gap: 4px;">
      <div style="font-size: 11.5px; color: #6B6A65;">건화물운임 BDI</div>
      <div style="display: flex; align-items: baseline; gap: 8px;">
        <div class="num" style="font-size: 19px; font-weight: 600;">2,776</div>
        <div class="num" style="font-size: 12px; font-weight: 600; color: #0E6B57;">-10.1%</div>
      </div>
      <div style="font-size: 10.5px; color: #9A9890;">08-20 기준</div>
    </div>
    <div style="background: #FFFFFF; padding: 12px 16px; display: flex; flex-direction: column; gap: 4px;">
      <div style="font-size: 11.5px; color: #6B6A65;">컨테이너운임 SCFI</div>
      <div style="display: flex; align-items: baseline; gap: 8px;">
        <div class="num" style="font-size: 19px; font-weight: 600;">1,942</div>
        <div class="num" style="font-size: 12px; font-weight: 600; color: #9A9890;">-0.4%</div>
      </div>
      <div style="font-size: 10.5px; color: #B5730C;">08-18 값 이월</div>
    </div>
  </div>

  <div data-el="4" style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 20px 22px; display: flex; flex-direction: column; gap: 12px;">
    <div style="display: flex; align-items: center; gap: 8px;">
      <div class="lbl">오늘의 요약</div>
      <div data-el="5" style="display: inline-flex; align-items: center; gap: 5px; background: #EFEDE7; color: #6B6A65; font-size: 11.5px; padding: 3px 8px; border-radius: 7px;">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
        상관관계 확인 · 인과관계 미확정
      </div>
    </div>
    <div style="font-size: 19px; font-weight: 500; line-height: 1.6; letter-spacing: -.01em; text-wrap: pretty;">
      중동 2개국에서 <span class="hl">판매 도매가 급감</span>하는 동안 같은 국가의 <span class="hl">항해중 재고가 늘었고</span>, 시점이 <span class="hl">호르무즈 해협 사건</span>과 겹칩니다.<span class="fn">1,2,3</span> 생산은 변동이 없습니다.
    </div>
  </div>

  <div style="display: flex; flex-direction: column; gap: 10px;">
    <div style="display: flex; align-items: center; justify-content: space-between;">
      <div class="lbl">도메인 상태</div>
      <button data-el="6" style="display: inline-flex; align-items: center; gap: 5px; height: 26px; padding: 0 9px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #6B6A65; font-size: 11.5px; border-radius: 7px; font-family: inherit; cursor: pointer;">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m18 15-6-6-6 6"></path></svg>
        접기
      </button>
    </div>
    <div data-el="7" style="display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 14px;">

      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 16px; display: flex; flex-direction: column; gap: 10px;">
        <div style="display: flex; align-items: center; gap: 8px;">
          <span style="width: 8px; height: 8px; border-radius: 50%; background: #9A9890; display: inline-block;"></span>
          <div style="font-size: 15px; font-weight: 600;">생산</div>
          <div style="font-size: 11.5px; color: #6B6A65; margin-left: auto;">모니터링</div>
        </div>
        <div style="font-size: 13.5px; line-height: 1.65; color: #1A1B19;">
          공장별 생산은 전월 수준을 유지했습니다. 임계값을 넘은 공장이 없습니다. 생산 대시보드에 국가 차원이 없어 중동행 물량만 따로 보지는 못합니다.<span class="fn">4</span>
        </div>
        <div style="font-size: 11px; color: #9A9890; border-top: 1px solid #EFEDE7; padding-top: 8px;">대시보드 판정 A1 · 기여 분해 없음</div>
      </div>

      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 16px; display: flex; flex-direction: column; gap: 10px;">
        <div style="display: flex; align-items: center; gap: 8px;">
          <span style="width: 8px; height: 8px; border-radius: 50%; background: #B5730C; display: inline-block;"></span>
          <div style="font-size: 15px; font-weight: 600;">재고</div>
          <div style="font-size: 11.5px; color: #B5730C; font-weight: 600; margin-left: auto;">주의</div>
        </div>
        <div style="font-size: 13.5px; line-height: 1.65; color: #1A1B19;">
          오만의 <span class="hl">항해중 재고가 418대</span>로 총재고의 42%를 차지합니다. 아랍에미리트도 28%로 올라 두 나라 모두 도매가 멈춘 것과 맞물립니다.<span class="fn">5,6</span>
        </div>
        <div style="font-size: 11px; color: #9A9890; border-top: 1px solid #EFEDE7; padding-top: 8px;">대시보드 판정 A2 · 기여 상위 오만 · 아랍에미리트</div>
      </div>

      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 16px; display: flex; flex-direction: column; gap: 10px;">
        <div style="display: flex; align-items: center; gap: 8px;">
          <span style="width: 8px; height: 8px; border-radius: 50%; background: #B3261E; display: inline-block;"></span>
          <div style="font-size: 15px; font-weight: 600;">판매</div>
          <div style="font-size: 11.5px; color: #B3261E; font-weight: 600; margin-left: auto;">확인 필요</div>
        </div>
        <div style="font-size: 13.5px; line-height: 1.65; color: #1A1B19;">
          오만과 아랍에미리트의 도매가 <span class="hl">전월 대비 급감</span>했습니다. 대시보드 분해로는 투싼·아반떼가 감소분의 대부분을 만들었습니다.<span class="fn">1,7</span>
        </div>
        <div style="font-size: 11px; color: #9A9890; border-top: 1px solid #EFEDE7; padding-top: 8px;">대시보드 판정 A3 · 기여 상위 투싼 · 아반떼</div>
      </div>

    </div>
  </div>

  <div style="display: flex; flex-direction: column; gap: 10px;">
    <div data-el="8" style="display: flex; align-items: center; justify-content: space-between;">
      <div class="lbl">변동 목록 · 국가 4건</div>
      <div style="font-size: 11.5px; color: #6B6A65;">신호등 순 정렬</div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 10px;">

      <div data-el="9" style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 18px 20px; display: flex; flex-direction: column; gap: 14px;">
        <div style="display: flex; align-items: center; gap: 10px; flex-wrap: wrap;">
          <div style="font-size: 16px; font-weight: 600;">오만</div>
          <span data-el="10" style="font-size: 11px; font-weight: 600; color: #B3261E; background: rgba(179,38,30,.08); border: 1px solid #F0CFCB; padding: 2px 7px; border-radius: 7px;">확인 필요</span>
          <span data-el="11" style="font-size: 11px; color: #0B5847; background: #E7F0ED; padding: 2px 7px; border-radius: 7px;">신규</span>
          <span data-el="12" style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">CBU 100%</span>
          <span data-el="13" style="font-size: 11px; color: #9A9890; background: #FBFAF8; border: 1px solid #E2DFD8; padding: 2px 7px; border-radius: 7px;">법인 미매핑</span>
        </div>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 12px;">
          <div data-el="14" style="background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 6px;">
            <div class="lbl">변동</div>
            <div style="display: flex; align-items: baseline; gap: 8px;">
              <div style="font-size: 13.5px;">판매 도매</div>
              <div class="num" style="font-size: 17px; font-weight: 600; color: #B3261E;">-100%</div>
              <div style="font-size: 11.5px; color: #6B6A65;">전월 대비</div>
            </div>
            <div class="num" style="font-size: 11.5px; color: #6B6A65;">418대 → 0대</div>
          </div>
          <div data-el="15" style="background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 6px;">
            <div class="lbl">대시보드 안에서의 분해</div>
            <div style="font-size: 13px; line-height: 1.6;">투싼 <span class="num" style="font-weight:600;">-52%p</span> · 아반떼 <span class="num" style="font-weight:600;">-31%p</span> · 나머지 <span class="num" style="font-weight:600;">-17%p</span></div>
            <div style="font-size: 11px; color: #9A9890;">A3 판매 리포트가 낸 값</div>
          </div>
        </div>

        <div style="display: flex; flex-direction: column; gap: 9px;">
          <div style="display: flex; align-items: center; gap: 8px; flex-wrap: wrap;">
            <div data-el="16" style="font-size: 14px; font-weight: 600;">호르무즈 해협 봉쇄 위협</div>
            <span data-el="17" style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">변동 시점과 1일 차이</span>
            <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">기사 8건 · 출처 5곳</span>
          </div>
          <div data-el="18" style="font-size: 13.5px; line-height: 1.7; color: #1A1B19; text-wrap: pretty;">
            도매가 멈춘 시점과 해협 사건의 보도 구간이 겹칩니다.<span class="fn">1</span> 같은 기간 항해중 재고가 418대로 늘어 선적은 됐으나 하역이 지연된 모습입니다.<span class="fn">5</span> 건화물운임은 10.1% 내렸습니다.<span class="fn">3</span> 오만 물량은 전부 완성차라 해상 구간 지연에 그대로 노출됩니다.
          </div>
        </div>

        <div style="display: flex; align-items: center; gap: 10px; border-top: 1px solid #EFEDE7; padding-top: 12px;">
          <button data-el="19" style="display: inline-flex; align-items: center; gap: 6px; height: 32px; padding: 0 12px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #1A1B19; font-size: 13px; font-weight: 500; border-radius: 7px; font-family: inherit; cursor: pointer;">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"></path></svg>
            근거 펼치기
          </button>
          <div style="font-size: 11.5px; color: #6B6A65;">후보 4건 · 시점 가까운 순</div>
        </div>
      </div>

      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 18px 20px; display: flex; flex-direction: column; gap: 14px;">
        <div style="display: flex; align-items: center; gap: 10px; flex-wrap: wrap;">
          <div style="font-size: 16px; font-weight: 600;">아랍에미리트</div>
          <span style="font-size: 11px; font-weight: 600; color: #B3261E; background: rgba(179,38,30,.08); border: 1px solid #F0CFCB; padding: 2px 7px; border-radius: 7px;">확인 필요</span>
          <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">CBU 94%</span>
          <span style="font-size: 11px; color: #0B5847; background: #E7F0ED; padding: 2px 7px; border-radius: 7px;">GLOVIS ME</span>
        </div>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 12px;">
          <div style="background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 6px;">
            <div class="lbl">변동</div>
            <div style="display: flex; align-items: baseline; gap: 8px;">
              <div style="font-size: 13.5px;">판매 도매</div>
              <div class="num" style="font-size: 17px; font-weight: 600; color: #B3261E;">-62%</div>
              <div style="font-size: 11.5px; color: #6B6A65;">전월 대비</div>
            </div>
            <div class="num" style="font-size: 11.5px; color: #6B6A65;">1,204대 → 458대</div>
          </div>
          <div style="background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 6px;">
            <div class="lbl">대시보드 안에서의 분해</div>
            <div style="font-size: 13px; line-height: 1.6;">두바이 대리점 <span class="num" style="font-weight:600;">-41%p</span> · 아부다비 <span class="num" style="font-weight:600;">-21%p</span></div>
            <div style="font-size: 11px; color: #9A9890;">A3 판매 리포트가 낸 값</div>
          </div>
        </div>

        <div style="display: flex; flex-direction: column; gap: 9px;">
          <div style="display: flex; align-items: center; gap: 8px; flex-wrap: wrap;">
            <div style="font-size: 14px; font-weight: 600;">호르무즈 해협 봉쇄 위협</div>
            <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">변동 시점과 2일 차이</span>
            <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">오만과 같은 사건</span>
          </div>
          <div style="font-size: 13.5px; line-height: 1.7; color: #1A1B19; text-wrap: pretty;">
            오만과 같은 사건에 걸려 있습니다.<span class="fn">1</span> 다만 감소 폭이 더 작고 항해중 재고 비중도 28%에 머물러, 하역 지연이 아직 일부 대리점에만 닿은 것으로 보입니다.<span class="fn">6</span>
          </div>
        </div>

        <div style="display: flex; align-items: center; gap: 10px; border-top: 1px solid #EFEDE7; padding-top: 12px;">
          <button style="display: inline-flex; align-items: center; gap: 6px; height: 32px; padding: 0 12px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #1A1B19; font-size: 13px; font-weight: 500; border-radius: 7px; font-family: inherit; cursor: pointer;">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"></path></svg>
            근거 펼치기
          </button>
          <div style="font-size: 11.5px; color: #6B6A65;">후보 3건 · 시점 가까운 순</div>
        </div>
      </div>

      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 18px 20px; display: flex; flex-direction: column; gap: 14px;">
        <div style="display: flex; align-items: center; gap: 10px; flex-wrap: wrap;">
          <div style="font-size: 16px; font-weight: 600;">파키스탄</div>
          <span style="font-size: 11px; font-weight: 600; color: #B5730C; background: rgba(181,115,12,.10); border: 1px solid rgba(181,115,12,.28); padding: 2px 7px; border-radius: 7px;">주의</span>
          <span style="font-size: 11px; color: #0B5847; background: #E7F0ED; padding: 2px 7px; border-radius: 7px;">상승</span>
          <span style="font-size: 11px; color: #9A9890; background: #FBFAF8; border: 1px solid #E2DFD8; padding: 2px 7px; border-radius: 7px;">노출 미확인</span>
        </div>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 12px;">
          <div style="background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 6px;">
            <div class="lbl">변동</div>
            <div style="display: flex; align-items: baseline; gap: 8px;">
              <div style="font-size: 13.5px;">재고 선적대기</div>
              <div class="num" style="font-size: 17px; font-weight: 600; color: #B5730C;">+38%</div>
              <div style="font-size: 11.5px; color: #6B6A65;">전월 대비</div>
            </div>
            <div class="num" style="font-size: 11.5px; color: #6B6A65;">612대 → 845대</div>
          </div>
          <div style="background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 6px;">
            <div class="lbl">대시보드 안에서의 분해</div>
            <div style="font-size: 13px; line-height: 1.6; color: #6B6A65;">국가 시트 없음. 완성차 원장 보완 집계로 만든 변동입니다.</div>
            <div style="font-size: 11px; color: #B5730C;">보완 집계</div>
          </div>
        </div>

        <div style="display: flex; flex-direction: column; gap: 9px;">
          <div style="display: flex; align-items: center; gap: 8px; flex-wrap: wrap;">
            <div style="font-size: 14px; font-weight: 600;">카라치항 하역 인력 파업</div>
            <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">변동 시점과 3일 차이</span>
            <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">기사 3건 · 출처 2곳</span>
          </div>
          <div style="font-size: 13.5px; line-height: 1.7; color: #1A1B19; text-wrap: pretty;">
            선적대기가 늘어난 구간과 파업 보도가 겹칩니다.<span class="fn">8</span> 다만 기사 3건에 출처 2곳으로 근거 기준에 미치지 못하고 차종 노출도 확인되지 않아 주의까지만 둡니다.
          </div>
        </div>

        <div style="display: flex; align-items: center; gap: 10px; border-top: 1px solid #EFEDE7; padding-top: 12px;">
          <button style="display: inline-flex; align-items: center; gap: 6px; height: 32px; padding: 0 12px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #1A1B19; font-size: 13px; font-weight: 500; border-radius: 7px; font-family: inherit; cursor: pointer;">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"></path></svg>
            근거 펼치기
          </button>
          <div style="font-size: 11.5px; color: #6B6A65;">후보 2건 · 시점 가까운 순</div>
        </div>
      </div>

      <div data-el="25" style="background: #FBFAF8; border: 1px dashed #DAD6CE; border-radius: 7px; padding: 16px 20px; display: flex; align-items: center; gap: 14px;">
        <div style="font-size: 15px; font-weight: 600; color: #6B6A65;">인도</div>
        <div style="display: flex; align-items: baseline; gap: 6px;">
          <div style="font-size: 13px; color: #6B6A65;">재고 체류</div>
          <div class="num" style="font-size: 15px; font-weight: 600; color: #6B6A65;">+24%</div>
        </div>
        <div style="font-size: 13px; color: #9A9890;">시간창 안에서 원인 후보를 찾지 못했습니다. 신호등을 붙이지 않습니다.</div>
        <div style="margin-left: auto; font-size: 11.5px; color: #9A9890;">원인 미확인</div>
      </div>

    </div>
  </div>

  <div data-el="26" style="display: flex; align-items: center; justify-content: space-between; border-top: 1px solid #E2DFD8; padding-top: 14px; font-size: 11px; color: #9A9890;">
    <div>모든 문장의 각주는 아래 근거 목록의 번호와 이어집니다. 인과관계는 확정하지 않습니다.</div>
    <div class="num">2026-08-20 · 배치 #4127</div>
  </div>

</div>
<div class="var">근거 패널 펼친 상태(20). 같은 카드 안에서 열리며 화면을 떠나지 않는다</div>
<style>
body { margin: 0; background: #F4F2EE; color: #1A1B19;
      font-family: 'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif;
      -webkit-font-smoothing: antialiased; }
    a { color: #0E6B57; text-decoration: none; }
    a:hover { color: #0B5847; text-decoration: underline; }
    .num { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-variant-numeric: tabular-nums; }
    .lbl { font-size: 10.5px; font-weight: 500; text-transform: uppercase; letter-spacing: .09em; color: #9A9890; }
    .fn { font-size: 10px; vertical-align: super; color: #0E6B57; font-weight: 600; }
</style>
<div data-el="20" style="width: 760px; padding: 24px; box-sizing: border-box;">
  <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 18px 20px; display: flex; flex-direction: column; gap: 14px;">

    <div style="display: flex; align-items: center; gap: 10px; flex-wrap: wrap;">
      <div style="font-size: 16px; font-weight: 600;">오만</div>
      <span style="font-size: 11px; font-weight: 600; color: #B3261E; background: rgba(179,38,30,.08); border: 1px solid #F0CFCB; padding: 2px 7px; border-radius: 7px;">확인 필요</span>
      <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 2px 7px; border-radius: 7px;">CBU 100%</span>
      <div style="margin-left: auto; display: flex; align-items: baseline; gap: 8px;">
        <div style="font-size: 13px; color: #6B6A65;">판매 도매</div>
        <div class="num" style="font-size: 16px; font-weight: 600; color: #B3261E;">-100%</div>
      </div>
    </div>

    <div style="display: flex; align-items: center; gap: 10px; border-top: 1px solid #EFEDE7; border-bottom: 1px solid #EFEDE7; padding: 12px 0;">
      <button style="display: inline-flex; align-items: center; gap: 6px; height: 32px; padding: 0 12px; border: 1px solid #0E6B57; background: #E7F0ED; color: #0B5847; font-size: 13px; font-weight: 500; border-radius: 7px; font-family: inherit; cursor: pointer;">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m18 15-6-6-6 6"></path></svg>
        근거 접기
      </button>
      <div style="font-size: 11.5px; color: #6B6A65;">후보 4건 · 변동 시점과 가까운 순</div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 12px;">

      <div data-el="21" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
        <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
          <div style="font-size: 13.5px; font-weight: 600;">호르무즈 해협 봉쇄 위협</div>
          <span style="font-size: 11px; color: #6B6A65;">사건 · 해상운송 · 시점 1일 차이 · 출처 5곳</span>
          <span class="fn" style="margin-left: auto;">1</span>
        </div>
        <div style="padding: 4px 0;">
          <div style="display: flex; align-items: baseline; gap: 10px; padding: 9px 14px;">
            <a href="#" style="font-size: 13px; flex: 1;">Hormuz tensions disrupt Gulf vehicle discharge schedules</a>
            <span style="font-size: 11px; color: #6B6A65; white-space: nowrap;">Khaleej Times</span>
            <span class="num" style="font-size: 11px; color: #9A9890; white-space: nowrap;">08-12</span>
          </div>
          <div style="display: flex; align-items: baseline; gap: 10px; padding: 9px 14px; border-top: 1px solid #F4F2EE;">
            <a href="#" style="font-size: 13px; flex: 1;">Shipping lines add war risk surcharge on Gulf routes</a>
            <span style="font-size: 11px; color: #6B6A65; white-space: nowrap;">Lloyd's List</span>
            <span class="num" style="font-size: 11px; color: #9A9890; white-space: nowrap;">08-14</span>
          </div>
          <div style="display: flex; align-items: baseline; gap: 10px; padding: 9px 14px; border-top: 1px solid #F4F2EE;">
            <a href="#" style="font-size: 13px; flex: 1;">Oman port discharge backlog grows for third week</a>
            <span style="font-size: 11px; color: #6B6A65; white-space: nowrap;">Times of Oman</span>
            <span class="num" style="font-size: 11px; color: #9A9890; white-space: nowrap;">08-19</span>
          </div>
          <div style="padding: 8px 14px 10px; font-size: 11.5px; color: #9A9890; border-top: 1px solid #F4F2EE;">기사 8건 중 상위 3건. 해협 사건은 인접국 표로 오만·아랍에미리트에 붙었습니다.</div>
        </div>
      </div>

      <div data-el="22" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
        <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
          <div style="font-size: 13.5px; font-weight: 600;">같은 국가 재고 변동</div>
          <span style="font-size: 11px; color: #6B6A65;">타 도메인 · A2 재고 · 같은 국가 같은 일자</span>
          <span class="fn" style="margin-left: auto;">5</span>
        </div>
        <div style="padding: 12px 14px; display: flex; align-items: center; gap: 20px;">
          <div style="display: flex; flex-direction: column; gap: 3px;">
            <div style="font-size: 11.5px; color: #6B6A65;">항해중 재고</div>
            <div class="num" style="font-size: 17px; font-weight: 600;">418대</div>
          </div>
          <div style="display: flex; flex-direction: column; gap: 3px;">
            <div style="font-size: 11.5px; color: #6B6A65;">총재고 대비</div>
            <div class="num" style="font-size: 17px; font-weight: 600;">42%</div>
          </div>
          <div style="display: flex; flex-direction: column; gap: 3px;">
            <div style="font-size: 11.5px; color: #6B6A65;">선적대기</div>
            <div class="num" style="font-size: 17px; font-weight: 600;">0대</div>
          </div>
          <div style="font-size: 11.5px; color: #9A9890; margin-left: auto; max-width: 200px; line-height: 1.5;">선적은 됐으나 하역이 되지 않은 상태로 읽힙니다.</div>
        </div>
      </div>

      <div data-el="23" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
        <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
          <div style="font-size: 13.5px; font-weight: 600;">건화물운임 BDI</div>
          <span style="font-size: 11px; color: #6B6A65;">시장 지표 · 날짜만 일치 · 국가 축 없음</span>
          <span class="fn" style="margin-left: auto;">3</span>
        </div>
        <div style="padding: 12px 14px; display: flex; align-items: center; gap: 20px;">
          <div style="display: flex; align-items: baseline; gap: 8px;">
            <div class="num" style="font-size: 17px; font-weight: 600;">2,776</div>
            <div class="num" style="font-size: 12px; font-weight: 600; color: #0E6B57;">-10.1%</div>
            <div class="num" style="font-size: 11px; color: #9A9890;">08-20</div>
          </div>
          <svg width="220" height="36" viewBox="0 0 220 36" fill="none" style="flex: none;">
            <polyline points="0,10 20,8 40,12 60,9 80,7 100,11 120,14 140,13 160,18 180,24 200,27 220,29" stroke="#2F6FA8" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"></polyline>
          </svg>
          <div style="font-size: 11.5px; color: #9A9890; margin-left: auto;">30일</div>
        </div>
      </div>

      <div data-el="24" style="display: flex; align-items: center; gap: 10px; border: 1px solid #E2DFD8; border-radius: 7px; background: #FBFAF8; padding: 11px 14px;">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m9 18 6-6-6-6"></path></svg>
        <div style="font-size: 13px; color: #6B6A65;">설명에 쓰이지 않은 후보 1건</div>
        <div style="font-size: 11.5px; color: #9A9890;">중동 전력망 증설 계약 · 기사 2건 · 시점 9일 차이</div>
      </div>

    </div>

    <div style="font-size: 11px; color: #9A9890; border-top: 1px solid #EFEDE7; padding-top: 12px; line-height: 1.6;">
      각주 번호를 누르면 해당 근거가 강조됩니다. 설명에 쓰인 사실은 전부 위 후보 안에 있습니다. 기사 제목을 누르면 새 창으로 원문이 열립니다.
    </div>

  </div>
</div>
<div class="var">예외 상태 여섯(27). E1 변동 없음 · E2 첫 배치 전 · E3 강등 · E4 판정 미수신 · E5 배치 실패 · E6 지표 이월</div>
<style>
body { margin: 0; background: #F4F2EE; color: #1A1B19;
      font-family: 'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif;
      -webkit-font-smoothing: antialiased; }
    a { color: #0E6B57; text-decoration: none; }
    a:hover { color: #0B5847; text-decoration: underline; }
    .num { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-variant-numeric: tabular-nums; }
    .lbl { font-size: 10.5px; font-weight: 500; text-transform: uppercase; letter-spacing: .09em; color: #9A9890; }
    .cap { font-size: 11.5px; color: #6B6A65; }
</style>
<div data-el="27" style="width: 1280px; padding: 28px 32px; box-sizing: border-box; display: flex; flex-direction: column; gap: 18px;">

  <div style="display: flex; align-items: baseline; gap: 10px;">
    <div style="font-size: 16px; font-weight: 600;">예외 상태</div>
    <div class="cap">화면이 비지 않아야 하는 다섯 경우</div>
  </div>

  <div style="display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 16px;">

    <div style="display: flex; flex-direction: column; gap: 8px;">
      <div class="lbl">1. 변동이 없는 날</div>
      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 26px 20px; display: flex; flex-direction: column; align-items: center; gap: 8px;">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="#0E6B57" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="9"></circle><path d="m9 12 2 2 4-4"></path></svg>
        <div style="font-size: 14px; font-weight: 600;">오늘 감지된 변동 없음</div>
        <div class="cap" style="text-align: center; line-height: 1.6;">세 도메인 모두 임계값 안입니다. 정상 상태이며 오류가 아닙니다.</div>
      </div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 8px;">
      <div class="lbl">2. 첫 배치 전</div>
      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 26px 20px; display: flex; flex-direction: column; align-items: center; gap: 8px;">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="9"></circle><path d="M12 7v5l3 2"></path></svg>
        <div style="font-size: 14px; font-weight: 600;">리포트가 아직 생성되지 않았습니다</div>
        <div class="cap" style="text-align: center; line-height: 1.6;">다음 배치는 <span class="num">내일 02:00</span>에 시작합니다.</div>
      </div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 8px;">
      <div class="lbl">3. 문장 생성 실패 · 강등</div>
      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
        <div style="display: flex; align-items: center; gap: 8px; background: rgba(181,115,12,.08); border-bottom: 1px solid rgba(181,115,12,.28); padding: 9px 14px;">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#B5730C" stroke-width="2" stroke-linecap="round"><path d="M12 8v5"></path><path d="M12 16h.01"></path><circle cx="12" cy="12" r="9"></circle></svg>
          <div style="font-size: 12px; color: #B5730C; font-weight: 500;">생성 실패, 판정값만 표시</div>
        </div>
        <div style="padding: 14px; display: flex; flex-direction: column; gap: 10px;">
          <div style="display: flex; align-items: center; gap: 10px;">
            <div style="font-size: 15px; font-weight: 600;">오만</div>
            <span style="font-size: 11px; font-weight: 600; color: #B3261E; background: rgba(179,38,30,.08); border: 1px solid #F0CFCB; padding: 2px 7px; border-radius: 7px;">확인 필요</span>
            <div class="num" style="font-size: 13px; color: #6B6A65; margin-left: auto;">판매 도매 -100%</div>
          </div>
          <div style="font-size: 13px; color: #6B6A65; line-height: 1.6; background: #FBFAF8; border: 1px solid #EFEDE7; border-radius: 7px; padding: 10px 12px;">
            오만 · 판매 도매 전월 대비 -100% · 항해중 재고 418대 · 호르무즈 해협 봉쇄 위협 기사 8건
          </div>
          <div class="cap">템플릿 문장입니다. 연관 설명이 붙지 않았습니다.</div>
        </div>
      </div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 8px;">
      <div class="lbl">4. 대시보드 판정 미수신</div>
      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
        <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; border-bottom: 1px solid #EFEDE7; padding: 9px 14px;">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#6B6A65" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 3v18h18"></path><path d="m7 14 4-4 3 3 5-6"></path></svg>
          <div style="font-size: 12px; color: #6B6A65; font-weight: 500;">A2 재고 리포트 판정 미수신</div>
        </div>
        <div style="padding: 14px; display: flex; flex-direction: column; gap: 10px;">
          <div style="display: flex; align-items: center; gap: 10px;">
            <span style="width: 8px; height: 8px; border-radius: 50%; background: #B5730C; display: inline-block;"></span>
            <div style="font-size: 15px; font-weight: 600;">재고</div>
            <span style="font-size: 11px; color: #B5730C; background: rgba(181,115,12,.10); border: 1px solid rgba(181,115,12,.28); padding: 2px 7px; border-radius: 7px; margin-left: auto;">보완 집계</span>
          </div>
          <div style="font-size: 13px; line-height: 1.65;">완성차 원장에서 직접 집계한 값으로 변동을 만들었습니다. 대시보드 수치와 다를 수 있습니다.</div>
          <div class="cap">기여 분해는 표시하지 않습니다.</div>
        </div>
      </div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 8px;">
      <div class="lbl">5. 배치 실패 · 이전 게시본 유지</div>
      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
        <div style="display: flex; align-items: center; gap: 8px; background: rgba(179,38,30,.06); border-bottom: 1px solid #F0CFCB; padding: 9px 14px;">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 9v4"></path><path d="M12 17h.01"></path><path d="M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0Z"></path></svg>
          <div style="font-size: 12px; color: #B3261E; font-weight: 500;">오늘 배치가 실패했습니다</div>
          <div class="num" style="font-size: 11px; color: #6B6A65; margin-left: auto;">마지막 성공 2026-08-19</div>
        </div>
        <div style="padding: 14px; font-size: 13px; line-height: 1.65; color: #6B6A65;">
          <span class="num" style="color:#1A1B19;">8월 19일</span> 리포트를 그대로 보여 줍니다. 화면 상단 기준일도 그날로 표시됩니다. 원인 후보와 근거는 그날 것입니다.
        </div>
      </div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 8px;">
      <div class="lbl">참고. 지표 바 지연</div>
      <div style="background: #FFFFFF; border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 10px;">
        <div style="display: flex; align-items: center; gap: 16px;">
          <div style="display: flex; flex-direction: column; gap: 3px;">
            <div class="cap">컨테이너운임 SCFI</div>
            <div style="display: flex; align-items: baseline; gap: 6px;">
              <div class="num" style="font-size: 17px; font-weight: 600;">1,942</div>
              <div class="num" style="font-size: 11.5px; color: #9A9890;">-0.4%</div>
            </div>
          </div>
          <div style="font-size: 11.5px; color: #B5730C;">08-18 값 이월</div>
        </div>
        <div style="font-size: 13px; line-height: 1.65; color: #6B6A65;">이월 한도를 넘기면 값을 비우고 기준일 지연으로만 표시합니다. 판정에는 쓰지 않습니다.</div>
      </div>
    </div>

  </div>
</div>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 페이지 | 페이지 | 제품명, "종합 리포트" | 없음 |
| 2 | 기준일과 소스 최신일 | 텍스트 | 리포트 기준일, 그리고 완성차·뉴스·시장 세 소스의 데이터 최신일을 각각 따로 | 없음 |
| 3 | 시장지표 바 | 4칸 그리드 | GPR, Brent, BDI, SCFI의 최신값·변화율·기준일 | 없음 |
| 4 | 헤드라인 | 카드 | 19px 한 문장, 강조 토큰, 각주 번호 | 각주는 해당 근거 강조 |
| 5 | 인과 배지 | 칩 | "상관관계 확인 · 인과관계 미확정" 고정 문구 | 없음 |
| 6 | 도메인 상태 접기 | 버튼 | 접힘 여부 | 3카드 영역 접고 폄. 상태는 브라우저에 기억 |
| 7 | 도메인 상태 카드 | 카드 3 | 신호등과 상태 텍스트, 종합 관점 문장, 하단에 판정 출처와 기여 상위. 생산 카드의 신호등은 "판정 대상 아님" | 각주는 근거 강조 |
| 8 | 변동 목록 머리 | 줄 | 국가 건수, "신호등 순 정렬" | 없음 |
| 9 | 변동 카드 | 카드 | 국가 하나의 변동 한 건 | 없음 |
| 10 | 신호등 칩 | 칩 | red는 "확인 필요", yellow는 "주의". 색과 텍스트를 함께 | 없음 |
| 11 | 변화 배지 | 칩 | "신규" 또는 "상승". 직전 게시본과 비교한 결과 | 없음 |
| 12 | 노출 칩 | 칩 | CBU 비중. 확인 못 하면 "노출 미확인" | 없음 |
| 13 | 법인 태그 | 칩 | 글로비스 법인명. 매핑 없으면 "법인 미매핑" | 없음 |
| 14 | 변동 박스 | 보조면 박스 | 지표명, 변화율, 기준 방식, 기간 단위, 원값 두 개 | 없음 |
| 15 | 분해 박스 | 보조면 박스 | A 판정의 기여 상위 항목. 출처 표기 | 없음 |
| 16 | 첫 후보 제목 | 제목 | 정렬 1순위 후보의 이름 | 없음 |
| 17 | 근접도 칩 | 칩 다수 | 시점 차이, 기사 수와 출처 수, 국가 일치 방식 | 없음 |
| 18 | 연관 설명 | 문단 | 후보와 변동을 잇는 설명 한 문단, 각주 | 각주는 근거 강조 |
| 19 | 근거 펼치기 | 버튼 | 접힘 여부, 후보 건수와 정렬 기준 | 20번 패널을 펼침 |
| 20 | 근거 패널 | 영역 | 후보 전부를 정렬 순서대로, 잘린 건수 | 없음 |
| 21 | 사건 후보 블록 | 블록 | 사건 제목, 유형, 근접도, 기사 3건 이상(제목·출처·게시일) | 기사 제목은 새 창 원문 |
| 22 | 타 도메인 후보 블록 | 블록 | 같은 국가 다른 도메인의 변동값 | 없음 |
| 23 | 시장지표 후보 블록 | 블록 | 지표 원값, 변화율, 30일 스파크라인 | 없음 |
| 24 | 미사용 후보 줄 | 접힌 줄 | 설명에 쓰이지 않은 후보의 수와 요약 | 펼치면 같은 형식의 블록 |
| 25 | 원인 미확인 행 | 점선 박스 | 변동은 있으나 후보가 없는 국가 | 없음 |
| 26 | 페이지 꼬리 | 푸터 | 각주 안내 문구, 기준일, 배치 번호 | 없음 |
| 27 | 예외 상태 자리 | 영역 | 아래 "예외 상태" 여섯 중 해당하는 것 | 상태마다 다름 |

##### 규칙

- 열람 시 LLM을 호출하지 않는다. 게시된 리포트를 한 번의 호출로 조회만 한다([[TBL-PRD-002#N5]], p95 2초).
- 외부 자원을 불러오지 않는다. 기사 원문은 새 창 링크로만 연다. 폐쇄망이다([[TBL-PRD-002#N1]], [[TBL-INFRA-002#C10]]).
- 데이터 최신일은 완성차·뉴스·시장 셋을 각각 적는다. 한 칸으로 합치지 않는다. 한 줄로 줄여야 하는 자리에서는 게시 응답이 준 대표 최신일을 쓰고 화면이 최댓값을 다시 계산하지 않는다.
- 변동 목록 정렬은 신호등 순이다. red, yellow, none 차례다. 같은 신호등 안에서는 국가 단위로 묶는다.
- 카드 안 후보 정렬은 날짜 차이 오름차순, 같으면 출처 수 내림차순, 그다음 기사 수 내림차순이다([[TBL-DOM-004#CauseCandidate]]). 이 기준을 카드 하단과 근거 패널 머리에 글자로 적는다. 문구는 게시 응답의 후보 정렬 규칙 한 칸을 그대로 쓴다.
- 동률일 때는 후보 식별자 사전순으로 갈린다. 이 넷째 기준은 화면 문구에 적지 않는다. 순서가 흔들리지 않게 하는 장치이지 읽는 사람이 알아야 할 기준이 아니다.
- 관련도 등급이나 점수를 뜻하는 칩, 별점, 막대, 색 농도를 두지 않는다. 화면에 나오는 숫자는 코드가 센 값뿐이다.
- 신호등은 색과 텍스트를 함께 쓴다. 점 하나만 두지 않는다.
- 도메인 상태 3카드의 신호등도 규칙 산출물이다. 배치 5단계가 도메인 안 변동들의 신호등 최댓값으로 정하고 판정 출처(A 판정·보완 집계·미수신)와 기여 상위를 함께 낸다. 8단계 LLM은 그 카드의 문장만 쓴다. LLM 세 역할이 전부 실패해도 신호등과 판정 출처와 기여 상위는 그대로 나온다([[TBL-INFRA-002#C19]], [[TBL-PRD-002#N2]]).
- 생산 카드의 신호등 자리에는 "판정 대상 아님"을 적는다. 생산은 목적지 국가가 없어 국가 축 판정 대상이 아니기 때문이다. 후보를 찾지 못한 "원인 미확인"과 뜻이 다르므로 두 표기를 바꿔 쓰지 않는다(3.4절).
- 후보가 상한에 걸려 잘렸으면 잘린 건수를 근거 패널에 적는다([[TBL-UC-002#UC-S5]] 6a).
- 생산 변동은 이 목록에 오르지 않는다. 목적지 국가가 없어 국가 축에 붙지 못한다([[TBL-PRD-002#R16]], [[TBL-INFRA-002#C16]]). 생산은 도메인 상태 카드 한 줄로만 들어가고 그 문장에는 외부 원인 인용이 없다([[TBL-PRD-002#R21]]).
- 재고 값이 유도값이면 카드에 유도임을 표시한다([[TBL-PRD-002#R9]], [[TBL-DOM-004#SalesStageFlow]]).
- A 판정이 국가 단위가 아니어서 원장 보완 집계로 만든 변동은 분해 박스에 "보완 집계"를 표시하고 기여 분해를 비운다([[TBL-UC-002#UC-H2]] 5b).
- 비교 기준을 변동 박스에 함께 적는다. 계획 대비, 전년 동월, 전월 대비 중 무엇인지 글자로 나온다([[TBL-PRD-002#R13]]).
- 기간 단위를 표시한다. 형태 B에서 온 값이면 일·월·누계·년 중 무엇인지 적는다([[TBL-INFRA-002#C15]]).
- 수치는 등폭 글꼴에 `tabular-nums`를 쓴다. 자릿수가 흔들리면 비교가 안 된다.
- 도메인 상태 접기 상태는 브라우저에 기억한다. 서버에 저장하지 않는다.
- 각주 번호는 후보와 설명 ID에 이어진다. 각주 없는 주장을 두지 않는다([[TBL-PRD-002#R21]], [[TBL-PRD-002#N4]]).

##### 예외 상태

여섯 가지다. 어느 경우에도 화면이 비지 않는다.

| # | 상태 | 트리거 | 화면 |
|---|---|---|---|
| E1 | 변동 없음 | 세 도메인 모두 임계값 안 | 카드 중앙에 "오늘 감지된 변동 없음". 정상 상태이며 오류가 아니라고 적는다. 시장지표 바와 도메인 상태는 그대로 |
| E2 | 첫 배치 전 | 게시된 리포트가 0건 | "리포트가 아직 생성되지 않았습니다"와 다음 배치 예정 시각 |
| E3 | 생성 실패 강등 | LLM 세 역할 중 실패([[TBL-UC-002#UC-S11]]) | 카드 머리에 주의색 띠 "생성 실패, 판정값만 표시". 문장 자리에 템플릿 한 줄. 신호등, 후보, 근접도, 도메인 상태 신호등은 그대로 |
| E4 | 대시보드 판정 미수신 | A 리포트 배치 실패([[TBL-UC-002#UC-S2]] 2a) | 도메인 카드 머리에 "A2 재고 리포트 판정 미수신", 카드에 "보완 집계" 칩. 기여 분해는 표시하지 않음 |
| E5 | 배치 실패 | 오늘 배치 중단, 이전 게시본 유지 | 상단에 경고색 띠 "오늘 배치가 실패했습니다"와 마지막 성공일. 본문은 그날 리포트 그대로이고 기준일도 그날로 표시 |
| E6 | 지표 바 지연 | 시장지표 이월 | 해당 지표 기준일 자리에 주의색 "08-18 값 이월". 이월이어도 네 칸에 값이 있고 칸을 비우지 않는다. 이월값은 판정에 쓰지 않는다([[TBL-PRD-002#R27]]) |

여섯 중 다섯은 게시 응답의 안내 목록과 1대1로 맞춘다. 화면이 스스로 상태를 판단하지 않고 게시된 값을 그대로 읽는다.

| 상태 | 안내 코드 | 어느 배치 단계에서 나오나 | 붙는 자리 |
|---|---|---|---|
| E1 | `noAnomaly` | 5 원인 후보·근접도·신호등 | 리포트 전체 |
| E3 | `generationDegraded` | 6~8 강등 | 리포트 전체. 해당 카드 머리 |
| E4 | `aJudgmentNotReceived` | 2 A 판정 읽기 | 해당 도메인 카드 |
| E5 | `batchFailed` | 1·3·4·5 멈춤 | 페이지 상단 |
| E6 | `marketCarriedOver` | 4 결합 시점의 지표 이월 | 시장지표 바의 해당 지표 |

E2는 게시본 자체가 없는 상태라 대응하는 안내 코드가 없다. 조회 결과가 0건인 것으로 화면이 판단한다.

안내 목록 한 줄의 도메인은 E4에서 어느 도메인 카드에 띠를 붙일지 고르는 데 쓰고, 마지막 성공일은 E5 띠에 쓴다. 문구는 화면 문구를 대체하지 않고 상세 안내로만 쓴다([[TBL-DOM-004#CReport]]).

##### 시나리오

**S-1 오늘의 리포트를 연다** [[TBL-UC-002#UC-H2]]
1. H가 VODA 단독 주소를 연다.
2. 시장지표 바, 헤드라인, 도메인 상태 3카드, 변동 목록이 2초 안에 그려진다.
3. 생산 카드의 신호등 자리에 "판정 대상 아님"이 적혀 있다. 재고와 판매 카드에는 신호등이 붙는다.
4. H가 도메인 상태 접기를 눌러 3카드를 접는다. 다음 방문에도 접힌 채로 열린다.

**S-2 근거를 펼쳐 순서를 확인한다** [[TBL-UC-002#UC-H3]]
1. H가 오만 카드의 근거 펼치기를 누른다.
2. 후보 4건이 시점 가까운 순으로 펼쳐진다. 후보마다 근접도 값 넷이 적혀 있다.
3. H가 기사 제목을 누르면 새 창으로 원문이 열린다. 화면은 그대로 있다.
4. H가 설명의 각주 5를 누르면 타 도메인 후보 블록이 강조된다.
5. H가 정렬 기준 문구를 읽고 왜 이 순서인지 확인한다. 등급 표시는 어디에도 없다.

**S-3 원인 후보가 없는 변동을 본다** [[TBL-UC-002#UC-S5]] 2a
1. 인도의 재고 체류 +24%가 목록 끝에 점선 박스로 나온다.
2. "시간창 안에서 원인 후보를 찾지 못했습니다"가 함께 나오고 신호등이 붙지 않는다.

**S-4 문장 생성이 실패한 날** [[TBL-UC-002#UC-S11]]
1. 카드 머리에 "생성 실패, 판정값만 표시" 띠가 붙는다. 게시 응답의 안내 목록에 `generationDegraded`가 들어 있다.
2. 설명 자리에 템플릿 한 줄이 들어간다. 신호등 "확인 필요", 후보 목록, 근접도 값, 도메인 상태 카드의 신호등은 평소와 같다.

**연관**: [[TBL-PRD-002#R25]] · [[TBL-PRD-002#R26]] · [[TBL-PRD-002#R27]] · [[TBL-PRD-002#R21]] · [[TBL-PRD-002#R16]] · [[TBL-PRD-002#R29]] · [[TBL-PRD-002#N2]] · [[TBL-PRD-002#N5]] · [[TBL-DOM-004#CReport]] · [[TBL-DOM-004#CauseCandidate]] · [[TBL-DOM-004#Evidence]] · [[TBL-DOM-004#WatchItem]]

### 2.2 A 리포트

세 화면이 같은 뼈대를 쓴다. 좌측 248px 사이드와 중앙 지면이다. 중앙 지면은 44px 헤더 아래에 요약, 지표 카드, 분해, 해설, 고정 문구가 차례로 놓인다. 전체 폭은 1180px이다.

세 화면 모두 자기 도메인 데이터만 쓴다. 뉴스와 시장지표를 인용하지 않는다([[TBL-PRD-002#R1]]). C 리포트로 가는 버튼이나 링크를 두지 않는다.

세 화면의 요약과 해설은 이 시스템이 새로 쓰는 문장이 아니다. 브리핑 갈래가 A 리포트로 게시한 문장을 각주 근거와 버전까지 통째로 받아 게시 스키마에 사본으로 두고, 세 화면은 그 사본을 읽어 그대로 보여 준다([[TBL-DOM-004#DomainReport]]). 요약과 해설, 고정 문구, 트래킹 지표, 분해, 못 만드는 지표 목록이 모두 사본에서 온다. 사이드 버전 목록의 번호도 사본이 가진 버전 번호다. 그래서 열람 경로가 게시 스키마 밖으로 나가지 않는다([[TBL-INFRA-002#C10]]). C 리포트가 읽는 것은 A 판정뿐이고 A가 쓴 문장은 C의 서술에 쓰지 않는다([[TBL-UC-002#UC-S2]]).

#### UI-2 A1 생산 리포트

| 항목 | 내용 |
|---|---|
| 경로 | `/intel/areport/production` |
| 주 유스케이스 | [[TBL-UC-002#UC-H1]] |
| 진입 / 이탈 | VODA 생산 대시보드의 A1 서명 주소 / 내려받기(PDF). 다른 화면으로 나가는 링크 없음 |

페이지. 생산 대시보드에 붙는 AI 인사이트. 사업계획 대비 달성률을 중심으로 공장과 차종까지 분해한다.

##### 배치

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;600;700&display=swap">
<style>.var{margin:32px 0 8px;font-size:11.5px;color:#A6A49C;border-left:3px solid #DAD6CE;padding-left:10px;font-family:system-ui,sans-serif}</style>

<style>
body { margin: 0; background: #F4F2EE; color: #1A1B19;
      font-family: 'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif;
      -webkit-font-smoothing: antialiased; }
    a { color: #0E6B57; text-decoration: none; }
    a:hover { color: #0B5847; text-decoration: underline; }
    .num { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-variant-numeric: tabular-nums; }
    .lbl { font-size: 10.5px; font-weight: 500; text-transform: uppercase; letter-spacing: .09em; color: #9A9890; }
    .hl { font-weight: 600; background: rgba(181,115,12,.14); border-radius: 3px; padding: 0 2px; }
    .fn { font-size: 10px; vertical-align: super; color: #0E6B57; font-weight: 600; }
    .body { font-size: 12.5px; color: #3C3B37; }
</style>
<div style="width: 1180px; padding: 24px; box-sizing: border-box; display: flex; align-items: flex-start; gap: 20px;">

  <aside data-el="1" style="width: 248px; flex: none; display: flex; flex-direction: column; gap: 20px; border: 1px solid #E2DFD8; background: #FBFAF8; border-radius: 11px; padding: 16px;">
    <div data-el="2" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">출처</div>
      <div style="display: flex; flex-direction: column; gap: 6px;">
        <div style="display: flex; align-items: center; gap: 6px;">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#6B6A65" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 3v18h18"></path><rect x="7" y="10" width="3" height="7"></rect><rect x="13" y="6" width="3" height="11"></rect></svg>
          <span class="body">완성차 생산 대시보드</span>
        </div>
        <div style="font-size: 11.5px; color: #6B6A65;">운영계획·사업계획·실적 × 일·월·누적·년</div>
        <div style="font-size: 11.5px; color: #6B6A65;">스냅샷 <span class="num">2026-08-20 01:38</span></div>
      </div>
    </div>

    <div data-el="3" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">트래킹 지표</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        <div style="display: flex; align-items: center; gap: 7px; background: #FFFFFF; border-radius: 8px; padding: 7px 10px; box-shadow: inset 0 0 0 1px #E2DFD8;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #B5730C;"></span>
          <span class="body" style="font-weight: 500;">사업계획 달성</span>
          <span class="num" style="font-size: 11.5px; color: #B5730C; margin-left: auto;">86.1%</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px; padding: 7px 10px;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
          <span class="body">완성차 비중</span>
          <span class="num" style="font-size: 11.5px; color: #6B6A65; margin-left: auto;">78%</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px; padding: 7px 10px;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
          <span class="body">수출 비중</span>
          <span class="num" style="font-size: 11.5px; color: #6B6A65; margin-left: auto;">37%</span>
        </div>
      </div>
      <div style="font-size: 11px; color: #A6A49C; line-height: 1.5;">달성률은 계획과 실적으로 직접 계산합니다. 원본 진도율 컬럼은 총계 행에만 값이 있습니다.</div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">분해 차원</div>
      <div data-el="4" style="display: flex; flex-wrap: wrap; gap: 5px;">
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">생산법인 28</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">공장지역 18</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">세부지역 41</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">차종그룹 37</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">CBU/CKD</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">내수/수출</span>
      </div>
      <div data-el="5" style="display: flex; flex-direction: column; gap: 6px;">
        <div style="display: flex; align-items: flex-start; gap: 6px; background: rgba(181,115,12,.08); border: 1px solid rgba(181,115,12,.28); border-radius: 7px; padding: 8px 10px;">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#B5730C" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
          <div style="font-size: 11px; color: #B5730C; line-height: 1.5;">목적지 국가 없음. 수출이 어느 나라로 가는지 데이터에 없습니다.</div>
        </div>
        <div style="display: flex; align-items: flex-start; gap: 6px; background: rgba(181,115,12,.08); border: 1px solid rgba(181,115,12,.28); border-radius: 7px; padding: 8px 10px;">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#B5730C" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
          <div style="font-size: 11px; color: #B5730C; line-height: 1.5;">파워트레인은 절반이 미분류라 분해에서 뺐습니다.</div>
        </div>
      </div>
    </div>

    <div data-el="6" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">버전</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        <div class="num" style="background: #FFFFFF; border-radius: 8px; padding: 6px 10px; font-size: 12px; font-weight: 500; box-shadow: inset 0 0 0 1px #E2DFD8;">v12 · 현재</div>
        <div class="num" style="padding: 6px 10px; font-size: 12px; color: #6B6A65;">v11 · 08-19</div>
      </div>
    </div>
  </aside>

  <main style="flex: 1; min-width: 0; border: 1px solid #E2DFD8; background: #FFFFFF; border-radius: 11px; overflow: hidden;">

    <div data-el="7" style="display: flex; align-items: center; gap: 10px; height: 44px; padding: 0 16px; border-bottom: 1px solid #E2DFD8;">
      <span style="background: #E7F0ED; color: #0B5847; font-size: 11px; font-weight: 500; padding: 2px 8px; border-radius: 5px;">A1 생산 리포트</span>
      <span style="font-size: 13px; font-weight: 600;">생산 실적 일일 인사이트</span>
      <span class="num" style="background: #EFEDE7; color: #6B6A65; font-size: 11px; padding: 3px 8px; border-radius: 20px;">v12</span>
      <span style="font-size: 11px; color: #6B6A65; margin-left: 8px;">28개 법인 · 누적 기준</span>
      <div style="margin-left: auto; display: flex; align-items: center; gap: 6px;">
        <button data-el="8" style="display: inline-flex; align-items: center; gap: 6px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #3C3B37; font-size: 12px; padding: 6px 12px; border-radius: 7px; font-family: inherit; cursor: pointer;">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 17V3"></path><path d="m6 11 6 6 6-6"></path><path d="M19 21H5"></path></svg>
          내려받기
        </button>
      </div>
    </div>

    <div style="padding: 22px; display: flex; flex-direction: column; gap: 22px;">

      <div data-el="9" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">요약</div>
        <div style="font-size: 16px; font-weight: 500; line-height: 1.65; letter-spacing: -.01em; text-wrap: pretty;">
          누적 생산이 사업계획의 <span class="hl">86.1%</span>에 머물러 <span class="hl">280,809대</span>가 덜 나왔습니다. 대수로는 한국이 138,268대로 가장 크지만, 달성률로는 <span class="hl">미국 신공장이 45.9%</span>로 가장 낮습니다.<span class="fn">1</span>
        </div>
      </div>

      <div data-el="10" style="display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px;">
        <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 8px;">
          <div style="display: flex; align-items: center; gap: 6px;">
            <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
            <div style="font-size: 13px; font-weight: 600;">누적 사업계획</div>
          </div>
          <div class="num" style="font-size: 22px; font-weight: 600;">2,020,531</div>
          <div style="font-size: 11px; color: #9A9890;">운영계획 기준은 1,754,150대</div>
        </div>
        <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 8px;">
          <div style="display: flex; align-items: center; gap: 6px;">
            <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
            <div style="font-size: 13px; font-weight: 600;">누적 실적</div>
          </div>
          <div class="num" style="font-size: 22px; font-weight: 600;">1,739,722</div>
          <div style="font-size: 11px; color: #9A9890;">계획 대비 -280,809대</div>
        </div>
        <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 8px;">
          <div style="display: flex; align-items: center; gap: 6px;">
            <span style="width: 6px; height: 6px; border-radius: 50%; background: #B5730C;"></span>
            <div style="font-size: 13px; font-weight: 600;">달성률</div>
          </div>
          <div style="display: flex; align-items: baseline; gap: 8px;">
            <div class="num" style="font-size: 22px; font-weight: 600;">86.1%</div>
          </div>
          <div style="height: 8px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
            <div style="width: 86%; height: 100%; background: #C9922E;"></div>
          </div>
        </div>
      </div>

      <div style="display: flex; flex-direction: column; gap: 12px;">
        <div style="display: flex; align-items: baseline; gap: 10px;">
          <div class="lbl">변동을 대시보드 안에서 분해</div>
          <div style="font-size: 11.5px; color: #6B6A65;">사업계획 대비 달성률 · 계획 10,000대 이상 법인</div>
        </div>

        <div data-el="11" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
          <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
            <div style="font-size: 12.5px; font-weight: 600;">법인별 달성률</div>
            <div style="font-size: 11px; color: #9A9890;">낮은 순</div>
          </div>
          <div style="padding: 12px 14px; display: flex; flex-direction: column; gap: 10px;">
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 150px; flex: none;">HMGMA <span style="color:#9A9890;">미국</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 46%; height: 100%; background: #C9922E;"></div>
              </div>
              <div class="num" style="width: 52px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">45.9%</div>
              <div class="num" style="width: 142px; flex: none; text-align: right; font-size: 12px; color: #6B6A65;">16,338 / 35,600</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 150px; flex: none;">BHMC <span style="color:#9A9890;">중국</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 73%; height: 100%; background: #C9922E;"></div>
              </div>
              <div class="num" style="width: 52px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">72.6%</div>
              <div class="num" style="width: 142px; flex: none; text-align: right; font-size: 12px; color: #6B6A65;">75,599 / 104,095</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 150px; flex: none;">HMMI <span style="color:#9A9890;">인도네시아</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 74%; height: 100%; background: #C9922E;"></div>
              </div>
              <div class="num" style="width: 52px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">73.8%</div>
              <div class="num" style="width: 142px; flex: none; text-align: right; font-size: 12px; color: #6B6A65;">24,112 / 32,680</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 150px; flex: none;">HMC <span style="color:#9A9890;">한국</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 85%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div class="num" style="width: 52px; flex: none; text-align: right; font-size: 12.5px;">85.0%</div>
              <div class="num" style="width: 142px; flex: none; text-align: right; font-size: 12px; color: #6B6A65;">786,532 / 924,800</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 150px; flex: none;">HMMC <span style="color:#9A9890;">체코</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 88%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div class="num" style="width: 52px; flex: none; text-align: right; font-size: 12.5px;">88.4%</div>
              <div class="num" style="width: 142px; flex: none; text-align: right; font-size: 12px; color: #6B6A65;">119,740 / 135,400</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 150px; flex: none;">HMB <span style="color:#9A9890;">브라질</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 89%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div class="num" style="width: 52px; flex: none; text-align: right; font-size: 12.5px;">88.8%</div>
              <div class="num" style="width: 142px; flex: none; text-align: right; font-size: 12px; color: #6B6A65;">96,689 / 108,900</div>
            </div>
          </div>
        </div>

        <div data-el="12" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
          <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
            <div style="font-size: 12.5px; font-weight: 600;">생산 구성</div>
            <div style="font-size: 11px; color: #9A9890;">누적 실적 기준</div>
          </div>
          <div style="padding: 14px; display: flex; flex-direction: column; gap: 14px;">
            <div style="display: flex; flex-direction: column; gap: 7px;">
              <div style="display: flex; align-items: baseline; gap: 8px;">
                <div class="body" style="font-weight: 500;">완성차와 반조립</div>
                <div style="font-size: 11px; color: #9A9890;">종합 리포트의 노출 판정에 쓰입니다</div>
              </div>
              <div style="display: flex; height: 20px; border-radius: 4px; overflow: hidden;">
                <div style="width: 78%; height: 100%; background: #2F6FA8;"></div>
                <div style="width: 22%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div style="display: flex; align-items: center; gap: 16px;">
                <div style="display: flex; align-items: center; gap: 5px;"><span style="width: 10px; height: 10px; border-radius: 2px; background: #2F6FA8;"></span><span style="font-size: 11.5px; color: #3C3B37;">완성차 78%</span></div>
                <div style="display: flex; align-items: center; gap: 5px;"><span style="width: 10px; height: 10px; border-radius: 2px; background: #8FB4D2;"></span><span style="font-size: 11.5px; color: #3C3B37;">반조립 22%</span></div>
              </div>
            </div>
            <div style="display: flex; flex-direction: column; gap: 7px;">
              <div style="display: flex; align-items: baseline; gap: 8px;">
                <div class="body" style="font-weight: 500;">내수와 수출</div>
                <div style="font-size: 11px; color: #9A9890;">목적지 국가는 데이터에 없습니다</div>
              </div>
              <div style="display: flex; height: 20px; border-radius: 4px; overflow: hidden;">
                <div style="width: 63%; height: 100%; background: #5C93BF;"></div>
                <div style="width: 37%; height: 100%; background: #C9922E;"></div>
              </div>
              <div style="display: flex; align-items: center; gap: 16px;">
                <div style="display: flex; align-items: center; gap: 5px;"><span style="width: 10px; height: 10px; border-radius: 2px; background: #5C93BF;"></span><span style="font-size: 11.5px; color: #3C3B37;">내수 63%</span></div>
                <div style="display: flex; align-items: center; gap: 5px;"><span style="width: 10px; height: 10px; border-radius: 2px; background: #C9922E;"></span><span style="font-size: 11.5px; color: #3C3B37;">수출 37%</span></div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div data-el="13" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">해설</div>
        <div style="font-size: 13.5px; line-height: 1.75; color: #1A1B19; text-wrap: pretty;">
          미달 폭이 가장 큰 곳은 한국입니다. 138,268대로 전체 미달분의 절반을 차지합니다. 다만 달성률로 보면 85.0%로 평균 수준이고, 계획 자체가 92만대로 크기 때문에 절대 대수가 커진 것입니다.<span class="fn">1</span> 눈에 띄는 쪽은 미국 신공장입니다. 계획 35,600대에 실적 16,338대로 절반에 못 미칩니다. 가동 초기의 상승 곡선인지 실제 차질인지는 이 데이터만으로 가릴 수 없습니다.<span class="fn">2</span> 중국과 인도네시아도 70%대에 머물러 있습니다.
        </div>
      </div>

      <div data-el="14" style="display: flex; align-items: flex-start; gap: 8px; border-top: 1px solid #EFEDE7; padding-top: 14px;">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
        <div style="font-size: 11.5px; color: #9A9890; line-height: 1.6;">이 리포트는 생산 대시보드 데이터만 씁니다. 계획은 사업계획 기준이며 운영계획 기준과 266,381대 차이가 있습니다. 어느 쪽을 기준으로 삼을지는 미정입니다.</div>
      </div>

    </div>
  </main>
</div>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 좌측 사이드 | 248px 고정 폭 보조면 | 출처, 지표, 차원, 버전 | 없음 |
| 2 | 출처 블록 | 블록 | 대시보드 이름, 측정값 구성, 스냅샷 일시 | 없음 |
| 3 | 트래킹 지표 | 목록 3 | 사업계획 달성률, 완성차 비중, 수출 비중. 현재 값과 임시 지정 표시 | 선택한 지표가 본문의 기준이 됨 |
| 4 | 분해 차원 칩 | 칩 다수 | 차원명과 값 개수 | 없음 |
| 5 | 제약 경고 | 주의색 박스 2 | 목적지 국가 없음, 파워트레인 제외. 못 만드는 지표 목록을 문구로 보여 주는 표시용 | 없음 |
| 6 | 버전 목록 | 목록 | 게시 사본이 가진 현재 버전과 직전 버전 | 누르면 그 버전 지면 |
| 7 | 지면 헤더 | 44px 줄 | 리포트 배지, 제목, 버전 알약, 범위 문구 | 없음 |
| 8 | 내려받기 | 버튼 | 없음 | 보고 있는 버전 지면을 PDF로. 서버가 요청마다 만든다 |
| 9 | 요약 | 16px 문단 | 브리핑 갈래가 쓴 한두 문장, 강조 토큰, 각주 | 각주는 해설 강조 |
| 10 | 지표 카드 | 카드 3 | 누적 사업계획, 누적 실적, 달성률 | 없음 |
| 11 | 법인별 달성률 | 막대 표 | 법인명, 달성률 막대, 실적/계획 원값 | 없음 |
| 12 | 생산 구성 | 누적 막대 2 | 완성차와 반조립, 내수와 수출 | 없음 |
| 13 | 해설 | 문단 | 브리핑 갈래가 쓴 13.5px 서술, 각주 | 각주 강조 |
| 14 | 고정 문구 | 회색 줄 | 데이터 범위와 계획 정본 미정 안내 | 없음 |

##### 규칙

- 요약, 해설, 각주 근거, 버전 번호는 브리핑 갈래가 게시한 A 리포트 사본에서 그대로 읽는다. 고정 문구와 분해 블록(표·막대)은 이 시스템 배치가 만들어 같은 사본에 둔 것이다. 이 화면이 문장을 새로 만들지 않는다(2.2절).
- 달성률은 계획과 실적으로 직접 계산한다. 원본 진도율 컬럼은 개별 행이 0이라 쓰지 않는다. 이 사실을 사이드에 적는다.
- 목적지 국가가 없다는 경고를 사이드 분해 차원 아래에 둔다. 지우지 않는다. 이 제약이 C 리포트에서 생산이 국가 카드에 오르지 못하고 도메인 상태 카드에서 "판정 대상 아님"으로 나오는 이유다([[TBL-INFRA-002#C16]]).
- 파워트레인은 44~53%가 미분류라 분해 차원 칩에 넣지 않는다.
- 못 만드는 지표의 정본은 게시 사본의 목록이고 제약 경고 박스는 그 목록을 문구로 보여 주는 표시다. 목록을 새로 만들지 않는다.
- 계획 정본이 정해지기 전에는 사업계획을 기본으로 쓰고, 운영계획 기준 값과의 차이 266,381대를 고정 문구에 적는다([[TBL-DOM-004#ThresholdSetting]]).
- 지표 카드의 막대 색은 상태에 따른다. 법인 달성률이 80% 아래면 주의색, 그 위면 차트색을 쓴다. 80%는 판정 임계가 아니라 화면 표시 기준이며 설정 v1과 함께 정한 기본값이다(현업 검토 요청). 색만으로 구분하지 않고 값을 함께 적는다.
- 내려받기는 보고 있는 버전 지면을 PDF로 받는다. 서버가 요청마다 만들고 저장하지 않으며 화면의 색 규칙은 옮기지 않고 값을 적는다. 사본이 없으면 버튼을 막는다. [[#UI-3]] [[#UI-4]]도 같다([[TBL-API-002#GET/api/intel/areport/pdf/{domainReportId}]]).
- 뉴스, 시장지표, 다른 도메인 값을 인용하지 않는다. 인용 0건이 인수 기준이다([[TBL-PRD-002#R1]]).
- 분해할 차원이 없으면 변동 판정만 보여 주고 "분해 불가"를 적는다([[TBL-UC-002#UC-H1]] 2a).
- 트래킹 지표 셋은 임시 후보다. 현업 인터뷰 전까지 임시 지정임을 사이드에 표시한다([[TBL-UC-002#UC-H1]] 1a).
- 게시 사본이 없으면 판정값과 차트만 보여 주고 "리포트 문장 미수신"을 표시한다([[TBL-UC-002#UC-H1]] 4a).

##### 시나리오

**S-1 생산 리포트를 본다** [[TBL-UC-002#UC-H1]]
1. H가 생산 대시보드를 연다. 지면이 함께 뜬다.
2. 브리핑 갈래가 쓴 요약이 사본에서 실려 누적 달성 86.1%를 말하고 운영계획 기준 99.2%와의 차이를 밝힌다.
3. H가 법인별 달성률 표에서 HMGMA 45.9%를 본다. 대수와 비율이 함께 적혀 있다.
4. 해설이 공장과 차종 사실만 말한다. 왜 그런지는 여기서 답하지 않는다.

**S-2 변동이 없는 날** [[TBL-UC-002#UC-H1]] 1b
1. 요약 자리에 "감지된 변동 없음"과 지표 현황만 나온다. 리포트는 그대로 나간다.

**연관**: [[TBL-PRD-002#R1]] · [[TBL-PRD-002#R16]] · [[TBL-PRD-002#G2]] · [[TBL-UC-002#UC-H1]] · [[TBL-DOM-004#DomainReport]] · [[TBL-DOM-004#DomainJudgment]] · [[TBL-INFRA-002#C16]]

#### UI-3 A2 재고 리포트

| 항목 | 내용 |
|---|---|
| 경로 | `/intel/areport/inventory` |
| 주 유스케이스 | [[TBL-UC-002#UC-H1]] |
| 진입 / 이탈 | VODA 재고 대시보드의 A2 서명 주소 / 내려받기(PDF) |

페이지. 재고 대시보드에 붙는 AI 인사이트. 재고 원천이 없다는 배너로 시작한다. 지금 보이는 값은 판매에서 유도했거나 하루치 스냅샷이다.

##### 배치

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;600;700&display=swap">
<style>.var{margin:32px 0 8px;font-size:11.5px;color:#A6A49C;border-left:3px solid #DAD6CE;padding-left:10px;font-family:system-ui,sans-serif}</style>

<style>
body { margin: 0; background: #F4F2EE; color: #1A1B19;
      font-family: 'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif;
      -webkit-font-smoothing: antialiased; }
    a { color: #0E6B57; text-decoration: none; }
    a:hover { color: #0B5847; text-decoration: underline; }
    .num { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-variant-numeric: tabular-nums; }
    .lbl { font-size: 10.5px; font-weight: 500; text-transform: uppercase; letter-spacing: .09em; color: #9A9890; }
    .hl { font-weight: 600; background: rgba(181,115,12,.14); border-radius: 3px; padding: 0 2px; }
    .fn { font-size: 10px; vertical-align: super; color: #0E6B57; font-weight: 600; }
    .body { font-size: 12.5px; color: #3C3B37; }
</style>
<div style="width: 1180px; padding: 24px; box-sizing: border-box; display: flex; align-items: flex-start; gap: 20px;">

  <aside data-el="1" style="width: 248px; flex: none; display: flex; flex-direction: column; gap: 20px; border: 1px solid #E2DFD8; background: #FBFAF8; border-radius: 11px; padding: 16px;">
    <div style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">출처</div>
      <div data-el="2" style="display: flex; align-items: flex-start; gap: 6px; background: rgba(179,38,30,.06); border: 1px solid #F0CFCB; border-radius: 7px; padding: 10px;">
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
        <div style="font-size: 11px; color: #B3261E; line-height: 1.55;">재고 대시보드 원천이 아직 없습니다. 인입 데이터에 재고 파일이 들어 있지 않습니다.</div>
      </div>
      <div style="font-size: 11.5px; color: #6B6A65; line-height: 1.55;">아래 지표는 판매 데이터에서 유도했거나 단발 스냅샷입니다.</div>
    </div>

    <div data-el="3" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">지금 만들 수 있는 것</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        <div style="display: flex; align-items: center; gap: 7px; background: #FFFFFF; border-radius: 8px; padding: 7px 10px; box-shadow: inset 0 0 0 1px #E2DFD8;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #B5730C;"></span>
          <span class="body" style="font-weight: 500;">유통 체류</span>
          <span class="num" style="font-size: 11.5px; color: #B5730C; margin-left: auto;">-34,104</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px; padding: 7px 10px;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
          <span class="body">재고 회전 MOS</span>
          <span class="num" style="font-size: 11.5px; color: #9A9890; margin-left: auto;">불안정</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px; padding: 7px 10px;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
          <span class="body">현지재고 스냅샷</span>
          <span class="num" style="font-size: 11.5px; color: #9A9890; margin-left: auto;">1일</span>
        </div>
      </div>
    </div>

    <div data-el="4" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">없는 것</div>
      <div style="display: flex; flex-direction: column; gap: 5px;">
        <div style="display: flex; align-items: center; gap: 7px;">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="2.4" stroke-linecap="round"><path d="M18 6 6 18"></path><path d="m6 6 12 12"></path></svg>
          <span class="body">항해중 재고</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px;">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="2.4" stroke-linecap="round"><path d="M18 6 6 18"></path><path d="m6 6 12 12"></path></svg>
          <span class="body">선적대기 재고</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px;">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="2.4" stroke-linecap="round"><path d="M18 6 6 18"></path><path d="m6 6 12 12"></path></svg>
          <span class="body">법인재고와 딜러재고 구분</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px;">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="2.4" stroke-linecap="round"><path d="M18 6 6 18"></path><path d="m6 6 12 12"></path></svg>
          <span class="body">국가별 재고 시계열</span>
        </div>
      </div>
    </div>

    <div data-el="5" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">버전</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        <div class="num" style="background: #FFFFFF; border-radius: 8px; padding: 6px 10px; font-size: 12px; font-weight: 500; box-shadow: inset 0 0 0 1px #E2DFD8;">v3 · 현재</div>
        <div class="num" style="padding: 6px 10px; font-size: 12px; color: #6B6A65;">v2 · 08-19</div>
      </div>
    </div>
  </aside>

  <main style="flex: 1; min-width: 0; border: 1px solid #E2DFD8; background: #FFFFFF; border-radius: 11px; overflow: hidden;">

    <div data-el="6" style="display: flex; align-items: center; gap: 10px; height: 44px; padding: 0 16px; border-bottom: 1px solid #E2DFD8;">
      <span style="background: #E7F0ED; color: #0B5847; font-size: 11px; font-weight: 500; padding: 2px 8px; border-radius: 5px;">A2 재고 리포트</span>
      <span style="font-size: 13px; font-weight: 600;">재고 현황 일일 인사이트</span>
      <span class="num" style="background: #EFEDE7; color: #6B6A65; font-size: 11px; padding: 3px 8px; border-radius: 20px;">v3</span>
      <span style="font-size: 11px; color: #B5730C; margin-left: 8px;">유도 지표로만 구성</span>
      <div style="margin-left: auto; display: flex; align-items: center; gap: 6px;">
        <button data-el="7" style="display: inline-flex; align-items: center; gap: 6px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #3C3B37; font-size: 12px; padding: 6px 12px; border-radius: 7px; font-family: inherit; cursor: pointer;">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 17V3"></path><path d="m6 11 6 6 6-6"></path><path d="M19 21H5"></path></svg>
          내려받기
        </button>
      </div>
    </div>

    <div style="padding: 22px; display: flex; flex-direction: column; gap: 22px;">

      <div data-el="8" style="display: flex; align-items: flex-start; gap: 10px; background: rgba(179,38,30,.06); border: 1px solid #F0CFCB; border-radius: 7px; padding: 14px 16px;">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#B3261E" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" style="flex: none; margin-top: 2px;"><path d="M12 9v4"></path><path d="M12 17h.01"></path><path d="M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0Z"></path></svg>
        <div style="display: flex; flex-direction: column; gap: 4px;">
          <div style="font-size: 13.5px; font-weight: 600; color: #B3261E;">이 리포트는 아직 자기 데이터가 없습니다</div>
          <div style="font-size: 12.5px; color: #3C3B37; line-height: 1.6;">인입 파일 여섯 개를 전부 확인했지만 재고를 항해중·선적대기·법인·딜러로 나눈 데이터가 없습니다. 아래는 판매 데이터에서 유도했거나 하루치 스냅샷인 값입니다. 판정 근거로 쓰기 전에 원천을 받아야 합니다.</div>
        </div>
      </div>

      <div data-el="9" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">유도 지표 · 유통 체류</div>
        <div style="font-size: 11px; color: #9A9890; margin-top: -4px;">화면은 줄어드는 방향을 앞의 빼기 부호로 그립니다. 저장되는 값은 양수입니다.</div>
        <div style="font-size: 16px; font-weight: 500; line-height: 1.65; letter-spacing: -.01em; text-wrap: pretty;">
          딜러에게 넘긴 물량과 팔린 물량 사이에 <span class="hl">34,104대</span>가 남아 있습니다. 재고라고 부를 수는 없지만 어디에 쌓이는지는 이 값이 가리킵니다.<span class="fn">1</span>
        </div>
        <div style="display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px;">
          <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 7px;">
            <div class="body" style="font-weight: 600;">도매 정본 누계</div>
            <div class="num" style="font-size: 21px; font-weight: 600;">677,201</div>
            <div style="font-size: 11px; color: #9A9890;">미주 29개국</div>
          </div>
          <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 7px;">
            <div class="body" style="font-weight: 600;">소매 누계</div>
            <div class="num" style="font-size: 21px; font-weight: 600;">643,097</div>
            <div style="font-size: 11px; color: #9A9890;">미주 29개국</div>
          </div>
          <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 14px; display: flex; flex-direction: column; gap: 7px;">
            <div class="body" style="font-weight: 600;">딜러 구간 차이</div>
            <div style="display: flex; align-items: baseline; gap: 8px;">
              <div class="num" style="font-size: 21px; font-weight: 600; color: #B5730C;">34,104</div>
              <div class="num" style="font-size: 12px; color: #B5730C;">5.0%</div>
            </div>
            <div style="font-size: 11px; color: #9A9890;">칠레 25.2%가 최대</div>
          </div>
        </div>
      </div>

      <div data-el="10" style="display: flex; flex-direction: column; gap: 12px;">
        <div style="display: flex; align-items: baseline; gap: 10px;">
          <div class="lbl">국가별 체류 비중</div>
          <div style="font-size: 11.5px; color: #6B6A65;">도매 대비 미판매 비율 · 도매 1,000대 이상</div>
        </div>
        <div style="border: 1px solid #E2DFD8; border-radius: 7px; padding: 12px 14px; display: flex; flex-direction: column; gap: 9px;">
          <div style="display: flex; align-items: center; gap: 10px;">
            <div class="body" style="width: 120px; flex: none;">칠레 <span class="num" style="color:#9A9890;">B07</span></div>
            <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
              <div style="width: 100%; height: 100%; background: #C9922E;"></div>
            </div>
            <div class="num" style="width: 120px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">25.2% <span style="color:#9A9890; font-weight:400;">3,064대</span></div>
          </div>
          <div style="display: flex; align-items: center; gap: 10px;">
            <div class="body" style="width: 120px; flex: none;">페루 <span class="num" style="color:#9A9890;">B24</span></div>
            <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
              <div style="width: 84%; height: 100%; background: #C9922E;"></div>
            </div>
            <div class="num" style="width: 120px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">21.2% <span style="color:#9A9890; font-weight:400;">2,473대</span></div>
          </div>
          <div style="display: flex; align-items: center; gap: 10px;">
            <div class="body" style="width: 120px; flex: none;">파나마 <span class="num" style="color:#9A9890;">B22</span></div>
            <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
              <div style="width: 43%; height: 100%; background: #8FB4D2;"></div>
            </div>
            <div class="num" style="width: 120px; flex: none; text-align: right; font-size: 12.5px;">10.8% <span style="color:#9A9890;">391대</span></div>
          </div>
          <div style="display: flex; align-items: center; gap: 10px;">
            <div class="body" style="width: 120px; flex: none;">캐나다 <span class="num" style="color:#9A9890;">B06</span></div>
            <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
              <div style="width: 33%; height: 100%; background: #8FB4D2;"></div>
            </div>
            <div class="num" style="width: 120px; flex: none; text-align: right; font-size: 12.5px;">8.3% <span style="color:#9A9890;">5,986대</span></div>
          </div>
          <div style="display: flex; align-items: center; gap: 10px;">
            <div class="body" style="width: 120px; flex: none;">미국 <span class="num" style="color:#9A9890;">B28</span></div>
            <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
              <div style="width: 26%; height: 100%; background: #8FB4D2;"></div>
            </div>
            <div class="num" style="width: 120px; flex: none; text-align: right; font-size: 12.5px;">6.6% <span style="color:#9A9890;">29,548대</span></div>
          </div>
        </div>
      </div>

      <div data-el="11" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">원천을 받으면 여기에 들어갈 것</div>
        <div style="border: 1px dashed #DAD6CE; border-radius: 7px; background: #FBFAF8; padding: 16px; display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 14px;">
          <div style="display: flex; flex-direction: column; gap: 5px;">
            <div class="body" style="font-weight: 600; color: #6B6A65;">항해중 비중</div>
            <div style="font-size: 11.5px; color: #9A9890; line-height: 1.55;">배에 실렸으나 내리지 못한 물량. 해상 구간 지연을 가장 먼저 보여 주는 값입니다.</div>
          </div>
          <div style="display: flex; flex-direction: column; gap: 5px;">
            <div class="body" style="font-weight: 600; color: #6B6A65;">선적대기 비중</div>
            <div style="font-size: 11.5px; color: #9A9890; line-height: 1.55;">항만에 쌓였으나 아직 싣지 못한 물량. 선복 부족을 보여 줍니다.</div>
          </div>
          <div style="display: flex; flex-direction: column; gap: 5px;">
            <div class="body" style="font-weight: 600; color: #6B6A65;">법인재고와 딜러재고</div>
            <div style="font-size: 11.5px; color: #9A9890; line-height: 1.55;">어느 단계에 쌓였는지 가릅니다. 지금은 유통 체류 한 덩어리로만 보입니다.</div>
          </div>
          <div style="display: flex; flex-direction: column; gap: 5px;">
            <div class="body" style="font-weight: 600; color: #6B6A65;">국가별 일별 시계열</div>
            <div style="font-size: 11.5px; color: #9A9890; line-height: 1.55;">언제부터 쌓이기 시작했는지 봅니다. 뉴스 시점과 맞대려면 이게 있어야 합니다.</div>
          </div>
        </div>
      </div>

      <div data-el="12" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">해설</div>
        <div style="font-size: 13.5px; line-height: 1.75; color: #1A1B19; text-wrap: pretty;">
          칠레와 페루는 선적과 도매가 정확히 같습니다. 보낸 물량이 그대로 딜러까지 갔고 거기서 멈췄다는 뜻입니다.<span class="fn">2</span> 반면 캐나다는 선적보다 도매가 4,358대 적어 일부가 아직 법인 단계에 있고, 딜러 단계에서도 5,986대가 남아 있습니다. 같은 체류라도 걸린 자리가 다릅니다. 다만 이 구분은 선적과 도매의 차이로 미루어 짐작한 것이고, 재고 원천이 들어오면 직접 확인해야 합니다.
        </div>
      </div>

      <div data-el="13" style="display: flex; align-items: flex-start; gap: 8px; border-top: 1px solid #EFEDE7; padding-top: 14px;">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
        <div style="font-size: 11.5px; color: #9A9890; line-height: 1.6;">이 리포트의 값은 판매 대시보드에서 유도한 것입니다. 뉴스와 시장지표는 다루지 않습니다. 재고 원천이 들어오면 유도 지표를 실측값으로 교체합니다.</div>
      </div>

    </div>
  </main>
</div>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 좌측 사이드 | 248px 보조면 | 출처, 가능한 것, 없는 것, 버전 | 없음 |
| 2 | 원천 부재 알림 | 경고색 박스 | 재고 대시보드 원천이 없다는 사실 | 없음 |
| 3 | 지금 만들 수 있는 것 | 목록 3 | 유통 체류(코드 `distributionStay`, 딜러 구간), 재고 회전 MOS(`inventoryTurnMos`), 현지재고 스냅샷(`localInventorySnapshot`)과 각각의 한계 | 없음 |
| 4 | 없는 것 | X표 목록 4 | 항해중 재고(`afloat`), 선적대기 재고(`awaitingShipment`), 법인재고와 딜러재고 구분(`entityVsDealer`), 국가별 재고 시계열(`countryDailySeries`) | 없음 |
| 5 | 버전 목록 | 목록 | 게시 사본이 가진 현재 버전과 직전 버전 | 그 버전 지면 |
| 6 | 지면 헤더 | 44px 줄 | 배지, 제목, 버전 알약, "유도 지표로만 구성" | 없음 |
| 7 | 내려받기 | 버튼 | 없음 | 보고 있는 버전 지면을 PDF로 |
| 8 | 원천 부재 배너 | 경고색 배너 | 왜 자기 데이터가 없는지, 무엇을 대신 쓰는지 | 없음 |
| 9 | 유통 체류 요약 | 문단과 카드 3 | 도매 정본 누계, 소매 누계, 딜러 구간 차이와 비율. 부호 규칙 캡션 | 없음 |
| 10 | 국가별 체류 비중 | 막대 표 | 국가, 대리점 앞 3자리 코드, 비율, 대수 | 없음 |
| 11 | 원천을 받으면 들어갈 것 | 점선 블록 4 | 채워질 자리와 각각이 무엇을 보여 줄지 | 없음 |
| 12 | 해설 | 문단 | 단계 차이로 미루어 짐작한 것임을 밝히는 서술 | 각주 강조 |
| 13 | 고정 문구 | 회색 줄 | 유도값 안내와 교체 계획 | 없음 |

##### 규칙

- 요약, 해설, 각주 근거, 버전 번호는 브리핑 갈래가 게시한 A 리포트 사본에서 그대로 읽는다. 고정 문구와 분해 블록(표·막대)은 이 시스템 배치가 만들어 같은 사본에 둔 것이다. 이 화면이 문장을 새로 만들지 않는다(2.2절).
- 화면은 원천 부재 배너로 시작한다. 지표보다 먼저 나온다. 없는 것을 있는 것처럼 보이게 하지 않는다.
- 이 화면의 "유통 체류"는 딜러 구간이다. 도매 정본에서 소매를 뺀 값이고 지표 코드는 `distributionStay`다. 미주 누계 실측 34,104대다(CDO 인입 샘플 CSV 추출본 기준, [[TBL-DOM-004#SalesStageFlow]]).
- 사이드에는 `-34,104`로 그린다. 감소 방향을 보이려고 앞에 빼기 부호를 붙이는 것이고 API 값과 저장 값은 양수 34,104다. [[#UI-4]] 단계별 흐름과 같은 규칙이며 두 화면이 같은 값을 다른 부호로 적지 않는다. 지표 카드에는 양수로 적고 캡션에 부호 규칙을 밝힌다.
- 선적에서 도매 정본을 뺀 법인 구간은 다른 지표다. 실측 2,350대이고 지표 코드는 `entityStageStay`, 화면 표기는 "법인 단계 체류"다. 이 화면에 함께 두려면 그 이름으로 따로 적고 유통 체류와 한 칸에 합치지 않는다. 지금은 [[#UI-4]] 단계별 흐름에서 본다.
- 못 만드는 지표를 사이드에 목록으로 보여 준다. 항해중 재고(`afloat`), 선적대기 재고(`awaitingShipment`), 법인재고와 딜러재고 구분(`entityVsDealer`), 국가별 재고 시계열(`countryDailySeries`) 넷이다([[TBL-PRD-002#R1]] 넷째 인수 기준).
- 모든 값에 유도임을 표시한다. 저장 행에도 재고 출처 구분이 함께 간다([[TBL-DOM-004#CountryPeriodFact]], [[TBL-INFRA-002#C17]]).
- MOS는 음수와 0이 섞여 있어 값 대신 "불안정"으로 적는다. 숫자를 그대로 보여 주지 않는다.
- "원천을 받으면 여기에 들어갈 것" 블록은 점선으로 두고 값을 넣지 않는다. 자리만 표시한다.
- 재고 원천이 들어오면 같은 자리를 실측값으로 바꾸고 유도 표기를 내린다([[TBL-UC-002#UC-H1]] 3a, [[TBL-PRD-002#R9]]).
- 뉴스와 시장지표를 인용하지 않는다.

##### 시나리오

**S-1 재고 리포트를 연다** [[TBL-UC-002#UC-H1]]
1. H가 재고 대시보드를 연다.
2. 배너가 먼저 읽힌다. 재고 원천이 없고 아래 값은 판매에서 유도한 것이라는 문장이다.
3. 딜러 구간인 유통 체류 34,104대와 국가별 비중이 나온다. 칠레 25.2%가 가장 크다. 선적과 도매 정본 사이의 법인 단계 체류는 이 화면에 없다.
4. 사이드에서 못 만드는 지표 넷을 확인한다.

**S-2 재고 원천이 들어온 뒤** [[TBL-UC-002#UC-H1]] 3a
1. 배너가 사라진다.
2. 점선 블록 자리에 항해중 비중과 선적대기 비중이 실측값으로 들어간다.
3. 유통 체류는 참고 지표로 내려가고 유도 표기가 사라진다.

**연관**: [[TBL-PRD-002#R1]] · [[TBL-PRD-002#R9]] · [[TBL-PRD-002#R14]] · [[TBL-UC-002#UC-H1]] · [[TBL-DOM-004#CountryPeriodFact]] · [[TBL-DOM-004#SalesStageFlow]] · [[TBL-INFRA-002#C17]]

#### UI-4 A3 판매 리포트

| 항목 | 내용 |
|---|---|
| 경로 | `/intel/areport/sales` |
| 주 유스케이스 | [[TBL-UC-002#UC-H1]] |
| 진입 / 이탈 | VODA 판매 대시보드의 A3 서명 주소 / 내려받기(PDF) |

페이지. 판매 대시보드에 붙는 AI 인사이트. 선적, 도매, 소매의 단계별 흐름과 어디서 막히는지를 보여 준다. 세 리포트 중 재료가 가장 좋다.

##### 배치

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;600;700&display=swap">
<style>.var{margin:32px 0 8px;font-size:11.5px;color:#A6A49C;border-left:3px solid #DAD6CE;padding-left:10px;font-family:system-ui,sans-serif}</style>

<style>
body { margin: 0; background: #F4F2EE; color: #1A1B19;
      font-family: 'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif;
      -webkit-font-smoothing: antialiased; }
    a { color: #0E6B57; text-decoration: none; }
    a:hover { color: #0B5847; text-decoration: underline; }
    .num { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-variant-numeric: tabular-nums; }
    .lbl { font-size: 10.5px; font-weight: 500; text-transform: uppercase; letter-spacing: .09em; color: #9A9890; }
    .hl { font-weight: 600; background: rgba(181,115,12,.14); border-radius: 3px; padding: 0 2px; }
    .fn { font-size: 10px; vertical-align: super; color: #0E6B57; font-weight: 600; }
    .body { font-size: 12.5px; color: #3C3B37; }
</style>
<div style="width: 1180px; padding: 24px; box-sizing: border-box; display: flex; align-items: flex-start; gap: 20px;">

  <aside data-el="1" style="width: 248px; flex: none; display: flex; flex-direction: column; gap: 20px; border: 1px solid #E2DFD8; background: #FBFAF8; border-radius: 11px; padding: 16px;">
    <div data-el="2" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">출처</div>
      <div style="display: flex; flex-direction: column; gap: 6px;">
        <div style="display: flex; align-items: center; gap: 6px;">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#6B6A65" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 3v18h18"></path><rect x="7" y="10" width="3" height="7"></rect><rect x="13" y="6" width="3" height="11"></rect></svg>
          <span class="body">완성차 판매 대시보드</span>
        </div>
        <div style="font-size: 11.5px; color: #6B6A65;">선적·도매·소매 4계열 × 일·월·누계·년</div>
        <div style="font-size: 11.5px; color: #6B6A65;">스냅샷 <span class="num">2026-08-20 01:40</span></div>
      </div>
    </div>

    <div data-el="3" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">트래킹 지표</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        <div style="display: flex; align-items: center; gap: 7px; background: #FFFFFF; border-radius: 8px; padding: 7px 10px; box-shadow: inset 0 0 0 1px #E2DFD8;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #B3261E;"></span>
          <span class="body" style="font-weight: 500;">유통 체류율</span>
          <span class="num" style="font-size: 11.5px; color: #B3261E; margin-left: auto;">-5.0%</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px; padding: 7px 10px;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
          <span class="body">계획 대비 진도율</span>
          <span class="num" style="font-size: 11.5px; color: #6B6A65; margin-left: auto;">96.4%</span>
        </div>
        <div style="display: flex; align-items: center; gap: 7px; padding: 7px 10px;">
          <span style="width: 6px; height: 6px; border-radius: 50%; background: #9A9890;"></span>
          <span class="body">전년 동월 대비</span>
          <span class="num" style="font-size: 11.5px; color: #6B6A65; margin-left: auto;">-1.8%</span>
        </div>
      </div>
      <div style="font-size: 11px; color: #A6A49C; line-height: 1.5;">인입 CSV의 실제 컬럼에서 뽑았습니다. 현업 인터뷰로 확정합니다.</div>
    </div>

    <div style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">분해 차원</div>
      <div data-el="4" style="display: flex; flex-wrap: wrap; gap: 5px;">
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">국가 29</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">차종그룹 37</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">대리점</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">차급 16</span>
        <span style="font-size: 11px; color: #6B6A65; background: #EFEDE7; padding: 3px 8px; border-radius: 7px;">CBU/CKD</span>
      </div>
      <div data-el="5" style="display: flex; align-items: flex-start; gap: 6px; background: rgba(181,115,12,.08); border: 1px solid rgba(181,115,12,.28); border-radius: 7px; padding: 8px 10px;">
        <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#B5730C" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
        <div style="font-size: 11px; color: #B5730C; line-height: 1.5;">국가와 차종이 함께 있는 파일은 미주 한 장뿐입니다. 전 세계 파일은 실적이 비어 있습니다.</div>
      </div>
    </div>

    <div data-el="6" style="display: flex; flex-direction: column; gap: 10px;">
      <div class="lbl">버전</div>
      <div style="display: flex; flex-direction: column; gap: 4px;">
        <div class="num" style="background: #FFFFFF; border-radius: 8px; padding: 6px 10px; font-size: 12px; font-weight: 500; box-shadow: inset 0 0 0 1px #E2DFD8;">v12 · 현재</div>
        <div class="num" style="padding: 6px 10px; font-size: 12px; color: #6B6A65;">v11 · 08-19</div>
      </div>
    </div>
  </aside>

  <main style="flex: 1; min-width: 0; border: 1px solid #E2DFD8; background: #FFFFFF; border-radius: 11px; overflow: hidden;">

    <div data-el="7" style="display: flex; align-items: center; gap: 10px; height: 44px; padding: 0 16px; border-bottom: 1px solid #E2DFD8;">
      <span style="background: #E7F0ED; color: #0B5847; font-size: 11px; font-weight: 500; padding: 2px 8px; border-radius: 5px;">A3 판매 리포트</span>
      <span style="font-size: 13px; font-weight: 600;">판매 실적 일일 인사이트</span>
      <span class="num" style="background: #EFEDE7; color: #6B6A65; font-size: 11px; padding: 3px 8px; border-radius: 20px;">v12</span>
      <span style="font-size: 11px; color: #6B6A65; margin-left: 8px;">미주 29개국 · 누계 기준</span>
      <div style="margin-left: auto; display: flex; align-items: center; gap: 6px;">
        <button data-el="8" style="display: inline-flex; align-items: center; gap: 6px; border: 1px solid #DAD6CE; background: #FFFFFF; color: #3C3B37; font-size: 12px; padding: 6px 12px; border-radius: 7px; font-family: inherit; cursor: pointer;">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 17V3"></path><path d="m6 11 6 6 6-6"></path><path d="M19 21H5"></path></svg>
          내려받기
        </button>
      </div>
    </div>

    <div style="padding: 22px; display: flex; flex-direction: column; gap: 22px;">

      <div data-el="9" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">요약</div>
        <div style="font-size: 16px; font-weight: 500; line-height: 1.65; letter-spacing: -.01em; text-wrap: pretty;">
          도매로 넘긴 물량과 실제 팔린 물량 사이에 <span class="hl">34,104대</span>가 남아 있습니다. 미주 전체로는 5.0%인데 <span class="hl">칠레 25.2%, 페루 21.2%</span>로 두 나라가 크게 벌어져 있습니다.<span class="fn">1</span> 반대로 브라질과 멕시코는 소매가 도매를 넘어 쌓였던 물량을 덜어내고 있습니다.<span class="fn">2</span>
        </div>
      </div>

      <div data-el="10" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">단계별 흐름 · 누계 실적</div>
        <div style="font-size: 11px; color: #9A9890; margin-top: -4px;">화면은 줄어드는 방향을 앞의 빼기 부호로 그립니다. 저장되는 값은 양수입니다.</div>
        <div style="display: flex; align-items: stretch; gap: 0;">
          <div style="flex: 1; border: 1px solid #E2DFD8; border-radius: 7px 0 0 7px; padding: 14px; display: flex; flex-direction: column; gap: 6px;">
            <div style="font-size: 12.5px; font-weight: 600;">선적</div>
            <div class="num" style="font-size: 21px; font-weight: 600;">679,551</div>
            <div style="font-size: 11px; color: #9A9890;">배에 실린 물량</div>
          </div>
          <div style="width: 54px; flex: none; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 3px; border-top: 1px solid #E2DFD8; border-bottom: 1px solid #E2DFD8; background: #FBFAF8;">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m12 5 7 7-7 7"></path></svg>
            <div class="num" style="font-size: 10.5px; color: #6B6A65;">-2,350</div>
          </div>
          <div style="flex: 1; border: 1px solid #E2DFD8; padding: 14px; display: flex; flex-direction: column; gap: 6px;">
            <div style="font-size: 12.5px; font-weight: 600;">도매</div>
            <div class="num" style="font-size: 21px; font-weight: 600;">677,201</div>
            <div style="font-size: 11px; color: #9A9890;">딜러에게 넘긴 물량</div>
          </div>
          <div style="width: 54px; flex: none; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 3px; border-top: 1px solid #E2DFD8; border-bottom: 1px solid #E2DFD8; background: rgba(181,115,12,.08);">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#B5730C" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"></path><path d="m12 5 7 7-7 7"></path></svg>
            <div class="num" style="font-size: 10.5px; color: #B5730C; font-weight: 600;">-34,104</div>
          </div>
          <div style="flex: 1; border: 1px solid #E2DFD8; border-radius: 0 7px 7px 0; padding: 14px; display: flex; flex-direction: column; gap: 6px;">
            <div style="font-size: 12.5px; font-weight: 600;">소매</div>
            <div class="num" style="font-size: 21px; font-weight: 600;">643,097</div>
            <div style="font-size: 11px; color: #9A9890;">최종 고객에게 팔린 물량</div>
          </div>
        </div>
        <div style="display: flex; align-items: flex-start; gap: 7px;">
          <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 2px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
          <div style="font-size: 11.5px; color: #9A9890; line-height: 1.6;">선적과 도매는 거의 붙어 있고 벌어지는 구간은 도매와 소매 사이입니다. 이 차이가 딜러에 쌓인 물량입니다. 별도 재고 데이터가 없어 이 값으로 대신합니다.</div>
        </div>
      </div>

      <div style="display: flex; flex-direction: column; gap: 12px;">
        <div style="display: flex; align-items: baseline; gap: 10px;">
          <div class="lbl">변동을 대시보드 안에서 분해</div>
          <div style="font-size: 11.5px; color: #6B6A65;">도매 대비 소매 격차율 · 도매 1,000대 이상 14개국</div>
        </div>

        <div data-el="11" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
          <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
            <div style="font-size: 12.5px; font-weight: 600;">쌓이는 쪽</div>
            <div style="font-size: 11px; color: #9A9890;">도매가 소매보다 많음</div>
          </div>
          <div style="padding: 12px 14px; display: flex; flex-direction: column; gap: 9px;">
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">칠레 <span class="num" style="color:#9A9890;">B07</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 100%; height: 100%; background: #2F6FA8;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">25.2% <span style="color:#9A9890; font-weight:400;">3,064대</span></div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">페루 <span class="num" style="color:#9A9890;">B24</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 84%; height: 100%; background: #2F6FA8;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">21.2% <span style="color:#9A9890; font-weight:400;">2,473대</span></div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">파나마 <span class="num" style="color:#9A9890;">B22</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 43%; height: 100%; background: #5C93BF;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px;">10.8% <span style="color:#9A9890;">391대</span></div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">캐나다 <span class="num" style="color:#9A9890;">B06</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 33%; height: 100%; background: #5C93BF;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px;">8.3% <span style="color:#9A9890;">5,986대</span></div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">미국 <span class="num" style="color:#9A9890;">B28</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 26%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px;">6.6% <span style="color:#9A9890;">29,548대</span></div>
            </div>
          </div>
          <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-top: 1px solid #EFEDE7; border-bottom: 1px solid #EFEDE7;">
            <div style="font-size: 12.5px; font-weight: 600;">덜어내는 쪽</div>
            <div style="font-size: 11px; color: #9A9890;">소매가 도매보다 많음</div>
          </div>
          <div style="padding: 12px 14px; display: flex; flex-direction: column; gap: 9px;">
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">푸에르토리코 <span class="num" style="color:#9A9890;">B35</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden; display: flex; justify-content: flex-end;">
                <div style="width: 54%; height: 100%; background: #4B8B7A;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px;">-13.7% <span style="color:#9A9890;">712대</span></div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">콜롬비아 <span class="num" style="color:#9A9890;">B08</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden; display: flex; justify-content: flex-end;">
                <div style="width: 42%; height: 100%; background: #4B8B7A;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px;">-10.5% <span style="color:#9A9890;">535대</span></div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 120px; flex: none;">브라질 <span class="num" style="color:#9A9890;">B05</span></div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden; display: flex; justify-content: flex-end;">
                <div style="width: 12%; height: 100%; background: #4B8B7A;"></div>
              </div>
              <div class="num" style="width: 130px; flex: none; text-align: right; font-size: 12.5px;">-3.1% <span style="color:#9A9890;">2,632대</span></div>
            </div>
          </div>
        </div>

        <div data-el="12" style="border: 1px solid #E2DFD8; border-radius: 7px; overflow: hidden;">
          <div style="display: flex; align-items: center; gap: 8px; background: #FBFAF8; padding: 10px 14px; border-bottom: 1px solid #EFEDE7;">
            <div style="font-size: 12.5px; font-weight: 600;">캐나다 차종별</div>
            <div style="font-size: 11px; color: #9A9890;">5,986대 중 상위 5</div>
          </div>
          <div style="padding: 12px 14px; display: flex; flex-direction: column; gap: 9px;">
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 96px; flex: none;">아반떼</div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 100%; height: 100%; background: #2F6FA8;"></div>
              </div>
              <div class="num" style="width: 140px; flex: none; text-align: right; font-size: 12.5px;">도매 12,473 소매 9,929</div>
              <div class="num" style="width: 58px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">2,544</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 96px; flex: none;">코나</div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 92%; height: 100%; background: #2F6FA8;"></div>
              </div>
              <div class="num" style="width: 140px; flex: none; text-align: right; font-size: 12.5px;">도매 13,993 소매 11,665</div>
              <div class="num" style="width: 58px; flex: none; text-align: right; font-size: 12.5px; font-weight: 600;">2,328</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 96px; flex: none;">베뉴</div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 42%; height: 100%; background: #5C93BF;"></div>
              </div>
              <div class="num" style="width: 140px; flex: none; text-align: right; font-size: 12.5px;">도매 7,306 소매 6,237</div>
              <div class="num" style="width: 58px; flex: none; text-align: right; font-size: 12.5px;">1,069</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 96px; flex: none;">싼타페</div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 29%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div class="num" style="width: 140px; flex: none; text-align: right; font-size: 12.5px;">도매 4,494 소매 3,762</div>
              <div class="num" style="width: 58px; flex: none; text-align: right; font-size: 12.5px;">732</div>
            </div>
            <div style="display: flex; align-items: center; gap: 10px;">
              <div class="body" style="width: 96px; flex: none;">쏘나타</div>
              <div style="flex: 1; height: 14px; background: #F4F2EE; border-radius: 3px; overflow: hidden;">
                <div style="width: 17%; height: 100%; background: #8FB4D2;"></div>
              </div>
              <div class="num" style="width: 140px; flex: none; text-align: right; font-size: 12.5px;">도매 1,859 소매 1,415</div>
              <div class="num" style="width: 58px; flex: none; text-align: right; font-size: 12.5px;">444</div>
            </div>
          </div>
        </div>
      </div>

      <div data-el="13" style="display: flex; flex-direction: column; gap: 10px;">
        <div class="lbl">해설</div>
        <div style="font-size: 13.5px; line-height: 1.75; color: #1A1B19; text-wrap: pretty;">
          미주 전체로는 도매와 소매 차이가 5.0%지만 나라별로 크게 갈립니다. 칠레와 페루는 넘긴 물량의 5분의 1이 아직 팔리지 않았고, 두 나라 모두 선적과 도매가 정확히 같아 물량이 딜러 단계에서 멈춰 있습니다.<span class="fn">1</span> 캐나다는 격차율은 8.3%지만 대수로는 5,986대로 커서, 아반떼와 코나 두 차종이 그중 4,872대를 차지합니다.<span class="fn">3</span> 반대로 푸에르토리코와 콜롬비아는 소매가 도매를 앞질러 이전에 쌓인 물량을 덜어내는 중입니다.<span class="fn">2</span>
        </div>
      </div>

      <div data-el="14" style="display: flex; align-items: flex-start; gap: 8px; border-top: 1px solid #EFEDE7; padding-top: 14px;">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#9A9890" stroke-width="2" stroke-linecap="round" style="flex: none; margin-top: 1px;"><circle cx="12" cy="12" r="9"></circle><path d="M12 8v5"></path><path d="M12 16h.01"></path></svg>
        <div style="font-size: 11.5px; color: #9A9890; line-height: 1.6;">이 리포트는 판매 대시보드 데이터만 씁니다. 왜 안 팔리는지는 여기서 답하지 않습니다. 바깥 원인은 종합 리포트에서 봅니다. 도매는 공식 집계 기준이며 실 도매(664,269대) 기준과 12,932대 차이가 있습니다.</div>
      </div>

    </div>
  </main>
</div>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 좌측 사이드 | 248px 보조면 | 출처, 지표, 차원, 버전 | 없음 |
| 2 | 출처 블록 | 블록 | 대시보드 이름, 4계열 구성, 스냅샷 일시 | 없음 |
| 3 | 트래킹 지표 | 목록 3 | 유통 체류율(도매 정본 대비 소매, `distributionStay`), 계획 대비 진도율, 전년 동월 대비 | 선택한 지표가 본문 기준 |
| 4 | 분해 차원 칩 | 칩 다수 | 국가 29, 차종그룹 37, 대리점, 차급 16, CBU/CKD | 없음 |
| 5 | 범위 경고 | 주의색 박스 | 국가와 차종이 함께 있는 파일이 미주뿐이라는 사실. 못 만드는 지표는 전 세계 국가×차종(`globalCountryByModel`) 하나 | 없음 |
| 6 | 버전 목록 | 목록 | 게시 사본이 가진 현재 버전과 직전 버전 | 그 버전 지면 |
| 7 | 지면 헤더 | 44px 줄 | 배지, 제목, 버전 알약, "미주 29개국 · 누계 기준" | 없음 |
| 8 | 내려받기 | 버튼 | 없음 | 보고 있는 버전 지면을 PDF로 |
| 9 | 요약 | 16px 문단 | 격차 대수와 상위 국가, 반대 방향 국가 | 각주 강조 |
| 10 | 단계별 흐름 | 3칸과 간격 2 | 선적, 도매 정본, 소매의 누계와 두 구간 차이. 선적 대비 도매 정본은 "법인 단계 체류"(`entityStageStay`, 2,350대), 도매 정본 대비 소매는 "유통 체류"(`distributionStay`, 34,104대). 부호 규칙 캡션 | 없음 |
| 11 | 국가별 격차 | 막대 표 두 묶음 | 유통 체류율 기준으로 쌓이는 쪽과 덜어내는 쪽. 비율과 대수 | 없음 |
| 12 | 차종별 분해 | 막대 표 | 한 국가 안의 차종별 도매·소매와 차이 | 없음 |
| 13 | 해설 | 문단 | 비율과 대수가 다른 이야기를 한다는 것을 밝히는 서술 | 각주 강조 |
| 14 | 고정 문구 | 회색 줄 | 범위 안내와 도매 정본 차이 | 없음 |

##### 규칙

- 요약, 해설, 각주 근거, 버전 번호는 브리핑 갈래가 게시한 A 리포트 사본에서 그대로 읽는다. 고정 문구와 분해 블록(표·막대)은 이 시스템 배치가 만들어 같은 사본에 둔 것이다. 이 화면이 문장을 새로 만들지 않는다(2.2절).
- 단계별 흐름의 두 구간에 각각 이름을 붙인다. 선적에서 도매 정본을 뺀 구간은 "법인 단계 체류"이고 코드는 `entityStageStay`, 실측 2,350대다. 도매 정본에서 소매를 뺀 구간은 "유통 체류"이고 코드는 `distributionStay`, 실측 34,104대다. 두 실측값은 CDO 인입 샘플 CSV 추출본의 미주 누계 기준이다. 두 값을 한 칸으로 합치거나 이름을 바꿔 쓰지 않는다([[TBL-DOM-004#SalesStageFlow]]).
- 화면은 감소 방향을 보이려고 앞에 빼기 부호를 붙여 그린다. API 값과 저장 값은 양수다. 부호는 렌더링 규칙이지 값이 아니다. 단계별 흐름의 `-2,350`과 `-34,104`, 트래킹 지표의 `-5.0%`가 그 자리이고, 넘어오는 값은 2,350 · 34,104 · 0.050이다.
- 트래킹 지표의 `-5.0%`는 캡션에 "도매 정본보다 소매가 적은 방향"을 함께 적는다. 같은 화면의 단계별 흐름이 같은 현상을 빼기 부호로 그리므로 이 줄만 양수로 적으면 한 화면 안에서 두 자리가 어긋나 보인다.
- 단계별 흐름의 구간 차이는 음수도 정상 값이다. 방향을 색과 부호로 함께 표시한다([[TBL-PRD-002#R9]]). 다만 국가별 표의 "덜어내는 쪽"(푸에르토리코 -13.7%)은 값 자체가 음수이고, 위 렌더링 부호와 같은 것으로 읽지 않는다.
- 국가별 표의 기준 지표는 유통 체류율이다. "쌓이는 쪽"과 "덜어내는 쪽" 두 묶음으로 나누고 음수를 한 표에 섞지 않는다.
- 국가 코드는 대리점 코드 앞 세 자리를 그대로 보여 준다. 미국 B28, 캐나다 B06이다.
- 차종은 그룹 단위로 집계하고 세부 차종은 분해에만 쓴다. 모델 교체가 -100%로 잡히지 않게 하기 위함이다(PRD 6.15, [[TBL-PRD-002#R13]]).
- 도매 정본이 정해지기 전에는 도매(공식)를 기본으로 쓰고 실 도매와의 차이 12,932대를 고정 문구에 적는다. 이 12,932대는 미주 누계 상세 행 기준이다(CDO 인입 샘플 CSV 추출본, 실 도매 664,269 대 도매(공식) 677,201). 같은 파일의 전체 총계 행은 차이가 14,203대이지만 총계 행은 상세 행과 범위가 달라 대조에만 쓴다. 범위가 다른 것은 미주만 남긴 추출본에 전 권역 총계 행이 남은 탓이다([[TBL-DOM-004#ThresholdSetting]]).
- 못 만드는 지표는 전 세계 국가×차종(`globalCountryByModel`) 하나다. 미주 밖 국가의 실적이 비어 있어서다. 사이드 범위 경고에 그대로 적고 목록에는 넣지 않는다([[TBL-PRD-002#R1]]).
- 뉴스와 시장지표를 인용하지 않는다. "왜 안 팔리는지는 여기서 답하지 않습니다"를 고정 문구에 둔다.

##### 시나리오

**S-1 판매 리포트를 본다** [[TBL-UC-002#UC-H1]]
1. H가 판매 대시보드를 연다.
2. 단계별 흐름에서 법인 단계 체류가 2,350대, 유통 체류가 34,104대로 나온 것을 본다. 벌어진 쪽은 유통 체류다.
3. 국가별 표에서 칠레 25.2%, 페루 21.2%를 확인한다.
4. 캐나다 차종별 표에서 아반떼와 코나가 4,872대를 만든 것을 본다.

**S-2 분해할 차원이 없는 지표** [[TBL-UC-002#UC-H1]] 2a
1. 변동 판정만 표시되고 그 자리에 "분해 불가"가 적힌다.

**연관**: [[TBL-PRD-002#R1]] · [[TBL-PRD-002#R9]] · [[TBL-PRD-002#R13]] · [[TBL-UC-002#UC-H1]] · [[TBL-DOM-004#SalesStageFlow]] · [[TBL-DOM-004#DomainJudgment]]

### 2.3 관리 화면

네 화면에는 시안이 없다. 아래 구성은 유스케이스 흐름에서 유도한 최소안이며 [확인 필요]다. 디자인 토큰은 3장을 그대로 쓴다. 관리자 로그인 뒤에만 열린다.

#### UI-5 데이터 적재

| 항목 | 내용 |
|---|---|
| 경로 | `/admin/ingest` |
| 주 유스케이스 | [[TBL-UC-002#UC-A1]] · 포함 [[TBL-UC-002#UC-S10]] |
| 진입 / 이탈 | 관리 화면 좌측 메뉴 / 적재 확정 후 [[#UI-6]]으로 이동 가능 |

페이지. 소스를 고르고 파일을 올려 형태 판별과 미리보기를 거쳐 적재를 확정한다.

##### 배치

```html
<main data-screen="UI-5" class="page admin" style="width:1180px;padding:24px">
  <header class="admin-head"><h1>데이터 적재</h1></header>
  <section data-el="1" class="card upload">
    <label class="lbl">소스</label>
    <select data-el="2"><option>완성차 생산</option><option>완성차 판매</option><option>뉴스</option><option>블룸버그</option><option>Marklines</option><option>크로스워크</option></select>
    <div data-el="3" class="dashed dropzone">파일을 올립니다</div>
  </section>
  <section data-el="4" class="card detect">
    <span class="lbl">형태 판별</span>
    <span class="chip chip-accent">형태 B · 피벗 리포트</span>
    <span class="cap">병합 헤더 3줄, 기간 블록 반복</span>
  </section>
  <section data-el="5" class="card preview">
    <span class="lbl">미리보기</span>
    <div class="box">기간 구분 일·월·누계·년 · 지표 종류 9 · 차원 조합 1,339 · 총계 행 1</div>
    <div class="box warn-box">총계 행 선적 1,710,715 · 상세 행 합 679,551 · 차이 경고</div>
    <label class="cap">파일 기준일 <input data-el="6" type="date"></label>
  </section>
  <section data-el="7" class="card dual-source">
    <span class="lbl">둘씩 들어온 값</span>
    <div class="grid-2"><div class="box">운영계획 / 사업계획</div><div class="box">실 도매 / 도매(공식)</div></div>
    <p class="cap">둘 다 저장합니다. 정본은 설정 테이블의 새 버전으로만 바뀝니다. 지금 기본값은 사업계획과 도매(공식)입니다.</p>
  </section>
  <section data-el="8" class="card regression">
    <span class="lbl">회귀 검사</span>
    <div class="box">행수 128,440 · 결측률 0.4% · 매칭률 97.1% · 중복률 0.0% · 직전 대비 매칭률 -0.3%p</div>
    <div data-el="9" class="warn-box" style="display:none">급변입니다. 적재는 마쳤고 배치 자동 실행을 멈췄습니다. 해제는 관리자가 합니다.</div>
  </section>
  <div class="admin-foot">
    <label><input data-el="10" type="checkbox"> 백필 모드. LLM 세 역할을 생략하고 게시하지 않습니다</label>
    <button data-el="11" class="primary">적재 확정</button>
    <span data-el="12" class="cap">같은 기준일 파일이 있어 덮어씁니다</span>
  </div>
  <section data-el="13" class="card ingest-history">
    <span class="lbl">최근 적재</span>
    <div class="box">일시 · 소스 · 형태 · 기간 단위 · 행수 · 결측률 · 매칭률 · 결과</div>
  </section>
</main>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 업로드 영역 | 영역 | 소스 선택과 파일 놓는 자리 | 없음 |
| 2 | 소스 선택 | 드롭다운 | 완성차 생산, 완성차 판매, 뉴스, 블룸버그, Marklines, 크로스워크 | 선택 |
| 3 | 파일 놓는 자리 | 드롭존 | 파일명, 크기 | 파일 선택창 |
| 4 | 형태 판별 결과 | 칩과 설명 | 형태 A인지 형태 B인지와 판별 근거 | 없음 |
| 5 | 미리보기 | 영역 | 형태별로 다른 검사 결과. 형태 B는 총계 행 값과 상세 행 합의 차이 경고 | 없음 |
| 6 | 파일 기준일 | 날짜 입력 | 형태 B에서만. 판별값 또는 관리자 입력 | 입력 |
| 7 | 둘씩 들어온 값 | 블록 | 계획 두 종류, 도매 두 기준이 함께 들어왔다는 사실 | 없음 |
| 8 | 회귀 검사 | 표 | 행수, 결측률, 매칭률, 중복률과 직전 대비 | 없음 |
| 9 | 급변 경고 | 주의색 박스 | 어느 수치가 얼마나 벌어졌는지, 자동 실행이 막혔다는 사실 | 없음 |
| 10 | 백필 모드 | 체크박스 | 후보와 신호등까지만 채우고 LLM 생략, 게시 없음 | 켬과 끔 |
| 11 | 적재 확정 | 버튼 | 없음 | 원본 저장, 표준화, 미매핑 수집, 회귀 검사 실행 |
| 12 | 덮어쓰기 안내 | 텍스트 | 형태 A는 기준일자 범위, 형태 B는 파일 기준일이 겹칠 때 | 없음 |
| 13 | 최근 적재 이력 | 표 | 일시, 소스, 형태, 기간 단위, 행수, 결측률, 매칭률, 결과 | 행을 누르면 상세 |

##### 규칙

- 형태 판별은 자동이다. 첫 행이 바로 헤더이고 기준일자 컬럼이 있으면 형태 A, 병합 헤더가 2~3줄이고 기간 블록이 반복되면 형태 B다([[TBL-INFRA-002#C6]]).
- 어느 쪽으로도 판별되지 않으면 어느 대목이 다른지 보여 주고 적재 버튼을 막는다. 새 형태 등록은 개발 작업이다([[TBL-UC-002#UC-A1]] 2a).
- 원본 파일은 판별 결과와 무관하게 먼저 보존한다([[TBL-PRD-002#N7]]).
- 총계 행은 상세 행과 분리해 보관하고 집계에 더하지 않는다. 미리보기에 총계 행 값과 상세 행 합을 나란히 적고 차이를 경고로 보인다. 배치 예시의 차이(총계 행 1,710,715 대 상세 행 합 679,551)는 미주만 남긴 CDO 인입 샘플 CSV 추출본의 것이고, 엑셀 표본의 전 권역 시트는 둘이 같다. 검증식은 상세 행 합만 쓴다([[TBL-INFRA-002#C6]], [[TBL-DOM-004#IngestFile]]).
- 둘씩 들어온 값은 "이 파일에 두 종류가 함께 들어왔다"는 사실만 보여 준다. 정본을 이 화면에서 고르지 않는다. 정본 선택은 설정 테이블의 새 버전으로만 바뀐다([[TBL-INFRA-002#C18]]).
- 멱등 단위를 화면에 적는다. 형태 A는 기준일자, 형태 B는 파일 기준일이다([[TBL-INFRA-002#C6]]).
- 회귀 검사가 급변으로 판정하면 수동 적재이므로 적재는 마치되 배치 자동 실행 플래그를 끄고 그 사실을 화면에 남긴다. 해제는 관리자가 한다([[TBL-UC-002#UC-S10]] 4, [[TBL-INFRA-002#C20]]).
- 형태가 직전과 달라진 것도 급변으로 본다.
- 결측은 NULL로 넣는다.

##### 시나리오

**S-1 피벗 리포트를 적재한다** [[TBL-UC-002#UC-A1]]
1. A가 완성차 판매를 고르고 파일을 올린다.
2. 형태 B로 판별된다. 병합 헤더가 펼쳐져 기간 구분 넷과 지표 종류 아홉이 목록으로 나온다. 총계 행과 상세 행 합의 차이가 경고로 보인다.
3. A가 파일 기준일을 확인한다.
4. A가 적재 확정을 누른다. 같은 기준일 파일이 있어 덮어쓴다는 안내가 먼저 뜬다.
5. 회귀 검사 결과가 이력에 남는다.

**S-2 회귀 검사 급변** [[TBL-UC-002#UC-A1]] 7a
1. 매칭률이 직전 대비 허용 폭 밖으로 떨어진다.
2. 적재는 끝나지만 배치 자동 실행이 멈추고 경고가 남는다.
3. A가 [[#UI-7]]에서 미매핑을 보강한 뒤 [[#UI-6]]에서 재실행한다.

**연관**: [[TBL-PRD-002#R3]] · [[TBL-PRD-002#R5]] · [[TBL-PRD-002#R7]] · [[TBL-PRD-002#R8]] · [[TBL-PRD-002#N7]] · [[TBL-PRD-002#N8]] · [[TBL-UC-002#UC-A1]] · [[TBL-UC-002#UC-S10]] · [[TBL-DOM-004#IngestFile]] · [[TBL-INFRA-002#C6]]

#### UI-6 배치 이력과 재실행

| 항목 | 내용 |
|---|---|
| 경로 | `/admin/batch` |
| 주 유스케이스 | [[TBL-UC-002#UC-A3]] · 포함 [[TBL-UC-002#UC-S1]] |
| 진입 / 이탈 | 관리 화면 좌측 메뉴, 또는 [[#UI-5]] 적재 후 / 없음 |

페이지. 배치 8단계의 결과를 보고 지정 단계부터 다시 돌리거나 특정 일자를 재생성한다.

##### 배치

```html
<main data-screen="UI-6" class="page admin" style="width:1180px;padding:24px">
  <header class="admin-head"><h1>배치 이력</h1></header>
  <section data-el="1" class="card run-list">
    <div class="box">기준일 · 시작 · 소요 · 상태 · 실패 단계 · 게시 여부 · 백필</div>
  </section>
  <section data-el="2" class="card stage-timeline">
    <span class="lbl">단계별 결과 <span class="cap">1~5 코드 판정 · 6~8 LLM 서술</span></span>
    <ol>
      <li class="ok">1 적재 <span class="num">00:04:12 · 128,440행</span></li>
      <li class="ok">2 A 판정 읽기 <span class="num">00:00:18 · 3본</span></li>
      <li class="ok">3 사건 묶음 <span class="num">00:01:02 · 사건 41</span></li>
      <li class="ok">4 결합 <span class="num">00:02:31 · 국가 29</span></li>
      <li class="ok">5 원인 후보·근접도·신호등 <span class="num">00:03:44 · 변동 12</span></li>
      <li class="degraded">6 사건 명명 <span class="num">강등 · JSON 위반</span></li>
      <li class="ok">7 연관 설명 <span class="num">00:02:10 · 토큰 84,200</span></li>
      <li class="ok">8 서술·검증·게시 <span class="num">00:01:55 · 토큰 61,700 · A 사본 3본</span></li>
    </ol>
  </section>
  <section data-el="3" class="card rerun">
    <span class="lbl">재실행</span>
    <label class="cap">기준일 <input type="date"></label>
    <label class="cap">시작 단계 <select data-el="4"><option>1</option><option>2</option><option>3</option><option>4</option><option>5</option><option>6</option><option>7</option><option>8</option></select></label>
    <button data-el="5" class="primary">재실행</button>
    <p class="cap">그 기준일의 데이터 스냅샷, 당시 A 판정, 당시 설정 버전으로 돌립니다.</p>
  </section>
  <section data-el="6" class="card diff">
    <span class="lbl">당시와 비교</span>
    <div class="box">필드 · 당시 · 지금 · 원인(매핑·설정·늦게 온 데이터·A 판정)</div>
    <p data-el="7" class="cap">변동·후보·근접도·신호등은 같은 입력이면 같아야 합니다. 설명과 문장은 새로 생성되므로 달라질 수 있습니다.</p>
    <button data-el="8" class="ghost">새 버전으로 게시</button>
  </section>
</main>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 배치 목록 | 표 | 기준일, 시작 시각, 소요, 상태, 실패 단계, 게시 여부, 백필 여부 | 행을 누르면 2번에 그 배치 |
| 2 | 단계 타임라인 | 8줄 목록 | 단계 이름, 소요, 처리 건수, LLM 호출 수와 토큰, 강등 표시 | 없음 |
| 3 | 재실행 영역 | 영역 | 기준일과 시작 단계 | 없음 |
| 4 | 시작 단계 | 드롭다운 | 1~8 | 선택 |
| 5 | 재실행 버튼 | 버튼 | 없음 | 지정 단계부터 실행 |
| 6 | 비교 표 | 표 | 당시 값과 지금 값, 차이 필드와 원인 | 없음 |
| 7 | 재현 안내 | 텍스트 | 무엇이 같아야 하고 무엇이 달라질 수 있는지 | 없음 |
| 8 | 새 버전 게시 | 버튼 | 없음 | 재생성 결과를 새 버전으로 게시. 워치리스트 변화 비교가 함께 돈다 |

##### 규칙

- 8단계 이름을 인프라 확정 이름 그대로 쓴다. 적재, A 판정 읽기, 사건 묶음, 결합, 원인 후보·근접도·신호등, 사건 명명, 연관 설명, 서술·검증·게시다([[TBL-INFRA-002#C19]]).
- 1~5단계는 코드, 6~8단계는 LLM임을 화면에서 구분해 보여 준다. 강등은 6~8단계에만 생긴다.
- 단계별로 멈춤·강등·예외가 다르다. 1·3·4·5는 멈춤, 6~8은 강등, 2는 보완 집계로 내려가는 예외다. 타임라인에 이 셋을 구분해 표시한다([[TBL-INFRA-002#C20]]).
- 기존 게시 리포트를 덮어쓰지 않는다. 재생성은 항상 새 버전이다([[TBL-UC-002#UC-A3]]).
- 같은 기준일의 배치가 진행 중이면 재실행을 받지 않고 안내한다([[TBL-UC-002#UC-A3]] 1a).
- 비교 표에서 변동, 후보, 근접도, 신호등이 달라졌으면 그 자체가 재현성 위반이다. 원인 컬럼을 반드시 채운다([[TBL-PRD-002#N3]]).
- 회귀 검사에 LLM 없는 실행 대조가 포함된다. 그 결과를 이력에서 볼 수 있어야 한다([[TBL-INFRA-002#C19]], [[TBL-PRD-002#R16]]).
- 재생성 중 정기 배치 시각이 오면 정기 배치를 대기시킨다. 대기 상태를 목록에 표시한다([[TBL-UC-002#UC-A3]] 2a).

##### 시나리오

**S-1 실패 지점부터 다시 돌린다** [[TBL-UC-002#UC-A3]]
1. A가 목록에서 실패한 배치를 고른다.
2. 타임라인이 4단계에서 멈춘 것을 보여 준다.
3. A가 시작 단계를 4로 두고 재실행한다.
4. 비교 표에 변동과 신호등이 당시와 같은지 나온다.
5. A가 새 버전으로 게시한다.

**S-2 LLM 없는 대조** [[TBL-INFRA-002#C19]]
1. A가 백필 모드로 같은 기준일을 돌린다.
2. 신호등과 후보 순서가 정상 실행과 같은지 비교 표에서 확인한다.

**연관**: [[TBL-PRD-002#R30]] · [[TBL-PRD-002#R18]] · [[TBL-PRD-002#N2]] · [[TBL-PRD-002#N3]] · [[TBL-PRD-002#N6]] · [[TBL-UC-002#UC-A3]] · [[TBL-UC-002#UC-S1]] · [[TBL-DOM-004#BatchRun]] · [[TBL-INFRA-002#C19]]

#### UI-7 마스터와 미매핑

| 항목 | 내용 |
|---|---|
| 경로 | `/admin/master` |
| 주 유스케이스 | [[TBL-UC-002#UC-A2]] |
| 진입 / 이탈 | 관리 화면 좌측 메뉴 / 과거 재계산은 [[#UI-6]]으로 |

페이지. 미매핑 목록을 내려받고 크로스워크를 올려 마스터를 새 버전으로 갱신한다.

##### 배치

```html
<main data-screen="UI-7" class="page admin" style="width:1180px;padding:24px">
  <header class="admin-head"><h1>마스터와 미매핑</h1></header>
  <section data-el="1" class="card match-rate grid-3">
    <div class="box kpi"><span class="lbl">판매 → 국가</span><span class="num">97.1%</span></div>
    <div class="box kpi"><span class="lbl">생산 → 국가</span><span class="num">목적지 없음</span></div>
    <div class="box kpi"><span class="lbl">뉴스 → 국가</span><span class="num">91.4%</span></div>
  </section>
  <section data-el="2" class="card unmapped">
    <span class="lbl">미매핑 목록</span>
    <div class="box">소스 · 값 · 등장 횟수 · 첫 등장일</div>
    <button data-el="3" class="ghost">엑셀로 내려받기</button>
  </section>
  <section data-el="4" class="card crosswalk-upload">
    <span class="lbl">크로스워크 업로드</span>
    <div class="dashed dropzone">보강한 시트를 올립니다</div>
  </section>
  <section data-el="5" class="card diff">
    <span class="lbl">매핑 차이</span>
    <div class="box">추가 24 · 변경 3 · 삭제 0 · 영향 국가 11 · 영향 공장 30</div>
    <div data-el="6" class="warn-box" style="display:none">삭제로 미매핑이 될 값이 N건입니다</div>
    <button data-el="7" class="primary">확정</button>
    <p class="cap">다음 배치부터 적용됩니다. 과거 판정은 바뀌지 않습니다.</p>
  </section>
  <section data-el="8" class="card master-history">
    <span class="lbl">마스터 버전</span>
    <div class="box">버전 · 적용일 · 변경 건수 · 적용자</div>
  </section>
</main>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 매칭률 | 카드 3 | 판매, 생산, 뉴스 각각의 국가 매칭률과 직전 대비. 생산은 목적지 국가가 없어 "목적지 없음" | 없음 |
| 2 | 미매핑 목록 | 표 | 소스, 값, 등장 횟수, 첫 등장일 | 정렬 |
| 3 | 엑셀 내려받기 | 버튼 | 없음 | 미매핑 목록을 파일로 |
| 4 | 크로스워크 업로드 | 드롭존 | 파일명 | 파일 선택창 |
| 5 | 매핑 차이 | 표 | 추가, 변경, 삭제와 영향 받는 국가·공장 | 없음 |
| 6 | 삭제 경고 | 주의색 박스 | 삭제로 미매핑이 될 값의 건수 | 없음 |
| 7 | 확정 | 버튼 | 없음 | 마스터를 새 버전으로 갱신 |
| 8 | 마스터 버전 이력 | 표 | 버전, 적용일, 변경 건수, 적용자 | 없음 |

##### 규칙

- 기존 매핑은 확정 전까지 손상되지 않는다. 차이를 먼저 보여 주고 확인을 받는다.
- 같은 값이 두 국가로 갈리면 올리기를 막고 충돌 행을 보여 준다([[TBL-UC-002#UC-A2]] 2a).
- 삭제되는 매핑이 있으면 그로 인해 미매핑이 될 값의 건수를 경고하고 확인을 받는다([[TBL-UC-002#UC-A2]] 3a).
- 갱신은 다음 배치부터 적용된다. 과거 판정을 소급해 바꾸지 않는다. 과거를 다시 계산하려면 [[#UI-6]]에서 재생성한다.
- 판매 국가코드는 대리점 코드 앞 세 자리다. 미국 B28AB, 캐나다 B06AA로 확인됐다([[TBL-PRD-002#R4]]).
- 글로비스 법인 매핑이 없는 동안 [[#UI-1]]의 법인 태그는 "법인 미매핑"으로 나온다. 현대차 판매법인으로 대체하지 않는다([[TBL-PRD-002#R17]]).

##### 시나리오

**S-1 미매핑을 보강한다** [[TBL-UC-002#UC-A2]]
1. A가 미매핑 목록을 엑셀로 내려받는다.
2. A가 크로스워크 시트를 보강해 올린다.
3. 추가 24건, 변경 3건, 삭제 0건과 영향 국가 11개가 표로 나온다.
4. A가 확정한다. 마스터가 새 버전이 되고 다음 배치부터 적용된다.

**연관**: [[TBL-PRD-002#R4]] · [[TBL-PRD-002#R17]] · [[TBL-PRD-002#N8]] · [[TBL-UC-002#UC-A2]] · [[TBL-DOM-004#Country]] · [[TBL-DOM-004#GlovisEntity]]

#### UI-8 열람 주소

| 항목 | 내용 |
|---|---|
| 경로 | `/admin/exposure` |
| 주 유스케이스 | [[TBL-UC-002#UC-A4]] |
| 진입 / 이탈 | 관리 화면 좌측 메뉴 / 없음 |

페이지. 열람 화면 넷의 제공을 켜고 끄고, VODA에 걸 서명 주소를 보고, 새 주소로 바꾼다. 시안이 없어 유스케이스에서 유도한 최소 구성이다.

##### 배치

```html
<main data-screen="UI-8" class="page admin" style="width:1180px;padding:24px">
  <header class="admin-head"><h1>열람 주소</h1></header>
  <p data-el="1" class="cap">화면마다 주소가 하나입니다. 주소를 가진 사람은 그 화면을 봅니다. 개인을 가리지 않습니다.</p>
  <table data-el="2" class="table">
    <tr><th>화면</th><th>상태</th><th>주소</th><th>마지막 변경</th><th></th></tr>
    <tr><td>C 리포트</td><td data-el="3">제공 중</td><td><input data-el="4" readonly value="(PUBLIC_BASE_URL)/intel/creport?token=…"></td><td data-el="5">2026-09-30 14:05 · A001</td>
        <td><button data-el="6">끄기</button> <button data-el="7">복사</button> <button data-el="8">새 주소로 바꾸기</button></td></tr>
    <tr><td>A2 재고 리포트</td><td>주소 없음</td><td>-</td><td>-</td><td><button>켜고 주소 만들기</button></td></tr>
  </table>
</main>
```

##### 요소

| # | 이름 | 종류 | 보여주는 것 | 누르면 |
|---|---|---|---|---|
| 1 | 안내 | 텍스트 | 화면 단위 주소이고 개인을 가리지 않는다는 사실 | 없음 |
| 2 | 화면 목록 | 표 | C 리포트, A1, A2, A3 네 줄. 순서 고정 | 없음 |
| 3 | 상태 | 텍스트 | 제공 중, 꺼짐, 주소 없음 | 없음 |
| 4 | 주소 | 읽기 전용 칸 | 서명 주소 완성본 | 누르면 전체 선택 |
| 5 | 마지막 변경 | 텍스트 | 시각과 관리자 사번 | 없음 |
| 6 | 켜기·끄기 | 버튼 | 처음이면 "켜고 주소 만들기" | 제공 전환. 처음이면 서명값 발급 |
| 7 | 복사 | 버튼 | 없음 | 주소를 클립보드로 |
| 8 | 새 주소로 바꾸기 | 버튼 | 없음 | 확인 뒤 서명값 회전. 이전 주소는 바로 막힌다 |

##### 규칙

- 화면은 넷으로 고정이다. 행이 없는 화면은 제공하지 않는 것과 같다([[TBL-DOM-006#screen_exposure]]).
- 새 주소로 바꾸기 전에 "지금 VODA에 걸린 주소는 바로 막히고 VODA 쪽 주소도 바꿔야 한다"는 확인을 거친다. 자동 회전은 없다.
- 끄면 주소는 남고 열람만 막힌다. 다시 켜면 같은 주소가 열린다.
- 서명값은 주소 칸 하나에만 보인다. 로그와 오류 문장에 남기지 않는다.
- 현업 화면에서 주소가 맞지 않거나 제공이 꺼져 있으면 리포트를 그리지 않고 안내만 낸다([[TBL-INFRA-002]] 5장).

##### 시나리오

**S-1 C 리포트를 VODA에 건다** [[TBL-UC-002#UC-A4]]
1. A가 열람 주소 화면을 연다. 네 화면 모두 주소 없음이다.
2. A가 C 리포트 줄의 "켜고 주소 만들기"를 누른다. 주소 완성본이 나온다.
3. A가 복사해 VODA의 C 리포트 버튼에 건다.
4. 현업이 그 주소로 들어와 C 리포트를 본다. 같은 주소로 A 리포트는 열리지 않는다.

**S-2 주소가 새었다** [[TBL-UC-002#UC-A4]] 2a
1. A가 A1 줄의 "새 주소로 바꾸기"를 누르고 확인한다.
2. 이전 A1 주소는 바로 막힌다. A가 VODA 쪽 주소를 새 주소로 바꾼다.

**연관**: [[TBL-PRD-002#R29]] · [[TBL-UC-002#UC-A4]] · [[TBL-UC-002#UC-H1]] · [[TBL-UC-002#UC-H2]] · [[TBL-INFRA-002#C12]]

## 3. 디자인 토큰과 공통 틀

아래 첫 html 블록은 모든 화면 앞에 함께 들어간다. 앱에서 그대로 가져온 토큰을 변수로 두고 배치가 쓰는 공통 클래스를 정의한다. 구현에서는 `frontend/src/styles.css`에 같은 변수를 전사하고 컴포넌트에 값을 직접 쓰지 않는다.

```html
<style>
  :root{--bg:#F4F2EE;--surface:#FFFFFF;--surface-2:#FBFAF8;--border:#E2DFD8;--border-input:#DAD6CE;--text:#1A1B19;--text-body:#3C3B37;--text-2:#6B6A65;--label:#9A9890;--text-muted:#A6A49C;--chip-bg:#EFEDE7;--accent:#0E6B57;--accent-bg:#E7F0ED;--accent-text:#0B5847;--warn:#B5730C;--danger:#B3261E;--danger-border:#F0CFCB;--chart-1:#2F6FA8;--chart-2:#5C93BF;--chart-3:#8FB4D2;--chart-4:#C9922E;--chart-5:#4B8B7A;--chart-6:#8B6BA8;--radius-card:11px;--radius-control:7px;--radius-sm:5px;--radius-pill:20px}
  body{margin:0;background:var(--bg);color:var(--text);font-family:'Pretendard Variable',Pretendard,'Noto Sans KR',-apple-system,'Malgun Gothic',sans-serif;font-size:13.5px;line-height:1.7}
  .page,.report{box-sizing:border-box;background:var(--bg)}
  .page-head{display:flex;align-items:baseline;gap:12px;margin-bottom:16px}
  .page-head h1,.admin-head h1{margin:0;font-size:20px;font-weight:600}
  .sub,.cap{font-size:11.5px;color:var(--text-2)}
  .asof{margin-left:auto;font-size:11.5px;color:var(--label)}
  .lbl{display:block;font-size:10.5px;font-weight:500;text-transform:uppercase;letter-spacing:.09em;color:var(--label);margin-bottom:6px}
  .num{font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;font-variant-numeric:tabular-nums}
  .card{background:var(--surface);border:1px solid var(--border);border-radius:var(--radius-control);padding:16px 18px;margin-bottom:20px}
  .box{background:var(--surface-2);border:1px solid var(--border);border-radius:var(--radius-control);padding:12px 14px}
  .kpi .num{display:block;font-size:21px;font-weight:600}
  .grid-2{display:grid;grid-template-columns:1fr 1fr;gap:12px}
  .grid-3{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
  .market-bar{display:grid;grid-template-columns:repeat(4,1fr);gap:12px;margin-bottom:20px}
  .metric{background:var(--surface);border:1px solid var(--border);border-radius:var(--radius-control);padding:12px 14px}
  .metric .num{display:block;font-size:18px;font-weight:600}
  .headline-text{font-size:19px;font-weight:500;line-height:1.6;letter-spacing:-.01em;margin:8px 0 0}
  .chip{display:inline-block;font-size:11.5px;padding:3px 9px;border-radius:var(--radius-control);background:var(--chip-bg);color:var(--text-body);margin-right:6px}
  .chip-accent{background:var(--accent-bg);color:var(--accent-text)}
  .chip-muted{background:var(--surface-2);color:var(--label);border:1px solid var(--border)}
  .state{display:inline-block;font-size:11.5px;font-weight:600;padding:3px 9px;border-radius:var(--radius-control);margin-right:6px}
  .state-red{color:var(--danger);background:rgba(179,38,30,.08);border:1px solid var(--danger-border)}
  .state-yellow{color:var(--warn);background:rgba(181,115,12,.10)}
  .state-na{color:var(--label);background:var(--surface-2);border:1px solid var(--border)}
  .sec-head{display:flex;align-items:baseline;gap:12px;margin-bottom:10px}
  .domain-card h3,.anomaly-card h2{margin:6px 0;font-size:15px}
  .card-head{display:flex;align-items:center;gap:8px;margin-bottom:12px}
  .cause-top{margin-top:12px}.cause-top h3{margin:0 0 6px;font-size:14px}
  .cause-text,.summary,.prose{margin:10px 0}
  .summary{font-size:16px;font-weight:500;line-height:1.65}
  .card-foot,.panel-head{display:flex;align-items:center;gap:12px;margin-top:12px}
  .no-cause{border:1px dashed var(--border-input);border-radius:var(--radius-control);padding:12px 14px;color:var(--text-2)}
  .page-foot{margin-top:24px;font-size:11px;color:var(--label)}
  .fn{font-size:10px;vertical-align:super;color:var(--accent);font-weight:600}
  .ghost{background:var(--surface);border:1px solid var(--border-input);border-radius:var(--radius-control);padding:6px 12px;font-size:12.5px;cursor:pointer}
  .primary{background:var(--accent-bg);color:var(--accent-text);border:1px solid var(--accent);border-radius:var(--radius-control);padding:6px 14px;font-weight:600;cursor:pointer}
  .cand{margin-bottom:12px}.cand-head{font-weight:600;margin-bottom:6px}
  .article-list{list-style:none;padding:0;margin:0}.article-list li{display:flex;gap:10px;padding:4px 0}.article-list a{color:var(--accent)}
  .value-row{display:flex;gap:16px}.cand-unused{color:var(--label);font-size:12px;padding:8px 0}
  .empty-card{text-align:center;padding:40px}.empty-card .title{font-size:16px;font-weight:600;margin:0 0 6px}
  .banner{border-radius:var(--radius-control);padding:12px 16px;display:flex;gap:12px;align-items:center;font-weight:500}
  .banner-warn{background:rgba(181,115,12,.10);color:var(--warn)}
  .banner-danger{background:rgba(179,38,30,.08);color:var(--danger);border:1px solid var(--danger-border)}
  .banner .title{margin:0 0 4px;font-weight:600}
  .warn{color:var(--warn)}
  .warn-box{background:rgba(181,115,12,.10);color:var(--warn);border-radius:var(--radius-sm);padding:8px 10px;font-size:11.5px;margin-top:8px}
  .danger-box{background:rgba(179,38,30,.08);color:var(--danger);border:1px solid var(--danger-border);border-radius:var(--radius-sm);padding:8px 10px;font-size:12px}
  .side{background:var(--surface-2);border:1px solid var(--border);border-radius:var(--radius-card);padding:18px 16px;font-size:12.5px;color:var(--text-body)}
  .side section{margin-bottom:18px}.side ul{list-style:none;padding:0;margin:0}.side li{padding:5px 0;display:flex;gap:6px;flex-wrap:wrap}.side li.active{font-weight:600;color:var(--text)}
  .x-list li::before{content:"×";color:var(--danger);margin-right:6px}
  .sheet{flex:1;background:var(--surface);border:1px solid var(--border);border-radius:var(--radius-card)}
  .sheet-head{display:flex;align-items:center;gap:10px;padding:0 22px;border-bottom:1px solid var(--border)}
  .sheet-body{padding:22px}.sheet-body section{margin-bottom:22px}
  .tag{background:var(--accent-bg);color:var(--accent-text);font-size:11px;font-weight:600;padding:3px 8px;border-radius:var(--radius-sm)}
  .title{font-weight:600}.pill{background:var(--chip-bg);border-radius:var(--radius-pill);padding:2px 10px;font-size:11.5px}
  .bar-row{display:flex;align-items:center;gap:10px;padding:5px 0}.bar-row .name{width:140px}.bar-row .bar{height:8px;background:var(--chart-1);border-radius:var(--radius-sm)}.bar-neg .bar{background:var(--chart-4)}
  .table-head{font-weight:600;margin:8px 0 4px}.stack{padding:4px 0}
  .flow{display:flex;align-items:center;gap:10px}.stage{flex:1}.stage .num{display:block;font-size:21px;font-weight:600}
  .gap{width:110px;text-align:center;background:var(--surface-2);border-top:1px solid var(--border);border-bottom:1px solid var(--border);padding:8px 4px}.gap-warn{background:rgba(181,115,12,.08)}.gap .num{display:block;font-size:12px;font-weight:600}
  .dashed{border:1px dashed var(--border-input);border-radius:var(--radius-control);padding:12px 14px;color:var(--label)}
  .dropzone{text-align:center;padding:28px}
  .fixed-note{font-size:11.5px;color:var(--label);border-top:1px solid var(--border);padding-top:12px}
  .admin-foot{display:flex;align-items:center;gap:16px;margin:16px 0 24px}
  ol{margin:0;padding-left:20px}ol li{padding:4px 0}ol li.degraded{color:var(--warn)}
  select,input[type=date]{border:1px solid var(--border-input);border-radius:var(--radius-control);padding:5px 8px;font-size:12.5px}
  .var{margin:32px 0 8px;font-size:11.5px;color:var(--text-muted);border-left:3px solid var(--border-input);padding-left:10px}
</style>
```

### 3.1 디자인 토큰 표

| 이름 | 변수 | 값 | 쓰는 곳 |
|:--|:--|:--|:--|
| 배경 | `--bg` | `#F4F2EE` | 페이지 바탕 |
| 카드 | `--surface` | `#FFFFFF` | 카드, 지면 |
| 보조면 | `--surface-2` | `#FBFAF8` | 사이드, 표 머리, 내부 박스 |
| 경계선 | `--border` | `#E2DFD8` | 카드 테두리, 구분선 |
| 입력 테두리 | `--border-input` | `#DAD6CE` | 버튼, 입력, 점선 |
| 글자 | `--text` | `#1A1B19` | 제목, 본문 강조 |
| 본문 | `--text-body` | `#3C3B37` | 사이드 본문 |
| 보조글자 | `--text-2` | `#6B6A65` | 설명, 캡션 |
| 라벨 | `--label` | `#9A9890` | 섹션 라벨, 단위 |
| 흐린글자 | `--text-muted` | `#A6A49C` | 주석 |
| 칩배경 | `--chip-bg` | `#EFEDE7` | 중립 칩, 알약 |
| 액센트 | `--accent` | `#0E6B57` | 링크, 각주, 강조 테두리 |
| 액센트배경 | `--accent-bg` | `#E7F0ED` | 리포트 배지, 활성 버튼 |
| 액센트글자 | `--accent-text` | `#0B5847` | 액센트 배경 위 글자 |
| 주의 | `--warn` | `#B5730C` | yellow 신호등, 유도값, 이월 |
| 경고 | `--danger` | `#B3261E` | red 신호등, 원천 부재 |
| 경고테두리 | `--danger-border` | `#F0CFCB` | 경고 박스 테두리 |

차트 색 여섯이다. 순서대로 쓴다. `--chart-1` `#2F6FA8`, `--chart-2` `#5C93BF`, `--chart-3` `#8FB4D2`, `--chart-4` `#C9922E`, `--chart-5` `#4B8B7A`, `--chart-6` `#8B6BA8`.

반경 넷이다. `--radius-card` 11px(A 리포트 사이드와 지면), `--radius-control` 7px(카드·버튼·칩·입력), `--radius-sm` 5px(작은 배지·막대), `--radius-pill` 20px(버전 알약).

| 타이포 | 값 |
|:--|:--|
| 본문 글꼴 | `'Pretendard Variable', Pretendard, 'Noto Sans KR', -apple-system, 'Malgun Gothic', sans-serif` |
| 수치 글꼴 | `ui-monospace, SFMono-Regular, Menlo, Consolas, monospace` + `font-variant-numeric: tabular-nums` |
| 섹션 라벨 | 10.5px / 500 / uppercase / letter-spacing .09em / `--label` |
| 헤드라인 | 19px / 500 / line-height 1.6 / letter-spacing -.01em |
| A 리포트 요약 | 16px / 500 / line-height 1.65 |
| 본문 서술 | 13.5px / line-height 1.7~1.75 |
| 사이드 본문 | 12.5px / `--text-body` |
| 캡션 | 11.5px / `--text-2` |
| 주석 | 11px / `--label` 또는 `--text-muted` |
| 강조 토큰 | 600 + 배경 `rgba(181,115,12,.14)` + 반경 3px |
| 각주 | 10px / super / `--accent` / 600 |

### 3.2 폭과 여백

| 화면 | 폭 | 안쪽 여백 |
|:--|:--|:--|
| [[#UI-1]] | 1280px | `28px 32px 40px` |
| [[#UI-2]] [[#UI-3]] [[#UI-4]] | 1180px. 사이드 248px + 간격 20px + 지면 나머지 | 바깥 24px, 지면 안쪽 22px |
| 근거 패널 | 카드 안쪽 폭 그대로. 단독으로 볼 때 760px | 18px 20px |
| 관리 화면 | [확인 필요] 1180px를 따르는 것으로 둔다 | 24px |

섹션 사이 간격은 [[#UI-1]] 20px, A 리포트 지면 22px이다. 카드 안 요소 간격은 10~14px이다. 폐쇄망 데스크톱 열람이 전제다. 좁은 화면 대응 범위는 [확인 필요]다.

### 3.3 공통 규칙

- 관련도 등급을 어디에도 두지 않는다. 등급 칩, 별점, 점수 막대, 색 농도로 관련 정도를 표현하지 않는다(PRD 6.14).
- 근접도 값은 칩으로 적는다. "변동 시점과 1일 차이", "기사 8건 · 출처 5곳", "해협 귀속" 형태다. 값 넷의 이름은 날짜 차이, 기사 수, 출처 수, 국가 일치 방식이다([[TBL-INFRA-002#C19]]).
- 정렬 기준을 화면에 적는다. "후보 4건 · 시점 가까운 순", "신호등 순 정렬", "낮은 순"처럼 글자로 둔다.
- 신호등은 색과 텍스트를 함께. red "확인 필요", yellow "주의", none "원인 미확인", notApplicable "판정 대상 아님"이다.
- 신호등은 규칙 산출물이다. 변동 카드의 신호등도 도메인 상태 카드의 신호등도 배치 5단계에서 코드가 정한다. LLM은 그 옆의 문장만 쓴다. 화면이 신호등을 다시 계산하지 않는다([[TBL-INFRA-002#C19]]).
- 수치는 등폭에 `tabular-nums`. 비율과 절대 대수를 함께 적는다.
- 못 만드는 것은 못 만든다고 적는다. 빈 자리를 추정값으로 채우지 않는다. 자리만 점선으로 표시한다.
- 유도값에는 유도 표기를 붙인다. 실측으로 바뀌면 표기를 내린다([[TBL-INFRA-002#C17]]).
- 기간 단위를 표시한다. 일·월·누계·년 중 무엇인지 화면에서 읽힌다([[TBL-INFRA-002#C15]]).
- 감소 방향을 빼기 부호로 그리는 것은 렌더링 규칙이다. 저장 값과 API 값은 양수다. 부호를 붙이는 자리에는 캡션으로 그 사실을 밝힌다([[#UI-3]] [[#UI-4]]).
- 외부 자원을 불러오지 않는다. 이미지, 글꼴, 스크립트를 외부에서 가져오지 않는다. 기사 원문만 새 창 링크다([[TBL-PRD-002#N1]]).
- 화면은 서버 호출을 직접 하지 않는다. `pages → api → 서버` 한 방향이다.
- LLM을 열람 시 호출하지 않는다. 게시된 결과를 조회만 한다([[TBL-PRD-002#N5]]).

### 3.4 상태 표현 모음

| 표현 | 모양 | 언제 |
|:--|:--|:--|
| 확인 필요 | 경고색 글자, `rgba(179,38,30,.08)` 배경, `--danger-border` 테두리 | 신호등 red |
| 주의 | 주의색 글자, `rgba(181,115,12,.10)` 배경 | 신호등 yellow |
| 원인 미확인 | 점선 테두리 박스, 회색 글자 | 신호등 none |
| 판정 대상 아님 | `--label` 글자, `--surface-2` 배경, 실선 테두리 | 신호등 notApplicable. 지금은 생산 카드가 여기 해당한다 |
| 신규 / 상승 | 액센트 글자, `--accent-bg` 배경 | 워치리스트 변화([[TBL-DOM-004#AlertEvent]]) |
| 법인 미매핑 | `--label` 글자, `--surface-2` 배경, 실선 테두리 | 법인 매핑 없음 |
| 노출 미확인 | 같은 모양 | CBU 비중 확인 실패 |
| 보완 집계 | 주의색 글자 | A 판정이 국가 단위가 아님 |
| 유도 | 주의색 글자 | 재고 파생값 |
| 값 이월 | 주의색 캡션 | 시장지표 이월. 값은 그대로 보인다 |
| 강등 | 카드 머리 주의색 띠 | LLM 역할 실패 |
| 리포트 문장 미수신 | 주의색 캡션 | A 리포트 게시 사본 없음 |

"원인 미확인"과 "판정 대상 아님"은 뜻이 다르다. 앞은 판정 대상인데 시간창 안에서 원인 후보를 찾지 못한 것이고, 뒤는 애초에 국가 축 판정 대상이 아닌 것이다. 생산은 목적지 국가가 없어 뒤에 해당한다. 두 표기를 바꿔 쓰지 않는다.

### 3.5 화면 구성은 고정이 아니다

발주자가 개요 시트를 디자인 방향으로 설명했다([[TBL-RFQ-002#Q15]]). 여기 적은 배치는 지금 데이터가 받쳐 주는 만큼의 구성이다. 다음 셋이 바뀌면 구성도 바뀐다.

| 바뀌면 | 어디가 바뀌나 |
|:--|:--|
| 생산에 목적지 국가가 들어온다 | [[#UI-1]] 변동 목록에 생산 카드가 오르고 도메인 상태 카드의 "판정 대상 아님"이 신호등으로 바뀐다. [[#UI-2]] 사이드 경고가 내려간다 |
| 재고 원천이 들어온다 | [[#UI-3]] 배너와 "없는 것" 목록이 내려가고 점선 블록이 실측값으로 찬다. [[#UI-1]] 재고 카드의 유도 표기가 사라진다 |
| 미주 밖 국가의 실적이 들어온다 | [[#UI-4]] 범위 문구가 바뀌고 못 만드는 지표 하나가 내려간다. [[#UI-1]] 변동 목록의 국가 수가 늘어난다 |

## 4. 화면 흐름

```mermaid
flowchart LR
  VODA["VODA 포털"] --> UI1["UI-1 C 리포트 메인"]
  UI1 -->|"근거 펼치기"| EV["근거 패널 · 같은 화면"]
  EV -->|"기사 제목"| EXT["새 창 · 기사 원문"]

  D1["생산 대시보드"] --> UI2["UI-2 A1 생산 리포트"]
  D2["재고 대시보드"] --> UI3["UI-3 A2 재고 리포트"]
  D3["판매 대시보드"] --> UI4["UI-4 A3 판매 리포트"]

  ADM["관리 화면 로그인"] --> UI5["UI-5 데이터 적재"]
  ADM --> UI6["UI-6 배치 이력·재실행"]
  ADM --> UI7["UI-7 마스터·미매핑"]
  ADM --> UI8["UI-8 열람 주소"]
  UI8 -.->|"서명 주소를 VODA에 건다"| VODA
  UI5 -->|"적재 후 재실행"| UI6
  UI7 -->|"과거 재계산"| UI6
```

A 리포트 셋에서 [[#UI-1]]로 가는 선은 없다. C는 VODA 단독 주소이고 A 리포트는 각 대시보드에 붙는다. 둘은 서로를 참조하지 않는다.

[[TBL-UC-002#UC-H3]]는 화면 이동이 아니다. [[#UI-1]] 카드 안에서 패널이 열릴 뿐이다.

관리 화면 넷은 서로를 오가지만 열람 화면과는 이어지지 않는다. [[#UI-8]]이 낸 서명 주소를 VODA가 걸고 현업은 그 주소로 [[#UI-1]]~[[#UI-4]]에 들어온다.

## 5. 미결사항

- [ ] 관리 화면 넷([[#UI-5]] [[#UI-6]] [[#UI-7]] [[#UI-8]])의 시안이 없다. 여기 적은 구성은 유스케이스에서 유도한 최소안이며 확정이 아니다
- [ ] 좁은 화면 대응 범위. 지금은 데스크톱 고정 폭 전제다(3.2절)
- [ ] 현업 피드백 버튼("맞다·아니다·모르겠다")을 [[#UI-1]] 카드에 넣을지([[TBL-PRD-002#R19]])
- [ ] 2본째 상세 화면을 만들지([[TBL-PRD-002#R28]]). 만들면 UI-9 이후 번호를 쓴다
- [ ] 라이선스 제한으로 기사 본문을 못 보일 때의 표시. 지금은 제목·출처·링크만 두는 것으로 적었다([[TBL-UC-002#UC-H3]] 3a)
- [x] A 리포트의 내려받기 파일 형식. 서버가 만드는 PDF로 정했다(유저 결정 2026-09-30)
- [ ] A 리포트 트래킹 지표 셋의 현업 확정. 확정 전까지 임시 지정 표시를 유지한다([[TBL-PRD-002#R1]])
- [ ] 신호등 임계값을 화면에 노출할지. 지금은 근접도 값만 보이고 기준선은 보이지 않는다
- [ ] 알림 채널이 정해지면 [[#UI-1]]에 알림 표시 자리가 필요한지([[TBL-PRD-002#R24]])
- [ ] 시안 파일의 카드 라벨과 이 문서의 요소 이름이 글자 단위로 같은지 재확인. 시안이 정본이다(0.1절)
