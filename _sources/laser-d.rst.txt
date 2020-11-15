Introduction
============

Laser-D is a computer programming language based on the D programming Language. The language differs from D in some major ways: Laser-D does not support 
garbage collection and does not provide a runtime. As a consequence, Laser-D does not support certain features of the D programming language. Laser-D 
does not support the standard D library phobos for the same reason.

Here is a hello world program in Laser-D.

::

    import core.stdc.stdio : printf;

    extern (C) void main()
    {
        printf("Hello World!\n");
    }


Notation
========
The syntax is specified using Extended Backus-Naur Form (EBNF)::

    Production  = production_name "=" [ Expression ] "." .
    Expression  = Alternative { "|" Alternative } .
    Alternative = Term { Term } .
    Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
    Group       = "(" Expression ")" .
    Option      = "[" Expression "]" .
    Repetition  = "{" Expression "}" .

Productions are expressions constructed from terms and the following operators, in increasing precedence::

    |   alternation
    ()  grouping
    []  option (0 or 1 times)
    {}  repetition (0 to n times)

Lower-case production names are used to identify lexical tokens. Non-terminals are in CamelCase. Lexical tokens are enclosed in
double quotes ``""``.

The form ``a … b`` represents the set of characters from ``a`` through ``b`` as alternatives. The horizontal
ellipsis ``…`` is also used elsewhere in the spec to informally denote various enumerations or code snippets that are not further specified. 


Basic Concepts
==============

Laser-D programs are structured as modules that can be compiled separately and linked with external libraries to create 
native libraries or executables.

Each Laser-D source file defines a single module. 

Each laser-D module contains one or more declarations of entities. 

An entity is a named variable, function, structure or union type, enumeration type, template, mixin template or an external symbol.

Entities in a Laser-D module can have linkage that specifies how these entities may be accessed from other Laser-D modules, or from programs in other 
other languages such as C or C++.

Source Code Representation
==========================

Source code must be Unicode text encoded in UTF-8.

Characters
----------

The following terms are used to denote specific Unicode character classes:

``newline`` 
    the Unicode code point U+000A

``unicode_char``
    an arbitrary Unicode code point except ``newline``

``universal-character-name``
    as defined in ISO/IEC 9899:1999(E) Appendix D of the C99 Standard

::

    nondigit = "A" … "Z" | "a" … "z" | "_" | universal-character-name .
    digit    = "0" … "9" .


Lexical Elements
================

Keywords
--------

The language has following keywords.

::

    abstract    alias        align          asm          assert      auto
    body        bool         break          byte         case        cast
    catch       cdouble      cent           cfloat       char        class
    const       continue     creal          dchar        debug       default
    delegate    delete       deprecated     do           double
    else        enum         export         extern       false       final
    finally     float        for            foreach      foreach_reverse    function
    goto        idouble      if             ifloat       immutable    import
    in          inout        int            interface    invariant    ireal
    is          lazy         long           macro        mixin        module
    new         nothrow      null           out          override     package
    pragma      private      protected      public       pure 
    real        ref          return         scope        shared       short
    static      struct       super          switch       synchronized
    template    this         throw          true         try          typeid
    typeof      ubyte        ucent          uint         ulong        union
    unittest    ushort       version        void         wchar        while
    with        __FILE__     __FILE_FULL_PATH__          __MODULE__   __LINE__
    __FUNCTION__             __PRETTY_FUNCTION__         __gshared    __traits     
    __vector    __parameters


Operators and Punctuation
-------------------------

::

    /     /=    .    ..    ...   &
    &=    &&    |    |=    ||    -
    -=    --    +    +=    ++    <
    <=    <<    <<=  >     >=    >>=
    >>>=  >>    >>>  !     !=    (
    )     [     ]    {     }     ?
    ,     ;     :    $     =     ==
    *     *=    %    %=    ^     ^=
    ^^    ^^=   ~    ~=    @    =>
    #


Identifiers
-----------

::

    identifier = nondigit { nondigit | digit } .


An identifier is a sequence chracters with a ``nondigit`` character followed by ``nondigit`` or ``digit`` characters.  

Lowercase and uppercase letters are distinct. There is no specific limit on the maximum length of an identifier.

Comments
--------

Comments serve as program documentation. There are two forms:

* Line comments start with the character sequence // and stop at the end of the line.
* General comments start with the character sequence /\* and stop with the first subsequent character sequence \*/.

String Literals
---------------

A string literal is either a double quoted string, a wysiwyg quoted string, a delimited string, a token string, or a hex string.

Names
=====
In a Laser-D program, identifiers can be used to denote:

* modules
* structures and union types
* template types and template parameters
* functions and function arguments
* aliases
* labels
* variables
* mixin templates
* import bindings
* attributes
* external symbols

The same identifier can denote different entities at different points in the program.

Scope
-----
An identifier is visible (i.e., can be used) only within a region of program text 
called its scope. There are following kinds of scopes: global, module, struct or union type, template type, function, block. 

Different entities designated by the same identifier either have different scopes, or the identifiers must be in an overload set.


Modules
=======

Each Laser-D source file represents a distinct module. Each module has a name, optionally qualified with a package prefix.
The module name, if not declared explicitly using the ``module`` declaration, is derived from the source file name.  

If present, the ``module`` declaration must be the first and only such declaration in the source file, 
and may be preceded only by comments and #line directives.

::

    ModuleDeclaration             = "module" ModuleFullyQualifiedName ";" .
    ModuleFullyQualifiedName      = { PackageName "." } ModuleName .
    ModuleName                    = identifier .
    PackageName                   = identifier .

The fully qualified module name forms part of the qualified name of every entity declared in that module. Thus two entities with identical names in different
modules can be disambiguated using the fully qualified names of the entities.

Types
=====

A type determines a set of values together with operations and methods specific to those values. 

::

    Type         = BasicType | DerivedType | TemplateType .
    BasicType    = "void" | "bool" | "byte" | "ubyte" | "short" | "ushort" | "int" | "uint" | "long" | "ulong" | 
                   "cent" | "ucent" | "char" | "wchar" | "dchar" | "float" | "double" | "real" |
                   "ifloat" | "idouble" | "ireal" | "cfloat" | "cdouble" | "creal" .
    DerivedType  = ArrayType | StructType | PointerType | ReferenceType | FunctionType | UnionType | DelegateType .



Void type
---------

::

    Keyword     Default Initializer         Description
    void        -	                        no type

Boolean types
-------------

::

    Keyword     Default Initializer         Description
    bool        false	                    boolean value

The ``bool`` type is a byte-size type that can only hold the value ``true`` or ``false``.

The operators that can accept operands of type ``bool`` are: ``& |``, ``^``, ``&=``, ``|=``, ``^=``, ``!``, ``&&``, ``||``, and ``?:``.


Integral types
--------------

::

    Keyword     Default Initializer         Description
    byte        0                           signed 8 bits
    ubyte       0u                          unsigned 8 bits
    short       0                           signed 16 bits
    ushort      0u                          unsigned 16 bits
    int         0                           signed 32 bits
    uint        0u                          unsigned 32 bits
    long        0L                          signed 64 bits
    ulong       0uL                         unsigned 64 bits
    cent        0                           signed 128 bits (reserved for future use)
    ucent       0u                          unsigned 128 bits (reserved for future use)

    char        'xFF'                       unsigned 8 bit (UTF-8 code unit)
    wchar       'uFFFF'                     unsigned 16 bit (UTF-16 code unit)
    dchar       'U0000FFFF'                 unsigned 32 bit (UTF-32 code unit)

Floating-point types
--------------------

::

    Keyword     Default Initializer         Description
    float       float.nan                   32 bit floating point
    double      double.nan                  64 bit floating point
    real        real.nan                    largest FP size implemented in hardwareImplementation Note: 80 bits for x86 CPUs or double size, whichever is larger

    ifloat      float.nan*1.0i              imaginary float
    idouble     double.nan*1.0i             imaginary double
    ireal       real.nan*1.0i               imaginary real
    cfloat      float.nan+float.nan*1.0i    a complex number of two float values
    cdouble     double.nan+double.nan*1.0i  complex double
    creal       real.nan+real.nan*1.0i      complex real


Pointer types
-------------

::

    PointerType = BaseType "*"
    BaseType = Type

A pointer type denotes the set of all pointers to variables of a given type, called the base type of the pointer. 
The value of an uninitialized pointer is ``null``. 

Enum types
----------


Array types
-----------

::

    ArrayType       = BaseType "[" ArrayLength "]" .
    ArrayLength     = Expression .
    BaseType        = Type .

An array is a numbered sequence of elements of a single type, called the element type. The number of elements is called the length of the array 
and is never negative. 

The length is part of the array's type; it must evaluate to a non-negative constant representable by a value of type int. 
The length of array ``a`` can be discovered using the built-in property ``length``. The elements can be addressed by integer indices 
``0`` through ``a.length-1``. Array types are always one-dimensional but may be composed to form multi-dimensional types. 

::

    int[10] a;   // a is an array of 10 ints, a.length == 10


Slice Types
-----------

::

    SliceType       = BaseType "[" "]" 
    BaseType        = Type

A slice is a descriptor for a contiguous segment of an underlying array and provides access to a numbered sequence of elements from that array. 
A slice type denotes the set of all slices of arrays of its element type. The number of elements is called the length of the slice and is 
never negative. 

The length of a slice ``s`` can be discovered by the built-in property ``length``; unlike with arrays it may change during execution. 
The elements can be addressed by integer indices ``0`` through ``s.length-1``. The slice index of a given element may be less than the 
index of the same element in the underlying array.

A slice, once initialized, is always associated with an underlying array that holds its elements. A slice therefore shares storage with its 
array and with other slices of the same array; by contrast, distinct arrays always represent distinct storage. 

An uninitialized slice has a ``length`` of ``0``.

In Laser-D it is the programmer's responsibility to ensure that the life time of a slice is less than or equal to the underlying array.

String type
-----------

::

    string          immutable array of char  (UTF-8)
    wstring         immutable array of wchar (UTF-16)
    dstring         immutable array of dchar (UTF-32)

A string type is simply an alias for an immutable array of ``char``, ``wchar`` or ``dchar``. 

While a string literal has a ``0`` byte terminator, the string type does not.

Laser-D's built-in comparison operators compare strings as a sequence of Unicode code-points. 

Struct types
------------


Union types
-----------





Implicit Conversions 
--------------------

A ``bool`` value can be implicitly converted to any integral type, with ``false`` becoming ``0`` and ``true`` becoming ``1``.

The numeric literals ``0`` and ``1`` can be implicitly converted to the bool values ``false`` and ``true``, respectively. 
Casting an expression to ``bool`` means testing for ``==0`` or ``!=0`` for arithmetic types, and ``==null`` or ``!=null`` for pointers 
or references.

