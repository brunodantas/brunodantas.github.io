---
layout: post
title: "Cosmic Django"
description: "Let's learn how to apply Cosmic Python's architecture patterns to Django development while maintaining the framework's batteries-included philosophy. Explore domain modeling, service layers, and clean architecture in Django."
date: 2025-09-12
categories: [django, python, architecture]
tags: [cosmic-python, domain-driven-design, tdd, django, clean-architecture, software-engineering]
author: brunodantas
image: /assets/images/cosmic.png
seo:
  type: BlogPosting
---

[Cosmic Python](https://www.cosmicpython.com/) (aka Architecture Patterns With Python) is a very interesting book with great concepts. Though reading it a Django fan, I kept thinking how all that would apply to Django. So I implemented the example project while reflecting on best practices and referencing other sources.

If you look at their Django implementation in Appedix D, the authors do something weird with the project structure. It's like they think Django is an ORM as opposed to a web framework with batteries included... They totally threw the batteries away and did their own thing, and that's the opposite of what I want to do.

In this post, we'll go over each of the book's architecture patterns and discuss their applicability to Django projects. I'll also reimplement the example project using Django and all the patterns I decide on using.

Let me know what you think @ the [project repo](https://github.com/brunodantas/cosmic-django/issues).

## Starting out: TDD

I started my project with the `startproject` command, although I always have [Cookiecutter Django](https://github.com/cookiecutter/cookiecutter-django) in mind. The latter is aimed towards big production-ready projects though.

Surprisingly, Cosmic Python is a sort of sequel to another book about Test Driven Development. I don't know anyone who does TDD, but I'm open to different perspectives.

I decided to start with the final version of the test definitions because I don't want to do all the architecture/requirement changes that they do along the book.

I won't get into detail on them, but if you like tests, you can see their final form in the [repo](https://github.com/brunodantas/cosmic-django).

One of my goals with this project is making all tests pass. This seems challenging, but at least then I could say my project works.

## Domain Modeling

> This is the part of your code that is closest to the business, the most likely to change, and the place where you deliver the most value to the business. Make it easy to understand and modify.

That's an interest concept. If you've read the book, you know that they put this model stuff away from the ORM/DB, interestingly. It sure feels like throwing the baby out with the bathwater in the case of Django. They [kinda admit it](https://www.cosmicpython.com/book/appendix_django.html#_why_was_this_all_so_hard) by basically saying Django is just for simple CRUD apps. Which feels wrong. I wonder if they mean that Django practices are too coupled and messy for larger projects. I suppose they can be, but they don't *have* to be.

All that makes me want to skip on separate domain models and use the ORM models like everyone does. But I have to say, it actually feels like a compelling idea to have another separation layer between the domain and the infrastructure. Because doing both on the Django `models.py` feels like those things are coupled, the model is doing too many things, and the model code becomes a mix of business logic and DB operations and whatnot. And I'd say one of the things I find most interesting about the book is how much it emphasizes the principle of [Separation of Concerns](https://en.wikipedia.org/wiki/Separation_of_concerns).

Let's see what others have to say about Django models.

> Models should encapsulate every aspect of an "object," following Martin Fowler's Active Record design pattern.  
https://docs.djangoproject.com/en/5.2/topics/db/models/

> An object that wraps a row in a database table or view, encapsulates the database access, and adds domain logic on that data.  
https://www.martinfowler.com/eaaCatalog/activeRecord.html

The Django docs are firmly on the camp of the so-called *fat models*.

Note that the Active Record pattern apparently does three things. Is that too much? [Wikipedia](https://en.wikipedia.org/wiki/Active_record_pattern#Criticism) says it's too much`[citation needed]`.

If you google `active record bad`, you see people saying that it's ok for simple CRUD apps.

[Two Scoops of Django](https://www.feldroy.com/two-scoops-of-django) says the popular pattern called *fat models* can lead to the anti-pattern called *god object*. That aligns with what I think, meaning that models that do too many things are bad. Their solution is to separate business logic into helper functions somewhere else, which can them be segregated into a `core` app. This makes me think of some things we'll see later, namely the Service Layer, but how about domain models?

Looking at what the Cosmic authors' [domain models](https://github.com/cosmicpython/code/blob/appendix_django/src/allocation/domain/model.py) end up storing, it's the same data as the [ORM models](https://github.com/cosmicpython/code/blob/appendix_django/src/djangoproject/alloc/models.py). What differs is that one has business logic methods and the other has DB related methods... plus annoying-to-write mappers between the two. There must be a way to avoid this duplication and mapping same-y objects back and forth.

Which brings us back to the idea of helper functions. It's like the perfect middle ground. We can have an Active Record AND have a `core` app with our helper/utils functions. Who needs OO methods anyway? So the Two Scoops recommendation wins, in my opinion.

More specifically, we'll put the functions in a `logic` module.

## Repository Pattern

> We'll introduce the Repository pattern, a simplifying abstraction over data storage, allowing us to decouple our model layer from the data layer.

This is a pretty interesting pattern. Separating the storage layer and decoupling even more, and then doing dependency inversion so that the Repository calls the domain model. We can do that. But should we? [The authors](https://www.cosmicpython.com/book/appendix_django.html#_what_to_do_if_you_already_have_django) aren't sure either, saying it's a lot of work in Django's case, and that it may be good if you plan on migrating away from Django. Which brings me to the following post.

> For example: in general, people don't often swap out their data access layer without also doing other massive rewrites. And in Django-based applications it's even less likely to try to swap out the ORM without other huge code changes happening at the same time, because the Django ORM is probably the single most tightly-integrated component of the entire framework — if you stop using it, you're throwing away so much other stuff that "why are we even still using Django" becomes a really significant question.  
[https://www.b-list.org/weblog/2020/mar/16/no-service](https://www.b-list.org/weblog/2020/mar/16/no-service)

That is, if you get to the point of replacing the Django ORM, you may have bigger problems...

The other advantage is being able to unit test things with a mock DB more easily, which doesn't seem like a big deal IMO.

Other than that, the dependency inversion is super cool, but I don't see enough reason to add another abstraction layer to hide away the main storage of the system. If it was a secondary storage though, it would make a lot more sense. So I'm skipping this pattern.

## Service Layer

> It often makes sense to split out a service layer, sometimes called an orchestration layer or a use-case layer.

This pattern is another interesting separation of concerns. From what I understand, this is where we connect the business logic to the technical aspects. This interacts with the Repository (the (ORM) model in our case), does validation, calls domain functions. Seems like a nice straightforward addition to our project.

If only this pattern wasn't [*considered harmful*](https://www.b-list.org/weblog/2020/mar/16/no-service/) by Django people though...

But it's ok, because the service layer fits well with the choice we made back in Domain Modeling. This translates to a `service` module in our `core` app, which is endorsed by Two Scoops. And I'm sorry to disappoint my forefathers.

## Unit of Work

> If the Repository pattern is our abstraction over the idea of persistent storage, the Unit of Work (UoW) pattern is our abstraction over the idea of atomic operations. It will allow us to finally and fully decouple our service layer from the data layer.

This sounds like Django's [`transaction.atomic`](https://docs.djangoproject.com/en/5.2/topics/db/transactions/#django.db.transaction.atomic) with extra steps. Correct me if I'm wrong, but I think we're safe to use `transaction.atomic` instead and call it one of the included batteries.

For the sake of separation of concerns, let's put all our UoWs in our [Custom Managers](https://docs.djangoproject.com/en/5.2/topics/db/managers/#custom-managers).

## Aggregate Pattern

> An AGGREGATE is a cluster of associated objects that we treat as a unit for the purpose of data changes.

This chapter talks about managing business logic invariants by containing them in a new class... I don't like this very much.

Personally, I'd rather apply [Design by Contract](https://en.wikipedia.org/wiki/Design_by_contract) for that. Our contracts can check for preconditions and invariants. Then we can use something like [`deal`](https://github.com/life4/deal), or if you're into Functional Programming, [`ensures`](https://github.com/brunodantas/ensures). I'll be using `ensures`.

## Events and the Message Bus

> If we have two things that can be transactionally isolated (e.g., an order and a product), then we can make them eventually consistent by using events.

Unless I'm missing something, this is another included battery of Django: [Signals](https://docs.djangoproject.com/en/5.2/topics/signals/). Some say you should avoid them like the plague due to maintenance concerns, and it's a powerful way of decoupling things. Just be careful and try to implement good monitoring on them.

Following the conventions of this chapter, the Message Bus will be the Signals. Our UoW i.e. Managers would publish events, which will be received by the Domain (`logic` module) and by Handlers (`signal_receivers`). On the first iteration at least, because things quite a bit afterwards.

## Event-Driven Architecture

> In this chapter, we'll start to make events more fundamental to the internal structure of our application.

Even more decoupling. This is about managing system complexity when adding complex use cases. Basically, we'll have an API that fires events, which will be picked up by service handlers, which then interact with the rest of the system.

## Command Pattern

> Commands are sent by one actor to another specific actor with the expectation that a particular thing will happen as a result. When we post a form to an API handler, we are sending a command. We name commands with imperative mood verb phrases like "allocate stock" or "delay shipment."

This is about events that return values. Django Signals can do that too.

## Microservices

> We use events to talk to the outside world. This kind of temporal decoupling buys us a lot of flexibility in our application integrations, but as always, it comes at a cost.

Seems like a great idea. Skipping this because I don't want to deal with microservices in this project.

Let's say I chose the [Majestic Monolith](https://signalvnoise.com/svn3/the-majestic-monolith/) and that we don't a team big enough to do microservices.

> My First Law of Distributed Object Design: Don't distribute your objects  
https://martinfowler.com/bliki/FirstLaw.html

## Command-Query Responsibility Segregation (CQRS)

> Most Users Aren't Going to Buy Your Furniture

The idea of View models is awesome, but the specific approach is gonna depend a lot on the application. Let's see the options from this chapter.

- Just use repositories: *we don't do that here*.
- Use custom queries with your ORM: I'm gonna pick this for simplicity, and especially to illustrate how great [`Queryset.values`](https://docs.djangoproject.com/en/5.2/ref/models/querysets/#django.db.models.query.QuerySet.values) is for read-only data. I've found at work that ORM object instantiation from usual querysets can have quite a lot of overhead.
- Use hand-rolled SQL to query your normal model tables: can be performant and nice if you like SQL.
- Add some extra (denormalized) tables to your DB as a read model: can be a great option for specific troublesome fields.
- Create separate read stores with events: adds complexity but is probably the best option for big projects.

## Dependency Injection

> Dependency injection (DI) is regarded with suspicion in the Python world. And we've managed just fine without it so far in the example code for this book!

This is a good concept, and I think we can go even further by doing a kind of [Metaprogramming](https://en.wikipedia.org/wiki/Metaprogramming) with [Django Settings](https://docs.djangoproject.com/en/5.2/ref/settings/), keeping multiple settings files as suggested by Two Scoops, and selecting the settings module in our `pytest.ini`. This seem more powerful than the suggested bootstrap script.
