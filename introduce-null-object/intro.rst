Depending on the language, calling a method on a variable which is null leads to
errors, system crashes or related behavior. In order to avoid this, developers 
wrap the variable in null checks. Repeating such null handling logic in one or
two places is not necessarily a problem, but sprinkling it over the entire codebase
is less than ideal.

Code with many null checks is usually harder to read and comprehend. Null
checks must also be repeated in all future code, so all developers have to be 
aware of duplicating the appropriate behavior.

Introducing a Null Object, an object which represents the behavior in case of "null",
should usually reduce or at least keep the code size even. If the code size increases,
it is usually a sign that a Null Object is not necessary.

A Null object can rawly be implemented in two flavors. Either by sub-classing, or
by implementing an interface. In the examples section, we will demonstrate both
variants.

.. note ::
    If you implement a Null Object by sub-classing, keep in mind that you need to
    add new null-handling logic each time a method is added in the parent class.
    Otherwise, non-null logic might be accidentally exposed. The interface approach
    does not suffer from this liability.