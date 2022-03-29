Blah Blah

In 2010 I discovered a blog posed by ... titled An Immutable .... In 2011 I was implementing a
system utilizing Event Sourcing, but I was struggling to make it immutable while carrying state
from on version of the model to the other. As I was dealing with Insurance Policies which carried a
lot of data, there was a lot of repetition.

In 2017, I was once again thinking about this particular problem and I think I've got it. As event
sourcing involves hydrating your objects from events, your models should then have constructores
that accepts a list (array) of events as the one and only parameter. This can be a private
constructor, and a static public constructor can be used to truly initialize the object. And after
within the object instance, when an event is applied, the object will construct a new object with
all the old events and the new event passed in as an array of events.

Have to handle snapshot/memoization.
