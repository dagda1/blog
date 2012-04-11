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
{%gist 2358173 %}
The above code contains and <a href="http://emberjs.com/">Ember.js</a> controller with a **start_parsing** method/action.  When the user enters a url, this method is called and am Ember **url_search** model object is created containing the url of the site to be scraped.  

- On **line 3**, a new websocket connection is attempted.
- The **onopen** event fires when a successful connection has been negotiated.  **Line 7** provides an event handler for the onopen event that transforms the **url_search** object into JSON and sends it to the websocket server.
- On **line 12**, an **onmessage** handler is provided that the websocket layer can use to transmit any results that are found while screenscraping the site.

Below is the **EM-WebSocket** server that the Ember controller listed above will communicate with:
{%gist 2358241 %}
- On **line 2**, we start the **EventMachine** event loop by calling the **run** method.  The event loop can be thought of an endless loop that can be stopped and started by designated events or the expiration of a timer.  Everything from line 2 onwards is the block that is passed to the run method and is executed during the lifetime of the event loop.
- On **line 3**, the connection details for the rabbitmq server are specified (more on this later).
- On **line 11** the WebSocket server is started via the start method.
- **Line 14** provides an event handler for the server side **onopen** event.
- **Line 19** is the server side onmessage event handler which will be triggered when the Ember controller of the previous gist, sends a message containing the JSON of the url search request.
- **Line 22** calls the yet to be exposed **publish** method that will send a message request to our rabbitmq server instance.

##The RabbitMQ RPC Client##
So far, we have initiated a new websocket connection with the server and we are ready to send the message request to the rabbitmq server.

I stated earlier that the RabbitMQ RPC pattern will allow the backend queue handler to communicate with the websocket layer.  The webscocket layer can be thought of as the **RabbitMQ client** and the backend queue handler can be thought of as the **RabbitMQ server**.  This layer will send a request message, which in this case is the JSON sent from the front end Ember controller that contains the **url_search** object.  In order to receive a response or responses from the server, we will need to specify a **callback** queue that the server can send messages to.  In the context of my application, the server will send data scraped from the specified web site back to the front end code that will render a new row in a table for each result found.

In order to put this into context of our application, below is the code that creates a callback queue for the server to communicate with and publishes the request message to the RabbitMQ server.
{%gist 2358396 %}
- On **line 2**, I am creating what is well known in messaging circles as a **correlation id**.  We will use **correlation id** or unique identifier to correlate (obviously) or associate the RPC responses with the request.
- On **line 4**, a RabbitMQ connection is made to the from the configuration hash specified in the pervious gist.
- On **line 6**, I am creating the callback queue that the RabbitMQ server will send responses to.  The important thing to note here is that I am specifying an empty string as the first argument for the queue creation.  If you do not specify a name then an anonymous queue is created, the server will assign the queue a random name which we can use later to let the RabbitMQ server queue handler know which queue to send the RPC messages to.
-  **Line 8** sets up the callback queue to receive messages from the RabbitMQ server.
-  **Lines 9** to **11** contains the code that handles any message sent from the server.  **Line 11**  sends any results back the UI code which will trigger the specified **onmessage** handler.
- On **Line 14** the **append_callback** method is called and a callback is specified in the **do** block which is defined on **line 15**.  The callback will be called when the **declare** event is fired.  We want to wait until this event has been fired because we want to ensure that the anonymous queue has been created and we have the name of the anonymous name to pass to the server.
**Line 15** publishes the message to the RabbitMQ server.  It is important to note the **:reply_to** and **:correlation_id** arguments that help tie the RPC calls together.  The RabbitMQ server will now know the details of the anonymous queue to send any responses to.

##The RabbitMQ RPC Server##
<a href="" target="_blank"></a>
<a href="" target="_blank"></a