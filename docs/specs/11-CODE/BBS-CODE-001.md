---
doc_id: BBS-CODE-001
type: CODE
title: CODE — 동아리 게시판 구현 슬라이스
status: draft
upstream: [BBS-MS-001]
---

# 구현 슬라이스: 동아리 게시판

## 1. 슬라이스

**세로로 자른다.** 한 카드가 끝나면 브라우저에서 그 기능이 실제로 돈다.

#### A 기반

DB 스키마 · 카카오 로그인 · 상단 바. 근거: [[BBS-DOM-001]] · [[BBS-MS-001#MemberService.login]]

- [ ] `members` 표와 마이그레이션
- [ ] 카카오 OAuth 왕복, 세션 쿠키
- [ ] 로그인하면 상단 바에 이름이 뜬다

#### B1 읽고 쓴다

목록·본문·글쓰기. 근거: [[BBS-MS-001#PostService.list]] · [[BBS-MS-001#PostService.create]] · [[BBS-UI-001#UI-1]] · [[BBS-UI-001#UI-2]]

- [ ] `posts` 표
- [ ] 목록(공지 고정) · 본문 · 글쓰기 화면
- [ ] 비회원은 글쓰기 대신 로그인

#### B2 고치고 지운다

근거: [[BBS-MS-001#PostService.edit]] · [[BBS-UC-001#UC-H3]]

- [ ] 본인·운영자만 버튼이 보인다
- [ ] `(수정됨)` 표시

#### B3 댓글

근거: [[BBS-MS-001#CommentService.add]] · [[BBS-UI-001#UI-2]]

- [ ] `comments` 표 · 답글 한 단계
- [ ] 목록 행의 댓글 수

#### B4 검색

근거: [[BBS-MS-001#PostService.list]] · [[BBS-PRD-001#R5]]

- [ ] 제목 검색 · 두 글자 제한 · 빈 결과 문구

#### C 신고와 운영

근거: [[BBS-MS-001#ReportService.file]] · [[BBS-MS-001#ReportService.resolve]] · [[BBS-UI-001#UI-5]]

- [ ] `reports` 표 · 중복 금지 unique
- [ ] 신고 다이얼로그 · 운영자 처리 화면
- [ ] 숨긴 글이 목록·검색·댓글에서 빠진다

## 2. 통합 테스트

| 무엇 | 어느 시나리오 |
|---|---|
| 공지를 올리고 댓글로 참석을 받는다 | [[BBS-SCN-001#S1]] |
| 작년 후기를 제목으로 찾는다 | [[BBS-SCN-001#S2]] |
| 광고 댓글을 신고해 숨긴다 | [[BBS-SCN-001#S3]] |

## 3. 커밋

| 종류 | 형식 |
|---|---|
| 명세 | `spec(문서ID): 요약` |
| 코드 | `code(슬라이스): 함수 — 요약` |
| 상태 | `status(문서ID): a → b` |
| 버그 | `fix(#이슈): 요약` |

## 4. 미결사항

- [ ] 첨부 사진 슬라이스를 어디에 끼울지 — B1 뒤인지 C 뒤인지
