Introduction
============

Laser-D is a computer programming language based on the D programming Language. The language differs from D in two major ways: Laser-D does not support 
garbage collection and does not provide a runtime. As a consequence, Laser-D does not support certain features of the D programming language.

Here is a hello world program in Laser-D.

::

    import core.stdc.stdio : printf;

    extern (C) void main()
    {
        printf("Hello World!\n");
    }


