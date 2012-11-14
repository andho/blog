---
date: '2011-09-21 16:46:00'
layout: post
slug: getting-to-know-php-namespaces
status: publish
title: Getting to know PHP namespaces
wordpress_id: '21'
comments: true
categories:
- PHP
- Programming
tags:
- namespace
- Php
- programming
---

.. role:: code
   :class: inline-code

Introduction
============

Namespaces has been introduced in PHP since version 5.3, and it has been gaining traction over the months. It's not just namespaces but all the features of PHP 5.3 is catching on and helping developers move forward from where they were stuck before.

.. more

Why use it?
===========

With the introduction of namespaces comes better code organization. If you want to your code to be manageable you have to name them descriptively. This means that class names might become very long. How name spaces help in these situations is by letting you choose to import a namespace and then you have to write only the part of the class name that comes after the namespace.

For example you have a number of classes that helps to deploy code to servers. So the namespace could be \Joes\Deployment\Strategy and in this namespace you can have several strategy classes: Ssh, Git, Script, Phpunit, etc. Now you might have defined those classes as follows.

.. sourcecode:: php
   :caption: library/Joes/Deployment/Strategy/Ssh.php

    namespace \Joes\Deployment\Strategy
    
    class Ssh extends \StrategyAbstract {
        ... implementation here
    }

.. sourcecode:: php    
   :caption: library/Joes/Deployment/Strategy/Git.php
    
    namespace \Joes\Deployment\Strategy
    
    class Git extends \StrategyAbstract {
        ... implementation here
    }

So in the above two files you have succesfully created two classes in the :code:`\Joes\Deployment\Strategy` namespace, i.e. Git and Ssh.
	
* The first line :code:`namespace \Joes\Deployment\Strategy` tells the interpretter that all functions and classes that comes under this line should be in the :code:`\Joes\Deployment\Strategy` namespace.
* The class definition :code:`class Ssh extends \StrategyAbstract {` has two important bits, first is that the class name is Ssh, but it is actually defined under the \Joes\Deployment\Strategy namespace. Secondly the \StrategyAbstract has a backslash at the front meaning that StrategyAbstract class is from the root namespace. All built-in classses and functions, and classes and functions defined without a namespace is in the root namespace.


Now the Ssh and Git class can be accessed from other files, but either the namespace has to be imported or you have to specify the full name of the class (including the namespace) each time you use it.

Usage without importing namespace:

    
.. sourcecode:: php

    $ssh_deploy = new \Joes\Deployment\Strategy\Ssh(
        '192.168.0.3',
        'root',
        '',
        '`mv dir newdir`'
    );


Here I have used the whole class name. You can skip the first backslash, like so: :code:`$ssh_deploy = new Joes\Deployment\Strategy\Ssh(...);` if you are in the root namespace but if you are in any other namespace you have to remember to put the starting backslash.

Usage after importing the namespace:

    
.. sourcecode:: php

    use \Joes\Deployment\Strategy;
    
    $ssh_deploy = new Strategy\Ssh('192.168.0.3', 'root', '', '`mv dir newdir`');

Or better yet:

.. sourcecode:: php

    use \Joes\Deployment\Strategy\Ssh as SshStrategy;
    
    $ssh_deploy = new Strategy\Ssh('192.168.0.3', 'root', '', '`mv dir newdir`');

Autoloading
===========

There is a `pretty good autoloader <http://www.invenzzia.org/en/download/open-power-autoloader>`_ that supports namespaces and old style of separating with _ (underscores). Here is a `good article about Open Power Autoloader (Opl) <http://www.zyxist.com/en/archives/140>`_. It explains the features and how to set it up.
