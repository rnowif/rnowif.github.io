---
layout: post
title: "Controlling the Time in Java"
date: 2017-07-20 14:06:15 +0100
comments: true
categories: ["tests", "java"]
authors: ["renaud"]
---

Time is a tricky thing, it's always changing. Having such moving parts into the codebase can be very annoying when testing, for instance. In this article, we will see how to control the time in Java.

<!-- more -->

Let's take a simple example of a pizza delivery service. Its policy is that pizzas should be delivered in 10 minutes. When the customer ask for a delivery, the delivery time is automatically calculated by the system based on this policy.

```java
public class DeliveryPolicy {
  public Delivery createDelivery(Pizza pizza) {
    LocalDateTime deliveryTime = LocalDateTime.now().plus(10, ChronoUnit.MINUTES);

    return new Delivery(pizza, deliveryTime);
  }
}
```

## How Do we Test it?

Since the time always changes we cannot test this method very precisely without changing its code. We could test that the delivery time is _approximatively_ 10 minutes in the future but like all fuzzy tests, it could fail from time to time depending on the context on which it is executed.

## Make Dependency Visible

The reason why this code is so difficult to test is that it has a hidden dependency: the clock. Since Java 8, all `now` methods of the Date API take a clock as an argument. We can then make this dependency visible.

```java
public class DeliveryPolicy {

  private final Clock clock;

  public DeliveryPolicy() {
    this.clock = Clock.systemDefaultZone();
  }

  public Delivery createDelivery(Pizza pizza) {
    LocalDateTime deliveryTime = LocalDateTime.now(clock).plus(10, ChronoUnit.MINUTES);

    return new Delivery(pizza, deliveryTime);
  }
}
```

## Dependency Injection to the Rescue

Now that the dependency is out there, we can inject it in the constructor, like any other dependency, and testing this method becomes trivial.

```java
public class DeliveryPolicy {

  private final Clock clock;

  public DeliveryPolicy(ClockProvider clockProvider) {
    this.clock = clockProvider.get();
  }

  public Delivery createDelivery(Pizza pizza) {
    LocalDateTime deliveryTime = LocalDateTime.now(clock).plus(10, ChronoUnit.MINUTES);

    return new Delivery(pizza, deliveryTime);
  }
}
```

```java
public class DeliveryPolicyTest {

  @Test
  public void should_schedule_delivery_ten_minutes_later() {
    ZonedDateTime now = ZonedDateTime.of(LocalDateTime.of(2017, 7, 18, 0, 0, 0), ZoneId.of("+01"));
    DeliveryPolicy policy = new DeliveryPolicy(() -> Clock.fixed(now.toInstant(), now.getZone()));

    Delivery delivery = policy.createDelivery(new Pizza());

    LocalDateTime tenMinutesLater = LocalDateTime.of(2017, 7, 18, 0, 10, 0);
    assertThat(delivery.getDeliveryTime()).isEqualTo(tenMinutesLater);
  }
}
```

## Usage with Spring Framework and Spring Boot

If you use Spring Framework in your application, you can create a `ClockProvider` bean that will give the default Clock. Moreover, if you use Spring Boot, it allows you to mock this bean very easily in integration tests with the `@MockBean` annotation.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DeliveryPolicyIT {

  @MockBean
  private ClockProvider clockProvider;

  @Autowired
  private DeliveryPolicy policy;

  @Test
  public void should_schedule_delivery_ten_minutes_later() {
    ZonedDateTime now = ZonedDateTime.of(LocalDateTime.of(2017, 7, 18, 0, 0, 0), ZoneId.of("+01"));
    Mockito.when(clockProvider.get()).thenReturn(Clock.fixed(now.toInstant(), now.getZone()));

    Delivery delivery = policy.createDelivery(new Pizza());

    LocalDateTime tenMinutesLater = LocalDateTime.of(2017, 7, 18, 0, 10, 0);
    assertThat(delivery.getDeliveryTime()).isEqualTo(tenMinutesLater);
  }
}
```

```java
@Configuration
public class ClockConfig {

  @Bean
  public ClockProvider clockProvider() {
    return () -> Clock.systemDefaultZone();
  }
}
```
## Conclusion

I used to think that the time, like random, was a very difficult thing to test. With the Java 8 Date API, it becomes trivial. You just have to acknowledge the dependency you have on the clock and treat it like any other dependency. Now, I almost never use a method of the Date API without passing a clock as a parameter. This allows me to control the time throughout the application very easily (in unit, integration or functional tests).
