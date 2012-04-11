---
layout: post
title: "Ruby, RabbitMQ RPC and WebSockets"
date: 2012-04-10 15:20
comments: true
categories: ruby websockets
---
##Scenario##
The rails project that I am currently working on presented an interesting problem which I am guessing is a reasonably common one. The problem is how to have a long running process run both independently of the browser and display results in the browser as and when they happen. This is my solution to the problem but I would love to hear how others have dealt with the same scenario.  

The focal point of the application in question initiates a long running process that crawls a web site that is specified by user input (see image below) and screen scrapes each page in the site with the intent of collecting any data that meets a certain search criteria.  This long running process could last many minutes and it is safe to say that each process will outlive the lifetime of a standard page request in terms of time.  The long running process will persist any results that are found to a backend data store. The first requirement is that the job completes if the user closes down their browser.  The next requirement is that if the user is connected to the browser, I want to visually display any results that are found by adding data rows to a data table as and when they happen and not have to wait until the process has executed to render all the results at once.  Below is a screenshot of how the user inputs their search terms and how the data is displayed.
{%img /images/realtime/results.png%}
While working on the window's platform, I have often used Microsoft's <a href="http://en.wikipedia.org/wiki/Microsoft_Message_Queuing" target="_blank">MSMQ</a> for reliable messaging so I just needed to do some research into what was available on non-windows platforms.  The message queue handler will be charged with doing the actual work of crawling around the website and persisting any results to the backend data store.  This will fulfil my requirement of having the job complete if the user closes down the browser.

That just left displaying the results of the screen scraping in the UI as and when they are found.  I could have used polling to periodically check for results on the backend store but this did not seem in keeping with the times and this also appeared to be a great excuse to play about with websockets.  I just needed someway of having the queue handler pass the results back to the websocket layer as and when they are found.
##Solution##
The creation of a queue handler and a websocket layer are easy problems to solve separately, the hard part was finding a means for the two distinct boundaries to communicate together.  

The websocket layer seemed very well covered on the ruby and rails platform and the <a href="http://rubyeventmachine.com/" target="_blank">EventMachine</a> based, async <a href="https://github.com/igrigorik/em-websocket" target="_blank">EM-Websocket</a> Ruby websocket server was an obvious choice.

When it came to queuing, I first looked at the popular <a href="https://github.com/defunkt/resque" target="_blank">resque</a> library as my background job processor or queue handler but I could not find anyway for the queue handler to speak to the websocket layer.  I then looked into <a href="http://www.rabbitmq.com/" target="_blank">RabbitMQ</a> which has many more features that are a better fit for my needs.  As there is a gem for everything in Ruby, I quickly tracked down the <a href="https://github.com/ruby-amqp/amqp" target="_blank">AMQP</a> gem which is a feature rich well maintained, fast, asynchronous RabbitMQ ruby client.  

<a href="http://www.amqp.org/" target="_blank">AMQP</a> stands for Advanced Message Queueing Protocol and is an open standard for passing business messages between applications or organisations.  You can think of it as an interface describing a common set of functionality that all message queuing systems must provide if they implement the AMQP interface.
##RabbitMQ RPC##
After a bit of digging about, I came across a messaging pattern that was a good fit for my problem which is the **RPC** pattern.  We all know about RPC (Remote Procedure Calls) in general computing terms but how does this translate to the message queuing world?

In the context of my application, the **RPC pattern** manifests itself like this:

The user will submit a url to perform the search that will be sent to the websocket layer via the following clientside coffeescript code:
{%gist 2358173 %}
The above code contains an <a href="http://emberjs.com/">Ember.js</a> controller with a **start_parsing** method/action on **line 2**.  When the user enters a url, this method is called and am Ember **url_search** model object is created containing the url of the site to be scraped.  

- On **line 3**, a new websocket connection is attempted.
- The **onopen** event fires when a successful connection has been negotiated.  **Line 7** provides an event handler for the onopen event that transforms the **url_search** object into a JSON string and sends it to the websocket server.
- On **line 12**, an **onmessage** handler is provided for the websocket layer to transmit any results that are found while screenscraping the site.

Below is the **EM-WebSocket** server that the Ember controller listed above will communicate with:
{%gist 2358241 %}
- On **line 2**, we start the **EventMachine** event loop by calling the **run** method.  The event loop can be thought of an endless loop that can be stopped and started by designated events or the expiration of a timer.  Everything from line 2 onwards is the block that is passed to the run method and is executed during the lifetime of the event loop.
- On **line 3**, the connection details for the rabbitmq server are specified (more on this later).
- On **line 11** the WebSocket server is started via the start method.
- **Line 14** provides an event handler for the server side websocket **onopen** event.
- **Line 19** is the server side onmessage event handler which is triggered when the Ember controller of the previous gist, sends a message containing the JSON of the url search request.
- **Line 22** calls the yet to be exposed **publish** method that will send a message request to our rabbitmq server instance.

##The RabbitMQ RPC Client##
So far, we have initiated a new websocket connection with the server and sent a JSON string that can be transformed into an object that describes what website to crawl and scrape results from.

I stated earlier that the RabbitMQ RPC pattern will allow the backend queue handler to communicate with the websocket layer.  The webscocket layer can be thought of as the **RabbitMQ client** and the backend queue handler can be thought of as the **RabbitMQ server**.  The websocket layer outlined above is our RabbitMQ client layer that will send a request message containing the JSON string sent from the front end Ember controller code outlined in the first gist.  

In order for the RabbitMQ client to receive esponses from the RabbitMQ server, we will need to specify a **callback** queue that the RabbitMQ server can send messages to.  In the context of my application, the RabbitMQ server will send data scraped from the specified web site back to the front end code that will render a new row in a table for each result found.

In order to put this into the context of my application, below is the code that both creates a callback queue for the RabbitMQ server to communicate with **and** publishes the request message to the RabbitMQ server.
{%gist 2358396 %}
- On **line 2**, I am creating what is well known in messaging circles as a **correlation id**.  The **correlation id** or unique identifier will be used to correlate (obviously) or associate the RPC responses with the request.
- On **line 4**, a RabbitMQ connection is made from the configuration hash specified in the pervious gist.
- On **line 6**, I am creating the callback queue that the RabbitMQ server will send responses to.  The important thing to note here is that I am specifying an empty string as the first argument for the queue creation.  If you do not specify a name then an anonymous queue is created. The RabbitMQ server will assign the queue a random name which can be used later to let the RabbitMQ server queue handler know which queue to send the RPC response messages to.
-  **Line 8** sets up the callback queue to receive messages from the RabbitMQ server.
-  **Lines 9** to **11** contains the code that handles any message sent from the RabbitMQ server.  **Line 11**  sends any results back the UI code if the WebSocket connection is still open. This will in turn, trigger a client side **onmessage** handler:
{%codeblock%}
socket.onmessage = (evt) ->
  Lead.get('leads_controller').addLead evt.data
{%endcodeblock%}
- On **Line 14** the **append_callback** method is invoked which takes a **block** of code as one of its arguments.  The **block** will be executed when the **declare** event of the queue creation is triggered.  It is important to wait until this event has been fired because we want to ensure that the anonymous queue has been created and the name of the anonymous queue is available to send to the RabbitMQ server.
- **Line 15** publishes the message to the RabbitMQ server.  It is important to note the **:reply_to** and **:correlation_id** arguments that help tie the RPC calls together.  The RabbitMQ server will now know the details of the anonymous queue to send any responses to.  It is worth repeating that the **reply_to** argument specifies the name of the anonymous queue that the RabbitMQ server layer can send its responses to.

##The RabbitMQ RPC Server##
So far, we have sent a message from the front end code containing any arguments required in a JSON string, we have also set up a callback queue that we can send any parsing results to and finally we have published the request message and specified the name of the anonymous response queue that we can send any results of the screen scraping to.

Below is the code for the rabbitmq server in nearly all of its entirety:
{%gist 2358775 %}
- **Line 17** defines the Thor task that contains the RabbitMQ server code.
- **Line 19** calls the AMQP start method that opens a connection to the RabbitMQ server.
- **Line 22** creates the queue that will listen for request messages from the RabbitMQ server.
- **Line 33** uses the subscribe method of the AMQP queue object takes a block as an argument that gets passed a message each time a message is published to the queue.  The block takes 2 parameters, the header which is the metadata of the message which in this case contains both the **correlation_id** and the **reply_to** details that were created in the RabbitMQ client layer and give us all the data we need to communicate any results back to the RabbitMQ client layer.
-  **Line 38** creates the parser object that does the actual crawling and screenscraping.
- **Line 40** calls the **find_potential_leads** method that takes a block and **yields** any leads found back to the block that allows me to publish any leads via the **reply_to** queue back to the RabbitMQ client layer. 



I have created the RabbitMQ server process as a <a href="http://railscasts.com/episodes/242-thor" target="_blank">Thor</a> task.  I like the name spacing capabilities of Thor that lets me execute namespace tasks with respect to their environment like:
{%codeblock%}
thor worker:dev:start_consumer
{%endcodeblock%}


