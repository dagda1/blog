---
layout: post
title: "Animating Paths and Arcs with d3.js"
date: 2016-06-12 07:31:48 +0100
comments: true
categories: JavaScript d3.js MathJax
---
{% img /images/sine2.png %}
You can see a working example of the end result of this post <a href="http://www.d3geometry.com/sine2" target="_blank">here</a> and above is a screenshot.

The full and current source can be found <a href="https://github.com/dagda1/d3-geometry/blob/master/client/app/components/sine2.js" target="_blank">here</a>.

Following up from my <a href="http://www.thesoftwaresimpleton.com/blog/2016/05/25/sine-wave/" target="_blank">last post</a>, I have created yet another sine wave animation (YASWA) and I want to blog about how I achieved a smooth animation for the <a href="https://www.dashingd3js.com/svg-paths-and-d3js" target="_blank">svg path shapes</a> and arcs with <a href="https://d3js.org/" target="_blank">d3.js</a>.

I am not going to go over how to set up the basic shapes in the animation, you can find out how that is done by referring to my <a href="http://www.thesoftwaresimpleton.com/blog/2016/05/25/sine-wave/" target="_blank">last post</a>.

The 3 path shapes that I found challenging to animate are the red sine wave that is progressively added and removed from the document as the unit circle rotates along the x axis and the two blue angled arcs that form the small angled arc at the centre of the centre of the larger circle and the blue arc that expands around the circumference of the larger unit circle.

The first step is to append the paths to an svg <a href="https://developer.mozilla.org/en/docs/Web/SVG/Element/g" target="_blank">group</a> element.  Their initial position is not important at this stage.

{% codeblock add.js %}
const state = {};

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

There is a ```state``` object (line 1) that is used as a container to hold a reference to all the svg shapes that will be transformed on each tick of the animation.  The shapes are added as properties of the ```state``` object.  The sine curve path shape is added to the ```state``` object on **line 10** and the two path shapes that will render the arcs are bound as properties of the ```state``` object on **lines 13** and **17** repectively.

**Line 3** creates a d3.js line function that will be used to generate the rather cryptic <a href="https://developer.mozilla.org/en/docs/Web/SVG/Tutorial/Paths" target="_blank">svg path mini language</a> instructions for the <a href="https://www.dashingd3js.com/svg-paths-and-d3js" target="_blank">d</a> attriute of the svg path shape that will plot the points of the sine curve.

A ```sineData``` array is initialised on **line 8** and the line function will be executed for every element in the array and the ```x``` and ```y``` accessor functions on **lines 5** and **6** will be executed exactly once for each element in the array.  Obviously as the array is initially empty, these functions will not be invoked until there actually is some data.

Now that the the svg shapes are on the document, the next step is to kick off the animation function:

{% codeblock initial.js %}
state = this.addShapes(state);

state.time = 0;

this.animate(state, {forward: true});
{% endcodeblock %}

**line 3** initialises a ```state.time``` variable that will be incremented on each tick of the animation.  This variable is central to all calculations that will be used to position.

The body of the ```animate``` function that is called on **line 5** of the above code snippit takes the following form:

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

**Lnes 11** to **18** make a simple check of whether the time is greater than 2π, in which case it is time to start animating backwards or if the animation is moving backwards and time is less than 0 then the shapes need to move forward.  The current direction is passed into the ```animate``` function with each call to ```requestAnimationFrame```.

**Line 20** calls <a href="https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame" target="_blank">requestAnimationFrame</a> to progress the animation each time.

### Animating the sine wave

My first attempt at incrementally drawing the sine wave was to add and remove the sinewave on each call to ```animate``` but this led to a very jerky horrible visual effect as the browser struggled to recreate the curve each time from the origin.

The solution was incredibly simple and I simply make a call to a function called ```progressSineGraph``` that is listed below:

{% codeblock progressSineGraph.js %}
progressSineGraph(state, direction) {
  if(direction.forward) {
    state.sineData.push(state.time);
  } else {
    state.sineData.pop();
  }

  state.sineCurve.attr('d', state.sine(state.sineData));
}
{% endcodeblock %}

The ```state``` object that contains references to all the shapes and properties that are needed to perform the animation is passed into the function along with the direction argument that specifies whether the shapes are animating forwards or backwards.

The code between **lines 2** and **6** either adds the current value of the ```time``` variable to the ```sineData``` array if the animation is progressing forward or removes the current head of the array if the animation is moving backwards.  The elements of this array will be used to plot the sine graph.

On **line 8**  the ```d``` attribute of the ```path``` shape is set to the ```state.sine``` function that is listed below.  As mentioned earlier, the ```d``` attribute contains the mini language instructions to plot the curve.  The line function will generate the instructions by calling the function below for each element in the array:

{% codeblock sine.js %}
const sine = state.sine = d3.svg.line()
        .interpolate('monotone')
        .x((d) => { return state.xScale(d); })
        .y((d) => { return state.yScale(Math.sin(d) + 1); });
{% endcodeblock %}

The line function will be called for each element int the array and ```x``` and ```y``` coordinates will be created by calling the ```x``` and ```y``` accessor functions that accept the current element as an argument.

I was pretty amazed how easy this technique is to animate shapes progressing or regressing.

The d3.js line function is a great abstraction and you can see what the generated output of the line function looks like below.  The ```state.sine``` function has created the ```d``` attribute of the path shape that contains the instructions to draw the sine curve.  I know which I would rather work with:

{% codeblock sinecurve.html %}
<path class="sine-curve" d="M0,0C0.14211661304959536,-0.15833395510988238,1.6268299307366627,-1.8125204868447078,1.9111111111111112,-2.1291935853485757S3.680057650459382,-4.0993994575052985,3.8222222222222224,-4.257738597705085"></path>
{% endcodeblock %}

It is also worth noting that <a href="https://en.wikipedia.org/wiki/Monotone_cubic_interpolation" target="_blank">montone interpolation</a> is set to ensure a smooth curve is drawn.

### Animating the Arcs
All of the shapes in the animation are interlinked in some certain way and all the calculations of where to place the shapes on each tick of the animation are based on the incrementing ```state.time``` counter.

The black line that rotates from the centre of the circle to the circumference can be thought of as the hypotenuse of a right angle triangle.  The hypotenuse in this document is an <a href="https://developer.mozilla.org/en/docs/Web/SVG/Element/line" target="_blank">svg line shape</a> with ```x1```, ```y1```, ```x2``` and ```y2``` properties that are set using cartesian coordinates to position the shape.

The two arcs that form the angle at the centre of the circle and at the circumference will also use the hypotenuse coordinates once they have been calculated.

Below is the code that positions the svg line shape on each tick of the animation:

{% codeblock hypotenuse.js %}
const xTo = state.xScale(state.time);

const dx = (state.radius * Math.cos(state.time));
const dy = (state.radius * -Math.sin(state.time));

const hypotenuseCentre = xTo - dx;

const hypotenuseCoords = {
  x1: hypotenuseCentre,
  y1: parseFloat(state.hypotenuse.attr('y1')),
  x2: xTo,
  y2: dy
};

state.hypotenuse
  .attr('x1', hypotenuseCoords.x1)
  .attr('x2', hypotenuseCoords.x2)
  .attr('y2', hypotenuseCoords.y2);
{% endcodeblock %}

Once the ```x1```, ```y1```, ```x2``` and ```y2``` corrdinates have been calculated, they are bound as properties of a ```hypotenuseCoords``` object that can be referrenced later.  **Lines 15** to **18** positions the shape.

We can now use basic trigonometry and the <a href="https://github.com/d3/d3/wiki/SVG-Shapes#arc" target="_blank">d3.js arc function</a> to create the arcs.  The arc function is also a to the d3.js svg line function as it also generates instructions for the path shape's ```d``` attribute but it also has some additional properties such as the ```innerRadius```, ```outerRadius```, ```startAngle``` and ```endAngle``` properties that are specific to generating arcs.

Below is the code that sets the properties pf the two arcs:
{% codeblock arc.js %}
let angle = Math.atan2(
  (hypotenuseCoords.y2 - hypotenuseCoords.y1),
  (hypotenuseCoords.x2 - hypotenuseCoords.x1)
);

if(angle > 0) {
  angle = (-2 * (Math.PI) + angle);
}

angle = angle + Math.PI / 2;

const innerArc = d3.svg.arc()
        .innerRadius(8)
        .outerRadius(12)
        .startAngle(Math.PI/2)
        .endAngle(angle);

const outerArc = d3.svg.arc()
        .innerRadius(state.radius - 1)
        .outerRadius(state.radius + 3)
        .startAngle(Math.PI/2)
        .endAngle(angle);
{% endcodeblock %}

The main calculation takes place between **lines 1** and **10** and I would love somebody to tell me there is a better way than this.  I always find that if I have used ```let``` to declare a variable and later reassign the variable after initialisation, this usually tells to me that I have got something wrong or I have taken the wrong path.

In order to set the angle each time, I need a to give the ```d3.svg.arc``` function a ```startAngle``` and an ```endAngle``` each time the function is called.

The ```startAngle``` properties of the 2 arcs are set on **lines 15** and **20** and I am using a static value of ```PI \ 2``` which is 90 degrees in radians.  This is because a ```startAngle``` of 0 will position the ```startAngle``` at 12 o'clock on the circle and I want it to be at 3 o'clock or 90 degrees.

On **line 1** the <a href="https://en.wikipedia.org/wiki/Atan2" target="_blank">atan2</a> function is called to find the angle that the hypotenuse makes with the ```startAngle``` of the arc or 0 on the y axis property.

Arctan2 is different than <a href="http://www.rapidtables.com/math/trigonometry/arctan.htm">arctan</a> because it takes 2 arguments and returns an angle in the correct quadrant.  If you are unfamiliar with the 4 trigonometric quadrants, you can read about it <a href="https://www.mathsisfun.com/algebra/trig-four-quadrants.html" target="_blank">here</a>.

The atan2 function takes into the account the two signs of the arguments that are passed in and places the angle in the correct quadrant.  In this example the angle of the ```hypotenuse``` is found by subracting the ```y2``` and ```y1``` properties and subtracting ```x2``` and ```x1``` of the hypotenuse and passing them into ```atan2```.  ```atan2``` returns the angle in radians between π and -π. Thus, ```atan2(1, 1) = π/4``` and ```atan2(−1, −1) = −3π/4```.

My first attempt at drawing the arcs did not contain the readjustment of the ```angle``` variable below;
{% codeblock angle.js %}
if(angle > 0) {
  angle = (-2 * (Math.PI) + angle);
}
{% endcodeblock %}
Without this readjustment, the arcs where being rendered like this:

{% img /images/arctan2.png %}

This is because ```atan2``` returns the angles between π and -π and the angle was being calculated correctly for rays in quadrant 1 and 2 but not what was required for quadrants 3 and 4.  The readjustment of ```(-2 * (Math.PI) + angle)``` gives same ray but in the correct direction when in quadrants 3 and 4 by adding the angle onto -2π.

###Epilogue
I am very happy with the end result.  If you can suggest a better way for any of the above then please a comment below.

I think I need to move on from sine waves.
