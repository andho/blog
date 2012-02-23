---
date: '2011-10-18 20:14:58'
layout: post
slug: command-query-responsibility-segregation-in-php-cqrs
status: publish
title: Command Query Responsibility Segregation in PHP - CQRS
wordpress_id: '210'
categories:
- Architecture
- Domain Driven Design
- PHP
- Programming
tags:
- command query responsibility segregation
- cqrs
- ddd
- domain driven design
- es
- event sourcing
- Php
---

I've been playing around with CQRS recently and the pure simplicity of the concept and what it brings to the table is astounding. I have been contemplating about how I would go about implementing the domain architecture for a new project that I would be starting soon. The first decision I made was that it will be done using DDD, making the domain explicit in the code. But then how will I implement DDD. I drew diagrams, studied the current systems, etc.

<!-- more -->

The most important thing in DDD that has to be tackled is how you store the data and how storing that data will be abstracted from the Domain. Ofcourse it will be through a Repository. So I went about thinking how I will implement changing an Aggregate Root in to a serializable format and send it to the database. It was at this time that I saw the article "[Towards an Immutable Domain Model](http://blog.zilverline.com/2011/02/01/towards-an-immutable-domain-model-introduction-part-1/)". It was the first time that I read about [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html), and what I felt was persistence through Event Sourcing was simple, yet more effective. And so I kept doing more research on this until I found CQRS, and it saved me a lot of trouble, because I was struggling with using just Event Sourcing as my sole data source. So now I could use Event Sourcing for my Write Model and a traditional RDBMS (for now) for the Read Model. It was coming together.

Now that I've read about CQRS I had to think about how it could fit into my project, was it going to be a good thing. The idea of separating the Write Model from the complexities of the Read Model would be a plus. The Business Logic wouldn't have the unnecessary burden of dealing with query methods. And that was all there was to CQRS really. So my project architecture became a DDD+ES+CQRS architecture.

A team of two started to implement the CQRS+ES layer as well as a simple part of the Domain to go with it, and in 3 weeks into it we have a pretty basic architecture that works, which implements just an Event Store, Repository, Event Bus, a Command Bus, an Aggregate Root abstract class to deal with uncommitted events for each Aggregate Root and an Aggregate Factory abstract class that deals with initializing and applying Events to bring an Aggregate Root into it's current state. Right now the system is synchronous, all Events are stored and then denormalized into the Read Model.

Some of what remains unimplemented of the architecture for now is an Event Queue, which the Event Bus will publish to instead of denormalizing the Events itself, Event Handlers, which will denormalize or do whatever else with the data that needs to be done, and how to deal with concurrency.

I'll post more on it as it comes along. Also watch a sample app I'm making using ES+DDD+CQRS and a bit of extra sugar (Immutable Domain Model) at [https://github.com/andho/php-cqrs](https://github.com/andho/php-cqrs)
