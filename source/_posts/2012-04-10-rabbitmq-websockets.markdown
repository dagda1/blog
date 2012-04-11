---
layout: post
title: "RabbitMQ RPC and WebSockets"
date: 2012-04-10 15:20
comments: true
categories: ruby websockets
---
##Scenario##
I am currently using Ruby on Rails for a project that I am working on.  Part of the application initiates a long running process that crawls a web site and screen scrapes results from each page as it spiders its way through the website.  This long running process could last many minutes in some cases and it is safe to say that most requests will outlive the lifetime of a simple web request in terms of time.  The long running process will add any results found to a persistent backend store.  I also want the job to complete if the user closes down their browser.  While the user is connected to the browser, I want to visually display any results found as and when they are found.
##Solution##



