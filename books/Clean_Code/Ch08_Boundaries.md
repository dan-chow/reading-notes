## Chapter 08: Boundaries

A cleaner way to use Map might look like the following. No user of Sensors would care
one bit if generics were used or not. That choice has become (and always should be) an
implementation detail.
public class Sensors {
private Map sensors = new HashMap();
public Sensor getById(String id) {
return (Sensor) sensors.get(id);
}
//snip
}
The interface at the boundary (Map) is hidden. It is able to evolve with very little impact on
the rest of the application. The use of generics is no longer a big issue because the casting
and type management is handled inside the Sensors class.



Learning the third-party code is hard. Integrating the third-party code is hard too.
Doing both at the same time is doubly hard. What if we took a different approach? Instead
of experimenting and trying out the new stuff in our production code, we could write some
tests to explore our understanding of the third-party code. Jim Newkirk calls such tests
learning tests.



There is another kind of boundary, one that separates the known from the unknown. There
are often places in the code where our knowledge seems to drop off the edge. Sometimes
what is on the other side of the boundary is unknowable (at least right now). Sometimes
we choose to look no farther than the boundary.



TODO figure capture
Figure 8-2
Predicting the transmitter


Code at the boundaries needs clear separation and tests that define expectations. We
should avoid letting too much of our code know about the third-party particulars. It’s better
to depend on something you control than on something you don’t control, lest it end up
controlling you.