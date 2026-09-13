---
doc_id: BBS-API-001
type: API
title: API — 동아리 게시판 REST
status: review
upstream: [BBS-UI-001]
---

# API 명세 REST: 동아리 게시판

## 1. 규칙

- 경로는 복수형 명사. `/api/posts`, `/api/comments`
- 인증은 세션 쿠키. **읽기 엔드포인트는 인증 없이 연다.** 근거: [[BBS-INFRA-001#C4]]
- 목록은 `?page=1&size=20`. 응답에 `total`을 담는다
- 시각은 전부 ISO8601 UTC

## 2. 에러

| 상태 | 언제 | 본문 |
|---|---|---|
| 400 | 입력 형식이 틀림 | `{code, message, field}` |
| 401 | 세션 없음 | `{code: "unauthorized"}` |
| 403 | 남의 글을 고치려 함 | `{code: "forbidden"}` |
| 404 | 없거나 숨겨진 글 | `{code: "not-found"}` |
| 409 | 같은 글을 두 번 신고 | `{code: "already-reported"}` |

**숨긴 글은 404다.** 403으로 주면 "숨겨진 글이 있다"는 사실이 샌다. 근거: [[BBS-UC-001#UC-S2]]

## 3. 엔드포인트

#### GET/api/posts 글 목록

근거: [[BBS-UI-001#UI-1]] · [[BBS-PRD-001#R1]]

```yaml
get:
  summary: 공지 묶음 + 일반 묶음
  parameters: [page, size, q]
  responses:
    200: { notices: [Post], posts: [Post], total: int }
```

`q`가 있으면 제목만 훑는다. 두 글자 미만이면 400. 근거: [[BBS-PRD-001#R5]]

#### GET/api/posts/{postId} 글 하나

근거: [[BBS-UI-001#UI-2]]

```yaml
get:
  responses:
    200: { post: Post, comments: [Comment] }
    404: 없거나 숨겨짐
```

#### POST/api/posts 글 쓰기

근거: [[BBS-UC-001#UC-H2]]

```yaml
post:
  body: { title: str(100), body: str(10000), is_notice: bool }
  responses:
    201: { post: Post }
    401: 비회원
    403: 운영자가 아닌데 is_notice=true
```

#### PATCH/api/posts/{postId} 글 고치기

근거: [[BBS-UC-001#UC-H3]]

```yaml
patch:
  body: { title, body }
  responses:
    200: { post: Post }   # edited_at 갱신
    403: 남의 글
```

#### DELETE/api/posts/{postId} 글 지우기

```yaml
delete:
  responses:
    204: 댓글도 함께 사라진다
    403: 남의 글
```

#### POST/api/posts/{postId}/comments 댓글 달기

근거: [[BBS-UC-001#UC-H4]]

```yaml
post:
  body: { body: str, parent_id: int|null }
  responses:
    201: { comment: Comment }
    400: parent가 이미 답글이다 (한 단계 제한)
```

#### POST/api/reports 신고

근거: [[BBS-UC-001#UC-H6]]

```yaml
post:
  body: { post_id|comment_id, reason: 광고|욕설|기타 }
  responses:
    201: { report: Report }
    409: already-reported
```

#### PATCH/api/reports/{reportId} 신고 처리

근거: [[BBS-UC-001#UC-A7]]

```yaml
patch:
  body: { outcome: hidden|ignored }
  responses:
    200: { report: Report }
    403: 운영자가 아니다
```

## 4. 미결사항

- [ ] 첨부 업로드 엔드포인트 — 사진 결정이 나면 추가
