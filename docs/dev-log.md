# SelectPaper Development Log

이 문서는 SelectPaper의 개발 과정을 버전별로 기록한 개발 리포트입니다.

SelectPaper는 입력 텍스트를 분석해 적합한 출판물 레퍼런스를 매칭하고, 해당 레퍼런스의 판형·여백·단 구성·서체·행간·각주 체계를 반영해 LaTeX 기반 편집 레이아웃을 생성하는 AI 편집 디자인 시스템입니다.

---

# v1 — Initial Optimization

**Date**: 2026.04.27  
**Version**: SelectPaper optimized  
**Focus**: 토큰 최적화 및 중복 코드 제거

---

## Goal

초기 SelectPaper 코드에서 불필요하게 토큰을 많이 사용하는 구조를 찾고, 기능 저하 없이 중복 작업과 미사용 코드를 제거한다.

---

## Design Shift

기존 구조는 기능 실험 과정에서 남은 함수와 데이터가 많았고, 실제 UI에서 사용하지 않는 로직도 코드에 남아 있었다.

v1에서는 기능을 확장하기보다, 이후 개발을 안정적으로 이어가기 위한 정리 작업을 우선했다.

---

## Changes

- 사용되지 않는 `profileText()` 함수 제거
- 사용되지 않는 `generate()` 함수 제거
- 사용되지 않는 `textProfile` state 제거
- 불필요한 state 초기화 타이포 제거
- `rationaleCache`를 이용한 설명 생성 캐싱 적용
- refine 입력에서 LaTeX 주석과 빈 줄 압축
- refine 대화 히스토리를 최근 항목 중심으로 제한

---

## Problem / Solution

| Problem | Cause | Solution |
|---|---|---|
| API 호출마다 토큰 사용량이 큼 | 전체 텍스트와 긴 LaTeX 코드가 반복 전달됨 | 본문 truncate, refine 압축, 캐싱 적용 |
| 사용하지 않는 함수가 남아 있음 | 초기 실험 코드가 유지됨 | dead code 제거 |
| 반복 생성 시 같은 설명을 다시 요청 | 캐싱 없음 | rationale cache 추가 |

---

## Impact

- 코드 길이 감소
- API 호출 비용 감소
- 이후 구조 개편을 위한 코드 기반 정리
- 기능은 유지하면서 불필요한 연산과 토큰 사용 제거

---

# v2 — Structured Input UI

**Date**: 2026.04.27  
**Version**: SelectPaper v2  
**Focus**: 입력 구조 개편

---

## Goal

단일 본문 입력 중심 구조를 제목, 소제목, 본문, 면주, 각주, 미주 등 실제 편집 요소에 가까운 입력 구조로 전환한다.

---

## Design Shift

기존 입력 방식은 “긴 텍스트 하나”를 중심으로 동작했다.  
v2에서는 텍스트를 편집 요소 단위로 분리해 AI가 각 요소의 역할을 이해할 수 있도록 구조화했다.

---

## Changes

- 입력 필드를 6개로 분리:
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

---

## Before / After

### Before

```text
Single text input + sample text selector
