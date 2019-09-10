---
title: '[jpa] best practice'
date: 2019-01-25 19:51:51
tags:
---

Spring orm이 제공하는 것 LocalContainerEntityManagerFactory
테이블의 기본키 이름은 테이블이름을 prefix로 가져가는것이 좋다
엔티티의 경우 참조라는 값이 있지만 RDB의 경우는 없기 때문
게다가 테이블간 관계설정할 때 외래키에 테이블의 이름을 쓰는 관례가 많으므로 헷갈리지 않기 위해 기본키도 테이블 이름을 prefix로 가져주는 것이 좋다

@PersistenceContext : 
스프링이나 J2EE를 사용하면 컨테이너가 직접 엔티티매니저를 관리하고 제공해준다.
이 어노테이션은 컨테이너가 관리하는 엔티티 매니저를 주입하는 어노테이션이다

@PersistenceUnit : EntityManagerFactory를 주입받고자 할 때 

@Transactional은 RuntimeException만 롤백한다

- 엔티티 설계에서 참조할 수 있는 부분. 편의메서드가 내가 작성한 것 만큼 빡빡하게 조건검사를 하지 않는다

public void setMember(Member member){
	this.member = member;
	member.getOrders().add(this);
}

public void addOrder(Order order){
	orders.add(order);
	order.setmember(this);
}
근데 책에서 설명할때는 bug를 방지하여 조건검사를 어느정도 넣어주는것을 봤는데, 이걸 제거해도 되는것인지?

회원 중복검사를 하는 로직은 service 메서드에서 private으로 구현했다
이미 등록된 회원일 경우 exception을 반환하는 식으로 처리했다
이러한 검증 로직이 있어도 멀티스레드 안정성을 보장하기 위해 UK를 걸어주는 것이 좋다

- 값이 setter를 통해 무분별하게 사용되지 않도록 메서드로 제한한다
public void addStock(int quantity) {
    this.stockQuantity += quantity;
}

public void removeStock(int quantity) {
    int restStock = this.stockQuantity - quantity;
    if (restStock < 0) {
        throw new NotEnoughStockException("need more stock");
    }
    this.stockQuantity = restStock;
}

public void cancel() {

    if (delivery.getStatus() == DeliveryStatus.COMP) {
        throw new RuntimeException("이미 배송완료된 상품은 취소가 불가능합니다.");
    }

    this.setStatus(OrderStatus.CANCEL);
    for (OrderItem orderItem : orderItems) {
        orderItem.cancel();
    }
}

- 관계를 한방에 맺어주는 편의메서드를 제공한다. 이로인해 객체간의 관계도 파악가능하다.
public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {

    Order order = new Order();
    order.setMember(member);
    order.setDelivery(delivery);
    for (OrderItem orderItem : orderItems) {
        order.addOrderItem(orderItem);
    }
    order.setStatus(OrderStatus.ORDER);
    order.setOrderDate(new Date());
    return order;
}

- 추가적인 정보를 제공해주는 메서드를 만든다
public int getTotalPrice() {
    int totalPrice = 0;
    for (OrderItem orderItem : orderItems) {
        totalPrice += orderItem.getTotalPrice();
    }
    return totalPrice;
}

- 다른 엔티티와 값을 동기화한다(재고 등)
public static OrderItem createOrderItem(Item item, int orderPrice, int count) { // 주문생성시 재고를 깎고

    OrderItem orderItem = new OrderItem();
    orderItem.setItem(item);
    orderItem.setOrderPrice(orderPrice);
    orderItem.setCount(count);

    item.removeStock(count);
    return orderItem;
}

public void cancel(){ // 주문취소시 재고를 더한다
	getItem().addStock(count);
}

—

- 도메인 모델 패턴과 트랜잭션 스크립트 패턴
도메인 모델 패턴은 대부분의 비즈니스 로직이 엔티티에 속해있고, 서비스는 역할을 위임하고 트랜잭션 바인딩 정도로 사용되것이다
트랜잭션 스크립트 패턴은 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것을 말한다

- 수정에 대해서는 ‘영속화 + 필요한 메서드 변경’ 을 사용하던 ‘병합’을 사용하던 상관없다고 말하고 있음 

<!-- more -->