---
layout: post
title: "When NOT To Mock"
date: 2016-02-22 18:39:20 +0100
comments: true
categories: [Test, Craftsmanship]
---

As a Test Driven Developer, and adept of the London School of TDD, I tend to use an extensive amount of mocks when I test my code. However, there are cases when I find useful NOT to mock.

The goal of this article is to describe theses cases and how to test the code without mocks.

<!-- more -->

## Database

When dealing directly with databases, the only thing I care about is whether or not the code interacts correctly with it. Except for some eventual optimization or protection against injections, the query itself is not that important.

For these reason, I prefer to use a real database and make assertions directly on it rather than make assertions on the generated query. Moreover, it is much more simple to build an embedded database than mock a `DataSource` and capture the query.

The following example uses Spring's `EmbeddedDatabaseBuilder` and `JdbcTemplate` to create and access an embedded database.

```java
public class UserRepositoryTest {

	private DataSource dataSource() {
		return new EmbeddedDatabaseBuilder()
			.setType(EmbeddedDatabaseType.HSQL)
			.addScript("db/sql/create-db.sql")
			.build();
    }

	@Test
    public void should_save_user_to_database {
		DataSource ds = dataSource();
        UserRepository userRepository = new UserRepository(ds);

		userRepository.save(new User("John", "Doe"));

        JdbcTemplate jdbcTemplate = new JdbcTemplate(ds);
        int count = jdbcTemplate.queryForObject("select count from user where first_name = 'John' and last_name = 'Doe'", Integer.class);
		assertThat(count, is(1));
    }
}
```

## Network

Code that communicates through the network may be hard to test in isolation. In my opinion, it is not necessarily useful

## File

## Controller

## Mail