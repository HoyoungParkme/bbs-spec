---
doc_id: BBS-UI-002
type: UI
title: UI — 동아리 게시판 와이어프레임
status: draft
upstream: [BBS-UI-001]
---

# 와이어프레임: 동아리 게시판

## 1. 형식

요소 번호는 `data-el`이다. 화면 설계의 화면 하나가 여기 배치 하나에 대응한다.

#### UI-1 글 목록 배치

근거: [[BBS-UI-001#UI-1]]

```html
<div class="page" data-el="1">
  <div class="bar" data-el="1.1">등산동아리 <input data-el="1.2" placeholder="제목 검색"> <a data-el="1.3">글쓰기</a></div>
  <ul class="notice" data-el="2">
    <li data-el="2.1">[공지] 10/5 북한산 백운대 · 박태현 · 09-28 · 12</li>
  </ul>
  <ul class="list" data-el="3">
    <li data-el="3.1">지리산 후기 · 김순자 · 09-20 · 4</li>
  </ul>
  <div class="pager" data-el="4">1 2 3 ›</div>
</div>
```

#### UI-2 글 본문 배치

근거: [[BBS-UI-001#UI-2]]

```html
<div class="page" data-el="1">
  <h1 data-el="1.1">10/5 북한산 백운대</h1>
  <div class="meta" data-el="1.2">박태현 · 09-28 (수정됨)</div>
  <div class="actions" data-el="1.3"><a data-el="1.4">고치기</a> <a data-el="1.5">지우기</a> <a data-el="1.6">신고</a></div>
  <div class="body" data-el="2">08:00 불광역 2번 출구 …</div>
  <ul class="comments" data-el="3">
    <li data-el="3.1">김순자 · 저 갑니다 <a data-el="3.2">답글</a></li>
  </ul>
  <form data-el="4"><textarea data-el="4.1"></textarea><button data-el="4.2">댓글</button></form>
</div>
```

#### UI-5 신고 처리 배치

근거: [[BBS-UI-001#UI-5]]

```html
<div class="page" data-el="1">
  <h1 data-el="1.1">신고 2건</h1>
  <table data-el="2">
    <tr data-el="2.1"><td>광고</td><td>대출 상담 …</td><td>김순자</td>
        <td><button data-el="2.2">숨김</button><button data-el="2.3">무시</button></td></tr>
  </table>
</div>
```
