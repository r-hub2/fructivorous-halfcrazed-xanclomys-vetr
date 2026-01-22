
Looks mostly good.  A few comments:

* Do NOT change any code that is not directly affected by the ATTRIB changes
* trim trailing spaces
* for ALIKEC_compare_dimnames:
  * You can compare the names attribute case directly to a "names" string instead of installing the symbol.
  * As soon as there is one attribute comparison failure, exit the entire nested loop structure (that's the point of do_continue == 2).
  * Make sure to also end when the tag is missing from sec, and remove the now redundant check at the end of the loop.
* for ALIKEC_list_as_sorted_vec: only include the VECSXP processing and ensure all the code that calls it is getting attributes with R_getAttributes which only returns a VECSXP.
* for abstract.tsp, we will accept that zeroed tsp attributes are illegal and should not be used so:
  * abstract should return 'tsp_vetr' instead of 'tsp' classed objects.
  * the C code should be updated to recognize either 'tsp' or 'tsp_vetr' templates to match to 'tsp' objects.

# Background

R core has been gradually removing package access to C level functions.  Most
recently they have made use of the ATTRIB function illegal (from section 6.22.6
of r-devel Writing R Extensions:

> The current implementation (R 4.5.0) represents attributes internally as a linked list. It may be useful to change this at some point, so external code should not rely on this representation. The low-level functions ATTRIB and SET_ATTRIB reveal this representation and are therefore not part of the API. Individual attributes can be accessed and set with Rf_getAttrib and Rf_setAttrib. Attributes can be copied from one object to another with DUPLICATE_ATTRIB and SHALLOW_DUPLICATE_ATTRIB. The CLEAR_ATTRIB function added in R 4.5.0 can be used to remove all attributes. These functions ensure can that certain consistency requirements are maintained, such as setting the object bit according to whether a class attribute is present.
> 
> The function R_getAttributes returns the same result as the R function attributes. R_getAttribCount returns the number of attributes and R_getAttribNames returns the names of an object’s attributes. R_hasAttrib can be used to query whether an attribute is present. The functions R_nrow and R_ncol return the number of rows or columns in a standard matrix or data frame. They may be extended to handle non-standard cases by dispatching to dim.
> 
> The function R_mapAttrib can be used to iterate over the attributes of an object:
> 
> SEXP R_mapAttrib(SEXP x, SEXP (*FUN)(SEXP, SEXP, void *), void *data);
> 
> The function FUN should return a C NULL if it wants the iteration to continue. If FUN returns a non-NULL value, then the iteration stops and that value is returned as the result of the R_mapAttrib call. R_mapAttrib is highly experimental. It should only be used if absolutely necessary as both the interface and the semantics may change at short notice.

from: https://cran.r-project.org/doc/manuals/r-devel/R-exts.html#index-R_005fgetAttributes-1

# Task

The package `vetr` (attached as a tar.gz) uses ATTRIB in several places.

For the use in ALIKEC_compare_class, note that we are comparing the attributes
of the class attributes (i.e. attributes(class(x))).  It's possible for these
the downstream logic can equally handle pairlists and vector lists, so perhaps
nothing other than the simple change to R_getAttributes is sufficient (but make
sure nothing modifies the returned values downstream).

For the uses in ALIKEC_compare_attributes_internal, you will need to
re-implement ALIKEC_list_as_sorted_vec.

For the use in ALIKEC_compare_dimnames you'll probably want to use something like ALIKEC_list_as_sorted_vec to make it easier to pairwise compare the attributes instead of using what we're using now (a O(n^2) version).

For the use in ALIKEC_abstract_ts see the documentation of the R level exported
`abstract` function and the R level implementation (i.e. to see that there
should not be a ts attribute set at the C level).

You will need to implement a backport for `R_getAttributes`, conditional on R
version (probably less than 4.6 will require it, but you'll need to check).

Lay out what you plan to do before implementing changes.  Once we agree on a
plan of attack, output only the changed files.
