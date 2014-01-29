---
author: Pedro Gil Candeias
layout: page
title: Projects
date: 2013-02-20 16:49:37 +0000
comments: true
categories:
---

Personal projects, past and present, a tech overview.

### [Greenspace2](https://github.com/pgscandeias/greenspace)

A very incomplete, yet playable, clone of Neptune's Pride. The PHP backend is riddled with bugs because it was kind of an afterthought. My focus was client side: Knockout.js for data binding and Raphael.js for graphics. A powerful combination, apparently able to deliver a strategy game.


### [My previous blog](https://github.com/pgscandeias/blog)

When Posterous announced it was to shut down, I decided it was time to move to a lighter, *simpler* (there's a trend here) blogging system that let me write in Markdown. So I made this one.

It runs on custom php code, similar in form to microframeworks like Slim or Silex but with no unused features. It's little more than a routing system and a thin wrapper around MongoDB. It's lightweight and very fast.


### [Greenspace](https://github.com/pgscandeias/greenspace_x)

PHP pays the bills, but that doesn't mean I can't do other stuff. Greenspace is an exploration of Python, Node.js, MongoDB and messaging systems, taking the form of a turn-based space strategy game.

A browser client done in jquery and handlebars.js connects to a node.js frontend which in turns uses RabbitMQ to route requests to a Python backend which runs all (ok most) the business logic and database operations. Each element of the stack is there mainly due to a desire to experiment:

* ***Jquery + Handlebars*** for convenience
* ***Node.js*** it's fast and solves concurrency
* ***RabbitMQ*** separates concerns between frontend and backend
* ***Python*** needs no justification, it's just a sweet langage
* ***MongoDB*** is a good fit for the desired data model


### [Threddie](http://threddie.com) 2010

A simplified brainstorming app that melds real time chat with reddit-style threaded discussion.

Client side it's powered by jquery, achieving semi-real time updates via traditional polling with ajax. Server side it's a custom php5.3 MVC framework, lacking in features but light and fast.


### [Mercato](http://gomercato.com) 2012 (discontinued)

An extremely simplified e-commerce platform for small vendors, available only in the portuguese language.

A Symfony2 app running with the Doctrine2 ORM over a MySQL database. Client side there's jquery, ajax & a bit of responsive web design.
