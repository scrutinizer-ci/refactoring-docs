Refactoring - Why?
==================

Before we go into refactoring, let's first talk about general design techniques and the problems
which come with them.

Problems during Code Design
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Design Patterns and Over-Engineering
------------------------------------
Design patterns are appealing. Most developers have already suffered from the consequences of a bad design,
so they invest a lot of time into upfront design and usage of design patterns to avoid ending up in a
similar situation again.

One problem which you might face is over-engineering. A certain design pattern is implemented in anticipation
of a future need; this need might never arise though. So, you make a design more sophisticated or
flexible than it needs to be which comes with an additional effort in maintaining such a system.

An overly complex system, usually leads to new developers not understanding your code or developers starting
to work on separate, discrete parts of the system potentially duplicating code because they do not know
each others' part of the system.


Design Patterns - Loosing Sight of Simple Solutions
---------------------------------------------------
Design patterns represent a simple and flexible way to design software applications. They are
tested, and proven development paradigms which can be seen as templates which you apply to a
concrete situation.

When you overly or only use design patterns, you might loose sight of simpler solutions, and
introduce indirection where it is not necessary or not necessary yet.

Just as an example, let's assume that there are different ways of downloading something. You might
be tempted to start to use the Strategy pattern and implement different download strategies while
it might be enough to simply use a conditional statement and a couple of private methods.


Under-Engineering
-----------------

Under-engineering basically means producing poorly designed software. Some reasons why this happens
frequently are:

- **Time Constraints**: not enough time, too many projects, close deadlines, etc.
- **Lack Of Knowledge**

While you might be able to quickly release a first version with poorly designed software, continuous
under-engineering will make adding new features slower and slower until the system has become so
fragile after a couple of release, that a re-write might look very tempting.


Solutions to Code Design Problems
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
In the previous paragraphs, we had a look at a few common problems when design code. Fortunately, there is a
better alternative to over-engineering and excessive application of design patterns on one side, and 
under-engineering on the other side.

Test-Driven Development
-----------------------
Instead of upfront thinking about an elaborate code design, you delay the design decision to a later point and
first focus on implementing the feature. The process is quite simple:

1. Add a test which expresses what your code is supposed to do.
2. Write code to make the test pass.
3. Only now think about improving the design.

`Test-driven development <http://en.wikipedia.org/wiki/Test-driven_development>`_ has been advocated by many
authors and is a extreme programming practice known for very short development cycles.

Evolutionary Design with Continuous Refactoring
-----------------------------------------------
While the first two steps above can be achieved relatively easily, the third step requires a through understanding
of how well designed code looks like.

This is where design patterns come in. Design patterns can be viewed as the targets of a refactoring, the state
which you want to reach. Refactoring is just the way to get you to that state.

Like we touched in the first paragraphs, blindly applying design patterns leads to over-engineered code which
is equally bad as under-engineered code. To truly master code design, it is not only important to know the
target state, for example a specific pattern, which you would like to implement, but the true knowledge lies 
in knowing why and when to use a certain pattern.


