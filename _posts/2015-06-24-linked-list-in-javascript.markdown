---
layout: post
title: Linked List in Javascript
author: Nikola Shahpazov
modified:
excerpt:
tags: []
image:
  feature:
date: 2015-06-24T12:46:11+03:00
---
<style>

.btn {
  border-radius: 12px;
  background-color: #336699;
  cursor: pointer;
}

.node {
  stroke: #fff;
  stroke-width: 1.5px;
}

.link {
  stroke: #999;
  stroke-opacity: .6;
}

</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js" charset="utf-8"></script>

<script>
  function Canvas(id, width, height, radius, charge) {
    this.radius = radius;

    this.data = {
      nodes: [],
      links: []
    };
    this.size = 0;

    this.force = d3.layout.force()
      .charge(charge)
      // .gravity(0.003)
      .linkDistance(40)
      .size([width, height]);

    this.svg = d3.select(id).append("svg")
        .attr("width", width)
        .attr("height", height);
  }

  // static color for all
  Canvas.color = d3.scale.category20();

  Canvas.prototype.redraw = function () {
    this.node = this.node.data(this.data.nodes);
    this.node.enter().insert("circle")
      .attr("class", "node")
      .attr("r", this.radius)
      .style("fill", function (d) {
        return Canvas.color(d.group);
      })
      .call(this.force.drag);

    this.link = this.link.data(this.data.links);
    this.link.enter().insert("line", ".node")
      .attr("class", "link");

    return this.force.start();
  }

  Canvas.prototype.addNode = function (node) {
    this.data.nodes.push(node);
    this.size++;
  };

  Canvas.prototype.addEdge = function (edge) {
    this.data.links.push(edge);
  };

  Canvas.prototype.start = function () {
    var self = this;
    this.force.nodes(this.data.nodes)
      .links(this.data.links)
      .start();

    this.link = this.svg.selectAll(".link")
    .data(this.data.links)
    .enter().append("line")
      .attr("class", "link")
      .style("stroke-width", function (d) {
        return Math.sqrt(d.value);
      });

      this.node = this.svg.selectAll(".node")
        .data(this.data.nodes)
        .enter().append("circle")
          .attr("class", "node")
          .attr("r", this.radius)
          .style("fill", function (d) { return Canvas.color(d.group); })
          .call(this.force.drag);

      // node.append("title")
      //     .text(function (d) { return d.name; });

      this.force.on("tick", function() {
        self.link.attr("x1", function (d) { return d.source.x; })
            .attr("y1", function (d) { return d.source.y; })
            .attr("x2", function (d) { return d.target.x; })
            .attr("y2", function (d) { return d.target.y; });

        self.node.attr("cx", function(d) { return d.x; })
            .attr("cy", function(d) { return d.y; });
      });
  };

  document.addEventListener("DOMContentLoaded", function (event) {
    var canvas1 = new Canvas('#first-canvas', 678, 200, 10, -200);
    canvas1.data.nodes.push({name: 'Lorem', group: 1});
    canvas1.start();

    // ** Update data section (Called from the onclick)
    document.querySelector("#first-btn")
      .addEventListener('click', function (ev) {
        var group = parseInt(Math.random() * 356);
        canvas1.addNode({name: 'Lorem Ipsum', group});
        canvas1.redraw();
      });

    // second canvas and button
    var secondCanvas = new Canvas("#second-canvas", 678, 300, 15, -200);
    secondCanvas.data.nodes.push({name: 'sss', group: 1});
    secondCanvas.start();

    document.querySelector('#second-btn').addEventListener('click', function (event) {
      var group = parseInt(Math.random() * 356);
      secondCanvas.addNode({name: "ala", group: group});
      secondCanvas.addEdge({source: secondCanvas.size - 1, target: secondCanvas.size});
      secondCanvas.redraw();
    });
  });
</script>

This is my first article, so I wanted it to set the mood for my entire blog and best describe what I'm passionate about, Computer Sciences in a modern web environment. I also wanted it to be an introduction article.
<p><br/></p>


I assume all of my readers know what an array is, so I decided to move to something very similar but in the same time describing an entirely different representation of a list.
By definition
<br>
"Linked list is a data structure consisting of a group of nodes which together represent a sequence. Under the simplest form, each node is composed of data and a reference (in other words, a link) to the next node in the sequence; more complex variants add additional links.
"

<p><br /></p>


<img src="http://www.nczonline.net/blog/wp-content/uploads/2009/04/408px-Singly-linked-list.svg_.png" alt="Linked List Diagram" width="408" height="41">

<br />
 So lets start with the first thing.

##The Node.

We want to represent a simple object consisting of data and a reference to another node. In our particular implementation I have decided to make it with to references, one to a previous node, and one to a next one.

{% highlight javascript %}
function Node(data) {
  this.data = data;
  this.data.ref = this;
  this.next = null;
  this.prev = null;
}
{% endhighlight %}

<div id="first-canvas"></div>
<a id="first-btn" class="btn">Add New One</a>
<br>

It can't get easier than that. A simple function constructor with data and a references to a next and previous node. But what we actually need is methods with which we can attach those references.

{% highlight js %}
// our reference setters
Node.prototype.setNext = function (next) {
  this.next = next;
};

Node.prototype.setPrev function (prev) {
  this.prev = prev;
};
{% endhighlight %}

##Linked List

OK. Lets wrap what we have so far in a LinkedList class constructor.

{% highlight js %}
function LinkedList() {
  this.first = null;
  this.last = null;
  this.size = 0;
}
{% endhighlight %}

At this point you are probably thinking. "Wait a minute! This thing starts to look a lot like an array! What is the point of all of this?!?"<br/>
<p><br></p>

I am showing you all this because there's a fundamental concept in the Linked List. And that is the reference. In an array we have cells with data where we can query whichever element of the array as long as we know it's index. Here we have a reference connected to a reference, connected to a reference and so on. It's an entirely different view on the matter of connection of objects. And with it we can learn interesting concepts contained in the Linked List.
<p><br /></p>

If you still interested in what else we can do with this data structure lets see what we have to do to insert a new element in our list.
<br />

{% highlight js %}
LinkedList.prototype.insert = function (data) {
  var node = new Node(data);
  this.size++;
  if (!this.first) {
    this.first = this.last = node;
    return;
  }

  node.setPrev(this.last);
  this.last.setNext(node);
  this.last = node;
};
{% endhighlight %}
For now we are only attaching nodes to the last inserted node in the list via the insert method.
Lets see how it looks when we use it.
<br />

<div id="second-canvas" class="node-canvas"></div>
<a id="second-btn" class="btn">Add New One</a>
<br />

As you have probably noticed a general thing the array has and we don't is a way to iterate through the list. But how should we do this? Should we log the element, should we push it somewhere? Why don't we make a generic method which lets the user decide what he wants to do with the object, being that let him pass a callback function which will be executed for every element in the list and in it he can specify what exactly wants to do with the node.
<br />

{% highlight js %}
LinkedList.prototype.forEach = function (callback) {
  var current = this.first,
      doesContinue;
  while (current) {
    doesContinue = callback(current, current.index);
    // lets us break out of the loop if the user returns false
    if (doesContinue === false) {
      break;
    }
    current = current.next;
  }
};
{% endhighlight %}
<br />
What we just did was designing a function which takes a function as an argument. In computer sciences this function is called "high order function". You have probably used this many times, almost every library on the web take advantage of that approach.


OK, so far we are able to add and iterate through the elements of the list, but what if we need to remove a record. If we want to remove an element we'll sure need to first find it.
<br />

{% highlight js %}
LinkedList.prototype.find = function (callback) {
  var result = null;
  this.forEach(function (el) {
    if (callback(el) === true) {
      result = el;
      return false;
    }
  });
  return result;
};
{% endhighlight %}

This is our find method which makes a linear search passing a callback to every element. If the callback at some point returns true it means we have met our requirements and we return that element.
<br />

{% highlight js %}
LinkedList.prototype.remove = function (callback) {
  if (size === 0) {
    throw Error("You can't remove from an empty list");
    return;
  }
  var node = this.find(callback);

  // case of a one element and we have it found
  if (node && size === 1) {
    this.last = this.first = null;
    size = 0;
    return;
  }

  // case of not finding the node
  if (node === null) {
    return null;
  }

  // case of having element somehere in the middle
  var prev = node && node.prev;
  var next = node && node.next;
  if (next && prev) {
    prev.setNext(next);
    next.setPrev(prev);
  }

  // case of having it last
  if (node === this.last && !next) {
    prev.setNext(null);
    this.last = prev;
  }

  // case of having it first
  if (node === this.first && !prev) {
    node.setPrev(null);
    this.first = node;
  }
  this.size--;
  return node;
};
{% endhighlight %}
<br />

I hope you liked my first article and you've learnt something from it. As you probably already see this data structure is not very usable in the real world but is good enough to present some interesting ideas for representing data.
