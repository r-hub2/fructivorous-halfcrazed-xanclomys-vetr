
R core has been gradually removing package access to some C level functions.  Most recently they have made use of the ATTRIB function illegal.  

This required using the newly provided (R 4.6.0) `R_getAttributes` instead, but
because that returns an UNPROTECTED VECSXP instead of the actual attribute
LISTSXP substantial code changes are required.

In addition to the straightforward changes, our prior manipulation of the "tsp"
attribute is no longer possible, so we had to come up with a kludge to address
it.

Attached is a copy of the previously published package, along with a diff of the
changes for a potential fix.

Review the changes and identify potential problems.  One issue we've already
noticed is:


      plst1 <- pairlist(a=character(), b=list(), c=NULL)
      plst5 <- pairlist(a=character(), b=list(), integer())

    > alike(plst1, plst5)
    [1] "`names(plst5)[3]` should be \"c\" (is \"\")"
    Warning: stack imbalance in 'withVisible', 88 then 90

Notice the stack imbalance warning

Additionally, the previous error was (notice the double [):

    [1] "`names(plst5)[[3L]]` should be \"c\" (is \"\")"                        


