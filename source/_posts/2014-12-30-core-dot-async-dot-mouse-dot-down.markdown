---
layout: post
title: "Clojurescript - Handling Mouse Events With core.async"
date: 2014-12-30 17:33:42 +0000
comments: true
categories: clojure clojurescript
---
I came across a situation recently where I needed to know the ```x``` and ```y``` co-ordinates of the mouse while the user was dragging over a canvas element.  The ```mousemove``` event will give me the ```x``` and ```y``` co-ordinates via the ```offfsetX``` and ```offsetY``` properties of the event object but I only want to record these values when the user is dragging, i.e. when the mouse is down and the ```mousedown``` event has been triggered and we are receiving ```mousemove``` events.   When the user stops dragging and the ```mouseup``` event is fired, I want to stop recording the co-ordinates.

The solution was to use the illusory synchronous appearance of <a href="http://clojure.com/blog/2013/06/28/clojure-core-async-channels.html">core.async</a>.  In my opinion, the key to having user friendly asynchronous code is to have it read as synchronous as possible and I think that ```core.async``` outshines ```promises```, ```generators``` and any other abstraction that I have encountered to read synchronously.

I am using the excellent <a href="https://github.com/swannodette/om" target="new">om</a> library to render the html and below I am declaring 3 channels in the ```IInitState``` lifecycle event that will be stored in the local state and can be retrieved in subsequent lifecycle evnets.
{% codeblock %}
om/IInitState
(init-state [_]
  {:mouse-down (chan)
   :mouse-up (chan)
   :mouse-move (chan 1 (map (fn [e] (clj->js {:x (.-offsetX e) :y (.-offsetY e)}))))})
{% endcodeblock %}
I am creating 3 channels to handle the ```mousedown```, ```mouseup``` and ```mousemove``` and storing them in the local state.

On ```line 4``` I am creating a channel and also supplying a transducer to the channel that will transform each event object that is placed on the channel.

In the ```IDidMount``` lifecycle event that is triggered when the component is mounted onto the dom and is displayed below, I am retrieving the channels from the local state and creating event ```listeners``` for the mouse events that I am interested in, ```MOUSEDOWN```, ```MOUSEUP``` and ```MOUSEMOVE```.  All these event handlers do is basically ```put!``` the event object onto the relevant channel:
{% codeblock %}
om/IDidMount
(did-mount [_]
  (let [canvas (q ".tutorial")
        ctx (.getContext canvas "2d")
        mouse-down (om/get-state owner :mouse-down)
        mouse-up (om/get-state owner :mouse-up)
        mouse-move (om/get-state owner :mouse-move)]

    ; we use put! because we are not in a go block
    (listen canvas EventType.MOUSEDOWN #(put! mouse-down %))
    (listen canvas EventType.MOUSEMOVE #(put! mouse-move %))
    (listen canvas EventType.MOUSEUP #(put! mouse-up %))

    (set! (.-lineWidth ctx) 3)))
{% endcodeblock %}
Now we come to the meat and two potatoes of the piece.  Below is the ```IWillMount``` handler that is called before the component is mounted onto the dom:
{% codeblock %}
om/IWillMount
(will-mount [_]
  (let [mouse-down (om/get-state owner :mouse-down)
        mouse-up (om/get-state owner :mouse-up)
        mouse-move (om/get-state owner :mouse-move)]

    (go-loop []
      (loop []
        (alt! [mouse-down] ([down]
                              (do (log "we hit mouse down and onto next loop.") nil))))
      (log "we won't reach here until mouse down.")
      (loop []
        (alt! [mouse-up] ([up] (do (log "we hit mouse up, final recur and back to previous loop.")  nil))
              [mouse-move] ([coords]
                              (do
                                (log coords)
                                (recur)))))
      (recur))))
{% endcodeblock %}
As before, in lines ```3-5``` I retrieve the channels from local state.

On ```line 7``` a ```go-loop``` is created.  A ```go-loop``` is a convenience or shorthand for ```(go (loop ...))```.  The ```go-loop``` appears to create an infinite loop but actually it creates a statemachine that turns synchronous/blocking looking code into asynchronous non-blocking code.  If there are no events then the ```go``` block is suspended.  The ```go``` macro will expand out calls to ```<!```, ```>!```, ```alts!``` or in our case ```alt!``` into an asynchronous non-blocking statemachine.

On ```line 8``` of the above, we use a combination of ```loop``` and ```alt!``` to suspend execution until a ```mousedown``` event is triggered.   Execution will not reach the second inner ```loop``` on ```line 12``` until the ```alt!``` expression on ```line 9``` receives the mousedown event and returns ```nil``` on ```line 10```.

```alt!``` works like a sort of poor man's pattern matching by dispatching execution to the right hand side s-expression of the left hand side channel that has received the event.

When the ```mousedown``` event is triggered, execution then proceeds to ```line 12``` in a nice synchronous manner where a second loop will listen for ```mouseup``` or ```mousemove``` events on via the ```alt!``` expression on ```line 13```.

If ```mousemove``` events are received then we are simply logging the transformed event object that is transformed via the transducer that was passed to the channel initialisation in ```IInitState```:
{% codeblock %}
:mouse-move (chan 1 (map (fn [e] (clj->js {:x (.-offsetX e) :y (.-offsetY e)}))))
{% endcodeblock %}

While the stream of ```mousemove``` events are received on the ```mouse-move``` channel on line 14, the ```recur``` statement on line 17 will keep execution on this inner loop.

When the user stops dragging and the ```mouseup``` event is fired and received on the ```mouse-up``` channel on ```line 13```.   ```nil``` is returned which breaks execution out of this inner loop and execution continues onto the ```recur``` on ```line 18``` which belongs to the ```go-loop``` and execution goes back to the first inner loop on ```line 8``` where the code appears to block as it listens for the next ```mousedown``` event.

I found this a very interesting approach and the code appears beautifully synchronous which of course it is not.  I also like the ability to supply a ```transducer``` to the core.async channel to transform the incoming input.  This decouples things nicely.


