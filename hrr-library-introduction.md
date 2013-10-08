
# HRR Library Introduction

This document serves as a simple introduction to the Python HRR Library.  All
the examples should run happily in the latest version of IPython.

## Importing the Library

Before we can begin we need to import the library.  My version of the library is
stored in ../hrr-plaground/, so using IPython's `%cd` command I can switch
directory and import.


    %cd ../hrr-playground/
    import Holographic # Import the library
    from Holographic import CleanUpMemory, Symbol
    %cd ../notebooks/

    /home/andrew/Documents/University/Postgraduate/hrr-playground
    /home/andrew/Documents/University/Postgraduate/notebooks


## Using the Basic Components

For simple experiments (or particularly complex ones) several primitive
functions are exposed by the library, these are:

* Vector Generation
    * **`vec_generate( d )`** Generates and returns a `d` dimensional vector
according to the distribution requirements given in (Plate 1991).
    * **`vec_genenerate_normalised( d )`** As above, but normalises the vector
before returning
* Vector Manipulation
    * **`vec_convolve_circular( a, b )`** Convolves `a` and `b` and returns the
result
    * **`vec_normalise( a )`** Returns a normalised copy of vector `a`

As a simple example, we can generate vectors `a` and `b`, and convolve them to
gain `c`.


    a = Holographic.vec_generate( 10 )
    b = Holographic.vec_generate( 10 )
    c = Holographic.vec_convolve_circular( a, b )
    
    print a
    print b
    print c

    [-0.31484757 -0.20656134 -0.21196469  0.45160621  0.08935454 -0.11363466
      0.36244234 -0.4069927   0.03278003 -0.61062851]
    [-0.08549678 -0.55158246 -0.04243535  0.25299539 -0.12297877  0.10400238
     -0.19965436 -0.19313891  0.01318006  0.54661504]
    [-0.01778487  0.20879541  0.14218944  0.14452017 -0.31006506  0.1886324
      0.10549855 -0.10335454 -0.04217629 -0.05768869]


However, this is pretty meaningless if we can't compare vectors or unbind them.
Enter the `Symbol` class.

## The `Symbol` Class

The `Symbol` class provides a handy wrapper around some common tasks, such as
generating, binding, unbinding and comparing symbols.
The constructor for `Symbol` requires a label for the symbol (e.g., "dog") and
*either* a pre-generated vector, or the dimensionality for which a vector is to
be generated to represent the symbol [optionally a generator function can be
passed].

We can construct `Symbol`s in any of these ways:


    s1 = Symbol( "dog", dimensionality = 10 ) # Construct a Symbol with a 10d random vector
    s2 = Symbol( "cat", dimensionality = 10, generator=Holographic.vec_generate_normalised ) # With 10d normalised vector
    s3 = Symbol( "mouse", vector = a ) # Using a predefined vector

### Binding and Unbinding `Symbol`s

Binding `Symbol`s together can be done by either calling the `bind` function:


    s4 = s1.bind( s2 ) # Bind s1 and s2 together

Or by using the `*` operator:


    s4 = s1 * s2

Unbinding can occur one of two ways, either by calling the `unbind` function:


    s5 = s4.unbind( s2 ) # Should be approximately equal to s1

Or by using the `*` operator and the `inverse` function:


    s6 = s4 * s2.inverse() # Should be approximately equal to s1

### Comparing `Symbol`s

`Symbol`s can be compared using the `comparison` function, this returns the
cosine of the angle between the vectors representing the symbols.


    print "s1, s1\t%.3f" % s1.comparison( s1 )
    print "s1, s2\t%.3f" % s1.comparison( s2 )
    print "s1, s3\t%.3f" % s1.comparison( s3 )
    print "s1, s4\t%.3f" % s1.comparison( s4 )
    print "s1, s5\t%.3f" % s1.comparison( s5 )
    print "s1, s6\t%.3f" % s1.comparison( s6 )

    s1, s1	1.000
    s1, s2	0.446
    s1, s3	0.358
    s1, s4	-0.892
    s1, s5	0.979
    s1, s6	0.979


## Making Large Examples Easier with the `CleanUpMemory` Class

A `CleanUpMemory` can be used to simplify the construction and comparison of
vectors in a large example.
The constructor for a `CleanUpMemory` resembles the constructor for a `Symbol`:
the number of dimensions to work in *must* be specified, and a function for
generating vectors may also be passed.


    mem = CleanUpMemory( 500 ) # Create a 500D CleanUpMemory

We can then simultaneously generate symbols and add them to the memory with the
`get_symbol` function, which requires a label for the symbol as an argument.


    banana = mem.get_symbol( "banana" )
    monkey = mem.get_symbol( "monkey" )
    eats = mem.get_symbol( "eats" )
    cat = mem.get_symbol( "cat" )
    chases = mem.get_symbol( "chases" )
    mouse = mem.get_symbol( "mouse" )

Importantly, each of the symbols we acquire from the memory in this way is an
instance of the `Symbol` class we saw above:


    type( banana )




    Holographic.Symbol



Which means that we can use the techniques we've already learned to bind and
unbind them as desired.


    trace = monkey * eats * banana + cat * chases * mouse
    print "%s" % trace

    ( ((monkey (*) eats) (*) banana) + ((cat (*) chases) (*) mouse) )


### Cleaning Up with `CleanUpMemory`

Say we wish to ask the question "what eats a banana?", we know that we can
construct the query as $(\mathbf{eats} \circledast \mathbf{banana})'$ and bind
it with the above trace to get the result.
The result can then be compared with everything in the memory to see which it is
most like, this is achieved with the `clean_up` function.

First, let's get the result.


    question = (eats * banana).inverse()
    answer = trace * question
    print "%s" % answer

    (( ((monkey (*) eats) (*) banana) + ((cat (*) chases) (*) mouse) ) (*) (eats (*) banana)')


The `clean_up` function accepts a symbol to clean and returns an ordered list of
pairs with each pair consisting of a symbol from the memory and its likeness to
the given symbol.
If all is well, then the first symbol in the list will be the desired result.


    mem.clean_up( answer )




    [(<Holographic.Symbol at 0x2c12190>, 0.44033064183783605),
     (<Holographic.Symbol at 0x2c12290>, -0.016443800712090801),
     (<Holographic.Symbol at 0x2c12310>, -0.030881824708566912),
     (<Holographic.Symbol at 0x2c12350>, -0.031512413503889586),
     (<Holographic.Symbol at 0x2c122d0>, -0.063450058390025862),
     (<Holographic.Symbol at 0x2c121d0>, -0.065957450486743938)]



How can we make more sense of this?
We saw above that we can print symbols to better represent them, the following
block of code does this:


    print "%s is most like:" % answer
    for (symbol, similarity) in mem.clean_up( answer ):
        print "\t%s\t\t% 2.3f" % (symbol, similarity)

    (( ((monkey (*) eats) (*) banana) + ((cat (*) chases) (*) mouse) ) (*) (eats (*) banana)') is most like:
    	monkey		 0.440
    	eats		-0.016
    	chases		-0.031
    	mouse		-0.032
    	cat		-0.063
    	banana		-0.066


So monkeys eat bananas.

_QED._

## Some Architectural Comments

I've tried to put everything together such that it will be easy to subclass to
try new ideas with.  For example, I have built some of the machines from (Plate
1995) using the library, I've also built more complex memory structures by
subclassing `CleanUpMemory`.
