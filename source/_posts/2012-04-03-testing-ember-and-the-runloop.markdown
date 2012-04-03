---
layout: post
title: "Testing Ember, A.K.A. How I Learned to Love the Runloop and Stop Worrying"
date: 2012-04-03 07:55
comments: true
categories: JavaScript Ember
---
I am currently using <a href="http://emberjs.com/">Ember.js</a> for my soon to be billion dollar side project and I have ran into some interesting behaviour while driving the development of the front end code test first.  I am using the excellent javascript bdd library <a href="http://pivotal.github.com/jasmine/" target="_blank">Jasmine</a> as my test runner.  I have also been using the excellent <a href="https://github.com/netzpirat/guard-jasmine">guard-jasmine</a> gem to automatically run my tests when any files are modified which really is a great experience.  

I had vaguely heard of the Ember <a href="http://blog.sproutcore.com/the-run-loop-part-1/">runloop</a> in a few articles I had read.  But I did not have to deal with directly until I started getting more and more unexpected behaviour in my tests.

This is best illustrated with some code.  Below is a very simple test
