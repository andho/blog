---
date: '2011-07-20 22:18:07'
layout: post
slug: modifying-private-or-protected-properties-in-php
status: publish
title: Modifying private or protected properties in php
wordpress_id: '6'
categories:
- PHP
- Programming
tags:
- Php
- private property
- protected property
- reflection
---

Yes it is possible and many of you might already know this. But I have searched for this a couple of times and never crossed this solution except until recently.

Now, when you talk about modifying private or protected properties you should be aware that these properties are private or protected for a reason. So I would like to tell you that this should be done only when it is absolutely necessary. This kind of code is not the kind of code that you come across when your strolling around the php street. This is inside that black forest that you see at the horizon.

.. more

I am using this functionality to implement a *magic* way to handle references in my object relational mapper. Although I am using this feature for now, I will always be looking for a more verbose solution to this, but now let's get to the point.

The answer is in the PHP Reflection API, the ReflectionProperty class in particular.

An Example
==========

Here is an example class we can work with:

.. sourcecode:: php
   :caption: BestClassEver.php

    class BestClassEver {
        public $goodStuff = 'bananas?';
        private $theBestStuff = 'secrets******';
    
        public function getTheBestStuff() {
            return $this->theBestStuff;
        }
    }

Now with this class you can easily access and/or modify the goodStuff property but theBestStuff can not be modified, you can only access it through the public method getTheBestStuff().

Reflecting on the BestStuff
===========================

So you need to use this class and absolutely this class, no subclassing (specially when multi inheritance is not possible). But you need the best stuff to be injected into it. Well, that's what Reflection API is for! (actually it is definitely not). Well here is what you need to do:

.. sourcecode:: php    

    $best_obj = new BestClassEver();
    echo $best_obj->getTheBestStuff();
    // echoes -> secrets******
    
    $beststuff_property = new ReflectionProperty('BestClassEver',
                                                 'theBestStuff');
    
    // magic is happening here -------------------------
    $beststuff_property->setAccessible(true);
    $beststuff_property->setValue($best_obj, 'me!');
    // magic is ending here ----------------------------
    
    echo $best_obj->getTheBestStuff();
    // echoes -> me!

That's it! :D

Now I hope you guys will use this as responsible adults. Hope it helps.
