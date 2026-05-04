# SelectPaper Development Log

이 문서는 SelectPaper의 개발 과정을 버전별로 기록한 개발 리포트입니다.

SelectPaper는 입력 텍스트를 분석해 적합한 출판물 레퍼런스를 매칭하고, 해당 레퍼런스의 판형, 여백, 단 구성, 서체, 행간, 각주 체계를 반영해 LaTeX 기반 편집 레이아웃을 생성하는 AI 편집 디자인 시스템입니다.

---

## v1 — Initial Optimization

**Date**: 2026.04.27  
**Version**: SelectPaper optimized  
**Focus**: 토큰 최적화 및 중복 코드 제거

### Goal

초기 SelectPaper 코드에서 불필요하게 토큰을 많이 사용하는 구조를 찾고, 기능 저하 없이 중복 작업과 미사용 코드를 제거한다.

### Design Shift

기존 구조는 기능 실험 과정에서 남은 함수와 데이터가 많았고, 실제 UI에서 사용하지 않는 로직도 코드에 남아 있었다.

v1에서는 기능을 확장하기보다, 이후 개발을 안정적으로 이어가기 위한 정리 작업을 우선했다.

### Changes

- 사용되지 않는 `profileText()` 함수 제거
- 사용되지 않는 `generate()` 함수 제거
- 사용되지 않는 `textProfile` state 제거
- 불필요한 state 초기화 타이포 제거
- `rationaleCache`를 이용한 설명 생성 캐싱 적용
- refine 입력에서 LaTeX 주석과 빈 줄 압축
- refine 대화 히스토리를 최근 항목 중심으로 제한

### Problem / Solution

| Problem | Cause | Solution |
|---|---|---|
| API 호출마다 토큰 사용량이 큼 | 전체 텍스트와 긴 LaTeX 코드가 반복 전달됨 | 본문 truncate, refine 압축, 캐싱 적용 |
| 사용하지 않는 함수가 남아 있음 | 초기 실험 코드가 유지됨 | dead code 제거 |
| 반복 생성 시 같은 설명을 다시 요청 | 캐싱 없음 | rationale cache 추가 |

### Impact

- 코드 길이 감소
- API 호출 비용 감소
- 이후 구조 개편을 위한 코드 기반 정리
- 기능은 유지하면서 불필요한 연산과 토큰 사용 제거

---

## v2 — Structured Input UI

**Date**: 2026.04.27  
**Version**: SelectPaper v2  
**Focus**: 입력 구조 개편

### Goal

단일 본문 입력 중심 구조를 제목, 소제목, 본문, 면주, 각주, 미주 등 실제 편집 요소에 가까운 입력 구조로 전환한다.

### Design Shift

기존 입력 방식은 “긴 텍스트 하나”를 중심으로 동작했다.

v2에서는 텍스트를 편집 요소 단위로 분리해 AI가 각 요소의 역할을 이해할 수 있도록 구조화했다.

### Changes

- 입력 필드를 6개로 분리
  - 제목
  - 소제목
  - 본문
  - 면주
  - 각주
  - 미주
- “Genre Hint”를 “Genre”로 변경
- 장르 옵션을 한국어 중심으로 정리
- 샘플 텍스트 선택 UI 제거
- 대량의 `SAMPLES` 상수 제거
- LaTeX 프롬프트에 구조화된 입력 블록 전달

### Before / After

**Before**

Single text input + sample text selector

**After**

TITLE  
SUBTITLE  
BODY  
RUNNING HEAD  
FOOTNOTE SAMPLE  
ENDNOTE SAMPLE

### Impact

- AI가 입력 텍스트의 역할을 더 명확히 이해할 수 있음
- 편집 구조에 가까운 입력 방식 확보
- 샘플 텍스트 의존성 제거


---

## v3 — Korean Typesetting Fix

**Date**: 2026.04.27  
**Version**: SelectPaper v3  
**Focus**: 한글 렌더링 오류 수정

### Goal

LaTeX 출력에서 한글이 자소 단위로 깨지거나 비정상적으로 벌어지는 문제를 해결한다.

### Problem

생성된 결과에서 한글 음절이 정상적으로 유지되지 않고 낱자 단위처럼 분해되어 보이는 문제가 발생했다.

### Cause

`microtype`의 `letterspace` 기능이 한글 조판에 적합하지 않았다.

한글 음절 단위가 아니라 문자 구성 요소 단위로 간격을 처리하면서 출력이 깨졌다.

### Changes

- `\usepackage{microtype}` 제거
- `\microtypesetup{letterspace=...}` 사용 금지
- `fontspec`의 `LetterSpace` 옵션 사용 방식으로 전환
- 프롬프트에 `microtype`, `textls` 사용 금지 규칙 추가

### Impact

- 한글 음절이 깨지는 문제 완화
- 한글 조판에 더 적합한 자간 처리 방식 적용
- 이후 모든 버전에서 한글 출력 안정성의 기반 마련

---

## v4 — Module Grid & Column Logic Redesign

**Date**: 2026.04.27  
**Version**: SelectPaper v4  
**Focus**: 단 구성 및 모듈 그리드 해석

### Goal

DB에 기록된 “10단”, “11단”, “12열”과 같은 값을 실제 조판에서 그대로 다단 텍스트로 해석하지 않고, 편집 디자인의 모듈 그리드로 이해하도록 개선한다.

### Problem

기존 코드는 `c.구성`에서 숫자만 추출해 그대로 `multicols` 값으로 사용했다.

예를 들어:

11단 → `\begin{multicols}{11}`

이런 방식은 실제 편집 디자인 관례와 맞지 않는다.

### Design Shift

단순 숫자 해석에서 벗어나, DB의 `layout_type`과 `c.구성`을 함께 읽어 실제 본문 영역과 주석 영역을 계산하는 방식으로 전환했다.

### Changes

- 단순 다단과 모듈 그리드 구분
- 5단 이하: 일반 다단으로 해석
- 6단 이상 또는 “열” 표기: 모듈 그리드로 해석
- `layout_type` 파싱 적용
- 본문 영역과 주석 영역의 mm 단위 폭 계산
- `paracol` 사용 지시 추가

### Before / After

**Before**

10단 → 실제 10단 텍스트

**After**

10단 → 모듈 그리드  
본문 6~8단 폭 + 주석 2~4단 폭 등으로 병합 사용

### Impact

- DB 데이터의 의미를 더 정확히 반영
- 실제 출판물의 모듈 그리드 관례에 가까워짐
- 본문과 주석 영역 분리가 가능해짐
- 고정 다단보다 더 유연한 조판 구조 확보

---

## v5 — Footnote Layout Improvement

**Date**: 2026.04.27  
**Version**: SelectPaper v5  
**Focus**: 각주 레이아웃 개선

### Goal

각주가 본문 다단 구조와 어긋나거나 불필요한 구분선을 생성하는 문제를 해결한다.

### Problems

- 각주 위에 원치 않는 선이 생성됨
- 본문이 다단이어도 각주는 전폭 또는 1단처럼 처리됨
- 각주 크기가 DB 값과 정확히 연결되지 않음

### Changes

- footnote configuration block 생성
- DB의 `footnote` 값을 읽어 각주 크기와 행간 계산
- 기본 `\footnoterule` 제거
- 각주가 컬럼 구조를 고려하도록 프롬프트 강화

### Impact

- 각주가 본문 구조와 더 자연스럽게 연결됨
- 기본 LaTeX 각주 선 제거
- DB 기반 각주 스타일 반영 가능

---

## v6 — Divider Line Removal

**Date**: 2026.04.27  
**Version**: SelectPaper v6  
**Focus**: 불필요한 구분선 제거

### Goal

출력물에 나타나는 모든 불필요한 선을 제거한다.

### Problem

LaTeX 기본 설정이나 AI 생성 코드에 의해 다음과 같은 선이 나타났다.

- 각주 구분선
- 헤더 라인
- 푸터 라인
- 임의의 `\hrule`, `\rule`

### Changes

- `\footnoterule` 무조건 제거
- `\headrulewidth=0pt`
- `\footrulewidth=0pt`
- `\hrule`, `\rule` 삽입 금지 규칙 추가

### Impact

- 불필요한 그래픽 요소 제거
- 사용자가 의도하지 않은 시각적 선이 출력되지 않도록 제어
- 레이아웃 결과의 예측 가능성 증가

---

## v7 — Unit Artifact Removal

**Date**: 2026.04.27  
**Version**: SelectPaper v7  
**Focus**: 단위 문자열 출력 오류 수정

### Goal

`1em`, `4mm` 같은 LaTeX 단위 문자열이 지면에 그대로 출력되는 문제를 해결한다.

### Problem

AI가 LaTeX 설정 명령을 본문 영역에 잘못 배치하면서 단위 문자열이 실제 텍스트처럼 출력되었다.

### Changes

- `1em` 기반 설정 제거
- footnote margin 관련 위험 설정 제거
- 단위 문자열을 visible text로 출력하지 말라는 규칙 추가

### Impact

- LaTeX 명령 일부가 본문 텍스트로 흘러나오는 문제 감소
- 단위 텍스트가 페이지에 찍히는 현상 완화
- 다음 버전의 preamble/body 분리 필요성 확인

---

## v8 — LaTeX Preamble Separation

**Date**: 2026.04.27  
**Version**: SelectPaper v8  
**Focus**: LaTeX 생성 구조 재설계

### Goal

AI가 프리앰블 명령과 본문 내용을 혼동하지 않도록 LaTeX 생성 구조를 분리한다.

### Design Shift

기존에는 AI가 전체 LaTeX 문서를 생성했다.

AI generates preamble + document body

v8에서는 JS가 프리앰블을 직접 생성하고, AI는 본문만 생성하도록 바꿨다.

JS generates preamble  
AI generates document body only  
JS combines both

### Changes

- 프리앰블을 JS 코드에서 직접 생성
- AI에게는 `\begin{document}` 이후 본문만 생성하도록 요청
- 후처리에서 `\begin{document}` 이전 텍스트 제거
- 최종 출력은 `preamble + bodyOnly` 방식으로 조립

### Impact

- LaTeX 설정 명령이 본문에 노출되는 문제를 구조적으로 차단
- AI 출력 범위를 줄여 안정성 향상
- 생성 결과의 예측 가능성 증가

---

## v9 — Footnote Automation & Endnote Removal

**Date**: 2026.04.27  
**Version**: SelectPaper v9  
**Focus**: 각주 자동 처리 및 미주 제거

### Goal

사용자가 각주와 미주를 별도 필드로 관리하지 않아도, 본문 안의 번호 표기를 자동으로 각주로 변환한다.

### Changes

- 미주 입력 필드 제거
- 각주 자동 파싱 함수 추가
- 지원 마커:
  - `¹`, `²`, `³`
  - `[1]`, `[2]`
  - `^1`, `^2`
- 본문 생성 시 `injectFootnotes` 적용
- invalid footnote margin 설정 제거

### Before / After

**Before**

본문 입력 + 각주 입력 + 미주 입력

**After**

본문 안의 번호 표기 + 각주 목록  
→ 자동으로 `\footnote{}` 변환

### Impact

- 사용자 입력 부담 감소
- 실제 원고 작성 방식에 가까워짐
- 미주 필드 제거로 인터페이스 단순화

---

## v10 — Line Spacing System Fix

**Date**: 2026.04.27  
**Version**: SelectPaper v10  
**Focus**: 행간 계산 방식 수정

### Goal

DB에 입력된 행간 값이 실제 LaTeX 출력에 정확히 반영되도록 한다.

### Problem

기존에는 다음 방식으로 행간을 계산했다.

`const linespread = p.b.행간 / p.b.크기;`

하지만 LaTeX의 `\linespread`는 폰트 크기에 직접 곱해지는 값이 아니라, 기본 `baselineskip`에 다시 곱해지는 값이다.

결과적으로 실제 행간이 DB 값보다 크게 출력되었다.

### Changes

- `\linespread` 제거
- `\fontsize{fontSize}{leading}\selectfont` 방식으로 변경
- refine 프롬프트의 행간 가이드 수정
- diff parser의 행간 해석 업데이트

### Impact

- DB의 행간 값이 더 정확히 반영됨
- 과도하게 벌어진 행간 문제 해결
- 조판 수치의 신뢰성 향상

---

## v11 — Initialization Error Fix

**Date**: 2026.04.27  
**Version**: SelectPaper v10 fix  
**Focus**: 변수 선언 순서 오류 수정

### Goal

각주 관련 refactor 이후 발생한 초기화 순서 오류를 해결한다.

### Problem

다음 오류가 발생했다.

`Cannot access 'hasFootnote' before initialization`

### Cause

`hasFootnote` 변수가 선언되기 전에 `footnoteBlock`에서 먼저 참조되었다.

### Changes

- `hasFootnote`, `fnSize`, `fnLeading` 선언 위치를 앞으로 이동
- 중복 선언 제거

### Impact

- 런타임 오류 해결
- 각주 처리 로직 안정화

---

## v12 — Long Text Input Expansion

**Date**: 2026.04.27  
**Version**: SelectPaper v11  
**Focus**: 긴 본문 입력 지원

### Goal

사용자가 본문 10장 정도의 긴 텍스트를 넣을 수 있도록 입력 및 생성 제한을 확장한다.

### Problems

- 본문이 `slice(0, 800)`으로 잘림
- 출력 token 한도가 낮음
- 타임아웃이 짧음
- textarea가 긴 입력에 불편함

### Changes

- 본문 전송량 제한 제거
- max token 한도 증가
- timeout 확장
- 본문 textarea rows 증가
- 글자 수와 예상 페이지 수 표시 추가

### Impact

- 장문 텍스트 입력 가능
- 긴 LaTeX 결과 생성 가능
- 사용자 입력 경험 개선

---

## v13 — Style Tab & Column Mode Controls

**Date**: 2026.04.27  
**Version**: SelectPaper v12  
**Focus**: 스타일 지시 UI 및 단 구성 제어

### Goal

사용자가 입력 탭과 별도로 스타일 지시를 설정할 수 있도록 인터페이스를 분리하고, 고정 단과 가변 단을 선택할 수 있게 한다.

### Changes

- Step 0 UI를 탭 구조로 변경
- 텍스트 입력 탭 추가
- 스타일 지시 탭 추가
- 단 구성 버튼 추가:
  - 데이터 자동
  - 1단
  - 2단
  - 3단
  - 4단
  - 가변
- 추가 스타일 지시 입력창 추가
- 열 표기는 무조건 모듈 그리드로 해석하도록 수정

### Design Note

“열”은 실제 본문을 열 개수만큼 쪼개는 의미가 아니라, 판면을 구성하는 모듈 그리드로 해석해야 한다.

따라서 12열은 12개의 본문 컬럼이 아니라 6/6, 8/4, 9/3 등으로 병합 가능한 구조로 처리한다.

### Impact

- 사용자 제어 가능성 증가
- 자동 추천과 수동 스타일 지시 병행 가능
- 단 구성 실험을 인터페이스에서 직접 수행 가능

---

## v14 — AI-Based Variable Layout Redesign

**Date**: 2026.04.28  
**Version**: SelectPaper v13  
**Focus**: 가변 레이아웃 개념 재정의

### Goal

가변 레이아웃을 사용자가 직접 섹션별로 지정하는 방식이 아니라, AI가 본문 내용을 이해하고 편집적으로 판단하는 방식으로 재설계한다.

### Problem

이전 스타일 탭의 가변 레이아웃은 사용자가 섹션을 직접 추가하고 단 수를 지정하는 구조였다.

하지만 의도한 가변 레이아웃은 본문 전체 안에서 AI가 헤드라인, 본문, 인용구, 강조 문장을 판단해 단 배리에이션을 주는 방식이었다.

### Changes

- 섹션 빌더 제거
- `styleConfig.sections` 제거
- variable mode를 AI 콘텐츠 분석 기반으로 변경
- DB-native variable layout에도 같은 방식 적용
- 스타일 탭 설명 문구 업데이트

### Variable Layout Rules

- 본문 전체를 먼저 읽고 구조를 파악한다.
- 제목, 헤드라인, 섹션 오프너는 전폭 1단 사용 가능.
- 본문 단락은 DB 기본 단 수를 따른다.
- 인용구, 강조 문장, 전환부는 2단 또는 전폭으로 조정 가능.
- 단 전환은 문장 중간이 아니라 문단 단위로 이루어져야 한다.
- 배리에이션은 편집적으로 의도된 것처럼 보여야 한다.

### Impact

- 가변 레이아웃이 수동 UI 기능이 아니라 AI 편집 판단 기능으로 재정의됨
- 사용자의 실제 의도에 더 가까운 구조가 됨
- 편집 자동화 시스템의 방향성이 더 명확해짐

---

## v15 — Real-Time Progress Panel & Token Review

**Date**: 2026.04.28  
**Version**: SelectPaper v14  
**Focus**: 출력 진행 상태 표시 및 토큰 낭비 제거

### Goal

AI가 결과물을 생성하는 동안 현재 어떤 작업을 수행 중인지 사용자에게 보여주고, 동시에 추가적인 토큰 낭비 구조를 점검한다.

### Changes

- `runLog` state 추가
- 단계별 진행 상태 표시
- 기존 로딩 UI를 실시간 진행 패널로 교체
- `generateRationale(p, cleanLatex)`에서 사용하지 않는 `cleanLatex` 파라미터 제거
- Regenerate 시 `runLog` 초기화

### Progress Steps

| Step | Status |
|---|---|
| Keyword Matching | 출판물 후보 매칭 |
| LaTeX Generation | 조판 코드 생성 |
| Semantic Matching | 의미 기반 매칭 |
| Layout Rationale | 레이아웃 설명 생성 |

### Impact

- 사용자가 생성 과정을 이해할 수 있음
- 불필요한 LaTeX 코드 전달 제거
- 약 2,000~4,000 tokens 절감
- 장시간 생성 시 사용자 불안 감소

---

## v16 — Output Stability & Page-Size Control

**Date**: 2026.04.28  
**Version**: SelectPaper v15 / v16  
**Focus**: 출력 안정화 및 판형 제어

### Goal

AI 출력에 불필요한 텍스트가 섞이지 않도록 하고, 선택된 출판물의 판형이 실제 PDF 출력에 정확히 반영되도록 한다.

### Problems

- LaTeX 코드에 “Hmm”, “Actually” 같은 AI 응답 텍스트가 섞임
- 선택된 판형이 아니라 A4 크기로 출력됨
- refine 단계에서 판형을 유지하지 않고 본문만 줄이는 방식으로 수정됨

### Causes

| Problem | Cause |
|---|---|
| 비LaTeX 텍스트 출력 | system prompt 부재 |
| A4 출력 | memoir 클래스의 기본 stock size |
| refine 판형 변경 | 판형/여백 고정 규칙 부재 |

### Changes

- LaTeX 생성 API에 system prompt 추가
- refine API에 system prompt 추가
- `\begin{document}` 이전 텍스트 제거 후처리 강화
- memoir preamble에 `\setstocksize` 추가
- `\settrimmedsize` 추가
- refine 프롬프트에 IMMUTABLE 규칙 추가
- 판형과 여백을 절대 변경하지 않도록 명시

### Before / After

**Before**

AI output may contain:  
Hmm, actually...  
`\documentclass...`

**After**

Output must contain only valid LaTeX code.

**Before**

Selected size: 105 × 150 mm  
Actual output: A4-like page

**After**

Selected size: 105 × 150 mm  
Actual output: 105 × 150 mm

### Impact

- 출력 안정성 향상
- Overleaf 복붙 가능성 증가
- 판형 제어 정확도 향상
- AI 생성물이 더 예측 가능한 코드 출력으로 전환됨

---

## Current Open Issues

- 가변 레이아웃의 편집 판단 정확도 추가 개선 필요
- 긴 텍스트에서 페이지별 리듬 조절 필요
- 이미지/도판이 포함된 편집 구조 지원 필요
- DB 분리 및 유지보수성 개선 필요
- 실제 PDF 컴파일 테스트 자동화 필요

---

## Next Goals

1. DB를 별도 파일로 분리하기
2. 최신 코드 기준으로 `core/` 정리하기
3. 레이아웃 테스트 샘플 추가하기
4. README에 사용 방법 추가하기
5. v16 이후 안정 버전을 기준으로 v17 개발 시작하기
- 
- 실제 사용자 원고 입력에 가까운 구조로 전환

---


---

## v17 — Margin Control Fix & Layout Behavior Analysis

**Date**: 2026.04.30  
**Version**: SelectPaper v17  
**Focus**: 상단 여백 제어 및 레이아웃 파라미터 구조 분석

### Goal

페이지 상단 여백이 과도하게 발생하는 문제를 해결하고,  
AI가 여백 수정 요청을 제대로 반영하지 못하는 구조적 원인을 분석 및 개선한다.

---

### Design Shift

기존 시스템에서는 판형 안정성을 유지하기 위해  
페이지 크기와 여백을 모두 IMMUTABLE로 고정했다.

하지만 이 구조는 사용자 요청(예: "상단 여백 줄여줘")을  
시스템적으로 무시하게 만드는 문제를 발생시켰다.

v17에서는 다음과 같이 제어 구조를 재정의했다:

- 판형 (paper size): 고정
- 여백 (margin): 수정 가능
- 콘텐츠 간격 (spacing): 수정 가능

---

### Changes

- refine 프롬프트에서 margin 관련 IMMUTABLE 규칙 제거
- page size와 margin을 분리된 제어 요소로 재구성
- 상단 여백을 `geometry`의 `top` 값으로 직접 제어 가능하도록 변경
- 본문 시작 위치를 `\vspace*{}`로 별도 제어 가능하게 유지
- layout spacing 관련 프롬프트 가이드 수정

---

### Problem / Solution

| Problem | Cause | Solution |
|---|---|---|
| 상단 여백이 과도하게 크게 생성됨 | `top=67mm` 고정값 사용 | `top` 값을 직접 수정 가능하도록 변경 |
| 채팅에서 여백 수정 요청이 반영되지 않음 | margin이 IMMUTABLE로 잠겨 있음 | margin 제어를 unlock |
| AI가 여백 대신 다른 값을 수정함 | `top` 변경이 불가능한 상태 | spacing과 margin 역할 분리 |
| 헤더 위치가 비정상적으로 아래에 배치됨 | `includehead=true` + large top margin | top margin 감소로 해결 |

---

### System Analysis

이 문제는 단순한 프롬프트 이해 문제가 아니라,  
시스템 제약으로 인해 발생한 구조적 오류였다.

기존 구조:

    [IMMUTABLE]
    - page size
    - margin

이 상태에서는 AI가 어떤 요청을 받아도:

- top margin 수정 불가
- 대신 headsep / vspace 등 다른 값 수정

즉, 잘못된 파라미터를 건드리는 오류가 발생했다.

---

### Layout Logic Clarification

현재 레이아웃 구조:

    top margin (geometry)
    ↓
    header (includehead)
    ↓
    body start (\vspace*)
    ↓
    main text

따라서:

- 상단 여백 = `top`
- 헤더-본문 간격 = `\vspace*{}`
- headsep은 거의 영향 없음 (현재 구조 기준)

---

### Impact

- 사용자 여백 제어 요청이 정상적으로 반영됨
- 레이아웃 파라미터의 역할이 명확히 분리됨
- AI 출력의 예측 가능성 증가
- "말은 이해하는데 결과가 이상한" 문제 해결

---

### Notes

- 이전 버전(v16)의 안정성 확보 과정에서 margin까지 잠긴 것이 문제의 원인
- v17은 편집 제어 가능성 복구 버전
- 이후 단계에서는 UI에서 margin 조정 기능을 제공하는 것이 필요함

- ---

## v18 — Typography Base Integration

**Date**: 2026.04.30  
**Version**: SelectPaper v18  
**Focus**: 타이포그래피 기초 규칙 시스템 내장

### Goal

AI가 기본적인 타이포그래피 원칙을 무시하지 않도록  
최소한의 기초 규칙을 시스템 레벨에서 강제한다.

---

### Design Shift

기존에는 LaTeX 생성 시 AI가 타이포그래피 판단을 직접 수행했다.

v18에서는 이를 변경하여:

- 타이포그래피 기본 규칙을 JS 상수(TYPO_BASE)로 내장
- AI는 그 위에서만 생성하도록 제한

---

### Changes

- TYPO_BASE 상수 추가 (행간, 면주 크기, 가변단 규칙 등)
- 글자 크기별 행간 자동 계산 함수 적용
- 판형 대비 면주 크기 자동 계산
- 구분선 제거 규칙 강화
- 텍스트 박스 사용 규칙 추가 (중앙 정렬 필수)
- 가변단 조합 (5단 → 3+2 / 4+1 등) 지원

---

### Impact

- 기본 타이포 오류 감소
- 슬기와민 스타일의 “기초 안정성” 확보
- 토큰 사용 없이 품질 유지 (rule hardcoding)

---

## v19 — Semantic Matching Pipeline

**Date**: 2026.04.30  
**Version**: SelectPaper v19  
**Focus**: 의미 기반 추천 시스템 구축

### Goal

단순 키워드 매칭이 아니라  
텍스트 의미와 디자인 의도를 기반으로 레퍼런스를 선택한다.

---

### Design Shift

기존:

    keyword → top1 → LaTeX → semantic (후처리)

v19:

    analyzeText → keyword top20 → semantic rerank → LaTeX

---

### Changes

- analyzeText() 추가 (텍스트 구조 8개 항목 추출)
- semanticRerank() 추가 (top20 → 최종 1)
- scoreKw 개선 (summary, layout, why_* 반영)
- structuredReason 출력 구조 추가
- pipeline 재구성 (semantic을 foreground로 이동)

---

### Problem

- semantic rerank 이후 LaTeX 재생성 없음 → drift 발생
- 키워드 seed bias 여전히 존재

---

### Impact

- recommendation과 결과물 연결 강화
- 시스템이 “디자인 추천 시스템”으로 정의됨

---

## v20 — Pipeline Stabilization & Data Integration

**Date**: 2026.04.30  
**Version**: SelectPaper v20  
**Focus**: DB 기반 추천 정확도 향상

---

### Goal

Google Sheet 기반 디자인 DB의 “의도 데이터”를  
추천 로직에 직접 반영한다.

---

### Design Shift

기존:

- 장르 / 키워드 중심 매칭

v20:

- 내용 요약 / 여백 의도 / 레이아웃 특징 중심 매칭

---

### Changes

- scoreKw → multi-field scoring 구조
- semanticRerank에 디자인 의도 필드 포함
- 후보 선택 구조 변경:

    top20 → semantic → top4 → final

- structuredReason 상세화
- textProfile 기반 보정 적용

---

### Impact

- 디자인 “왜”를 반영한 추천 가능
- DB 활용도가 크게 증가

---

## v21 — Layout Stability & Token Optimization

**Date**: 2026.05.01  
**Version**: SelectPaper v21  
**Focus**: 레이아웃 안정화 및 토큰 절감

---

### Goal

조판 오류(헤더 잘림, 줄나눔 문제)를 해결하고  
프롬프트 비용을 줄인다.

---

### Changes

#### Layout Fix

- includehead=true 적용
- \headheight 명시
- \headsep 고정

#### Line-breaking Fix

- \tolerance=9999 제거 → 400 적용
- widow/orphan penalty 추가
- emergencystretch fallback 유지

#### Token Optimization

- latexPrompt 구조 압축
- preambleSummary 축소
- analyzeText 스키마 압축
- verbatim 규칙 제거

→ 약 **300 토큰/회 절감**

---

### Impact

- 레이아웃 안정성 확보
- 비용 효율 개선
- 출력 예측 가능성 증가

---

## v22 — Korean Typesetting Engine & Style Composition

**Date**: 2026.05.01  
**Version**: SelectPaper v22  
**Focus**: 한글 조판 엔진 개선 + 스타일 합성 구조

---

### Goal

한글 조판 품질을 근본적으로 개선하고  
단일 레퍼런스 복사 구조를 탈피한다.

---

### Design Shift

기존:

    single reference → 전체 스타일 복사

v22:

    top4 references → 속성별 선택 → styleSpec 구성

---

### Changes

#### Korean Typesetting

- HANGUL_LINEBREAK_SKIP 도입
- \sloppy / \tolerance=9999 완전 제거
- hyphenation 완전 차단
- cleanKoreanHyphenation() 후처리 추가
- chooseAlignmentPolicy() → ragged fallback

#### Layout Quality

- line-level orphan 완화
- preview에서도 keep-all 적용

#### Style Composition

- semanticRerank → top4 반환
- composeStyleSpec() 추가
- 속성별 스타일 선택 구조 도입
    - page / margin / column / typography / folio 분리
- confidence 기반 fallback

#### Data Audit

- DB source: 하드코딩 :contentReference[oaicite:0]{index=0}  
- DB count: 254
- 슬기와민 관련 데이터 다수 포함
- Google Sheet 미연동 상태

#### Token Optimization

- 중복 규칙 제거
- 후보 정보 압축
- promptGuard 최소화

→ 약 **350 토큰/회 절감**

---

### Impact

- 한글 조판 품질 대폭 개선
- 스타일 “클론 문제” 해결
- 시스템이 단순 추천 → **편집 엔진**으로 전환

---

## Overall Evolution (v18–v22)

```
TYPO BASE (v18)
→ SEMANTIC SYSTEM (v19)
→ DATA-DRIVEN MATCHING (v20)
→ LAYOUT STABILITY (v21)
→ KOREAN TYPE ENGINE + STYLE COMPOSITION (v22)
```

---

### 핵심 변화 요약

- 규칙 → 의미 → 데이터 → 안정성 → 조판 엔진
- “예쁜 결과 생성” → “편집 디자인 시스템”

