---
date: '2011-11-05 13:41:17'
layout: post
slug: junit-xml-formatter-for-phpspec-compatible-with-jenkins
status: publish
title: Junit.xml formatter for PHPSpec, compatible with jenkins
wordpress_id: '233'
comments: true
categories:
- Continuous Integration
- Programming
- Testing
tags:
- ci
- continuous integration
- formatter
- jenkins
- junit.xml
- PHPSpec
---

## I needed a Junit Formatter


Currently I am doing my testing using [PHPSpec](http://www.phpspec.net/), and Continuous Integration using [Jenkins CI](http://jenkins-ci.org/). As PHPSpec doesn't currently have Junit.xml output format the XUnit plugin for Jenkins cannot show the test results, so I set out to create a Junit Formatter for PHPSpec.

There was already a [feature request for a Junit Formatter](https://github.com/phpspec/phpspec/issues/33) on the [PHPSpec github](https://github.com/phpspec/phpspec) page. And a user named xenji (who also made the request) had[ started to work on it](https://github.com/xenji/phpspec), so I decided to check it out. I pulled the changes from his repo and ran my tests with -f j, which indicates that I want the output in junit format and I got a plain text result. So I checked the source and it was just a skeleton mostly. As I needed the Junit format really soon I started to hack on this.

<!-- more -->

## So I hacked it


In my hurry, I forgot to create a new branch for this feature so all the commits ended up on the develop branch. Will be difficult to pull them out separately.

When I thought about it, the Junit formatter would be the same as the Html formatter, it's just the tags and stuff that would change. So I checked the Html formatter and it was using the observer pattern to hook into three events, SuiteStarted, SuiteFinished and RunExample. I moved these events to the Progress formatter which is the base class of Html and Junit formatter and used those methods to inject the correct output.

The next thing I needed was the actual format for the Junit.xml accepted by Jenkins. My first clue was from [this stackoverflow question](http://stackoverflow.com/questions/4922867/junit-xml-format-specification-that-hudson-supports). It had the everything correct except one minor detail. The 'classname' attribute in the 'testcase' tag was supposed to be 'class', which I didn't know until much later. So obviously Jenkins didn't - couldn't - accept this file to show the test results.

Not realizing, this I went on searching for a true format and came upon [an XSD file for the Junit.xml file](http://windyroad.org/2011/02/07/apache-ant-junit-xml-schema/). I ran [cci_bash]xmllint --noout --schema junit.xsd junit.xml[/cci_bash] and well, it didn't pass. So I started to fill in the missing tags and attributes. After I got the XML to validate with the XSD, I tried to build the project in Jenkins again and it failed :S.

My next idea was foolproof, compare the phpspec junit output with phpunit output. So I started to imitate the phpunit format one attribute at a time. At the end I had the following attributes added; file, time, line and assertions. But the only thing I needed to change from the original one from the stackoverflow question was 'classname' attribute to 'class'. After it worked with 'class' I removed the file, time, line, and assertions and it still worked with Jenkins. So I have kept the lines to add 'file', 'time', 'line', 'assertions' commented until phpspec makes those information available to the formatter.

Enjoy the new Junit formatter at [https://github.com/andho/phpspec](https://github.com/andho/phpspec). The code is in the development branch so be careful using it in production although I am doing exactly that. I'll try to separate the changes into a separate branch and send a pull request to phpspec repo.
