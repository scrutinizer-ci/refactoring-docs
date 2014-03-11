A Collecting Parameter is an object that you pass to methods in order to collect 
information from those methods. Therefore, often this refactoring is combined with 
the :refactoring:`compose-method` refactoring.

Instead of calling many methods in sequence and collecting their results in
local variables, you instead pass an object to these methods and the methods
can store their results in the passed object.

This refactoring is very similar to the :refactoring:`move-accumulation-to-visitor`
refactoring. A visitor may be the better approach if you collect data from diverse
interfaces and classes.