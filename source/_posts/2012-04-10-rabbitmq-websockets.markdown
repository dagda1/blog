---
layout: post
title: "RabbitMQ RPC and WebSockets"
date: 2012-04-10 15:20
comments: true
categories: ruby websockets
---
##Scenario##
The rails project that I am currently working on presented an interesting problem which I am guessing is a reasonably common one. The problem is how to have a long running process run both independently of the browser and display results in the browser as and when they happen. This is my solution to the problem but I would love to hear how others have dealt with it.  

The focal point of the application in question, initiates a long running process that crawls a web site that is specified by user input (see image below) and screen scrapes each page in the site, collecting any data that meets a certain search criteria.  This long running process could last many minutes and it is safe to say that most jobs will outlive the lifetime of a standard page request in terms of time.  The long running process will persist any results found to a backend data store and one of the requirements is that the job completes if the user closes down their browser.  The final requirement is that if the user is connected to the browser, I want to visually display any results found as and when they are found.
{%img /images/realtime/results.png%}
I have often used Microsoft's <a href="http://en.wikipedia.org/wiki/Microsoft_Message_Queuing" target="_blank">MSMQ</a> while working on the windows platform for reliable messaging so I just needed to do some research into what was available on non-windows platforms.  The message queue handler will be charged with doing the actual work of crawling around the website and persisting any results to the backend data store.  This will fulfil my requirement of having the job complete if the user closes down the browser.

That just left displaying the results of the screen scraping in the UI as and when they are found.  I could have used polling to periodically check for results on the backend store but this did not seem in keeping with the times and this also appeared to be a great excuse to play about with websockets.  I just needed someway of having the queue handler pass the results back to the websocket layer as and when they are found.
##Solution##
The creation of a queue handler and a websocket layer are easy problems to solve separately, the hard part was finding a means for the two distinct boundaries to communicate together.  

The websocket layer seemed very well covered on ruby and rails and the <a href="http://rubyeventmachine.com/" target="_blank">EventMachine</a> based, async <a href="https://github.com/igrigorik/em-websocket" target="_blank">EM-Websocket</a> Ruby websocket server was an obvious choice.

When it came to queuing, I first looked at the popular <a href="https://github.com/defunkt/resque" target="_blank">resque</a> library as my background job processor or queue handler but I could not find anyway for the queue handler to speak to the websocket layer.  I then looked into <a href="http://www.rabbitmq.com/" target="_blank">RabbitMQ</a> which has many more features that are a better fit for my needs.  As there is a gem for everything in Ruby, I quickly tracked down the <a href="https://github.com/ruby-amqp/amqp" target="_blank">AMQP</a> gem which is a feature rich well maintained, fast, asynchronous RabbitMQ ruby client.  

<a href="http://www.amqp.org/" target="_blank">AMQP</a> stands for Advanced Message Queueing Protocol and is an open standard for passing business messages between applications or organisations.  You can think of it as an interface describing a common set of functionality that all message queuing systems must provide if they implement the interface.
##RabbitMQ RPC##
After a bit of digging about, I came across a messaging pattern that was a good fit for my problem which is the **RPC** pattern.  We all know about RPC (Remote Procedure Calls) in general computing terms but how does this translate to the message queuing world?

In the context of my application, this **RPC pattern** manifests itself like this:

The user will submit a url to perform the search that will be sent to the websocket layer:
{%gist 2358173%}
The websocket layer or messaging client will send a request message to our rabbitmq messaging server that will handle the message.  



<a href="" target="_blank"></a>
<a href="" target="_blank"></a>


