---
date: '2011-11-24 21:39:40'
layout: post
slug: cqrs-command-testing-with-phpspec
status: publish
title: CQRS command testing with PHPSpec
wordpress_id: '225'
comments: true
categories:
- Behavior Driven Development
- PHP
- Programming
- Testing
tags:
- bdd
- behavior driven development
- cqrs
- es
- event sourcing
- PHPSpec
- testing
---

.. role:: strike
   :class: strike-through

In this :strike:`tutorial` "hypothetical tutorial" I am going to show you how to use **PHPSpec** to test your commands using gherking language style tests, or in layman's terms Given-When-Then style test. A final test written in this way will look like this:

.. more
    
.. sourcecode:: php
   :caption: TweetSpec.php

   <?php
   
   class DescribeTweet extends CqrsContext {
   
     public function itShouldBeRetweetable {
       $tweet_id = 1;
       $user_id = 1;
       $msg = 'Hello, this is a #CQRS tweet';
       $retweeter = 2;
   
       $tweet_created = new RetweetEvent($user_id, $msg);
   
       $retweet = new RetweetTweetCommand($tweet_id, $retweeter);
   
       $retweeted = new TweetRetweeted($retweeter);
   
       $this->givenForAggregateId($tweet_id)
         ->andTheFollowingPriorEvents($tweet_created)
         ->whenISendTheFollowingCommand($retweet)
         ->thenTheFollowingEventsShouldBeRaised($retweeted);
     }
   
   }

**Note:** I might make some assumptions about your CQRS implementation here, but I'm sure you can adapt this to your needs.

Read on if you find this type of testing interesting, easier or more concise. Please leave a comment why, if you think it is not.

The CQRS Context
================

The CQRS Context is simply that there should be a CommandBus that handles commands, a Repository that saves Aggregate Roots, and the dependencies of these two classes. With the CommandBus and Repository we can test "Given" a set of events and "When" the command is run "Then" the following set of Events should have been saved to the Event Store (a dependency of Repository). So that's what we are gonna do here.

**Note:** This CQRS Context assumes that the Domain is Event Driven and Event Sourcing is used for Domain State Persistence. I mention this as there is a lot of debate going on trying to keep CQRS separate from ES, DDD, Command Pattern and such. This implementation is just one of many ways to do CQRS but happens to be my implementation. You can take this idea to your own implementation, the purpose is to provide an idea for implementing this. If your Domain isn't Event Driven then you will need to inspect the Persisted State in a different way then Events, in the "then" part of the test harness.

**Disclaimer:** This code is not tested at all, but just written to give an idea of how to do this, although the code here is a slight modification of what I already use on a project.

For this to work, the Context needs to be defined, so we extend the PHPSpec\Context class. I will call it CqrsContext.

.. sourcecode:: php    
   :caption: CqrsContext.php

   <?php
   
   class CqrsContext extends PHPSpec\Context {
   
     private $_aggregateId;
     private $_store;
     private $_storage;
     private $_bus;
     
     private $_testStage = null;
     public function __call($method, $args) {
       $type = preg_replace('/ /', '', ucwords(array_shift($args)));
     
       switch ($method) {
         case 'given':
           $real_method = 'given' . $type;
           break;
         case 'when':
           $real_method = 'when' . $type;
           break;
         case 'then':
           $real_method = 'when' . $type;
           break;
         case 'and':
           if (is_null($this->_testStage)) {
             throw new Exception("You can't use 'and' before using 'given', 'when' or 'then'");
           }
           $real_method = $this->_testStage . $type;
         default:
           throw new Exception("No, you can't do that");
       }
   
       if (!is_callable($this, $real_method)) {
         throw new Exception('Undefined method ' . $method . ' called on blah blah');
       }
   
       return call_user_func(array($this, $real_method), $args);
     }
   
     public function before() {
       $this->_storage = new Cqrs\Event\Storage\ArrayStore();
       $this->_store = new Cqrs\Event\Store($this->_storage);
       $repository = new Cqrs\Repository($this->store);
       $this->_bus = new Cqrs\Command\Bus($repository);
     }
   
     public function after() {
       // helper method in ArrayStore specificially for this
       $this->_storage->clear();
       $this->_testStage = null;
     }
   
     public function givenForAggregateId($aggregateId) {
       $this->_aggregateId = $aggregateId;
   
       $this->_testStage = 'given';
   
       return $this;
     }
   
     public function givenTheFollowingPriorEvents($event1=null, $event2=null) {
       $args = func_get_args();
   
       if (is_null($this->_aggregateId)) {
         throw new \Exception("Don't know for which AggregateRoot this"
           . " Event is for\nPlease specify using givenForAggregateId"
           . " before calling givenTheFollowingEvents");
       }
   
       foreach ($args as $event) {
         if (!$event instanceof EventInterface) {
           throw new \Exception('Object of EventInterface Required');
         }
   
         $this->_store->setEvent($aggregateId, $event);
       }
   
       $this->_aggregateId = null;
   
       $this->_testStage = 'given';
   
       return $this;
     }
   
     public function whenISendTheFollowingCommand(Cqrs\Command\Command $command) {
       $this->_storage->recordNewEvents();
       $this->_bus->execute($command);
   
       $this->_testStage = 'when';
   
       return $this;
     }
   
     public function thenTheFollowingEventsShouldHaveBeenRaised(Cqrs\Event\EventInterface $event) {
       $events = $this->_storage->getNewEvents();
       $expected = func_get_args();
   
       // the events array contains key for each aggregate containing an array of events
       foreach ($events as $aggregate_events) {
           foreach ($aggregate_events as $actual_event) {
             $expected_event = array_shift($expected);
   
             // note I've change equal matcher to use non strict comparison
             $this->spec($actual_event)->should->beEqualTo($expected_event);
           }
       }
   
       $this->_testStage = 'then';
   
       return $this;
     }
   
   }

That is it, using this class you can run tests described at the top of this post. Thank you.



