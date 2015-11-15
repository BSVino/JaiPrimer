
Compilation keyword #bake to specialize polymorphic functions.
Polymorphic function can contain also 'internal' polymorphic type but they can only be used after 'baking' them.

Compilation keyword #modify is used to make constraints/transformations on polymorphic types.
	For example, it allows to define default output polymorphic type, make only one implementation functions for different type u8..32-->u64, complex mapping to reduce the number of generated functions
	NdT: Powerful, but can be used to build function difficult to understand. It uses parameters names as input.
	
Compilation keyword #body_text takes the parameters types, then return a string contain the body of the function 
It's planned to have another keyword #body_if for 'inline' compile time code generation.

#bake_values: a compile time specialisation of some function with some (or all) parameters hardcoded.
	Useful to control inlining, somewhat duplicate (force) inline, but easier to use with functions with lots of arguments.
	
$$: an autobaker for function parameters prefixed by $$ and called with literals (currently no analysis to check if the parameter is known at compile time).

End of new features, demonstration of map which show 'high-levelness' of Jay, but requires to free the allocated memory or there is a memory leak (no GC).
And of map taking an allocator param which allow usage of stack (faster than heap) for 'inline map'.

In the Q&A video, Blow think that 'libraries' will be pre-parsed code but not binary, to allow these features working.

04/08/2015 Demo: Bounds check, here strings, overloading. [renox, I didn't have the patience to listen to the full Q&A]

* Array Bound Checking (ABC):
By default arrays(both SoA and AoS) and strings are bound checked, for optimisation there is a 'no array bound checking' directive: #no_abc (which works either on a statement or on a block),
you can also disable globally ABC, or enforce always ABC (ignoring the #no_abc directive).

SoA pointers are also bound checked as they're 'fat pointers' aka arrays.


* Here strings:
# string <end keyword>
...
<end keyword>
Currently no indentation support and the <end keyword> must be at the beginning of the line.
Strings are Unicode strings. Can be empty.

* overloaded function:
Overloading works as usual (and also with 'inheritance': Jai 'using' feature).
One main improvement: with integer literals, the smaller int type which can support the literal is called.
Works with multiple parameters.
If there are several possible overload with the same level of conversion, it's a compilation error.
Integer conversion are preferred to float conversion.

There are 'surprising' features related to overloading on which JB ask feedback:
1) overloading is scoped: if you can find a match in the local scope, it is selected, even if there is another better match in the outside scope
baz :: (a: u8) { ... }
{
    baz :: (a: u16) { ... }
    baz(100) // this use the u16 version.
}

2) overloading only work on constant function definition (not on functions pointers, lambda).

3) overloading induce a 'file scope'? [renox. I didn't catch this point]

In the Q&A: parametrized type cannot be overloaded currently.


