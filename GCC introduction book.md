
- Compilation has 4 phases (preprocessing -> compilation -> assembly -> linking) all done in order
- GCC can include multiple assemblers and output (object file) is linked with the linker producing an executable
- Exported functions are undefined by the assembler and linker fills the addresses of them

- The file suffix determines the compilation:
	- `file.c` - source file (must be preprocessed)
	- `file.i` - source file (must not be preprocessed)
	- `file.h` - header file to be turned into precompiled header
	- `file.s` - assembler code (must be preprocessed)
	- `obj` - object file (fed straight to linker)
	- `-x [lang][none] | --language [lang][none]` - specify language instead of relying on suffix , `c` `c-header` `assembler` , None tells gcc to check the suffix (none by default)

- file options & functions for gcc:
	- `-c [file] | --compile[file]` - compile source file , the output is an object file (For each file specified)
	- `-s [file] | --assemble[file]` - assemble file , ouput assembly file of non-assembly code
	- `-e [file] | --preprocess[file]` - preprocess file , output is a preprocessed file of the original file 
	- `-o [file] | --output [file]` - specify the output file (name , location) , result is based on the file options used `gcc -c test.c test` result is `test.o`(This can be done automatically by the compiler) - object file (result of compilation) . If specified with a name that exists, pre existing file will be overwritten 
	- `-Wall` - enable warnings and suggestions , errors have the format `file:line-number:message`

- We can specify multiple files when compiling or doing any file process , ex: `gcc -c main.c hello.c -o hello`, header file with the definition in the local file dir `#Include "header.h"`, they are included directly by the compiler and dont need to specify it when building.

- In the case of multiple files compiled together , they are compiled separately and then linked together , only edited files are recompiled if a change occurs (Saving time for large projects)

- Make file is a tool used to automate the budling process by following a basic scripting language that does all the work (using Make tool ), make specifies compilation rules
	 `Target: dependency Command`  , Think of it as To build `target`, first make sure `dependencies` are up to date, then run `command`, we can make variables as well by using `VARIABLE=VALUE` and using them by `$(VARIABLE)`  [^1], If a file changes , make makes sure all files are up to data automatically and are regenerated 
*Make files are out of scope here*

- Static libs are collection of object files compiled together, in order to use a function from a static lib we should specify it when compiling 
  `gcc main.o hello.a -o main` , `hello.a` is our static lib. Linker checks the libraries that have the function needed from left to right of our command line so a lib should be after the file that uses a function from it 

- `-l[name]` in gcc tells the compiler that there is a library with the name *name* in the library search path `usr/local/include | usr/include` , we can use it instead of typing the full path of lib , if the lib doesnt exist , we can use `-L[PATH]` to include the path in the search , example:
  `gcc -lgdb` - gdb lib exists in path
  `gcc -L/usr/Desktop/Lib -lgdb` - gdb exists in the specified path

- respectively same with `L` there is `-I` for header include path

- The environment variables are also check after the include / lib paths , so we can use it to add header and lib paths

- Variables can be assigned `VARIABLE=VALUE`
- Variables can be used by `$VARIABLE`

- Multiple paths can be saved in a single variable using : `path1:path2:path3` and '.' represents the current path

- we can control the specific C version used using the `-std` option , `-std=c99`

- `-Wall` represents multiple warning flags `-Wformat` , `-Wunused` , `-Wimplicit`, `-Wreturn-type`, `-Wuninitialized`
- There are other warning flags that can be used 

- We can used the command `-D` to define a macro with value and use it in the code `-DNAME=VALUE`, no value is equal to 1

- GCC provides the option to store debugging information by using `-g` the debug option works by storing name , code line number of function in something called a symbol table

- Core dump (file produced by a system when an app crashed) combined with the symbol table , can be used to know where the app crashed

- before dumping you must set the core page size in the terminal using `ulimit -c` to display your set page size , `ulimit -c unlimited` for unlimited page size

- We use the gdb debugger to investigate about the program, it prints diagnostic information along with the ability to debug code parts (we must set the debug flag)

- we can use GDB along with the exe and the core file to diagnose `GDB exe core` 
- `backtrace`  command used to check the functions and arguments passed
- `break` command sets a breakpoint at a function name or a memory location
- `run` command runs the app in the debugger
- `step` used to move forward by the debugger
- `next` used to skip a nested function call unlike skip
- `set variable` used to give variables a value in the debugger temporarily , ex `set variable p = (int*)malloc(sizeof(int))`
- `finish` continues the execution till the end of the function and shows the return value
- `continue` continues the execution till the next breakpoint

- Gcc is an optimizing compiler , it takes many factors and trys to produce the best code , there are 2 common types of optimization *common subexpression elimination* and *Function inlining*

- common subexpression elimination is involved in computing expressions and tends to use fewer instructions (by evaluating redundant expressions once and substitute it) , it is powerful as it increases speed and reduces code size
![[Pasted image 20260510125958.png]]

- function inlining increases the efficiency of frequently called function by removing the function overhead. It works by copying the content of the function to the place where it was called (inlining). It can be significant when the function called is relatively small and called oftenly , For big function , having more instruction is better than moving the body inline. `inline` is used to explicitly state that the function should be inlined

- There are more optimizations called *speed-space tradeoff* which trades space for speed of vice

- Loop unrolling is a speed-space tradeoff. It occurs within a for loop where we eliminate the end of loop condition on each iteration which in return saves a large fraction of time , This can be done on small loops (Loop unrolling increases the program’s speed by eliminating loop control instruction and loop test instructions)[^2]

- Optimization levels in GCC can control the compilation time , memory usage , and executable size

- The optimization is chosen by the `-O[Number]` , Number is a range from 0->3 (Each has its own optimizations):
	- 0/1 - Most common forms of optimizations such as scheduling (no speed-space tradeoffs) , 1 can take less time compiling
	- 2 - More optimizations are made , No speed-space tradeoffs , Faster compilation and less size (due to less instructions)
	- 3 - Turns on expensive optimizations (Speed-space tradeoffs) , faster executable , bigger binary

- `-funroll-loops` - turns on loop unrolling ,increases binary size (can be beneficial or not )
- `-Os` - optimizes for best binary size (sometimes faster executables)

- GCC has a profiler tool to (`gprof`) to measure the performance of a program (it shows the function that takes the most amount of time) 

- To use the profiler , we must compile the program with the `-pg` flag (adds more instruction to record the time taken for each function), profiling data is written to a file in the same directory

- `file` this command prints out information about a file like its header type , arch , what cpu it's made for

- `nm` this command used to examine symbol tables inside of executables (locations of function and variables by name) , `T` means function defined in the object file , `U` means function that's undefined 
- 
 ---
[^1]: ```
```
cc = gcc
cf = -Wall

all: main

main.o: main.c
	$(cc) $(cf) -c main.c

main: main.o
	$(cc) main.o -o main

clean:
	rm -f main main.o
```

[^2]: 
```c
	  for(int i = 0; i < 5; i++) //rolled loop (normal loop)
	  
	  y[0] = 0; //unrolled loop
	  y[1] = 1;
	  y[2] = 2;
	  y[3] = 3;
	  y[4] = 4;
	  
	for(int i = 0; i < 10; i+=2){ //unrolled loop with defined n
		  y[i] = 0;
		  y[i + 1] = 1;
	  }
	  
	int i = 0;
	for (; i + 3 < n; i += 4) { //unrolled loop with undefined n  
		  sum += arr[i];  
		  sum += arr[i + 1];  
		  sum += arr[i + 2];  
		  sum += arr[i + 3];  
	}  
	  
	/* Remaining iterations */  
	for (; i < n; i++) {  
		  sum += arr[i];  
	}
	

```
