---
doc_id: BBS-MS-001
type: MS
title: MINISPEC — 동아리 게시판
status: review
upstream: [BBS-SEQ-001, BBS-DOM-003]
---

# MINISPEC: 동아리 게시판

## 1. 함수 목록

#### PostService.list 글 목록

**시그니처** `list(page: int = 1, size: int = 20, q: str | None = None) -> PostPage`

근거: [[BBS-UC-001#UC-G1]] · [[BBS-API-001#GET/api/posts]]

**처리**
1. `q`가 있고 두 글자 미만이면 `! bad-request {field: q}`
2. `DB: posts where is_hidden = false` (+ `q`면 `title ILIKE %q%`)
3. `is_notice desc, created_at desc` 정렬 · `page`만큼 건너뛰고 `size`만큼
4. `→ PostPage(notices, posts, total)` — 공지는 페이지와 무관하게 늘 붙인다

**테스트 관점** 공지가 2쪽에서도 위에 온다 · 숨긴 글이 안 나온다 · `q` 한 글자면 400

#### PostService.create 글 쓰기

**시그니처** `create(author: Member, title: str, body: str, is_notice: bool) -> Post`

근거: [[BBS-SEQ-001#SEQ-1]]

**처리**
1. `len(title) > 100` 또는 `len(body) > 10000`이면 `! bad-request`
2. `is_notice and not author.is_admin`이면 `! forbidden`
3. `DB: insert posts (edited_at = created_at)`
4. `→ Post`

**테스트 관점** 일반 회원이 `is_notice=true` → 403 · 101자 제목 → 400

#### PostService.edit 글 고치기

**시그니처** `edit(actor: Member, post_id: int, title: str, body: str) -> Post`

근거: [[BBS-UC-001#UC-H3]]

**처리**
1. `post = get(post_id)` · 없거나 숨겨졌으면 `! not-found`
2. `actor.id != post.author_id and not actor.is_admin`이면 `! forbidden`
3. `DB: update title, body, edited_at = now`
4. `→ Post`

**테스트 관점** 남의 글 → 403 · 운영자는 남의 글도 고친다 · `edited_at`이 달라진다

#### PostService.hide 글 숨기기

**시그니처** `hide(post_id: int) -> None`

근거: [[BBS-UC-001#UC-S2]] · [[BBS-SEQ-001#SEQ-3]]

**처리** `DB: update posts set is_hidden = true`. **지우지 않는다**

#### CommentService.add 댓글 달기

**시그니처** `add(post_id: int, author: Member, body: str, parent_id: int | None) -> Comment`

근거: [[BBS-SEQ-001#SEQ-2]]

**처리**
1. 글이 없거나 숨겨졌으면 `! not-found`
2. `parent_id`가 있으면 그 댓글을 읽어 **그것의 `parent_id`가 null이 아니면** `! bad-request` (한 단계 제한)
3. `DB: insert comments`
4. `→ Comment`

**테스트 관점** 답글의 답글 → 400 · 숨긴 글에 댓글 → 404

#### ReportService.file 신고 접수

**시그니처** `file(reporter: Member, post_id: int | None, comment_id: int | None, reason: str) -> Report`

근거: [[BBS-UC-001#UC-H6]]

**처리**
1. `post_id`와 `comment_id` 중 정확히 하나가 아니면 `! bad-request`
2. `DB: insert reports` — unique 위반이면 `! already-reported`
3. `→ Report`

**테스트 관점** 같은 대상 두 번 → 409 · 둘 다 null → 400 · 둘 다 채움 → 400

#### ReportService.resolve 신고 처리

**시그니처** `resolve(admin: Member, report_id: int, outcome: str) -> Report`

근거: [[BBS-UC-001#UC-A7]]

**처리**
1. `not admin.is_admin`이면 `! forbidden`
2. `DB: update reports set outcome`
3. `outcome == "hidden"`이면 `PostService.hide(대상)`
4. `→ Report`

**테스트 관점** 일반 회원 → 403 · `hidden`이면 대상 글이 목록에서 빠진다

#### MemberService.login 카카오 로그인

**시그니처** `login(kakao_id: str, nickname: str) -> Member`

근거: [[BBS-UC-001#UC-S1]] · [[BBS-INFRA-001#C3]]

**처리** `kakao_id`로 찾고 없으면 만든다. 있으면 `nickname`을 갱신한다 · `→ Member`

**테스트 관점** 두 번째 로그인에 회원이 안 늘어난다 · 카카오에서 이름을 바꾸면 따라간다

## 2. 미결사항

- [ ] `PostService.list`의 검색을 ILIKE로 둘지 trgm 인덱스로 갈지 — 5000건에선 ILIKE도 버틴다
