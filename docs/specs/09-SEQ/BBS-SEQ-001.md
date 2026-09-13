---
doc_id: BBS-SEQ-001
type: SEQ
title: SEQ — 동아리 게시판 시퀀스
status: approved
upstream: [BBS-API-001, BBS-DOM-003]
---

# 시퀀스: 동아리 게시판

## 1. 생명선

| 약칭 | 무엇 |
|---|---|
| B | 브라우저 |
| W | 웹 라우터 |
| P | PostService |
| C | CommentService |
| R | ReportService |
| M | MemberService |
| DB | PostgreSQL |

## 2. 시퀀스

#### SEQ-1 글을 쓴다

근거: [[BBS-UC-001#UC-H2]] · [[BBS-API-001#POST/api/posts]]

```mermaid
sequenceDiagram
  B->>W: POST /api/posts {title, body, is_notice}
  W->>M: 세션 → 회원?
  M-->>W: Member 또는 없음
  alt 비회원
    W-->>B: 401
  else 회원
    W->>P: create(member, title, body, is_notice)
    P->>P: 제목 100자 · 본문 10000자 검사
    alt is_notice 인데 운영자가 아님
      P-->>W: forbidden
      W-->>B: 403
    end
    P->>DB: INSERT posts (edited_at = created_at)
    DB-->>P: id
    P-->>W: Post
    W-->>B: 201 {post}
  end
```

**읽을 때 볼 것**: 길이 검사가 서비스 안이다. 라우터는 형식만 본다.

#### SEQ-2 댓글을 단다

근거: [[BBS-UC-001#UC-H4]] · [[BBS-API-001#POST/api/posts/{postId}/comments]]

```mermaid
sequenceDiagram
  B->>W: POST /api/posts/{id}/comments {body, parent_id}
  W->>M: 세션 → 회원?
  W->>C: add(post_id, member, body, parent_id)
  alt parent_id 가 있다
    C->>DB: SELECT parent_id FROM comments WHERE id=?
    alt 부모가 이미 답글이다
      C-->>W: bad-request (한 단계 제한)
      W-->>B: 400
    end
  end
  C->>DB: INSERT comments
  C-->>W: Comment
  W-->>B: 201
```

#### SEQ-3 신고하고 숨긴다

근거: [[BBS-UC-001#UC-H6]] · [[BBS-UC-001#UC-A7]] · [[BBS-UC-001#UC-S2]]

```mermaid
sequenceDiagram
  B->>W: POST /api/reports {post_id, reason}
  W->>R: file(member, post_id, reason)
  R->>DB: INSERT reports
  alt 중복 (unique 위반)
    DB-->>R: conflict
    R-->>W: already-reported
    W-->>B: 409
  end
  Note over R,DB: — 나중에, 운영자가 —
  B->>W: PATCH /api/reports/{id} {outcome: hidden}
  W->>R: resolve(admin, report_id, hidden)
  R->>DB: UPDATE reports SET outcome
  R->>P: hide(post_id)
  P->>DB: UPDATE posts SET is_hidden = true
  W-->>B: 200
```

**지우지 않는다.** 숨김은 플래그다. 근거: [[BBS-DOM-001#posts]]

## 3. 대응표

| 시퀀스 | 유스케이스 | 엔드포인트 |
|---|---|---|
| [[#SEQ-1]] | [[BBS-UC-001#UC-H2]] | [[BBS-API-001#POST/api/posts]] |
| [[#SEQ-2]] | [[BBS-UC-001#UC-H4]] | [[BBS-API-001#POST/api/posts/{postId}/comments]] |
| [[#SEQ-3]] | [[BBS-UC-001#UC-H6]] · [[BBS-UC-001#UC-A7]] | [[BBS-API-001#POST/api/reports]] |

## 4. 되먹일 것

- [[BBS-API-001]]의 `POST/api/posts`는 403 조건을 "운영자가 아닌데 is_notice"로만 적었는데, 시퀀스를 그려 보니 **비회원은 401**이 먼저다. API 쪽에 401을 명시해야 한다
