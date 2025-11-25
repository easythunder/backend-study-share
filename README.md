# 📚 Backend Study Repository

백엔드 개발 역량 강화를 위해 **DB / Spring / 알고리즘 / Linux** 영역을 체계적으로 정리하는 스터디 저장소입니다.  
브랜치 전략·PR 프로세스·커밋 규칙을 명확하게 정의해 협업 효율을 높입니다.

---

## 📁 디렉토리 구조
```
backend-study/  
├─ DB/mysql/  
├─ Server/spring/  
├─ Algorithm/  
└─ OS/linux/
```

### 📌 각 디렉토리 역할

- **DB/mysql/**  
  MySQL 쿼리, 인덱스, 트랜잭션, JOIN, ERD 설계 등 DB 핵심 개념

- **Server/spring/**  
  Spring Core / Boot / MVC / JPA / 실습 코드

- **Algorithm/**  
  알고리즘 풀이(BOJ/Programmers), 자료구조 개념

- **OS/linux/**  
  리눅스 명령어, 프로세스/메모리, OS 이론 정리

---

## 🌿 브랜치 전략

### 1️⃣ main
- 스터디의 **최종/검증된 정리본**  
- 안정적인 문서/코드만 유지
- PR 승인 후에만 반영


### 2️⃣ summary
- 팀 공용 요약/정리본 저장
- 주제별 핵심 개념 모아놓는 브랜치
- Markdown 위주
- PR 승인 후에만 반영

### 3️⃣ study-이름
- 개인 자유 학습 공간
- 실습 코드, 예제, 필기, 기록 등 전부 가능
- 원하는 만큼 push 가능

### 4️⃣ feature/*
- 디렉토리 구조 개편, 레포 전체 리팩토링 등 구조 변경 작업 전용



## 📌 원칙

- `study` → `summary` : 개인 학습 내용 검토 요청  
- `summary` → `main` : 한 챕터 당 학습내용 최종 반영  
- `feature/*` → `main` : 구조 변경/리팩토링 병합  

---

## 📝 Commit Message 규칙

### 🔍 커밋 메시지 규칙 예시

`emoji` `type:` `한글로 제목 작성`

- 새로운 내용 추가 → `:sparkles: feat: ...`
- 피드백 반영/설명 보강 → `:bulb: feat: ...`


### 🔠 Type (소문자로 작성)

| Type | 의미 |
|------|------|
| feat | 공부 내용 추가 |
| docs | 문서/운영 내용 |
| fix | 잘못된 내용 수정 |
| refactor | 구조/정리 방식 변경 |
| chore | 파일/폴더/세팅 변경 |
| style | 포맷팅/마크다운 스타일 |


### ✨ Git Emoji 규칙

| 이모지 | 코드 | 의미 |
|--------|-------|------|
| ✨ | :sparkles: | 완전히 새로운 내용/기능 추가 |
| 💡 | :bulb: | 개념 보강, 예제 추가, 피드백 반영(보완용 feat) |
| ✏️ | :pencil2: | 오타 수정 |
| 🐛 | :bug: | 잘못된 내용 수정 |
| ♻️ | :recycle: | 구조 변경, 리팩토링 |
| 🎨 | :art: | 포맷/스타일 개선 |
| 🔧 | :wrench: | 세팅/환경/폴더 변경 |
| 📝 | :memo: | 문서 작업 |
| 🚀 | :rocket: | 신규 프로젝트 생성 |

