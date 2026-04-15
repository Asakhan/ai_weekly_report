# 주간보고서 자동화 — 슈퍼컴퍼니 서비스개발1팀

## 프로젝트 개요
슈퍼컴퍼니 서비스개발1팀의 주간 업무 보고서를 자동으로 작성하고 팀장에게 발송하는 프로젝트.

---

## 팀 정보
- 회사명: 슈퍼컴퍼니
- 팀명: 서비스개발1팀
- 팀장 (보고 수신자): Thomas Moon <thomas@itengineers.net>
- 보고자: Daniel Kim <daniel@itengineers.net>
- 보고 주기: 매주 금요일 오전
- 어투: 격식체

---

## 소스 데이터 위치
- STT 텍스트 파일 (회의 녹음 변환본): C:\Users\Administrator\Documents\회의녹음\
- 회의록 (Google Drive): https://drive.google.com/drive/folders/1G8Ok9vDdaFJc6rU6Hp8HkwtrF_bjiy70
- 업무 결과물 (Google Drive): https://drive.google.com/drive/folders/19NwSJBilPaRL2G0O_h-8oRaBcbkzidQS
- 업무 관련 메일: Gmail (최근 1주일 수발신 메일)
- 고객사 문의 메일: Gmail 수신함 (외부 도메인 발신 미답변 메일)
- 경쟁사 동향: Brave Search (웹 검색)

---

## 보고서 양식 & 저장
- 양식 파일: C:\Users\Administrator\Documents\ppt_templates\주간보고서.pptx
- IMPORTANT: 양식 파일 수정·덮어쓰기 절대 금지. 반드시 복사본에 작업할 것.
- 저장 위치: C:\Users\Administrator\Documents\보고서결과\
- 파일명 규칙: 주간보고서_YYYYMMDD_v1.pptx (수정 시 v2, v3으로 버전 올림)
- 이메일 제목 형식: [주간보고] Daniel _ YYYY년 MM월 N주차 업무 요약

---

## 사용하는 Skill
- 주간보고서 작성 전체 워크플로우: @skills/주간보고서/SKILL.md
- 회의록 작성 및 업로드: @skills/회의록/SKILL.md
- PPT 생성·편집: @skills/pptx/SKILL.md

---

## "주간보고서 작성해줘" 실행 워크플로우

IMPORTANT: 4주차부터 멀티에이전트 구조로 전환됨.
오케스트레이터 에이전트(.claude/agents/orchestrator.md)가 전체 흐름을 제어한다.
아래는 오케스트레이터가 실행하는 단계 요약이다.

### Phase 1 — 병렬 데이터 수집 [자동 실행]
아래 3개 에이전트를 동시에 실행한다:
- email-collector: daniel@itengineers.net 수발신 메일 수집 + 고객사 미답변 메일 분리
- meeting-collector: Drive 회의록 수집 + 로컬 STT 파일 확인 → 누락 회의록 자동 작성·업로드
- work-collector: Drive 업무 결과물 폴더 수집

수집 완료 후 사용자에게 수집 결과를 보고하고 진행 여부를 확인한다. (HITL #1)

### Phase 2 — 고객사 메일 처리 + 경쟁사 동향 검색 [순차 실행]
- customer-mail-agent: 미답변 고객사 메일 답변 초안 작성 → 건별 사용자 컨펌 → Gmail 발송
- competitor-research-agent: Brave Search로 경쟁사 동향 검색 → 우리 작업과 비교 분석

### Phase 3 — 보고서 작성 + 자동 검토 [자동 실행]
- report-writer: @skills/pptx/SKILL.md 활용, slide-mapping.md 기준으로 PPT 생성
  - 저장: C:\Users\Administrator\Documents\보고서결과\주간보고서_YYYYMMDD_v1.pptx
  - IMPORTANT: 원본 양식 파일(ppt_templates/) 수정·덮어쓰기 절대 금지
- report-reviewer: 체크리스트 자동 검토 → 미흡 시 수정 요청 (최대 2회)

### Phase 4 — 컨펌 및 발송 [사용자 확인]
- 보고서 완성 내용 확인 (HITL #2)
- 발송 직전 최종 확인 (HITL #3)
- 이메일 발송: @skills/주간보고서/references/email-template.md 양식 사용
- 발송 완료 후 파일 보관: C:\Users\Administrator\Documents\보고서결과\발송완료\

IMPORTANT: HITL #3 최종 컨펌 전 이메일 발송 절대 금지
IMPORTANT: 개인정보(연락처, 주민번호 등) 보고서 포함 금지

---

## 예외 처리

| 상황 | 처리 방법 |
|------|---------|
| Gmail 수집 0건 | "이번 주 관련 메일 없음"으로 기록 후 Step 2로 계속 진행 |
| STT 파일 0개 | "STT 파일 없음"으로 기록 후 Step 3으로 계속 진행 |
| Google Drive 접근 실패 | 오류 내용을 기록하고 해당 소스 제외 후 계속 진행. Step 6에서 사용자에게 고지 |
| PPT 작성 실패 | 오류 내용을 사용자에게 보고하고 중단 |
| Step 6 컨펌에서 "아니요" | 수정 요청 내용 반영 → 버전 올려 재저장 → Step 6 재실행 |
| 이메일 발송 실패 | 오류 내용 보고 후 재시도 여부 확인 |

---

## 절대 하지 말 것
- IMPORTANT: Step 6 컨펌 전 이메일 발송 금지
- IMPORTANT: 개인정보(연락처, 주민번호 등) 보고서 포함 금지
- 원본 양식 파일(ppt_templates/) 수정·덮어쓰기 금지
- 보고서결과/ 폴더 외 다른 위치에 파일 저장 금지
- Step 1~5 진행 중 사용자에게 진행 여부 묻지 말 것
