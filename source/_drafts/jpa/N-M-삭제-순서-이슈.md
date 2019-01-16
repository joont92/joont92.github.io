---
title: 'N:M 삭제 순서 이슈'
date: 2018-09-06 18:53:49
tags:
---

엔티티 : 포스트(post), 태그(tag)
관계 : N:M
연결 테이블 : post_tag

post 게시물에서 사용 tag를 변경할 경우
기존의 tag를 모두 삭제한 후 입력받은 태그를 insert

※ post 엔티티에서 setTags를 호출하면 기본적으로
N:M 사이에 있는 연결 테이블의 데이터가 전부 삭제되고,
setTags의 인자로 받은 자식을 insert함 (다대다 특성)

- 상황
수정 메서드에서
기존의 태

<!-- more -->