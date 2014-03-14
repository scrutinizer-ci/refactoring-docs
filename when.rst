Refactoring - When?
===================

Like we discussed in the introductory chapter, refactoring is by no means a silver bullet to all design problems.
It is a technique to reach a desirable target state from your current position, and it is essential if you want to
follow an evolutionary approach to code design which does neither suffer from under-engineering nor over-engineering.

In the following, we want to list some indicators to help you when you want to refactor code.

The code becomes annoying
-------------------------
Without refactoring, the design of code gets worse over time. As your code is changed, for example to meet short-term
goals or maybe without full comprehension of the design of your code base, the code gradually looses structure. This
will make it harder to see the design when reading code which in turn will make it harder to preserve the remaining
structure.

**Refactoring is a means to give structure back to your code.**

The code becomes hard to understand
-----------------------------------
There are basically two different kinds of users of your code. One user is the computer which reads your code and
precisely does what you have told it. The other type of user are your colleagues, your fellow developers and even you yourself;
basically anyone who may read the code in the future.

As you write your code, your understanding of the problem might change and maybe the naming was not perfect at first.

**Refactoring helps to express a higher level of understanding through your code.**


Refactoring makes an implementation easier/faster
-------------------------------------------------
This might sound counter-intuitive at first. After all, refactoring takes time, how can it help to develop faster?

The reasoning behind this is that a good design is essential for rapid development. Without a good design, you can
progress rapidly for a while until the poor design slows you down. Then, you usually start to spend time on finding and
fixing bugs instead of adding new features. Because of this, changes take longer.

**Refactoring stops the design of your code from deteriorating.**
