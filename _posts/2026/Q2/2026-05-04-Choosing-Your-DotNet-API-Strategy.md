---
layout: post
title:  "Choosing Your .NET API Strategy in 2026: MediatR vs. ServiceStack vs. WolverineFX"
date:   2026-05-04 12:00:00 +1000
categories: [dotnet, architecture]
tags: [dotnet, cqrs, mediatr, servicestack, wolverinefx, architecture, webapi]
permalink: /blog/2026/Choosing-Your-DotNet-API-Strategy/
---

Building a .NET web API in 2026 isn't just about choosing between Controllers or Minimal APIs anymore. It's about how you manage the flow of data and logic through your application. Three approaches dominate the conversation: the classic **MediatR with CQRS**, the high-performance **ServiceStack**, and the modern challenger **WolverineFX**.

This post came out of research I was doing while starting a few new projects and evaluating the best .NET 10 and React templates. The more I dug in, the more I realised the architectural decision around how you handle requests is just as important as any other tech choice.

Here's a breakdown of how they stack up.

---

## The Comparison at a Glance

| Feature | CQRS + MediatR | ServiceStack | WolverineFX |
|---|---|---|---|
| **Philosophy** | Decoupling & Clean Architecture | Message-based productivity | Low-ceremony & durable messaging |
| **Boilerplate** | High (many files/classes) | Low (all-in-one ecosystem) | Very Low (uses convention) |
| **Async Messaging** | Needs a 3rd party (e.g. MassTransit) | Built-in Service Client | Built-in (Inbox/Outbox patterns) |
| **Performance** | Standard (reflection-heavy) | High (optimised serialisation) | Very High (source generators) |

---

## 1. The Industry Standard: CQRS + MediatR

MediatR is the "safe" choice. It implements the Mediator pattern, allowing your API controllers or Minimal API endpoints to send a **Request** object and receive a **Response** without knowing who handles it.

```csharp
// Command
public record CreateOrderCommand(string CustomerId, List<OrderItem> Items) 
    : IRequest<OrderResult>;

// Handler
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, OrderResult>
{
    public async Task<OrderResult> Handle(CreateOrderCommand request, CancellationToken ct)
    {
        // Business logic lives here, not in the controller
        return new OrderResult(Guid.NewGuid());
    }
}

// Minimal API endpoint
app.MapPost("/orders", async (CreateOrderCommand cmd, IMediator mediator) =>
    Results.Ok(await mediator.Send(cmd)));
```

**The Pro:** It forces a strict separation of concerns. Your business logic stays out of your controllers, and every operation has a clear, discoverable handler.

**The Con:** It can feel like "class explosion." A non-trivial application will end up with a separate file for every Command, Query, and Handler in the system. The reliance on reflection also adds some overhead.

**Best For:** Teams following **Clean Architecture** or **Domain-Driven Design (DDD)** where decoupling is the top priority and a large community ecosystem matters.

---

## 2. The Productivity Powerhouse: ServiceStack

ServiceStack is an entire ecosystem, not just a library. It's opinionated, fast, and designed to minimise the friction between the server and the client.

```csharp
// One class defines the request, response, and route
[Route("/orders", "POST")]
public class CreateOrder : IReturn<OrderResponse>
{
    public string CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
}

// The service handles it
public class OrderService : Service
{
    public OrderResponse Post(CreateOrder request)
    {
        // ServiceStack handles serialisation, validation, auth, and caching
        return new OrderResponse { Id = Guid.NewGuid() };
    }
}
```

**The Pro:** It provides everything out of the box - authentication, caching, and auto-generated typed clients - without the overhead of piecing together multiple libraries. The auto-generated client support for TypeScript and C# is a genuine productivity win for full-stack teams.

**The Con:** It is a **commercial product** for teams beyond the free tier, and it has a distinct "ServiceStack way" of doing things that diverges from standard Microsoft ASP.NET Core idioms. Onboarding developers familiar with standard MVC patterns has a learning curve.

**Best For:** Small to mid-sized teams that want to move fast, avoid glue code, and want a consistent end-to-end story from the API to the client-side DTOs.

---

## 3. The Modern Contender: WolverineFX

Wolverine (from the JasperFx family) is the newest of the three, designed to be what MediatR would look like if it were built today for a cloud-native world. It acts as both a **command bus** and a **message broker**, with a focus on low ceremony and high performance.

```csharp
// No interface needed - Wolverine finds handlers by convention
public static class CreateOrderHandler
{
    // Convention: method named "Handle" with a matching command type
    public static async Task<OrderResult> Handle(
        CreateOrder command, 
        IDocumentSession session)
    {
        var order = new Order(command.CustomerId, command.Items);
        session.Store(order);
        await session.SaveChangesAsync();
        return new OrderResult(order.Id);
    }
}

// Minimal API wired directly to Wolverine
app.MapPost("/orders", (CreateOrder cmd, IMessageBus bus) => bus.InvokeAsync(cmd));
```

**The Pro:** It uses **Source Generators** to eliminate the performance cost of reflection used by MediatR. It also includes **Durable Messaging** with built-in Inbox/Outbox patterns, making it straightforward to build systems that don't lose data if a service crashes - a feature that would require a separate library like MassTransit to achieve with MediatR.

**The Con:** The community is significantly smaller than MediatR's, and there are fewer "how-to" guides and StackOverflow answers to lean on. Being newer also means patterns and best practices are still maturing.

**Best For:** Developers who want **maximum performance** and are building **Event-Driven Architectures** or microservices where reliable, durable messaging is a first-class requirement.

---

## Final Verdict: Which Should You Use?

There's no universally correct answer, but here's a practical decision tree:

- **Go with MediatR** if you are in an enterprise environment where Clean Architecture is the standard, you have a large team, and community support and available talent are key factors.

- **Go with ServiceStack** if you want a refined, industrial-grade framework that handles everything from the API layer to auto-generated client-side DTOs with minimal plumbing, and the commercial licence fits your budget.

- **Go with WolverineFX** if you want to be on the cutting edge of .NET performance, need built-in tools for asynchronous messaging and background processing, and are comfortable being an early adopter of a less-established ecosystem.

All three are production-ready choices. The decision comes down to your team's priorities: **community and convention**, **end-to-end productivity**, or **performance and native messaging**.

So what did I go with? For my new projects, I chose **WolverineFX**. The promise of source generator performance and built-in durable messaging was too compelling to ignore, especially as I build more event-driven systems. However, I still have a soft spot for MediatR and a penchant for ServiceStack. It can be really hard to break old habits and patterns, but the .NET ecosystem is evolving rapidly, and it's exciting to see new tools that challenge the status quo.