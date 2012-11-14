---
date: '2012-01-20 14:46:03'
layout: post
slug: towards-an-immutable-domain-model-in-php
status: open
title: Towards an immutable domain model, in PHP
wordpress_id: '178'
published: false
comments: true
---

I first saw the application of immutability to a domain written on ZilverBlog: [Towards an immutable domain model](http://blog.zilverline.com/2011/02/01/towards-an-immutable-domain-model-introduction-part-1/). I instantly found that this would be really helpful in modelling a complex system. This article is about how I (tried to) implemented an Immutable Domain Model in PHP.

<!-- more -->

## But why, you say?


Eric Evans says in his book 'Domain Driven Design' that making objects immutable is a good way to reduce complexity. If you are keeping up with the programming world, you will notice that the hype is towards Functional Programming, departing from the hype about Object Oriented Programming Paradigm.


## Okay so here we go! Immutability here we come!


So what I am gonna do is implement the same thing that was done in the ZilverBlog article, but I'm trying to do it in PHP. This will be difficult as PHP is not a functional programming language and is not as powerful as Scala (which is used in ZilverBlog). Here is the initial Invoice class, there are Doctrine2 based annotations so that it would be a transition from an ORM based Domain to an Event Sourced Immutable Domain.

    
    Invoice.php
    [cce_php]
    <?php
    
    class Invoice {
      private $_items;
      private $_totalAmount = 0;
      private $_sentDate;
    
      public function isSent() {
        return !is_null($this->_sentDate);
      }
    
      public function removeItem() {
        if ($this->isSent()) {
          throw new Exception('Items cannot be changed after invoice is sent');
        }
    
        $this->_totalAmount -= $this->_items[$index];
        unset($this->_items[$index]);
      }
    
    }
    [/cce_php]




Now the first thing we need to do is to introduce Event Sourcing. Other wise persistence will be a mess, you'll see.


## Monads! Ummm...


I haven't

Noway easy way to implement monads, I want self mutating objects.
