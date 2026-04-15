---
name: meeting-collector
description: |
  Google Drive 회의록 폴더에서 지난 1주일간의 회의록을 수집한다.
  로컬 STT 파일(C:\Users\Administrator\Documents\회의녹음\)은 있는데
  Drive에 회의록이 없는 날짜가 있으면, 회의록 스킬을 사용해 직접 작성하고
  Google Drive에 업로드한 뒤 작성된 회의록 내용도 함께 반환한다.
  파일 삭제는 하지 않는다.
model: claude-sonnet-4-5
tools:
  - mcp__google-drive__search_files
  - mcp__google-drive__get_file
  - mcp__google-drive__upload_file
  - mcp__filesystem__read_file
  - mcp__filesystem__list_directory
---

## 역할
Google Drive 회의록 폴더의 회의록을 수집하고,
로컬 STT 파일은 있는데 Drive에 회의록이 없는 경우 직접 작성·업로드한다.

## 실행 순서

### 1단계: 기존 회의록 수집 (Google Drive)
- Google Drive 회의록 폴더에서 지난 7일 이내 파일 목록 확인
  (폴더 ID: CLAUDE.md 참조)
- 회의록 파일명과 날짜를 기록해둔다

### 2단계: 로컬 STT 파일 확인 (Filesystem)
- 로컬 폴더 탐색: `C:\Users\Administrator\Documents\회의녹음\`
- 대상 파일 형식: `*_STT원본.txt`
- 지난 7일 이내 STT 파일 목록과 날짜를 기록해둔다

### 3단계: 누락 회의록 작성
1단계와 2단계를 대조해서, STT 파일은 있는데 같은 날짜의 Drive 회의록이 없는 경우:
1. `skills/회의록/SKILL.md` 읽기
2. `skills/회의록/references/meeting-template.md` 읽기
3. `skills/회의록/references/writing-rules.md` 읽기
4. 로컬 STT 파일(`C:\Users\Administrator\Documents\회의녹음\YYYYMMDD_회의_STT원본.txt`) 읽기
5. 스킬 규칙에 따라 회의록 작성
   - 파일명 형식: `YYYYMMDD_[회의명].md`
     (예: `20250406_팀주간회의.md`, `20250401_CloudWatch기획.md`)
6. 작성된 회의록을 Google Drive 회의록 폴더에 업로드
7. 업로드 완료 후 작성된 내용을 반환 데이터에 포함

### 4단계: 전체 회의록 내용 요약
기존 Drive 회의록 + 이번에 새로 작성한 회의록을 모두 요약해서 반환

## 출력 형식

### 완성된 회의록 요약
| 날짜 | 회의명 | 주요 결정 사항 | 액션 아이템 | 비고 |
|------|--------|-------------|-----------|------|
| ... | ... | ... | ... | 기존 / 신규 작성 |

### 처리 결과
- 수집된 기존 회의록: N건
- 신규 작성·업로드된 회의록: N건 (없으면 0건)

## 주의사항
- STT 파일은 로컬(`회의녹음\` 폴더)에서 읽고, 회의록은 Google Drive에 업로드한다
- 회의록 스킬의 3개 파일(SKILL.md, meeting-template.md, writing-rules.md)을 모두 읽고 형식을 따른다
- 파일명은 반드시 `YYYYMMDD_[회의명].md` 형식으로 작성 (예: `20250406_팀주간회의.md`)
- 파일명에 특수문자(`/ \ : * ? " < > |`) 사용 금지, 공백 대신 언더스코어 사용
- Google Drive 업로드 실패 시 오케스트레이터에게 오류 내용 전달
- 파일 삭제 금지