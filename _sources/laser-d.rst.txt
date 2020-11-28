Introduction
============

Laser-D is a computer programming language based on the D programming Language. The language differs from D in some major ways: Laser-D does not support 
garbage collection and does not provide a runtime. As a consequence, Laser-D does not support certain features of the D programming language. Laser-D 
does not support the standard D library phobos for the same reason.

Here is a hello world program in Laser-D.

.. code-block:: D

    import core.stdc.stdio : printf;

    extern (C) void main()
    {
        printf("Hello World!\n");
    }

Notation
========
The syntax is specified using Extended Backus-Naur Form (EBNF). This notation is borrowed from the Go Language specification.

::

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

An entity is a named variable, function, type, template, or mixin template. 

Entities in a Laser-D module can have linkage specification that defined how these entities may be accessed from other Laser-D modules, or from programs in other 
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

    abstract     alias        align        assert       auto         
    bool         break        byte         case         cast         
    cdouble      cfloat       char         class        const        
    continue     creal        dchar        debug        default      
    delegate     do           double       else         enum          
    export       extern       false        float        for           
    foreach      foreach_reverse           function     goto         
    idouble      if           ifloat       immutable    import       
    in           inout        int          ireal        is            
    long         mixin        module       null         out           
    override     pragma       real         ref          return       
    scope        shared       short        static       struct       
    switch       template     this         true         typeof       
    ubyte        uint         ulong        union        unittest     
    ushort       version      void         wchar        while
    __FILE__                   __FILE_FULL_PATH__       __MODULE__   __LINE__
    __FUNCTION__               __PRETTY_FUNCTION__      __gshared    __traits     
    __parameters

The following are reserved words for compatibility with D, however these are not keywords in Laser-D.

::

    asm          body         catch        cent         delete        
    deprecated   final        interface    invariant    lazy          
    new          nothrow      package      private      protected    
    public       pure         super        synchronized throw        
    try          typeid       ucent        with
    __vector


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


An identifier is a sequence characters starting with a ``nondigit`` character followed by ``nondigit`` or ``digit`` characters.  

Lowercase and uppercase letters are distinct. There is no specific limit on the maximum length of an identifier.

Comments
--------

Comments serve as program documentation. There are two forms:

* Line comments start with the character sequence ``//`` and stop at the end of the line.
* General comments start with the character sequence ``/*`` and stop with the first subsequent character sequence ``*/``.

String Literals
---------------

A string literal is either a double quoted string, a wysiwyg quoted string, a delimited string, a token string, or a hex string.

todo

Names
=====
In a Laser-D program, identifiers can be used to name:

* modules
* structures and union types
* template types and template parameters
* functions and function parameters
* aliases
* labels
* variables
* mixin templates
* import bindings
* attributes
* external entities

The same identifier can denote different entities at different points in the program.

Scope
-----
A name is visible (i.e., can be used) within a region of program text called its scope. There are following kinds of scopes: 
global, module, struct or union type, template, function, block. 

Names declared in one module can be imported into another module using the ``import`` statement.

Different entities named by the same identifier either have different scopes (although there is a further restriction for functions as described below), 
or the names must be in an overload set.

For example, a variable and a type of the same name conflict if declared inside the same scope.

.. code-block:: D

    int foo;
    struct foo { // Error: struct foo conflicts with variable foo
        int a;
    }

Similarly, a function and a type declared with the same name conflict if declared inside the same scope.

.. code-block:: D

    struct foo {
        int a;
    }
    void foo() {} // Error: function foo conflicts with struct foo

Additionally, inside a function, a name declared inside a block cannot hide or shadow another name declared in an enclosing block, including function parameters. 

.. code-block:: D

    void foo(int x) {
        int x; // illegal, x shadows parameter x
        int y;
        { int y; } // illegal, y shadows enclosing scope's y
    }

Overload sets
+++++++++++++
A function name can be redeclared in the same scope provided the declarations have different parameters. The multiple declarations form an overload set.

.. code-block:: D

    module A;
    void foo() { }
    void foo(long i) { } // A.foo() and A.foo(long) form an overload set

An overload set declared in one module is distinct from an overload set declared in a different module.

.. code-block:: D

    // file A.d
    module A; 
    void foo(long i) { }

    // file B.d
    module B;
    void foo(int i) { }

    // file C.d
    module C;
    import A;
    import B;
    void bar()
    {
        foo(1);   // error, matches A.foo(long) and B.foo(int)
    }

Declarations
------------
For a name to be available, its declaration must be available in the module in an appropriate scope, or via an import from another module

Modules
=======

Each Laser-D source file represents a distinct module. Each module has a name, optionally qualified with a package prefix.
The module name, if not declared explicitly using the ``module`` declaration, is derived from the source file name.  

If present, the ``module`` declaration must be the first and only such declaration in the source file, 
and may be preceded only by comments and ``#line`` directives.

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

    Type         = BasicType | DerivedType .
    BasicType    = VoidType | BooleanType | IntegralType | FloatingPointType .
    DerivedType  = ArrayType | SliceType | EnumerationType | StructType | UnionType |
                   PointerType | ReferenceType | C++Type .

VoidType
--------

::

    Keyword     Default Initializer         Description
    -------     -------------------         -----------
    void        -	                        no type

BooleanType
-----------

::

    Keyword     Default Initializer         Description
    -------     -------------------         -----------
    bool        false                       boolean value

The ``bool`` type is a byte-size type that can only hold the value ``true`` or ``false``.


IntegralType
------------

::

    Keyword     Default Initializer         Description
    -------     -------------------         -----------
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

FloatingPointType
-----------------

::

    Keyword     Default Initializer         Description
    -------     -------------------         -----------
    float       float.nan                   32 bit floating point
    double      double.nan                  64 bit floating point
    real        real.nan                    largest FP size implemented in hardware

    ifloat      float.nan*1.0i              imaginary float
    idouble     double.nan*1.0i             imaginary double
    ireal       real.nan*1.0i               imaginary real
    cfloat      float.nan+float.nan*1.0i    a complex number of two float values
    cdouble     double.nan+double.nan*1.0i  complex double
    creal       real.nan+real.nan*1.0i      complex real


EnumerationType
---------------
todo

ArrayType
---------

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

    int[10] a;    // a is an array of 10 ints, a.length == 10
    int[3] x;     // x is an array of 3 ints
    int[3][5] x;  // x is an array of 5 arrays of 3 ints


SliceType
---------

::

    SliceType       = BaseType "[" "]" .
    BaseType        = Type .

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

A string type is an alias for an immutable array of ``char``, ``wchar`` or ``dchar``. 

While a string literal has a ``0`` byte terminator, the string type does not.

Laser-D's built-in comparison operators compare strings as a sequence of Unicode code-points. 

Pointer types
-------------

::

    PointerType = BaseType "*" | FunctionPointerType | DelegateType .
    BaseType = Type .

A pointer type denotes the set of all pointers to variables of a given type, called the base type of the pointer. 
The value of an uninitialized pointer is ``null``. 

FunctionPointerType
-------------------
::

    FunctionPointerType  = ( Result | auto ) "function" "(" Parameters ")" .
    Result               = Type .
    Parameters           = [ ParameterList ] .
    ParameterList        = ParameterDecl { "," ParameterDecl } .
    ParameterDecl        = [ { ParameterAttribute } ] Type [ identifier ] .
    ParameterAttribute   = "in" | "out" | "ref" .

Example::

    int function(char) x; // x is a pointer to
                     // a function taking a char argument
                     // and returning an int
    int function(char)[] x; // x is an array of
                     // pointers to functions
                     // taking a char argument
                     // and returning an int


Parameter Attributes
++++++++++++++++++++

``in``	
    A parameter marked as ``in`` behaves as if has been passed by value. 
``ref``	
    A ``ref`` parameter is passed by reference.
``out``	
    An ``out`` parameter is passed by reference and initialized upon function entry with the default value of its type

The attributes ``in``, ``ref`` and ``out`` are mutually exclusive.


StructType
----------
A struct declaration introduces the struct name as a type into the scope where it is declared and hides any struct, 
variable, function, or other declaration of that name in an enclosing scope.


UnionType
---------
todo

DelegateType
------------

::

    DelegateType  = Result "delegate" "(" Parameters ")" .

The delegate type holds a pointer to a struct method. It enacpsulates both a reference to the struct object and the method. 
    
C++Type
-------

The declaration of a C++ class introduces the class name as a type into the scope where it is declared. 

Declarations
============
todo

Constant declarations
---------------------
todo

Variable declarations
---------------------
todo

Function declarations
---------------------
todo

Struct and union declarations
-----------------------------
::

    StructDeclaration = "struct" StructType [ "{" { MemberDecl } "}" ] .
    StructType        = identifier .
    MemberDecl        = DataMember | Constructor | Destructor | Method | StaticMethod .
    DataMember        = Type identifier [ "=" Initializer ] ";" .
    Constructor       = "this" "(" Parameters ")" FunctionBody .
    Destructor        = "~" "this" "(" ")" FunctionBody .
    Method            = ( Result | auto ) identifier "(" Parameters ")" FunctionBody .
    StaticMethod      = "static" Method .

The data members of a struct are allocated so that later members have higher addresses within a struct object. 
Implementation alignment requirements might cause two adjacent members not to be allocated immediately after each other.


Initialization
++++++++++++++
A variable of struct type is by default initialized to the values specified as the initializer for each data member, or if no such
value was specified, then to the default initial value of the data member type.

.. code-block:: D

    struct S { int a = 4; int b; }
    S x; // x.a is set to 4, x.b to 0

Constructors
++++++++++++
A constructor is defined with a function name of ``this`` and must have no return value. At least one parameter must be specified.
A struct constructor can be invoked by the name of the struct followed by its parameters.

.. code-block:: D
    
    struct S
    {
        int x, y = 4, z = 6;
        this(int a, int b)
        {
            x = a;
            y = b;
        }
    }

    extern (C) void main()
    {
        S a = S(4, 5); // calls S.this(4, 5):  a.x = 4, a.y = 5, a.z = 6
        S b = S();  // default initialized:    b.x = 0, b.y = 4, b.z = 6
        S c = S(1); // error, matching this(int) not found
    }


C++ Class Declaration
---------------------

A C++ abstract class or class type can be declared using following syntax::

    C++Type             = "extern" "(" "C++" ")" [ "abstract" ] "class" C++Type [ "{" { MemberDecl } "}" ] .

Variables of C++ class type are pointers rather than value objects.

.. code-block:: D

    extern (C++) class A {int a;}
    extern (C) void main() {
        A a;
        assert(a); // will fail as a is null
    }


Template declarations
---------------------

::

    Template                    = TemplateDeclaration | StructTemplateDeclaration | UnionTemplateDeclaration | FunctionTemplateDeclaration .
    TemplateDeclaration         = "template" TemplateName TemplateParameters [ Constraint ] TemplateBody .
    TemplateParameters          = "(" TemplateParameter { "," TemplateParameter } [ "," TemplateSequenceParameter ] ")" .
    TemplateParameter           = TemplateTypeParameter | TemplateAliasParameter | TemplateValueParameter | TemplateThisParameter .
    TemplateTypeParameter       = identifier TemplateTypeSpecialization TemplateTypeDefault .
    TemplateTypeSpecialization  = [ ":" Type ] .
    TemplateTypeDefault         = [ "=" Type ]
    TemplateSequenceParameter   = identifier "..." .
    StructTemplateDeclaration   = "struct" TemplateName TemplateParameters [ Constraint ] "{" { MemberDecl } "}" .
    FunctionTemplateDeclaration = ( Result | auto ) TemplateName TemplateParameters [ Constraint ] "(" Parameters ")" FunctionBody .
    TemplateName                = identifier .
    TemplateBody                = "{" MemberDecl "}" .

Templates are a compile-time code generation mechanism.

A templates takes template parameters that can be values, names of types or entities, or sequences. 

A template introduces a scope, and the template body can contain structs and unions, types, enums, variables, functions, and other templates.

Value parameters must be of an integral type, floating point type, or string type and specializations for them must resolve to an integral constant, 
floating point constant, null, or a string literal.

Templates are instantiated (i.e. generated) in the scope where the template is declared and any names used inside the template, unless declared in the template
scope, shall be looked up in the enclosing scopes of the module where the template is declared.

The struct template declaration is equivalent to a template declaration containing a single struct declaration where the struct name and the template name are the same.

The function template declaration is equivalent to a template declaration containing a single function where the function name and the template name are the same.

Template instantiations
+++++++++++++++++++++++
todo



Expressions
===========
todo

Literals
--------
todo

Primary expressions
-------------------

::

    PrimaryExpression   = [ "." ] identifier | [ "." ] TemplateInstance | "this" | "null" | "true" | "false" |
                          IntegerLiteral | FloatLiteral | CharacterLiteral | StringLiteral | FunctionLiteral |
                          AssertExpression | BasicType "." identifier | BasicType "(" Arguments ")" |
                          TypeOfExpression | IsExpression | "(" Expression ")" | SpecialKeyword | TraitsExpression .
    TypeOfExpression    = "typeof" "(" ( Expression | "return" ) ")" .


IsExpression
++++++++++++

todo

TraitsExpression
++++++++++++++++

todo

Index expressions
-----------------

::

    IndexExpression     = PostfixExpression "[" Arguments "]" .


Slice expressions
-----------------

::

    SliceExpression     = PostfixExpression "[" [ Slices ] "]" .
    Slice               = AssignmentExpression [ .. AssignmentExpression ] .
    Slices              = Slice { "," Slice } .

Binary expressions
------------------


Operators
---------


::

    binary_op     = "||" | "&&" | rel_op | add_op | mul_op | bit_op .
    rel_op        = "==" | "!=" | "<" | "<=" | ">" | ">=" .
    add_op        = "+" | "-" .
    mul_op        = "*" | "/" | "%" .
    bit_op        = "<<" | ">>" | ">>>" | "|" | "&" | "^" .

Arithmetic operators
--------------------

::

    +        sum
    -        difference
    *        product
    /        quotient
    %        modulus

    <<       left shift
    >>       signed right shift
    >>>      unsigned right shift

    |        bitwise or
    &        bitwise and
    ^        bitwise xor

Comparison operators
--------------------

::

    <        less than
    <=       less or equal to
    >        greater than
    >=       greater than or equal to
    ==       equal
    !=       not equal
    is       equal


Logical operators
-----------------

::

    &&       and
    ||       or
    !        not





Address operators
-----------------
todo

Unary expressions
-----------------
todo

Conversions 
-----------

A ``bool`` value can be implicitly converted to any integral type, with ``false`` becoming ``0`` and ``true`` becoming ``1``.

The numeric literals ``0`` and ``1`` can be implicitly converted to the bool values ``false`` and ``true``, respectively. 
Casting an expression to ``bool`` means testing for ``==0`` or ``!=0`` for arithmetic types, and ``==null`` or ``!=null`` for pointers 
or references.

todo

Statements
==========
todo

Scope statement
---------------
todo

Scope block statement
---------------------
todo

Labeled statement
-----------------
todo

Block statement
---------------
todo

Expression statement
--------------------
todo

Declaration statement
---------------------
todo

If statement
------------
todo

While statement
---------------
todo

Do statement
------------
todo

For statement
-------------
todo

Foreach statement
-----------------
todo

Switch statement
----------------
todo

Continue statement
------------------
todo

Break statement
---------------
todo

Return statement
----------------
todo

Goto statement
--------------
todo

Scope Guard statement
---------------------
todo

Pragma statement
----------------
todo

Mixin statement
---------------
todo

Conditional Compilation
=======================
todo

Compile Time Introspection
==========================
todo


