---
date: '2011-07-27 21:43:45'
layout: post
slug: behavior-driven-development-with-php-and-phpspec-web-runner
status: publish
title: Behavior Driven Development with PHP and PHPSpec Web Runner
wordpress_id: '24'
categories:
- Behavior Driven Development
- PHP
- Programming
- Testing
tags:
- bdd
- http runner
- Php
- PHPSpec
- PHPUnit
- programming
- testing
- web runner
---

.. role:: code
   :class: inline-code

..

Behavior Driven Development, BDD for short.
===========================================

If you have been using `PHPUnit <https://github.com/sebastianbergmann/phpunit>`_ for Unit Testing you already know the benefits of automated testing and once you get used to setting up Unit Tests for your code it gets easier. Now PHPUnit is great if you are following Test Driven Development, but if you want to write even more verbose code you could use BDD.

In BDD the Unit tests are the documentation. You write the class names and methods of the unit test exactly as you would write a requirements specification for that functionality, and this will make tests itself the documentation for the project.

.. more

It also helps when writing the actual unit test code because the method name explains clearly what you need to do. It will also help to show to the stake holders that your code passes the acceptance tests if the test results read like natural language and is 1:1 with the specification.

Checkout this great article on `BDD with PHP using Behat <http://techportal.ibuildings.com/2011/07/27/behaviour-driven-development-in-php-with-behat/>`_.

PHPSpec
=======

`PHPSpec <http://www.phpspec.net/>`_ is also a Testing Framework like PHPUnit written for PHP5.3.3 and above. It is not as mature as PHPUnit but is pretty well written, as of this writing it is in **1.2.2beta**, which is <del>not</del> available at their download site. You can get the latest source at their `Github page <http://www.github.com/phpspec/phpspec>`_. I have not come across any problems using it although it lacks a good Mocking framework to go with it. Currently I am using `Mockery <https://github.com/padraic/mockery>`_, but it is not stable yet, they also have `phpspec-mocks <https://github.com/phpspec/phpspec-mocks>`_ which I haven't been able to test yet.

Web Runner
==========

There isn't something really called web runner but I'd say it's the appropriate name. Some people like to run their Unit Tests in the command line but me, I like to run the unit tests in the browser. Here is why:

Advantages
----------

* Nice look, you can see your tests results in a beautifully designed page with nice colors. Red for bad and green for good.
* Easy to debug your unit tests, because both Netbeans and Zend Studio provide an easy way to debug remote code from the browser. I have recently come across an article that enables `debugging zend framework unit tests with xdebug and netbeans <http://robertbasic.com/blog/debugging-zend-framework-unit-tests-with-xdebug-and-netbeans/>`_.
* Auto refresh? Don't think this is a good idea :)

Disadvantages (or advantages of CLI runner)
-------------------------------------------

* PHPSpec is made to run in the CLI as are most other testing tools, so they are likely to be more features and ease in using the CLI.
* Most docs or tutorials will use the CLI.
* Debugging might be possible even on CLI, see the link in advantages.
* You can see live progress in the CLI, where as, in the browser you get the final output.
* Continous integration tools use CLI so your testing code has to work on the CLI.

Getting Started
===============

On second thought you can get better documentation at the `PHPSpec Documentation <http://www.phpspec.net/documentation/>`_, although it doesn't show how to setup the web runner (update: documentation now has info on setting up Http Runner). They have updated their documentation and is now pretty complete, where as when I started writing this it was almost empty and no instructions on setting it up. So if you want to know how to setup the web runner instead of console runner then read on, but by the time you read this they might already have updated their documentation. If so please leave me a comment and I will link it here.

I am going to show here the steps to get PHPSpec setup and how to test your code with it. I am going to get the code from github.com and setup it up to use the Web Runner which allows you to view the test result in the browser. Most of you might prefer running the test from the command line but I haven't found a way to specify a bootstrapper other than writing a script to run the phpspec inside it. When I get it running in the command line how I want, I will post it here.

**Note:** PEAR is the recommended way to install PHPSpec (or anyother testing framework). Why? Because it belongs in your development toolset and is not project specific, and every developer (PHP) should have it in their path.

1. Getting the code
-------------------

First you need to get the code. There are three ways to do that, you can see that in the `download page <http://www.phpspec.net/pages/download/>`_ (or `Installing PHPSpec <http://www.phpspec.net/documentation/installing-phpspec.html>`_). I am going to highlight using a non-Pearified tar ball (or zip), as they call it, to get the source.

So, you can get the source code and keep it where ever your (linux) user has permission to read it (not applicable to windows, mostly). For the sake of completeness, I am going to keep the source in :code:`~/src/phpspec`. The :code:`~` means 'home directory' in bash shell, also refered to by the :code:`$HOME`. From here on after the 'Source Directory' and '~/src/phpspec' will be used synonimously.

First lets create the src directory:
    
.. sourcecode:: console
   :caption: shell

    mkdir ~/src

Getting source by downloading.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This option is for those who don't want to, or don't know how to, use git. You can get the source tar ball at `phpspec github page <http://www.github.com/phpspec/phpspec>`_. Go there and you can find a big 'Download' button on the right. Click it and it will give you download options. As the tags are offered in zip, I am going to continue this with zip based instructions. Once you get the zip file copy it into :code:`~/src`. Then run the following command from the source folder to extract it:

.. sourcecode:: console
   :caption: shell
    unzip PHPSpec-<version>.zip

Getting the source by using git.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As PHPSpec uses git for hosting their code, we can use git to get the source code, which is the way I prefer.

Goto :code:`~/src` from the command line and then run:
    
.. sourcecode:: console
   :caption: shell

    git clone https://github.com/phpspec/phpspec.git phpspec

This will actually get the most recent code commits which might be unstable. You can get a release (or tag) with the following code:

.. sourcecode:: console
   :caption: shell

    cd phpspec
    git tag
    git checkout <tag-name>

The :code:`git tag` command will show all the tags available. Then you can run :code:`git checkout <tag-name>` to checkout the specific tag. For example :code:`git checkout phpspec-1.2.2beta` will checkout the lastest version as of now.

2. Changing php.ini include path
--------------------------------

If you installed using PEAR, the pear include path, usually at :code:`/usr/share/pear`, will contain your code and is already in the php.ini include path. Otherwise you can put the :code:`~/src/phpspec/src/` into your php.ini include path. Changing the php.ini file will not be covered in this tutorial, but you can probably find out how to if you search the web (aka.googling).

3. Setting up the command line (optional)
-----------------------------------------

**Note:** You will already have the command line setup if you used the PEAR installer from the Installing PHPSpec page I have linked above.

Now this step is optional. If you want you can make it run from the command line with this.

.. sourcecode:: console    
   :caption: shell

    ln -s /usr/bin/phpspec /home/yourusername/src/phpspec/scripts/phpspec.php
    chmod +x ./phpspec.php

**Disclaimer:** this doesn't work for now, but will let you in on some changes you can bring to this file in 'Setting up the webrunner' section. And this should be fixed in a future version also.

Here I have used :code:`/home/yourusername/src` instead of :code:`~/src` because this command works best if you are providing an absolute path.

And one more thing, you will need to add the path :code:`/home/yourusername/src/phpspec/src` to you php.ini include path. I think this won't be necessary in newer versions as I have commited a patch to detect the src directory automatically.

Then you can run the tests like this.
 
 .. sourcecode:: console  
   :caption: shell

    phpspec /path/to/testfile.php


4. Setting up the webrunner
---------------------------

What I have done for the webrunner is use the same script in :code:`~/src/phpspec/scripts/phpspec.php` and changed it a little.

See the modified file below. I will highlight what I have changed after the file. This file has to be under the web root. I have put it in :code:`/var/www/phpspec/webrunner.php`, my web root being :code:`/var/www`. All files following are created under this same directory.

.. sourcecode:: php
   :caption: webrunner.php

    <?php
    
    ini_set('display_errors', 1);
    error_reporting(E_ALL|E_STRICT);
    
    $dir = '/home/yourusername/src/phpspec/src/';
    
    require_once $dir . 'PHPSpec/Loader/UniversalClassLoader.php';
    $loader = new \PHPSpec\Loader\UniversalClassLoader();
    $loader->registerNamespace('PHPSpec', $dir);
    $loader->register();
    unset($dir);
    
    $testdir = './';
    
    $phpspec = new \PHPSpec\PHPSpec(array($testdir, '-f', 'h'));
    $phpspec->execute();

The lines I have changed/added are:

* **Line 6 (added)**: This sets the path where the PHPSpec files are. If you installed using **PEAR**, or put the source folder in the **php.ini include path**, you don't need this line.
* **Line 8 (changed)**: I put :code:`$dir .` right before the file name so the require_once directive is getting the full path to the file. Not necessary if you installed using **PEAR**, or put the source folder in the **php.ini include path**.
* **Line 10 (changed)**: The second argument to :code:`$loader->registerNamespace('PHPSpec', $dir);`, the $dir variable, was :code:`'/usr/bin/pear'` which I changed to :code:`$dir` so that it points to the correct phpspec library path. This is necessary even if you have installed using **PEAR** (and your PEAR path is not '/usr/bin/pear') or set the **php.ini include path**. If you installed using **PEAR** you need to find out where you pear path is, and set it to $dir in line 6.
* **Line 12 (added)**: Unsetting the $dir variable, not really necessary but why leave this stale variable dangling.
* **Line 14 (added)**: the path where the tests are, assuming the tests are where webrunner.php is, it is set to :code:`'./'`
* **Line 16 (changed)**: Original script passed $argv as the parameter, but $argv comes from the command line, so we need to pass our own argument array in. The first value of the array is :code:`$testdir` which we set in line 14. So we are passing the file/directory to test. PHPSpec will traverse the directory looking for files if a directory is passed. The second argument is '-f', which is used to specify the format. It needs a format identifier passed after it, which is the third argument, 'h' which is short for 'html'. This tells the PHPSpec runner to use an Html formatter, instead of the default Progress formatter.

I mentioned in the 'Setting up the command line' section that I will highlight some changes that will make it run correctly. Well above are the changes except for Line 16. And excluding Line 16 also excludes line 14. Thats it.

What the code will do is run all the test classes under that directory that starts with 'Describe'. But first we need to write some tests.

5. Now reap the benefits
------------------------

Now let's get to testing, write this in a new file :code:`DescribeSummerSum.php`. PHPSpec requires the test class to be prefixed with 'Describe', other wise it won't detect it as a test, so you can have many classes in any of these files and they won't be treated as tests. Why the name DescribeSummerSum? The file describes sum method of summer class. Makes sense.

.. sourcecode:: php
   :caption: DescribeSummerSum.php

    <?php
    
    class DescribeSummerSum extends PHPSpec\Context {
    
        protected $summer;
    
        public function before() {
            $this->summer = new Summer();
        }
    
        public function after() {
            $this->summer = null;
        }
    
        public function itShouldReturnTheSumOfTwoNumbers() {
            $a = 1;
            $b = 2;
    
            $this->spec($this->summer($a, $b)->should->equal(3);
        }
    }

Here, we are testing a class called Summer. The test says the Summer's sum method should return the sum of two numbers. Now you can point your browser to the bootstrap file: `webrunner.php <http://localhost/phpspec/webrunner.php>`_. That is assuming all these files were created under /phpspec/ under your webservers document root. You can expect something like this (if not, drop me a comment).

.. image:: http://blog.andho.com/wp-content/uploads/2011/07/Selection_008.png
   :alt: Summer not found fatally

6. Small Steps
--------------

Well ofcourse it would fail, we haven't created the Summer class yet. Now let us create :code:`Summer.php` and the Summer class:

.. sourcecode:: php
   :caption: Summer.php

    class Summer {
    
        public function sum($a, $b) {
    
        }
    }

The interface is finished, let's test what happens. Now refresh the webrunner.php page. Oh wait, remember to require this file on the top of your :code:`DescribeSummerSum.php` file. You should see something cool like the following:

.. image:: http://blog.andho.com/wp-content/uploads/2011/07/PHPSpec-results-Mozilla-Firefox_010.png
   :alt: Failed PHPSpec result

7. Making an effort to make it work
-----------------------------------

Now let's fill out the implementation.

.. sourcecode:: php    
   :caption: Summer.php

    class Summer {
    
        public function sum($int1, $int2) {
            return $int1 + $int2;
        }
    
    }

Aah, Summer is beautiful. Now let's run our test again. Now refresh the testing page. You should be expecting something like this:

.. image:: http://blog.andho.com/wp-content/uploads/2011/07/PHPSpec-results-Mozilla-Firefox_011.png
   :alt: Successfull PHPSpec result

Soooooo, that's it.

Hats Off
========

.. Padraic Brady's hyphenated a, unicode character kills restructured text render

Big thanks to `Padraic Brady <http://blog.astrumfutura.com>`_, Marcello Duarte and the PHPSpec Development team, for this amazing project.
