---
author: Pedro Gil Candeias
layout: post
title: Reusing my own code with Composer
date: 2013-06-14 13:11:15 +0000
comments: true
categories:
---

## Composer

[Composer](https://getcomposer.org) is a package manager for php that's gained a lot of traction lately. It ties in nicely with version control systems like git, greatly simplifies dependency management, and has a lot of great packages at [Packagist](https://packagist.org/).

[Symfony](http://symfony.com), an extremely popular php framework, has been using Composer since version 2.1, which is only logical as both tools leverage the [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) interoperability standard. [Silex](http://silex.sensiolabs.org) too, Symfony's lightweight brother, uses Composer for installation and dependency management.


## In practice

Which brings me nicely to a product I'm currently building. It's a landing page platform for online marketing campaigns, and it's split in two sub-projects:

* The client and administration backoffices, made in Symfony;
* A separate website in Silex just for serving the landing pages and gathering data.

Both websites are written in php and talk to the same database, which means there's a clear opportunity for reusing the database access code. I could simply copy my Symfony entities over to the Silex application, but then I'd need to keep track of changes in my head and manually ensure both projects were up to date.

Or I could move the entities into their own bundle, turn it into a separate project versioned with git, and use Composer to install and manage it in both websites like a boss.


### Isolating the entities

I began by moving them into a separate bundle called simply My\EntitiesBundle, dropping them nicely in the `src/My/EntitiesBundle/Entity` folder and running a quick find & replace to update their namespaces and all the `use` statements. Because Symfony2 bundles follow the PSR-0, I was already halfway done.


### Creating a package

1) Move the bundle somewhere outside of the Symfony app. This will be the package.

    cd {symfony root}
    mkdir -p ../entitiesbundle/lib/My
    mv src/My/EntitiesBundle ../entitiesbundle/lib/My

2) Then create `composer.json` at the package root

    {
        "name": "my/entitiesbundle",
        "type": "library",
        "description": "Doctrine entities for the Tranquility project",
        "license": "private",
        "authors": [
            {
                "name": "Pedro Gil Candeias",
                "email": "pedro@pedrogilcandeias.com"
            }
        ],
        "require": {
            "php": ">=5.3.0",
            "friendsofsymfony/user-bundle": "*",
            "doctrine/orm": "~2.2,>=2.2.3",
            "doctrine/doctrine-bundle": "1.2.*"
        },
        "autoload": {
            "psr-0": {"My": "lib/"}
        }
    }

Notice the `require` property. It tells Composer which packages this one depends on, so that it can install them. A php version can also be specified; Composer doesn't handle php upgrades but it can throw an error if it finds the current php installation doesn't match the requirements.

Also notice the `autoload` property. It's there to tell PSR-0 compatible autoloaders that the `My` namespace maps to the {bundle root}/lib folder.

3) Initialize a git repository for the package

    cd {package root}
    git init
    git add .
    git commit -m "Initial commit"

Composer leverages git tags and branches, mapping them to version and stability levels. This lets you specify exactly what version you want for your project.

In recap, I ended up with this file structure...

    entitiesbundle/      Doctrine Entities (Composer package)
      |_ .git
      |_ .gitignore
      |_ composer.json
      |_ lib/
        |_ My
          |_ EntitiesBundle
            |_ ...

... and a package ready for distribution. If I wanted to I could now push it to github, bitbucket or pretty much anywhere else: Composer can use repositories other than Packagist, as we'll see in a minute.


### Installing the package

Time to tell the Silex app to install it. This meant of course opening its `composer.json`. I should explain both packages and projects use that file. The one in a package holds its metadata like name, author identification, namespace and dependencies; the one in a project is basically just a list of dependencies. This is what the file on the landing pages project looked like initially:

    {
        "require": {
            "silex/silex": "1.0.*@dev",
            "twig/twig": ">=1.8,<2.0-dev",
        }
    }

Pretty simple, basically just requiring silex and twig. They both declare a bunch of dependencies of their own, but Composer resolves and downloads all that for me.

Here's the file after declaring my package:

    {
        "repositories": [
            {
                "type": "vcs",
                "url": "/Users/pedro/project/tranquility/entitiesbundle"
            }
        ],
        "require": {
            "silex/silex": "1.0.*@dev",
            "twig/twig": ">=1.8,<2.0-dev",
            "my/entitiesbundle": "dev-master",
            "dflydev/doctrine-orm-service-provider": "1.0.*@dev"
        }
    }

That `repositories` bit tells Composer where to look for packages *in addition* to Packagist. In my case, it's a local path under version control.

`"my/entitiesbundle": "dev-master"` is the line that actually declares my package as a dependency. `"dev-master"` tells it to pick up whatever's in the master branch, regardless of stability.

`"dflydev/doctrine-orm-service-provider": "1.0.*@dev"` is a nice service provider for Silex (also Pimple and Cilex) that simplifies the use of Doctrine ORM. But that's beyond the scope of this piece.


### Fun time

    cd {silex root}
    composer update

And we're done! Assuming the [Doctrine ORM Service Provider](https://github.com/dflydev/dflydev-doctrine-orm-service-provider) is set up correctly, I can now reuse my doctrine entities in the Landing Pages sub-project. Whenever I make a change to them, just have to commit it and run `composer update`. This last step can be automated with a post-commit hook.


## Conclusion

We all know code duplication is bad and modularity is good. The adoption of PSR-0 goes a long way towards making code distributable and therefore reusable, but by itself is insufficient. A good package management tool like Composer helps in that regard.

Building modular code also encourages the developer to think of a web application not as a tightly integrated stack, but as a collection of components each doing a specific job. This is good because decoupled code is testable code. And testable code, covered by a good test suite, saves time on any non-trivial project under active development.

Reusing my Doctrine Entities in the Silex sub-project might not have been strictly necessary, as I could have simply queried the database directly, but it was a great way of learning about Composer from the perspective of actually building packages for it and made me reflect on practical ways of implementing modular design for my code.