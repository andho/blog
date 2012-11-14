---
date: '2011-12-23 15:39:44'
layout: post
slug: talking-about-composer-a-package-manager-for-php
status: publish
title: Talking about Composer, a package manager for PHP
wordpress_id: '271'
comments: true
categories:
- PHP
- Programming
tags:
- autloading
- composer
- dependency management
- PEAR
- Php
- psr-0
---

There has been hype building around **Composer** and I am hyped about it. It's main user base is coming from within the **Symfony** community and looking at it's code, you can see the **Symfony** influence in it. And I tell y'all, this tool is pretty cool.

**Composer** is really more of a dependency manager and can be (usually is) used withing a project to define it's dependencies, and makes it easier for other developers to get those dependencies easily with a single command. It is not so mature yet but the development is rapid.

I also mention an idea I have about using composer for installing **PHP** toolsets. Read on.

.. more

What this post packs
====================
	
* Bitching about **PEAR**	
* **PEAR** vs **Composer**
* What **Composer** offers

  * Easy installation
  * Easy usage
  * Autoloading help
* Building toolsets with **Composer**

I bitch about PEAR here
=======================

The **PHP** community has been getting by with **PEAR**, and it's woe. It is slow to install something from a **PEAR** repo and sometimes I am not able to install any packages using **PEAR** at all. Examples would be **Behat** and **PHPUnit**. I was not able to install these packages through **PEAR** for whatever reasons for a time period of 24 hours or so, on two different occasions. Alternatively I had to get the packages from **github** and install them manually somehow.

Compared to Composer, PEAR is like a woman
==========================================

Admit it!!! **PEAR** is complicated. You have to add a channel first. Then you have to write the correct combination of characters that makes up the name of the channel and the package to install it. Wait...you forgot something. You need to upgrade the **PEAR** installer to the latest version. Okay, I'll just upgrade all **PEAR** packages: ``pear upgrade``........................

So let me run the installation again. What!!! There is no stable version available. I have to specify that I want beta? How do I do that. Hmmm. Let's just slap a '-beta' at the end of the package name and see if it works. Oh yeah, nailed it.

Moving on to another project. Oh..your unit tests are written for another version of **PHPUnit**? Oh well...I'm not gonna mention installation of PEAR itself.

Like I said, women are complicated.

Meet Composer, a bro for the throes
===================================

I'm not saying I prefer a guy to manage my dependencies, but **Composer** is welcome to do it. Here I talk about why I like **Composer**.

Installation
------------

Now imagine the installation instructions for **PEAR**, if it was not packaged with your OS package manager, or on **Windows** not packaged with **XAMPP**/**WAMPP**. Following are the installation instructions for composer:
	
    1. Download the |composer.phar|_ executable.

It's the best at "what it do"
-----------------------------

Say whaaat! You want **PHPUnit 3.5** to test your project. No prob, **Composer**'s got it under control. Okay listen up, here is how it's gonna roll. Project A wants its test to be run with **PHPUnit 3.5**. But Project B wants it's tests to be run with **PHPUnit 3.6**. And **Composer** wants you to chill, lay out a **composer.json** file for each project like so.

.. code-block:: javascript
   :caption: project-a/composer.json

    {
      "require": {
        "phpunit/phpunit": "=3.5"
      }
    }

.. code-block:: javascript
   :caption: project-b/composer.json

    {
      "require": {
        "phpunit/phpunit": "=3.6"
      }
    }

**Note: PHPUnit does not really support composer, yet!**

Now for each project, you have to get your dependencies: ``php composer.phar install``. **Composer** will install **PHPUnit** because it's specified in the composer.json as a requirement. After installation, you can most possible hypothetically run **PHPUnit**, the correct version for the project, with ``./vendor/bin/phpunit``.

Autoloading
-----------

**Composer** supports setting up autoloaders for the dependent libraries mentioned in your composer.json. It supports a variety of autoloading schemes too, including PSR-0.

It also supports copying a submodule into a subdirectory of the project directory. Let me explain with an example.

You have ProjectA which uses the namespace MyProject\Main\Stuff. You have a dependency on another repo 'Support', shared by many other projects in the MyProject namespace. You want it to load into MyProject\Support namespace. With **git** you just create a submodule in MyProject/library/MyProject/Support, and the autoloader registered with 'MyProject' namespace pointing to 'MyProject/library' as the base will work. That is assuming a PSR-0 compliant autoloading scheme used by your codebase.

But...this is not a big but...but this forces you to have that submodule repo with a rather naive directory structure. Take the class ``SupportOne`` of your submodule which is in the namespace ``MyProject\Support``. This file ``SupportOne.php`` has to be in the root of you 'Support' repository so that it can be submoduled into ``MyProject/library/MyProject/Support``. Got it, good, because I don't know how to explain it any better.

For more info about this checkout this `post on google groups <https://groups.google.com/forum/#!topic/composer-dev/NRTjqhu3e_o>`_.

Now something entirely new, toolsets
====================================

Sometimes you want some testing tools; **PHPUnit**, **PHPSpec**, **Mockery**, etc. It's all well and good when these are in your project as a dependency that can be installed using using **Composer**. But here's a better idea. As mentioned in `this post <http://nelm.io/blog/2011/12/composer-part-1-what-why/>`_ (last section), you can create a project inside your home folder and create a ``composer.json`` to get all the testing tools and put ``~/testing-tools/vendor/bin`` into your shell's include path.

That brings me to the idea of toolsets. Things like testing toolset, CI toolset, code introspection toolset, etc. What I'm talking about is git repos for each of these toolsets which the user can clone and compose, and viola he has all the tools installed with just a couple of commands. With **Composer**, it is easy to compose toolsets.

**Update**: I have created a PHP BDD toolset. `Check it out <https://github.com/andho/php-bdd-toolset>`_.

.. |composer.phar| replace:: **composer.phar**
.. _composer.phar: http://getcomposer.org/composer.phar
