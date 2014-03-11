You might have heard that classes should have a single responsibility. Even if
you try to follow this dogma, in practice classes grow. Some operations are,
some more data is added, and sooner or later your class has more than one
reponsibility.

The **Extract Class** refactoring is best applied if there are logical units of
data which are grouped together, or operations which are just performed on a 
subset of the data.