# TBOL Sources

Within this directory hierarchy are the sources of the various TBOL programs recovered from either in
distribution `STAGE.DAT` files, recovered from submitted `STAGE.DAT` or `CACHE.DAT` files, or authored directly.

The recovered files were disassembled with the as-yet unpublished TBOL disassembler.  There are two different binary
formats output by the original compiler - one that includes the program name, compile timestamp, and compiler version;
the other version does not include these items.  In the case of the former version, that detail will be present in the
header comment in the disassembled output.

## Structure Recovery

These `.TBOL` files are, initially, structured versions of the corresponding `.PGM` files as recovered.  The difference
bing that binary form of the TBOL programs, by necessity, represent control structures and expressions as a sequence of
conditional branch statements and other statements that alter the course of execution:

* CJEQ - Conditional Jump Equal
* CJNE - Conditional Jump Not Equal
* CJLT - Conditional Jump Less Than
* CJGT - Conditional Jump Greater Than
* CGLE - Conditional Jump Less than Or Equal To
* CJGE - Conditional Jump Greater than Or Equal To
* JUMP
* CALL

The first pass is generally reversing these simple branch instructions into more expressive control structures
(`IF/THEN/ELSE`, `WHILE/THEN`).

## Variable Recovery

The second pass of recovery entails assigning meanigful variables names.

In the original source, it was common for every TBOL program to include one or more "COPY BOOK" files, comparable to 
contemporary "include" files.  One of these "COPY BOOK" files would likely include the "data" header, 
which constitutes the definition of all data structures held in the run-time data array - essentially the heap.

Additional "COPY BOOK" files may be used to assign mnemoic values with `DEFINE` directives.  For example:

```DEFINE FOO, 'BAR';```

Such a statement would replace all existences of "FOO" in the applicable source with "BAR".
One common use case for this is the `XXCGTSYS` COPY BOOK, which has one `DEFINE` directive for each Global
External Variable name.

Little is understood of the use of `Group Data Names` as they are described in the patent, so no
effort will initially be made to recover such elements.  Rather when a variable's purpose becomes known,
a synthesized `COPY BOOK` for the relevant `.TBOL` files will be created, and a combination of `DEFINE` and `DATA` 
directives recorded therein for the variable in question. 

Furthermore, this `COPY BOOK` will be populated with comments for discovered but as yet unresolved variables; that is,
when the use of a variable (say `$92`) is observed, it'll be added to the COPY BOOK for tracking purposes.

### Determining Purpose

Variable purpose is largely determined by the content placed into the variable.  In some cases, variables are treated as
constants, so their purpose is fairly obvious and naming the variable straight foward.

An example of this is moving a constant value into a variable:

```
MOVE '0', $24;
MOVE 'XXOPMSGSPGM\x04\x0c', $25;
```
In this example, the following `DATA` section may be created:
```
DATA variables =
    { $1 .. $23 }
    ZERO,           {$24}
    XXOPMSGS_PGM,   {$25}
    { $26 .. }
```

Thus, all relevant instances of the variable (as shown above the `DATA` structure definition)  could be replaced with:
```
MOVE '0', ZERO;
MOVE 'XXOPMSGSPGM\x04\x0c', XXOPMSGS_PGM;
```

`DEFINE` is also helpful for many literal values that are tested.  For example, there are many comparisons to NULL like
so:

```
IF variable <> '\x00' THEN DO ... END;
```

Such literals might be assigned with a `DEFINE`, and the example rewritten like so:
```
DEFINE NULL, '\x00';

IF variable <> NULL THEN DO ... END;
```

Another common pattern is to iterate over a numeric sequence and programmaticly construct constants;

## Conventions

Where possible, source recovery will stick to the following conventions:

* Eliminate all GOTO statements
* Achieve a single exit from the function
  * I.e., an `IF/THEN/ELSE` where the positive case immediately returns will be logically inverted such that the `THEN` clause constitutes the body of the function, and the `ELSE` eliminated if the next logical statement is the end of the function.
  * This convention will not be followed it it makes the logic more cumbersome.
* Remove redundant `RETURN` statements.  A `RETURN` is implied by `END_PROC`, and so any `RETURN` statement immediately preceeding `END_PROC` will be eliminated.
