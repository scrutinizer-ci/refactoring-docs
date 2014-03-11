A conditional dispatcher is a conditional statement (typically ``switch``) which 
is used to route and handle a request. 

Conditional dispatchers are not necessarily bad, some are well suited for the 
task they are performing. This applies mostly to dispatchers which are small, but
even small Conditional Dispatchers might be better refactored to the Command pattern.

If one of the following is the case for you, you can benefit from using a Command
pattern over a Conditional dispatcher:

- *Need for Runtime Flexibility*: You want to pass in requests which you do not know of (for
  example, because they are provided by third-party libraries).

- *Long Conditional Dispatchers with big chunks of code*

The **Command pattern** simply places the action related code into a separate class,
the Command class. A command class typically has a ``execute`` or ``run`` method.

If you are not sure yet whether you should keep a Conditional Dispatcher or use
a Command pattern, keep the Conditional Dispatcher. The Command pattern is very 
easy to refactor to if the need arises.