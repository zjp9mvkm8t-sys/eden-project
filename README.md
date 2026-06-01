# 에덴 기숙학원의 비밀
### ARG × 방탈출 웹게임 | 디지털 스토리텔링 실습 프로젝트

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-배포중-5546CC?style=flat-square)](https://zjp9mvkm8t-sys.github.io/eden-project/)

---

## 🌐 대시보드 바로가기
**→ [프로젝트 대시보드 보기](https://zjp9mvkm8t-sys.github.io/eden-project/)**

---

## 📖 프로젝트 개요

한성대학교 디지털 스토리텔링 실습 Chapter 10·11·12 학습 내용을 실제 게임으로 구현한 프로젝트입니다.

- **Chapter 10** — ARG(대체현실게임) Rabbit Hole 설계
- **Chapter 11** — 방탈출 스토리 구성, 불일치 해소 원리
- **Chapter 12** — 방탈출 기본 문제 유형 7가지 적용

---

## 🗂 파일 구조

```
eden-project/
├── index.html              # 프로젝트 대시보드 (GitHub Pages 메인)
├── eden_main.html          # 게임 메인페이지 (타이틀 + 시나리오 선택)
├── eden_academy_official.html  # ARG 위장 홈페이지
├── INSTRUCTION.md          # 프로젝트 지침서
├── SKILL.md               # 구현 기술 가이드
├── decision.log           # 전체 결정 사항 로그
└── README.md              # 이 파일
```

---

## 🎮 시나리오

| 시나리오 | 제목 | 장르 | 퀘스트 |
|---------|------|------|--------|
| A | 에덴의 가면 | 심리 스릴러 × 바이오호러 | 11개 |
| B | 무균실의 기억 | SF 디스토피아 × 탈출 미스터리 | 7개 |
| C | 선택받은 자들 | 도덕 공포 × 사회 비판 스릴러 | 6개 |

---

## 🐇 ARG Rabbit Hole

| 장치 | 채널 | 단서 |
|------|------|------|
| RH-01 | SNS 실종 제보 | 마지막 카톡 |
| RH-02 | 홈페이지 소스코드 | PID_0047 / LAB_B3 |
| RH-03 | 유튜브 모스부호 | SOS+HELP+0047 |
| RH-04 | 전단지 UV 잉크 | 거칠어 보이는 자를 믿어라 |
| RH-05 | 에브리타임 DM | 자물쇠 비번 5130 |

---

## ⚙️ 개발 현황

- [x] Phase 0 — 사전 기획 전체
- [x] Phase 1 — 메인페이지 프레임워크
- [ ] Phase 2 — 게임 화면 + Stage 1 퀘스트
- [ ] Phase 3 — Stage 2 (카이사르 암호 포함)
- [ ] Phase 4 — Stage 3 (논리 퀴즈 + 분기)
- [ ] Phase 5 — 엔딩 시스템 + 통합 테스트

---

## 🔑 핵심 설계 원칙

- **TINAG** (This Is Not A Game) — ARG 현실 위장
- **불일치 해소 원리** — 2겹 반전 (수인·아란)
- **Flow 이론** — Stage별 난이도 점진 상승
- **카이사르 암호 이동 칸수 = 4** (EDEN → IHIR)
