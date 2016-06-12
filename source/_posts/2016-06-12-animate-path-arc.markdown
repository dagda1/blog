---
layout: post
title: "Animating Paths and Arcs with d3.js"
date: 2016-06-12 07:31:48 +0100
comments: true
categories: JavaScript d3.js MathJax
---
You can see a working example of the product of this post <a href="http://www.d3geometry.com/sine2" target="_blank">here</a> and below is a screenshot:

{% img /images/sine2.png %}

The full and current source can be found <a href="https://github.com/dagda1/d3-geometry/blob/master/client/app/components/sine2.js" target="_blank">here</a>.

Following up from my <a href="http://www.thesoftwaresimpleton.com/blog/2016/05/25/sine-wave/" target="_blank">last post</a>, I have created yet another sine wave animation and I want to blog about how to animate <a href="https://www.dashingd3js.com/svg-paths-and-d3js" target="_blank">path shapes</a> with <a href="https://d3js.org/" target="_blank">d3.js</a>.

I am not going to go over how to set up the basic shapes in the animation, you can find out how that is done by referring to my <a href="http://www.thesoftwaresimpleton.com/blog/2016/05/25/sine-wave/" target="_blank">last post</a>.

The 3 path shapes that I found challenging to animate are the red sine wave that is expanded as the unit circle animation rotates along the x axis and the two blue angled arcs that are situated on the centre of the unit circle and on the circumference respectively.

The first step is to append the paths to an svg <a href="https://developer.mozilla.org/en/docs/Web/SVG/Element/g" target="_blank">group</a> element.

{% codeblock add.js %}
    const sine = state.sine = d3.svg.line()
            .interpolate('monotone')
		        .x((d, i) => { return state.xScale(d); })
		        .y((d, i) => { return state.yScale(Math.sin(d) + 1); });

    const sineData = state.sineData = [];

    state.sineCurve = state.xAxisGroup.append('path')
      .attr('class', 'sine-curve');

    state.innerAngle = state.xAxisGroup
      .append("path")
      .attr("class", "arc");

    state.outerAngle = state.xAxisGroup
      .append("path")
      .attr("class", "arc");
{% endcodeblock %}

I have a ```state``` hash that I use as a container to reference all elements that I will need on each tick of the animation.  The sine curve path element is added on **line 8** and the two arcs are added on **lines 11** and **15** repectively.

**Line 1** creates a d3.js line function that will be used by the sine curve path shape to generate the rather cryptic <a href="https://developer.mozilla.org/en/docs/Web/SVG/Tutorial/Paths" target="_blank">svg path mini language</a> instructions that the svg element will use to draw the sine graph.

A ```sineData``` array is initialised on **line 6**, the line function will be executed for every element in the array and the ```x``` and ```y``` accessor functions on **lines 3** and **4** will be executed exactly once for each element in the array.  Obviously as the array is initially empty, these functions will not be invoked.

We now have the elements on the document, the next step is to kick off the animation function:

{% codeblock initial.js %}
    state = this.addShapes(state);

    state.time = 0;

    this.animate(state, {forward: true});
{% endcodeblock %}

**line 3** initialises a ```state.time``` variable that will be incremented on each tick of the animation.  This var is central to all calculations that will be used to position the shapes on each tick.

The body of the ```animate``` function that is called on **line 5** of the above snipped takes the following form:

{% codeblock animate.js %}
  animate(state, direction) {
    if(direction.forward) {
      state.time += state.increase;
    } else {
      state.time -= state.increase;
    }

    // position shapes


    if(direction.forward && state.time > (Math.PI * 2)) {
      direction = {backwards: true};
    }

    if(direction.backwards && state.time < 0) {
      state.time = 0;
      direction = {forward: true};
    }

    requestAnimationFrame(this.animate.bind(this, state, direction));
  }
{% endcodeblock %}

The ```state.time``` counter is either incremented or decremented on each tick of the animation depending on whether the animation is moving forwards or backwards.

**Lnes 11** to **18** make a simple check of whether the time is greater than 2π, in which case it is time to start animating backwards or if the animation is moving backwards and time is less than 0 then the shapes need to move forward.

**Line 20** calls <a href="https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame" target="_blank">requestAnimationFrame</a> to progress the animation each time.

My first attempt at incrementally drawing the sine wave was to add and remove the sinewave on each call to ```animate``` but this led to a very jerky horrible visual effect as the browser struggled to render the graph each time.
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
<a href="" target="_blank"></a>
