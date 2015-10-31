---
layout: post
title: "Perpendicular bisectors of a triangle with d3.js"
date: 2015-10-31 17:17:30 +0000
comments: true
categories: JavaScript d3.js
---
Following up from my <a href="http://www.thesoftwaresimpleton.com/blog/2015/10/24/altitude/" target="_blank">last post</a> on how to draw the altitude of a side of a triangle through a vertex, I wanted to draw the 3 perpendicular bisectors of a triangle and the circumcircle of the triangle.

Let us get some definitions for these terms, **the perpendicular bisectors of a circle** are described as:
{% blockquote %}
the lines passing through the midpoint of each side of which are perpendicular to the given side.
{% endblockquote %}

Below is a triangle with one perpendicular bisector running through side ```AB```
{% img /images/single_bisector.png %}.

The circumcircle of a triangle is:
{% blockquote %}
The point of concurrency of the 3 perpendicular bisectors of each side of the triangle.
{% endblockquote %}

The centre point of the circumcircle is the point of intersection of all the perpendicular bisectors of a triangle.

Below is a triangle with all 3 perpendicular bisectors and the circumcircle drawn with ```d3.js```.

{% img /images/perpendicular_bisector.png %}

The first step was to draw one perpendicular bisector of a triangle.

I chose 3 arbitary points for the vertices of the triangle.

{% codeblock points.js %}
const points = {
  a: {x: xScale(1), y: yScale(1)},
  b: {x: xScale(5), y: yScale(19)},
  c: {x: xScale(17), y: yScale(6)}
};
{% endcodeblock %}

This is all the information I need, to calculate the perpendicular bisectors and the circumcircle.

If I wanted to find the perpendicular bisector of ```AB``` using pen and paper, I would perform the following steps:

-  I would find the gradient or slope (for US readers) of the point ```AB```.
-  I would then find the perpendicular gradient or slope which would give me the ratio of rise over run that the perpendicular line flows through. If lines are perpendicular then M<sub>1</sub> x M<sub>2</sub> = -1.
-  I would find the midpoint of the line using the distance formula ((x<sub>1</sub> + x<sub>2</sub> / 2), (y<sub>1</sub> + y<sub>2</sub> / 2)).
-  I could then plug these values into <a href=""https://www.mathsisfun.com/equation_of_line.html" target="_blank">the equation of a line</a> which takes the form of ```y = mx + c```.

I have blogged previously in <a href="http://www.thesoftwaresimpleton.com/blog/2015/09/20/first-d3/" target="_blank">this post</a> about how to set up the graduated x and y axis and a more managable scale for positioning vertices etc.

My first step was to find the perpendicular bisector of the line ```AB```.

Below are two helper functions that take javascript point objects as arguments with x and y properties that map to coordinates and return either a gradient/slope or the perpendicular gradient/slope that occurrs between the 2 coordinates:

{% codeblock gradient.js %}
const gradient = function(a, b) {
  return ((b.y - a.y) / (b.x - a.x));
};

const perpendicularGradient = function (a, b) {
  return -1 / gradient(a, b);
};
{% endcodeblock %}

Below is a helper function to find the midpoint between two vertices or points:

{% codeblock midpoint.js %}
const midpoint = function(a, b) {
return {x: ((a.x + b.x) / 2), y: ((a.y + b.y) / 2)};
};
{% endcodeblock %}

Using these values, I can then find the y-intercept or the point where the perpendicular line will cut the y-axis.

Below is a function that will find the y-intercept given a vertex and a gradient/slope:

{% codeblock yintercept.js %}
function getYIntercept(vertex, slope) {
  return vertex.y - (slope * vertex.x);
}
{% endcodeblock %}
You can think of the above function as rearranging ```y = mx + c``` to solve for ```c``` or ```c = y - mx```.

All that is left is to find the x-intercept or the point where the bisector line cuts the x-axis.

Below is the code that brings this all together:

{% codeblock perpendicular-bisector.js %}
function perpendicularBisector(a, b) {
  const slope = perpendicularGradient(a, b),
        midPoint = midpoint(a, b),
        yIntercept = getYIntercept(midPoint, slope),
        xIntercept =  - yIntercept / (slope);
{% endcodeblock %}

The x-intercept on ```line 5``` is again re-arranging the equation of the line formula ```y = mx + c``` to solve for x.

The finshed function looks like this and there are a number of if statements I had to add for the conditions when the slope or gradient function might end up undefined or equalling infinity when it encounters horizontal or vertical values that have catches with the formula.  I would love to know if there is an algorithm that will avoid such checks:

{% codeblock perpendicularBisector.js %}
function perpendicularBisector(a, b) {
  const slope = perpendicularGradient(a, b),
        midPoint = midpoint(a, b),
        yIntercept = getYIntercept(midPoint, slope),
        xIntercept =  - yIntercept / (slope);

  if((yIntercept === Infinity || yIntercept === -Infinity)) {
    return drawTriangleLine(g, {
      x1: xScale(midPoint.x),
      y1: yScale(0),
      x2: xScale(midPoint.x),
      y2: yScale(20)
    });
  }

  if((a.x === b.x) || isNaN(xIntercept)) {
    return drawTriangleLine(g, {
      x1: xScale(0),
      y1: yScale(midPoint.y),
      x2: xScale(20),
      y2: yScale(midPoint.y)
    });
  }

  if(xIntercept < 0 || yIntercept < 0) {
    return drawTriangleLine(g, {
      x1: xScale(xIntercept),
      y1: yScale(0),
      x2: xScale(20),
      y2: yScale((slope * 20) + yIntercept)
    });
  }

  drawTriangleLine(g, {
      x1: xScale(xIntercept),
      y1: yScale(0),
      x2: xScale(0),
      y2: yScale(yIntercept)
    });

  return {vertex: midPoint, slope: slope};
}
{% endcodeblock %}

The ```drawTriangleLine``` function looks like this and simply adds a ```d3.js``` line:

{% codeblock drawTriangleLine.js %}
const drawTriangleLine = function drawTriangleLine(group, vertices) {
  group.append('line')
    .style('stroke', 'green')
    .attr('class', 'line')
    .attr('x1', vertices.x1)
    .attr('y1', vertices.y1)
    .attr('x2', vertices.x2)
    .attr('y2', vertices.y2);
};
{% endcodeblock %}

Every time I call the ```perpendicularBisector``` function, I return an object that contains a vertex and point that I can use to draw the circumcircle.

{% codeblock return.js %}
return {vertex: midPoint, slope: slope};
{% endcodeblock %}

All that is left is to draw the circumcircle and here is the function I wrote to do just that:

{% codeblock circumcircle.js %}
function drawCirumCircle(lineA, lineB) {
  if(!lineA || !lineB) {
    return;
  }

  const x1 = - lineA.slope,
      y1 = 1,
      c1 = getYIntercept(lineA.vertex, lineA.slope),
      x2 = - lineB.slope,
      y2 = 1,
      c2 = getYIntercept(lineB.vertex, lineB.slope);

  const matrix = [
    [x1, y1],
    [x2, y2]
  ];

  const circumCircleCentre = solveMatrix(matrix, [c1, c2]),
      dist = distance(convertPoint(points.b), circumCircleCentre);

  g.append('circle')
   .attr('cx', xScale(circumCircleCentre.x))
   .attr('cy', yScale(circumCircleCentre.y))
   .attr('r', xScale(dist))
   .attr('class', 'circumcircle')
   .attr('fill-opacity', 0.0)
   .style('stroke', 'black');
}
{% endcodeblock %}

In order to find the centre of the circumcirle or the point of intersection of the perpendicular bisectors, the function takes two arguments ```lineA``` and ```lineB``` which are two of the perpendicular bisectors of the traingle.  The function then arranges these line objects into ```y = mx + c``` format on lines 6 to 11 of the above.  I then solve these equations simulataneously using matrices and specifically using <a href="http://www.purplemath.com/modules/cramers.htm" target="_blank">cramer's rule</a> to find the point where the line intersect.

Once I have the 2x2 matrix assembled on lines 13-16, I then pass it to the ```solveMatrix``` function with the 2 y-intercept values that will apply cramer's rule:

{% codeblock solveMatrix.js %}
function det(matrix) {
  return (matrix[0][0]*matrix[1][1])-(matrix[0][1]*matrix[1][0]);
}

function solveMatrix(matrix, r) {
   const determinant = det(matrix);
   const x = det([
      [r[0], matrix[0][1]],
      [r[1], matrix[1][1]]
    ]) / determinant;

   const y = det([
     [matrix[0][0], r[0]],
     [matrix[1][0], r[1]]
   ]) / determinant;

  return {x: Math.approx(x), y: Math.approx(y)};
}
{% endcodeblock %}

I now have the point of intersection of the perpendicular bisectors.  All I need to know now is the radius of the circle.  The calculation I used is to use the <a href="http://www.purplemath.com/modules/distform.htm" target="_blank">distance formula</a>.  From the point of intersection we just found to one of the vertices of the triangle.

Below is a helper function for the distance formuala:

{% codeblock distance.js %}
function distance(a, b) {
  return Math.floor(Math.sqrt(Math.pow((b.x - a.x), 2) + Math.pow((b.y - a.y), 2)));
}
{% endcodeblock %}

All I have to do now is draw the circle from the two knowns, i.e. the point of intersection and the radius:

{% codeblock circle.js %}
  const circumCircleCentre = solveMatrix(matrix, [c1, c2]),
      dist = distance(convertPoint(points.b), circumCircleCentre);

  g.append('circle')
   .attr('cx', xScale(circumCircleCentre.x))
   .attr('cy', yScale(circumCircleCentre.y))
   .attr('r', xScale(dist))
   .attr('class', 'circumcircle')
   .attr('fill-opacity', 0.0)
   .style('stroke', 'black');
{% endcodeblock %}

Here is a <a href="http://jsbin.com/jixozu/1/edit?js,output" target="_blank">working jsbin</a> that illustrates all I have wrote about.

I have also added drag and drop so you can drag the vertices around by the red circles and watch it all redraw.

Please leave a comment below if I could have achieved this in a more efficient way.
