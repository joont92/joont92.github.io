---
title: 'Custom PageRequest'
date: 2018-09-06 18:53:59
tags:
---

Spring Data JPA에서 페이징을 위해 사용되는 Pageable의 구현체인 PageRequest를 커스터마이징하여 사용할 수 있다.
PageRequest는 page와 size를 전달받고 size에 맞게 시작값을 자동으로 세팅하여 페이징한다.

ex)
page = 1, size = 10 > limit 11,10
page = 0, size = 10 > limit 0,10

PageRequest는 AbstractPageRequest를 extends하고,
AbstractPageRequest의 getOffset 메서드를 보면 page * size 의 형태로 리턴하기 때문
이를 override 해서 시작페이지를 지정하게끔 할 수 있다.

```java
public class LimitPageRequest extends PageRequest implements Pageable{
    /*
     *
     */
    private static final long serialVersionUID = -7867774849562902151L;

    private final int page = super.getPageNumber();

    public LimitPageRequest(int page, int size) {
        super(page, size);
    }

    public LimitPageRequest(int page, int size, Direction direction, String... properties) {
        super(page, size, direction, properties);
    }

    public LimitPageRequest(int page, int size, Sort sort) {
        super(page, size, sort);
    }

    @Override // overrider to AbstractPageRequest method
    public int getOffset() {
        return page; // getPageNumber에서 이 메서드를 사용하는 듯
    }
}
```

<!-- more -->