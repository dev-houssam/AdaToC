# AdaToC
Simple Ada interface to C

```ada


Next: Calling Conventions, Up: Mixed Language Programming   [Contents][Index]
3.11.1 Interfacing to C

Interfacing Ada with a foreign language such as C involves using compiler directives to import and/or export entity definitions in each language – using extern statements in C, for example, and the Import, Export, and Convention pragmas in Ada. A full treatment of these topics is provided in Appendix B, section 1 of the Ada Reference Manual.

There are two ways to build a program using GNAT that contains some Ada sources and some foreign language sources, depending on whether or not the main subprogram is written in Ada. Here’s an example with the main subprogram in Ada:

/* file1.c */
#include <stdio.h>

void print_num (int num)
{
  printf ("num is %d.\\n", num);
  return;
}

/* file2.c */

/* num_from_Ada is declared in my_main.adb */
extern int num_from_Ada;

int get_num (void)
{
  return num_from_Ada;
}

--  my_main.adb
procedure My_Main is

   --  Declare then export an Integer entity called num_from_Ada
   My_Num : Integer := 10;
   pragma Export (C, My_Num, "num_from_Ada");

   --  Declare an Ada function spec for Get_Num, then use
   --  C function get_num for the implementation.
   function Get_Num return Integer;
   pragma Import (C, Get_Num, "get_num");

   --  Declare an Ada procedure spec for Print_Num, then use
   --  C function print_num for the implementation.
   procedure Print_Num (Num : Integer);
   pragma Import (C, Print_Num, "print_num");

begin
   Print_Num (Get_Num);
end My_Main;

To build this example:

    First compile the foreign language files to generate object files:

    $ gcc -c file1.c
    $ gcc -c file2.c

    Then compile the Ada units to produce a set of object files and ALI files:

    $ gnatmake -c my_main.adb

    Run the Ada binder on the Ada main program:

    $ gnatbind my_main.ali

    Link the Ada main program, the Ada objects, and the other language objects:

    $ gnatlink my_main.ali file1.o file2.o

You can merge the last three steps into a single command:

$ gnatmake my_main.adb -largs file1.o file2.o

If the main program is in a language other than Ada, you may have more than one entry point into the Ada subsystem. You must use a special binder option to generate callable routines that initialize and finalize the Ada units (Binding with Non-Ada Main Programs). You must insert calls to the initialization and finalization routines in the main program or some other appropriate point in the code. You must place the call to initialize the Ada units so that it occurs before the first Ada subprogram is called and must place the call to finalize the Ada units so it occurs after the last Ada subprogram returns. The binder places the initialization and finalization subprograms into the b~xxx.adb file, where they can be accessed by your C sources. To illustrate, we have the following example:

/* main.c */
extern void adainit (void);
extern void adafinal (void);
extern int add (int, int);
extern int sub (int, int);

int main (int argc, char *argv[])
{
   int a = 21, b = 7;

   adainit();

   /* Should print "21 + 7 = 28" */
   printf ("%d + %d = %d\\n", a, b, add (a, b));

   /* Should print "21 - 7 = 14" */
   printf ("%d - %d = %d\\n", a, b, sub (a, b));

   adafinal();
}

--  unit1.ads
package Unit1 is
   function Add (A, B : Integer) return Integer;
   pragma Export (C, Add, "add");
end Unit1;

--  unit1.adb
package body Unit1 is
   function Add (A, B : Integer) return Integer is
   begin
      return A + B;
   end Add;
end Unit1;

--  unit2.ads
package Unit2 is
   function Sub (A, B : Integer) return Integer;
   pragma Export (C, Sub, "sub");
end Unit2;

--  unit2.adb
package body Unit2 is
   function Sub (A, B : Integer) return Integer is
   begin
      return A - B;
   end Sub;
end Unit2;

The build procedure for this application is similar to the last example’s:

    First, compile the foreign language files to generate object files:

    $ gcc -c main.c

    Next, compile the Ada units to produce a set of object files and ALI files:

    $ gnatmake -c unit1.adb
    $ gnatmake -c unit2.adb

    Run the Ada binder on every generated ALI file. Make sure to use the -n option to specify a foreign main program:

    $ gnatbind -n unit1.ali unit2.ali

    Link the Ada main program, the Ada objects and the foreign language objects. You need only list the last ALI file here:

    $ gnatlink unit2.ali main.o -o exec_file

    This procedure yields a binary executable called exec_file. 

Depending on the circumstances (for example when your non-Ada main object does not provide symbol main), you may also need to instruct the GNAT linker not to include the standard startup objects by passing the -nostartfiles switch to gnatlink.

Next: Calling Conventions, Up: Mixed Language Programming   [Contents][Index]

```
