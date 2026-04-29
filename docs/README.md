
# SelectPaper

SelectPaper는 입력 텍스트를 분석해 출판물 레퍼런스와 매칭하고, LaTeX 기반 편집 레이아웃을 생성하는 AI 편집 디자인 시스템입니다.

텍스트의 장르, 밀도, 읽기 방식, 편집 구조를 바탕으로 적합한 출판물 스타일을 추천하고, 판형·여백·단 구성·서체·행간·각주 처리 등을 반영한 조판 코드를 생성하는 것을 목표로 합니다.

---

## Overview

SelectPaper is an AI-assisted editorial layout generation system.

It analyzes structured input text and matches it with a publication design database, then generates LaTeX-based editorial layouts for further use in Overleaf or print-oriented workflows.

The project evolved from a simple AI layout generator into a constraint-based editorial design system.

---

## Key Features

- 텍스트 기반 출판물 레퍼런스 매칭
- 장르별 편집 디자인 DB 기반 스타일 추천
- 판형, 여백, 단 구성, 서체, 행간, 자간 값 자동 반영
- LaTeX 코드 생성
- Overleaf 복붙용 출력 구조
- 고정 단 / 가변 단 레이아웃 지원
- 모듈 그리드 및 열 구조 해석
- 각주 자동 파싱
- 한글 조판 문제 대응
- 토큰 사용량 최적화
- 실시간 생성 진행 상태 표시

---

## Project Structure

```text
SelectPaper/
├── README.md
├── CHANGELOG.md
├── docs/
│   └── dev-log.md
├── versions/
│   ├── v1/
│   │   └── SelectPaper.jsx
│   ├── v2/
│   │   └── SelectPaper.jsx
│   ├── ...
│   └── v16/
│       └── SelectPaper.jsx
└── core/
    └── SelectPaper.jsx
