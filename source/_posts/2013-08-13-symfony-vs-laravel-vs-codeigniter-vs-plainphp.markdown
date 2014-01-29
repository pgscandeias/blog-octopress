---
author: Pedro Gil Candeias
layout: post
title: Symfony vs Laravel vs Codeigniter vs Plain PHP
date: 2013-08-13 10:12:24 +0000
comments: true
categories:
---

I had a couple of hours to kill on a train journey recently and decided to compare a few php frameworks just for kicks. The goal of the test was twofold:

* Measure performance overhead
* Compare the coding styles encouraged by each framework

A Debian7 / nginx  / php 5.5 VM was spun up using Vagrant, with pretty much the default settings. Opcache was enabled, but otherwise no caching was involved. A mysql 5.5 database was created, with a 'custmers' table holding 200 rows. Benchmarks were to be conducted on a page displaying the first N customers, as specified by a 'count' querystring parameter. A benchmark was conducted upon a static file in order to get a baseline performance reading.

**Disclaimer**: these tests were **not** conducted with anything resembling statistical rigour. And, as is the case with all things benchmarky, your mileage may and will bloody well vary.

With that out of the way...


*Setup*

    Host:   OSX 10.8.4 / Intel i5 2.3Ghz / 8Gb / SSD
    VM:     VirtualBox + Debian Wheezy / 1Gb

    Stack:  nginx 1.4.2 / php5-fpm / php 5.5.0 / mysql 5.5.31

*Read top $count records from the customers table and render a template with that information.*

    ab -c 5 -n 1000 http://bench.(*).dev/customers?count=5


## Results

    Framework           ORM         Templating  Req/s       Time /req

    static file         -           -           2654.53     1.884

    Symfony 2.3.5       Doctrine    Twig        50.83       98.365
    Laravel 4           Eloquent    Blade       54.43       92.006
    Codeigniter 2.1.4   CI_Model    php         302.28      16.541
    Bespoke PHP         -           php         433.33      11.538
    Single file php     -           php         1624.66     3.078



**Bespoke PHP** was a piece of custom code consisting of a light routing library matching url patterns to anonymous functions in the style of microframeworks like Slim and Silex.

**Single file php**, like the name implies, was a single file with database access at the top and html mixed with php at the bottom. Just like old times.


## Thoughts

**Symfony** has the greatest overhead, as expected. Every single request runs through lots and lots of classes, providing hook points for injecting code at any point; an opcode cache mitigates the problem, but there's no way around running lots of code.

The good news is that, with PSR-0 autoloading and a decent dependency injection container, it's quite clever about what it runs after the bare minimum. Even if you have a huge application with lots of models, controllers and assorted libraries, it only loads what it needs for each request. So yes, all of Symfony's flexibility implies quite some base overhead, but resource consumption doesn't necessarily grow with the application.

There's a lot of quality bundles available at [Packagist](https://packagist.org/search/?q=symfony), documentation is second to none and, despite heavy development, stuff rarely breaks on new releases. When something does break, chances are someone caught it before you did and documented the solution - which is usually pulled into the next release very quickly.

Symfony configuration does tend to be quite dense and its namespacing practices do lean towards the verbose. But on the other hand, everything is nicely decoupled. Testing and reusing code comes very naturally in a Symfony project.


**Laravel** is touted as a great alternative to Codeigniter, mixing its simple interface with the power of Composer-driven package management. It uses a LOT of Symfony components, to the point where I find using Laravel is a bit like running Symfony with different ORM and templating engines. It sits somewhere between Silex and Symfony in complexity and available toolset.

Why they introduced yet another ORM and templating engine, though, is beyond me. Blade has an inconsistent interface, using brackets for output and @ signs for control structures. Eloquent is... a bit weird. Better than Codeigniter's active record implementation for sure, but it felt about as ugly to use. Laravel also makes extensive use of Singletons, which I find very strange in a framework that's based on Symfony. Singletons are a bit of a cheat, essentialy serving as global variables. They have their uses, but have little raison d'être when a dependency injection container is available.

I was a bit surprised at Laravel benchmarking neck and neck with Symfony in terms of speed. But considering both frameworks are based on the same foundation, this test was quite simple and Symfony only loads what it needs, the results make sense.


**Codeigniter** benefits tremendously from its inherent simplicity in this sort of benchmark. In this scenario, CI executes little else beyond database initialization and route matching before calling the controller so it does go like the clappers as advertised. Trouble is, it's not very bright. Its router has no knowledge of HTTP methods other than GET, its request handler only adds POST to that list, there are no options to serialize responses to anything other than html and it proudly advocates plain old php as the templating engine. Which is a fine choice if you're looking for raw speed but not very good for maintainability at all. And the less said about CI's idea of an ActiveRecord implementation, the better.

Then there's the killer. Codeigniter, having no idea the PSR0 exists or even that php5.3 is out, does not employ namespacing. To prevent class name collisions, its core classes have silly olde style names like `CI_Model` and `CI_DB_MySQL`. And should you wish to use libraries, or plugins, or helpers, which so far as I can tell differ only in the folder they're installed in, you must declare them all in the bootstrapper. Which duly loads them to memory. It's easy to see how this can become a problem on larger applications with lots of libraries.

On simple projects, Codeigniter is good enough. On more compelex ones, its shortcomings can be overcome by simply relegating it to the role of router and using Composer to install a better ORM, an actual templating engine, whatever libraries are needed, etc. But then why use CI at all? It'll just be a bad router getting in the way of your application.

Codeigniter was great in its day. That day is now well and truly past, and no amount of `Hello World` speed can make up for its flaws.


## Closing words

The very act of using a framework is a choice of convenience over speed. Frameworks bring order to chaos, speed to both early and late stages of development. That convenience costs performance, but we use them anyway because, in most places, developer time is way more expensive than server time. In most places, most of the time, you can easily add more servers to your frontend cluster or beef up your database machines. But hiring new skilled developers is hard enough; convincing good developers to work in messy, unstructured codebases these days is a hard sale indeed.

I like Symfony. It makes development logical and, dare I say it, easy. I've been using it for most of my personal projects and all of my freelancing ones for over two years now. There's a steep learning curve, but the exceptional documentation makes it easy to climb. And it encourages the use of good programming practices like inversion of control, dependency injection, single resposibility principle, and more. Performance wise, commodity servers like DigitalOcean's $5 vps can run it at well below 100ms per request under moderate loads, making scale issues moot for web apps in their early stages. I wouldn't run an ad network on it, though.

If I did have to deploy something like an ad network in php, I'd probably go for bespoke code on the frontends *but* code the backoffices in Symfony simply due to how good the toolset is and how maintainable Symfony projects end up being. I'm not convinced Laravel, with its limited configuration options and opinionated development practices is a better choice, especially since it doesn't bring a significant performance boost to the table. It may be easier to get into, though, which is in itself a huge plus and just might make it the right framework for many projects.

As for Codeigniter, well, we have a big project running on it at work and lets just say thing aren't getting any easier.