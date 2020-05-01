---
layout: post
title: "ASP.Net Clean Architecture"
date: 2018-10-22 10:04:23 +1300
comments: true
categories: ["C#", ".NET", "ASP.NET", "architecture", "clean code"]
---

When creating a new project, it is always a challenge to design a clean, coherent and modular architecture. There are guidelines out there to help us achieve this goal but the implementation is not always straightforward. In this blog post, I will propose an implementation of the Uncle Bob's [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) on an ASP.Net project. The source code of this project can be found on my [GitHub](https://github.com/rnowif/Expenses).

<!-- more -->

The main idea of the clean architecture is to reduce the coupling between the core business code and the external world (Web, Database, Frameworks). In order to do that, the project can be divided in 3 main modules, that will be described below: `Domain`, `Web` and `Data`.

## Domain

This module contains all the core business code. It does not depend on anything else than the .NET SDK and contains sub-modules.

### `Entities`

They are the building blocks of the domain and encapsulate the business concepts. Entities are not coupled with any ORM framework. Indeed the domain model can be quite different from the database model!

### `Use Cases`

Following the Uncle Bob's definition, they contains application specific business rules and orchestrate the flow of data to and from the entities and implement higher level business rules. 

It is important to understand that the domain model should not leak outside of this module. In order to do that, use cases should not take entities are arguments for their methods, but a list of raw arguments. For instance, the use case that submit a new expense looks like the following:

```csharp
public interface ISubmitExpense
{
    void Execute(Guid userId, string description, long priceWithoutTax, long priceIncludingTax);
}
```

It is responsible to build an `Expense` object and apply the relevant business logic. 

### `Repositories`

In the domain module, repositories are only interfaces that are used by the use cases to access data without any knowledge of the concrete implementations: data can be retrieved from databases, files or external Web APIs, it should not affect the core business logic at all. To reinforce this, repositories take entities as arguments and return entities as well. It is the responsibility of the concrete implementation to handle conversion if needed.

## Data

The data module contains the database and ORM configuration and the repositories implementations. Typically, we find the EntityFramework configuration in this module as well as the annotated classes that will be mapped with database entries. Repositories implementation are responsible for converting "database objects" in domain objects:

```csharp
public void Create(Expense expense)
{
  // Convert the domain object "Expense" into a database object "DbExpense"
  _dbContext.Expenses.Add(DbExpense.FromExpense(expense));
  _dbContext.SaveChanges();
}
```

This prevents the ORM framework to leak into the domain.

## Web

The web module contains all the controllers. It is responsible for handling HTTP requests, converting JSON or XML payloads to objects and invoking use cases:

```csharp
[HttpPost]
public void SubmitExpense([FromBody] SubmitExpenseCommand expense)
{
  // The controller just invoke the use case with data extracted from the body of the HTTP request
  _submitExpense.Execute(expense.UserId, expense.Description, expense.PriceWithoutTax, expense.PriceIncludingTax);
}
```

It should not contain any business logic whatsoever. As a consequence, controllers are very lightweight and easy to test.

## Dependency injection

In order to achieve low coupling between the modules, interfaces are injected into the constructor of the different classes:

```csharp
public class SubmitExpense : ISubmitExpense
{

	private readonly IExpenseRepository _repository;

	public SubmitExpense(IExpenseRepository repository)
	{
	  _repository = repository;
	}
}
```

The plumbing is handled by the `Startup.cs` class where all the implementations of the interfaces are declared:

```csharp
public void ConfigureServices(IServiceCollection services)
{
  // ...

  // Register "SubmitExpense" as the implementation of "ISubmitExpense"
  services.AddTransient<ISubmitExpense, SubmitExpense>();
  services.AddTransient<IExpenseRepository, ExpenseRepository>();
}
```

## Conclusion

In this architecture, the emphasis is put on the domain. Every other modules should adapt to it but it does not depend on anything. This results in lightweight classes that are very easy to understand and test in isolation.  
Moreover, the low coupling makes it very easy to change the infrastructure. Indeed, the domain module would not change a bit if we decided to have a CLI instead of a Web App or if the data should be retrieved from an external Web API instead of a PostgreSQL database.