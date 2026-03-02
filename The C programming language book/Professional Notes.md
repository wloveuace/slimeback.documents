---
share_link: https://share.note.sx/vng5zupk#Vv07nyGSNuihcOh4j/Gi+Xz9Ca3q7S/oL+DZRlxK0Rw
share_updated: 2026-02-19T11:25:57+02:00
---
This Note will hold info about various topics and beginner friendly ( they are made to fill in unknown gaps )

- **expressions** are conditions that either are true or false or initializing variables or anything that produce a value (`5, x, x + 3, a > b, x = 10, ++i, f(3)`)
- **statement** is a complete instruction that the program executes.

- declaring is setting a variable name with a data type `int x;`
- initializing is when giving a variable (thats declared) a value `int x = 10`;

- an number thats too big to fit in an int will be saved as a long
  
- Long constants have an suffix of L `123456L` (this is called a terminal)
  Unsigned constants have suffix of U `12345U`, 2 terminals can be used example: 
  `123456UL` (unsigned long)

- Numbers with decimals `1.22` are taken as a Double , unless suffix F (float) is provided,
  `1.22F`, e can be used to represent a decimal `1.e2` (`1e2`) is valid , eN adds zeros to the number so `1e2` is equal to `100`

- A leading 0 to a number means Octal value , a leading 0x to a number means hexadecimal value
  
- Chars can be used in arithmetic operations just like numbers because they are internally just numeric values

- escape sequences have a prefix of \x , where x is a certain char -
   refer to the escape sequence list[^12]  | escape sequences can also represent a specific constant  value of an octal `\0hh` (hh are octal values)  , or hex `\xhh` (hh are hex values)[^13]

- \#Define or  called constant expressions are evaluated during compile time rather than runtime so they can be used in Code to represent certain values

- Enums are another type of constant expressions that provide naming to int values, Names in enums must be distinct but values can be the same 

- Enums or defines (in numeric values) ? in enums, values for constants can be auto generated , compilers dont check for the numeric value in an enum constant, enum values can be checked , debuggers can print enum values

- String literals can be concatenated at compile time, thats why "Hello " "World" is the same as "Hello World"

- 'X' isnt same as "X" because "X" is a string literal that has \0 at the end so "X" is the size of 2 (X + \0) but 'X' is an actual single char

- sizeof(char) <= sizeof(short) <= sizeof(int) <= sizeof(long) <= sizeof(long long)
	And minimum widths:
		`short` ≥ 16 bits (2bytes)
		`int` ≥ 16 bits (2bytes)
	    `long` ≥ 32 bits (4bytes)
	    `long long` ≥ 64 bits (8bytes)
	> Those are **minimum requirements**, not fixed sizes.
	> they are change from one OS to the other for-example in windows int = long = 4bytes, but in linux int = 4 and long = 8 bytes
	

- x % y produces the remainder when x is divided by y, and thus is zero when y divides x exactly, The % operator cannot be applied to a float or double.

- In C, expressions are **grouped according to operator precedence**.  
   Some operators also guarantee **left-to-right evaluation**, e.g., `&&`, `||`, `,`, `?:`.  
   Other operators (arithmetic, relational, bitwise) do **not** guarantee left-to-right evaluation; only grouping is defined by precedence.
  
- variables can hold values of expressions like: `d = c >= '0' && c <= '9';
	 this is evaluated like `d = (c >= '0') && (c <= '9')`, so each expression will give 1 or 0 and then `and` the 2 values so d = 1 if both = 1

- in C you can have a same variable name pointing to 2 different variables in case they are in 2 different scopes, updating or accessing the same name will access of update the local one because it has higher priority since its the same scope , example:
  ```c
	int j = 2; //global variable
	
	int main() {
		int* x = &j, j; // x is a pointer to the global variable and we have a new *local* variable 'j' which is a pointer to junk data
		j = 1; //here we are changing actully the local j (because of priority) not the global pointed with x
	
		printf("%d, %d", *x, j); //this prints 2 , 1
	}
  ```

- **Parentheses enforce order** when needed. For example: `(c = getchar()) != '\n'`, The assignment happens first because of the parentheses, then the comparison is evaluated.

- Higher precedence rules tell the compiler how to combine terms correctly by implicitly adding parentheses in the right places, it decides grouping and answers “which operands belong to which operator?[^1] so for example `for (i=0; i < lim-1 && (c=getchar()) != '\n' && c != EOF; ++i)` has a condition `i < lim-1 && (c=getchar()) != '\n' && c != EOF`, so in order for the compiler to give correct evaluation it follows the Precedence rule so its becomes `((i < (lim - 1)) && (((c = getchar()) != '\n') && (c != EOF)))`[^2]

- if an operator like + or * that takes two operands (a binary operator) has operands of different types, the \"lower'' type is promoted to the "higher'' type before the operation proceeds.[^4], Notice that floats in an expression are not automatically converted to double, The main reason for using float is to save storage in large arrays, or, less often, to save time on machines where double-precision arithmetic is particularly expensive.  [^5]
  example:
  ```c
	if (1.0F < 1.0) {
		printf("Hello");
	}
	else {
		printf("World");
	}
  ```
  Here we are comparing a DOUBLE and a FLOAT, expected value should be "Hello", but based on the rule we just explained above, "World" is actually printed 

- **automatic conversions** are those that convert a narrower operand to a wider operand such as int to float

- **manual conversions** when using the `(type) var`

- some conversions aren't allowed in C, mostly conversions that involve changing a wider data type to a lower data type, most of them are prohibited but still allowed (like assigning a float to an int) , another illegal operation like using float value in a subscript (index of an array)

- The unusual aspect is that ++ and -- may be used either as prefix operators (before the variable, as in ++n), or postfix operators (after the variable: n++). In both cases, the effect is to increment n. But the expression ++n increments n before its value is used, while n++ increments n after its value has been used.
  ```c
  n = 5;
  x = n++; // x = 5 then n = 6
  x = ++n; // x = 6 and n = 6
  ```

- athematic operators that are placed postfix in a return statement are evaluated before returning, example `return stack[counter--]` this places stack\[counter] in a register and then decrements then return the register value

- bitwise operations must be fully parenthesized to give proper results.
- Bits are numbered from **right to left**
- Position `0` = Least Significant Bit (LSB)
- Position `31` = Most Significant Bit (for 32-bit int)

- Declaring argument to be unsigned ensures that when its right shifted , empty bits will be filled with 0

- A **mask** is a number that has **1s in the positions you want to keep**, and **0s in the positions you want to remove**. Using `&` with a mask **clears all bits where the mask is 0**, and **keeps bits where the mask is 1**. example `n = n & 0177` -> `n = n & 1111111` sets only the lower 7 bits[^6]

- The bitwise OR operator | is used to turn bits on example , `x = x | SET_ON` sets all x bits to 1

- The bitwise exclusive OR operator ^ sets a one in each bit position where its operands have different bits, and zero where they are the same.

- the shift operators >> and << shifts the value of x left by  the number of bit position given by the right operand , << shifts the value of x 

- The unary operator ~ yields the one's complement of an integer; that is, it converts each 1-bit into a 0-bit and vice versa. For example `x = x & ~077` sets the last six bits of x to zero.

###### masks
- `1U << p` making 1 set bit mask at pos p
- `(1U << n) - 1` making n set bits
 ```
 1 << 5  = 00100000
 minus 1 = 00011111
 ``` 
 - `((1U << n) - 1) << (p + 1 - n)` making `n` set bits ending at position `p`

- positioning `(p + 1 - n)` , p is starting pos and n is number of bits
###### Setting, Clearing, Toggling Bits
- `x |= (1U << p)` set bit p in x
- `x &= ~(1U << p)` clear bit p in x
- `x ^= (1U << p)` invert bit p in x

###### masks + setting, clearing, toggling
- `x |= ((1U << n) - 1) << (p + 1 - n)` set n bits from pos p
- `x &= ~(((1U << n) - 1) << (p + 1 - n))` clear n bits from pos p 

- way to use masks and bitwise operations is to shift to the right most and mask the value

- conditional expressions can be in the form `expr1 ? expr2 : expr3`, the expression expr1 is evaluated first. If it is non-zero (true), then the expression expr2 is evaluated, and that is the value of the conditional expression. Otherwise expr3 is evaluated, and that is the value[^7]

- function calls like y = f(x) + g(x) are not guarantee to be executed in order
	this is part of C undefined behavior , for example when doing so `s[i] = i++` we are reading the i and we adding up 1 in the same expression which can cause undefined behavior either the compiler reads 1st and increment of the opposite also when doing something like
		```
		int x = INT_MAX;
		x += 1; //int wrap around ( undefined behaviour )[^8]
		```

- The for statement for (init; condition; update) statement is equivalent to ```
  ```c
	init;
	while (condition) {
	    body;
	    update;[^9]
	}
  ```
`Continue keyword behaviour will differ from this while loop implemenation and the for loop`

- when adding init expressions in for loop, they have to be the same data type
  ```c
  for(int i = 0, char c = 0;;) \\invalid
  for(int i = 0, c = 0;;) \\valid
  ```

- Grammatically, the three components of a for loop are expressions. Most commonly, expr1 and expr3 are assignments or function calls and expr2 is a relational expression. Any of the three parts can be omitted, although the semicolons must remain. If expr1 or expr3 is omitted, it is simply dropped from the expansion. If the test, expr2, is not present, it is taken as permanently true
  
	  for(;;) - is a while loop
	  for(int i=0;;i++) - is a while loop
	  for(;true;) - is a while loop
	  for(;false;) - wont execute

- comma (,) which most often finds use in the for statement. A pair of expressions separated by a comma is evaluated left to right, and the type and value of the result are the type and value of the right operand. Thus in a for statement, it is possible to place multiple expressions in the various parts` Note, comma in function arguments isnt the comma operator and doesnt guarantee left to right evalutions `

- `while` and `for` loops They check the condition **before** executing the body. If the condition is false at the start, The body runs **zero times**. For loops does the same thing but the initial expression is always performed (refer to the for loop internal structure) **These are called Top-tested loops** , do-while loop is different, The condition is checked **after** the body ran once, so if the initial expression is true of false, the body always executes once before checking. `do-while` is called **Bottom-tested loop**

- The `while` loop cant have an initialization expression inside it
  ```c
  while(char c = string) // invalid
  
  char c;
  while(c = string) //valid
  ```

- The **break** statement provides an early exit from for, while, and do, just as from switch. A break causes the innermost enclosing loop or switch to be exited immediately. The **continue** statement is related to break, but less often used; it causes the next iteration of the enclosing for, while, or do loop to begin.

- The `case` label in a switch **must be a constant expression.**

- Using continue in Do - while loops makes the Control goes to checking the condition directly , but for For loops Control goes **to the increment step** (`i++`) before checking the condition.

- there are a few situations where `gotos` may find a place. The most common is to abandon processing in some deeply nested structure, such as breaking out of two or more loops at once. The break statement cannot be used directly since it only exits from the innermost loop

- functions can be declared and defined later (definition means initialized), definitions must match the declaration

- when a function doesn't take arguments its better to add void in the definition because something like
  ```c
void atoi() {};

int main() {
	atoi(123);
} //compiles normally
  ```
  but adding void in the arguments causes a compilation error`too many arguments`
  ```c
void atoi(void) {};

int main() {
	atoi(123);
} //compilation error
  ```

- if a return value specified in the declaration then the expression returned can be casted automatically to the specified return type(if the return type is bigger)

- **definitions** these are global variables(declared outside a function or a scope) They also act as declarations for the rest of that file, They can be used by any function in that source file, Memory is actually **allocated (storage is set aside)** `int sp;` means Create a global integer named `sp` and reserve memory for it.

- **declarations only** these are variables and functions that have the `extern` key word which means “Somewhere else, there exists a variable named `sp` of type `int`. I just want to use it here.” No memory is allocated, No variable is created, it just tells the compiler the variable exists elsewhere in some another source file[^10]

- global variables that can be in any function, it doesn't get destroyed / created within each function ( not automatic ) rather they are fixed for every function and can be used as a way of communication between functions, they have their definition outside a function scope

- The `extern` keyword declares a variable or function that is defined in another file.  
  It tells the compiler that the storage is allocated elsewhere and does not create a new variable.  
  There must be exactly one definition of that variable in the entire program.  
  For arrays, the size can be omitted in the `extern` declaration but must be specified in the definition. 

- `Static` variables and functions are for the private use of the functions in their respective source files, and are not meant to be accessed by anything else. The static declaration, applied to an external variable or function, limits the scope of that object to the rest of the source file being compiled. It is very useful when you dont want to reset the variable in loop operations but keep the variable in loop scope

- **automatic variables** They are created automatically when the function starts, Destroyed automatically when the function ends, Stored on the **stack** ex:
  ```c
  void foo(){
	  int x = 0; // automatic variable
	  x++;
	  printf("%d", x);
  }
  
  foo() // 1
  foo() // 1 
  foo() // 1
  ```
- **Static Local Variables** they are local to a particular function just as automatic variables are, but unlike automatics, they remain in existence rather than coming and going each time the function is activated.
  ```c
  void counter() {
    static int x = 0;   // static local variable
    x++;
    printf("%d\n", x);
  }

  counter() // 1
  counter() // 2
  counter() // 3
  ```
[^11]

- External and static variables are initialized to 0 by default, automatic variables are initialized to garbage (undefined) values when no explicit initialization made

- `register` declaration advises the compiler that the variable in question will be heavily used. The idea is that register variables are to be placed in machine registers, which may result in smaller and faster programs. But compilers are free to ignore the advice, The register declaration can only be applied to automatic variables and to the formal parameters of a function

- the register keyword is just a compiler advice which it can ignore ( generally modern compilers ignore this keyword )

- when using nested scopes, automatic variable declaration and initialization is done each time the scope is entered and static variables are only initialized the 1st time the block of scope is entered

- **For external and static variables, the initializer must be a constant expression; the initialization is done once, conceptionally before the program begins execution. For automatic and register variables, the initializer is not restricted to being a constant: it may be any expression involving previously defined values, even function calls.**

- arrays can be initialized directly after declaration , and if the size of the array is omitted but its initialized, the compiler counts the number of elements at compile time. If the array has more initializers (values) less than specified, the compiler fills the empty with 0s, on the other hand if it has too many the compiler will give an error "too many initializers"

- Recursion in functions follow the scope rules so each time the function is called the automatic variables are reset

- C provides certain language facilities by means of a preprocessor, which is conceptionally a separate first step in compilation. The two most frequently used features are \#include, to include the contents of a file during compilation, and \#define, to replace a token by an arbitrary sequence of characters.

- preprocessors are the initial steps that are evaluated before actually compiling the code

- \#include "Filename" searches for the filename in the current source directory, \#include \<Filename> searches for the file in implementation defined rule , includes can be nested so a file can include other files with it

- \#defines can have multiple lines by adding the \ operator at the end of each line to tell the compiler that the define has more lines in it, defined tokens can hold different values later, defines can have arguments like functions and use them in the statement they are holding

- \# is a preprocessor operator that changes a passed in macro argument into a string so :
  `#define foo(x) #x foo(hello) -> "hello"`

- \## is another preprocessor operator concats arguments passed to a macro
  ``#define foo(x, y) x##y foo(hello, world) -> helloworld`

- there are preprocessor conditional operations that can be used to check statements during preprocessing , \#if can be used to evaluate integer constants if the expression is non zero then the lines until \#endif, \#elif, \#else are included. **mainly used in checking if a header is included or not**

- pointer is a variable that contains an address of another variable

- pointers and arrays are closely related and can be exploited in the same way

- memory is divided into cells(bytes) that can be treated as blocks like 2 chars is 2 bytes and can be treated as a short

- the address of operator '&' cant be applied to constant, expressions, or register variables

- the dereference operator '\*' when applied to a pointer it access the object that the pointer is pointing to

- every pointer points to a specific data type, except the void pointer which can point to anything but cant be dereferenced

- when passing an argument by its reference to a function, it allows the function to change the value that the variable is holding instead of getting passed just the value itself

- if a pointer is pointing to a object in an array so ptr+1 gonna point to the next element in the array automatically  without the need to add the actual size of the datatype, non the less it can be used to access the elements of the array, there remarks are true regardless of the type size

- Assigning a pointer to the start of an array can be done via `ptr = &arr[0]` or `ptr = arr`, which are equivalent in this context because `arr` decays to a pointer to the first element. You can then use `ptr[i]` to access elements just like `arr[i]`.[^15]

- A difference between arrays and pointers that should be known is that arrays arent variables like pointers so expressions like arr++ or x = arr are invalid

- when an array name is passed to a function, what is passed is the pointer to the 1st element in the array, so from now on we now any array name parameter is a pointer

- pointers and integers are interchangeable , the exception here is 0, zero can be a pointer and we can compare pointers with 0 , ZERO in C is NULL 

- The valid pointer operations are assignment of pointers of the same type, adding or subtracting a pointer and an integer, subtracting or comparing two pointers to members of the samearray,andassigningorcomparingtozero.Allotherpointerarithmeticisillegal.Itisnot legal to add two pointers, or to multiply or divide or shift or mask them, or to add float or double to them, or even, except for void\*, to assign a pointer of one type to a pointer of another type without a cast.

- \*p++ and \*++p fetches the pointer then increments and increments then fetches the pointer , this should be taken as a note because they are used alot in practical C code

- `char str[] = "hello"; ` Memory is allocated for the array, the string literal `"hello"` is **copied into the array**, You can modify the contents
- `char *str = "hello";` `"hello"` is stored in **read-only memory**, `str` just points to that memory, You are NOT allowed to modify it

- in c The `*` (declarator pointer operator, *can be the dereference operator* )  binds to the variable name, not the type. so something like `int* a, b, c;`
  is same as 
  ```
  int* a; //pointer
  int b; //normal int
  int c; // normal int
  ```
	another example we discussed earlier:
	```c
	int j = 2; //global variable
	
	int main() {
		int* x = &j, j; // x is a pointer to the global variable and we have a new *local* variable 'j' which is a pointer to junk data
		j = 1; //here we are changing actully the local j (because of priority) not the global pointed with x
	
		printf("%d, %d", *x, j); //this prints 2 , 1
	}
	
	/*
	int* x = &j, j; 
	is actully int* x = &j; int j; not int* x = &j; and int* j; 
	```

- loops in C always check the expression if it is zero or not without even checking, for example:
  ```c
  while (*s++ = *t++)
  while ((*s++ = *t++) != '\0')
  //same logic in both loops
  ```
  `*s++ = *t++` evaluates to the copied character
  Assign  `*t`  to `*s`
  check value `*t` gives
  if 0 then stop the loop (`*s will hold \0`)

- Since pointers are normal variables, they can be stored in an array, the arrays can hold multiple pointers ex, `char* stringsPointers[256]` this is an array of char pointers, where `stringsPointers[i] -> pointer at index i` and `*stringsPointers[i] -> charachter that the pointers in pointing to in index i`. `stringsPointers` can also be treated as a pointer itself since arrays are similar to pointers[^16]

- double pointers like `char**` can be used as an array in dereference (using index) 

- Multidimensional arrays are rectangular arrays that have rows and columns , that can be used to replace array of pointers even-though its not quite used as much as arrays of pointers, we can craft a multidimensional array in this form `int arr[row][cols]`, you can vision it as a rectangle , `arr[0][1]` this means ( get me the 1st item in the row which is in the 2nd column ) ** its the 2nd column since arrays have initial indexing of 0 not 1 **

- In c multi-dimensional arrays are really just one dimensional array that has multiple arrays in it, that is why we can use array of pointers so we can have an array that has pointers that point to other arrays such as the `char* arr[]` this is an array that has string literal arrays in it

- when passing multi-dimensional arrays to a function, the number of columns must be specified `foo(int [][10])` , This is because The array decays into a pointer to its first row so The compiler must know how large each row is , meanwhile the number of rows isnt needed, the compiler will have it look like this `void foo(int (*arr)[10])` -> arr is a pointer to an array of 10 integers

- For `int arr[3][4]` → `arr` is an array of **3 elements**, where each element is an array of **4 ints**.
- For `int* arr[4]` → `arr` decays to **pointer to int pointer**

- when you pass an array to a function, it **decays into a pointer to its first element**. so `foo(int* nums[])` will be `foo(int** nums)` , this is called **array decay**

- **array decay** is the implicit conversion of an array type into a pointer to its first element when its passed to a function as an argument

- In C we can have argument variables by adding 2 parameters in the main function `int argc`, `char** argv | char* argv[]`, where `argc` is the number of elements passed , `argv` is a pointer to an array of strings, `argv[0]` is always the program main so `argc` is always 1 , accessing the `args` can be done with `argv[i]`

- when `char* argv[]` is passed to the main function we can do pointer athematic since `argv` will be `char** argv `

- In C, a function itself is not a variable, but it is possible to define pointers to functions, which can be assigned, placed in arrays, passed to functions, returned by functions, and so on.

- The C function pointer follows this structure : `ret-type (*name)(arguments)` , example:
	`int (*foo)(int x, int* y)` -> this foo function returns an int and take int x argument and a pointer to an int (y) ** \(\*name) brackets are a must and cant be removed **

- The ANSI C book has made a pointer syntax reference that can help ease complicated declarations [^17]

### pointer declaration rules

- C has grammar rules that it follows :
 ```c
 dcl:        optional *'s  direct-dcl

direct-dcl: name
            (dcl)
            direct-dcl()
            direct-dcl[optional size]
 ``` 

these rules allow us to break down C complex syntax into simple terms 

**declaration**: zero or more or `*` followed by a **direct declarator**
**direct declarator**: can be a name, a parenthesized declarator, () -> function, \[] -> array

- The clockwise/spiral rule:
  ##### Step 1 — Start at the identifier.
  ##### Step 2 — Look right if possible
    If you see `[]` → say “array of”
    if you see `()` → say “function returning”
  ##### Step 3 — If you can’t go right, go left.
    If you see `*` → say “pointer to”
  ##### Step 4 — Respect parentheses.
    Parentheses control what you’re allowed to process next.
  Repeat until the whole declaration is consumed.

example: `char (*(*x[3])())[5]`
	x-> identifier
	x\[3] -> array of 3 elements
	\*x\[3] -> array of 3 pointers pointing to...
	`(*x[3])()` -> pointers to functions returning...
	`(*(*x[3])())` -> function returning a pointer to...
	`(*(*x[3])())[5]` -> pointer to an array of 5 ...
	`char` -> chars
so: **x is an array of 3 pointer functions that return an array of 5  pointers of type char**
	
[^18]

chatgpt explanation 
![[Pasted image 20260219223537.png]]
![[Pasted image 20260219223543.png]]

- i know the above point is dense , you deserve a break :) 🍵

solve this at the end `int *(*(*p[10])(double)))[10];`

- `p` is an **array of 10 pointers to functions**
- Each function **takes a `double` as an argument**
- Each function **returns a pointer to an array of 10 int pointers**

- Struct group multiple different objects (which can have different data types) into a single entity object, they can hold a name , they can be passed to functions , they can be copied, they can be assigned values
  
  - Structs have the keyword `struct` followed by a direct declaration a list of objects
    ```c
    struct player{
	    int x; // member
		int y; // member
		char* name; // member
    }
    ```

- The variables in a struct are called members, they can have the same name as a non-member variable since they can be distinguished 

- Structs declare new types ( sort of a blue print ) that reserves **no memory** yet unless you also declare variables.
	example:
	```c
	struct {  
		int x;  
		int y;  
	} a, b, c, *p, arr[20];
	```
	meaning `a`, `b`, and `c` are variables, Each one has its own memory, Each one is of that struct type, same for `*p` which is a pointer to the struct type, and finally `arr` is an array of size 20 for that struct type

- another way of declaring struct variables is using `struct struct-name var-name` , 
  `struct player player1` , this does the same thing as the above example

- A struct can be initialized by following its definition a list of initializers to the member variables 
  `struct player player1 = {10, 10, "John"};`

- Struct members can be used in expressions by using the '.' (dot) operators , for example:
  ```c
  struct player player1 = {10, 12, "john"};
  player.x //thats the x member (10)
  player.y // thats the y member (12)
  player.name //thats the name member ("john")
  ```
  they can be used in all expressions like assignment, unary, logic etc

- structs can be nested, meaning they can hold another structs as member variables

- struct pointers are just like normal pointers `struct player* player1` thats a pointer to player struct

- when dealing with struct pointers we can either use its members in a classic way via dereferencing ex: 
  ```c
  struct player player1 = {10, 10, "john"};
  struct player *ptr = &player1;
  printf("%d", (*ptr).x);
  ```
  but this way can be complicated so we can use the "->" pointer member operator which does the work of dereferencing automatically,
  `printf("%d", ptr->x) //valid`

-  keep in mind, the -> and . operators bind very tightly since they are the highest Precedencies , so something like `++ptr->x` will be `++(ptr->x)` so the value of x will increase but not the pointer

- we can always think of a struct as a data type like any other one, just like an int , we can have pointers, arrays, automatic variables so `int arr[20]` , structs can have the same thing `struct player arr[20]` that creates 20 player instances and saves it in an array so we can access like this `arr[0].x` 

- struct arrays can be followed by sets of struct initializers 
  ```c
  struct { int x; int y; } arr[] = { {10, 20} , {30, 40}, {50, 60} };
  ```

- structures don't have their size strict to the number of elements inside the struct as a compiler adds alignment in the structs that are sort of hidden unnamed holes that optimize accessing data of the struct, for example
  ```c
  struct idk{
	  char x;
	  int y;
  }
  ```
  this struct will take 8 bytes not 5, the `sizeof` operator return the proper size after alignment, you may ask why , because CPU reads data as 4 bytes (DWORD) so x + 3 bytes of y is invalid so it adds 4 unused bytes after x
  ```c
    struct idk{
	  char x;
	  //hidden 3 bytes here
	  int y;
  }
  ```

- A struct cant have an instance of itself inside itself so `struct foo { struct foo instance; } ` is invalid but it can contain a pointer to itself so 
  `struct foo { struc foo* instance; } ` is completely since its contains a pointer variable 

- In a compiler or preprocessor, the **symbol table manager** is responsible for storing and retrieving identifiers using a hash table. It typically provides two main functions: **`install`** and **`lookup`**.

	- The **`install`** function inserts a new entry into the hash table. It takes a name (the identifier) and a pointer to the associated object or value.
    
	- The **`lookup`** function searches the hash table for a given name and returns the corresponding entry if it exists.
    
	For example, when the preprocessor encounters:
	`#define age 123`

	It inserts `age` into the hash table as the key (the identifier name), and stores a pointer to the replacement value (`123`) as the associated data. This allows the preprocessor to quickly find `age` later and substitute it with `123` during preprocessing.

- `typedef` is a keyword in C that allows creating new data type names , the new name will be a synonym for the original data type so it can be used in declarations, casts, etc . Example:
  ```c
  typedef int length;
  
  length x; //same as int x;
  length y; // same as int y;
  
  ```

  it can be used in struct declarations to avoid using the struct key word so:
  ```c
  typedef struct foo{
	  int x;
	  int y;
  }tfoo;
  
  typedef struct foo tfoo; //is valid and eqv to the 1st part
  
  tfoo struct1 = {0 , 0}; //this is eqv to struct foo struct1 = {0, 0};
  ```

- It is important to emphasize that `typedef` does not create a new type; it creates a type alias. Unlike `#define`, which performs pure textual substitution during preprocessing, `typedef` is handled by the compiler and understands C’s type system.

	Because of this, `typedef` can correctly represent complex types such as function pointers. For example:
	
	`typedef int (*func)(char*, char*);`
	
	This defines `func` as an alias for a pointer to a function that takes two `char*` arguments and returns an `int`. After this declaration, we can declare function pointers simply by writing:
	
	`func f1;`
	
	instead of the more complex:
	
	`int (*f1)(char*, char*);`
	
	this is about creating **one reusable function pointer type**, and then using that type for many  different function pointers — as long as the functions have the same signature.[^19]

- Unions is a variable that can hold multiple objects to different types and sizes in a single area of storage, the size of that storage will be in the size of the biggest data type in the union ( the compiler keeps track of the size and alignment required ) , its useful for saving storage when you know you are going to use single objects in the union at a time, union declaration looks like a struct:
  ```c
  union uni{
	  int x;
	  char y;
	  int z;
  }
  ```
  members of a union have offset of 0 to the base address of the union since its a single element at a time

- Programmers using unions should keep track of what type is currently stored in the union 

- Union can have operations just like structs `copying, assigning, accessing members` , they also can be nested inside structs

- a bit filed is a set of adjacent bits within a defined storage unit , "Word" ( 2bytes ) , the syntax is similar to that of a struct and is just as a way to access bit values instead of shifting and masking
  example:
  ```c
  struct flags{
	  unsinged int protected: 1;
	  unsinged int reserved: 1;
	  unsinged int commited: 1;
  }
  ```
  that defines 3 containers each of a single bit field, fields are defined as ints and can be used in any athematic operation normally, bitfields overlapping a word size is implementation defined , they also can be assigned from left to right or right to left , thats why everything about bitfields is implementation defined 

- A text stream consists of lines where each line is ended with the new line character

- `getchar` function returns a single char form output stream , moves the file pointer, returns EOF if its the end of file

- `putchar` function prints a single char on the screen

- `printf` function takes 2 arguments (the format specifier, tokens) format is a string that has conversions `%A` where A is the key[^20] , `note printf(String)` is valid but the string cant have % so we use `printf("%s", String)`

- Variable arguments are a tricky part of C because they are necessary but often not explained well.  
    Take this example:
    
    `int printf(const char *fmt, ...);`
    
    The `printf` function takes a format string and `...` (three dots).  
    Those `...` mean the function accepts a variable number of arguments.
    
- The macros used to work with variable arguments are defined in the `stdarg.h` header.  
    Since the variable arguments are unnamed, we cannot access them directly, so we use special macros.
    
- `va_list` is a type that represents the state needed to iterate over the variable arguments.  
    It does **not** hold arguments — instead, it keeps track of the current position in the argument list. You can think of `va_list` like an **iterator or cursor** that moves through the extra arguments one by one, its implementation defined weather its a pointer, array, list.
    It is also called **argument pointer**
    
- `va_start` initializes the `va_list`.  
    It must be given the name of the last fixed parameter (in this case `fmt`).  
    It sets up the `va_list` so that it points to the first variable argument.
    
- `va_arg` is used to retrieve each argument one by one.  
    You must specify the type of the argument when retrieving it, and pass it to the `va_arg` function so it can increment the ap correctly and return the proper type
    each call to `va_arg` will increment the ap forward to the next argument 
    
- `va_end` must be called when you're done, to clean up.
    
- Important rule: There must be **at least one named parameter before `...`**, because `va_start` needs it to determine where the variable arguments begin.  
    In our example, that parameter is `const char *fmt`.

summary of Variable arguments:
`va_list` is the argument pointer
`va_start` assigns the argument pointer to the 1st variable pointer
`va_arg` increment the argument pointer to point to the next variable, we must pass the type of the current element so `va_arg` knows what type to return and how much to increment to point to the next argument
`va_end` is used to clean up , must be called before function returning or going out of scope

- `scanf` reads input from input stream and formats the input into tokens , where you provide a format and a pointer to a buffer that takes the input for example:
  ```c
  input stread -> 123 "Hello world" 0222
  
  int number1;
  int number2;
  char string[256];
  scanf("%d %s %d", &number1, &number2, string);
  // number1 will be 123 , number2 will be 0222 , string will be Hello world
  ```
  it takes input, tokenize it based on format, save each token in the memory pointed by the address (the address must be big enough), `scanf` stops when it finished all the format specifiers or the input doesn't contain the token needed and returns the number of successful matches

- `sscanf` is similar to `scanf` difference is it doesn't read from the input stream, it reads from a string given and does the same function as the `scanf`
  `int sscanf(char* string, char* format, ...)` , string is the string we want to get tokens out of and format should be familiar from `scanf`

- both `s/scanf` follow the conversion specifications which directs the flow of tokenizing , general form `%[*][width][length]specifier` 
	
	### `%` Starts a conversion specification.
	### `*` (Assignment Suppression) If present, the input is read **but not stored**.
	### `width` Maximum number of characters to read.
	### `length` Modifies the expected argument type, refer to the [^20]
	### `%[ ]` (Scan set) Reads characters that match a set, ex `scanf("%[a-z]", str);`  
	reads lower case letters only, `scanf("%[^0-9]", str);` Reads until a digit appears.

- Not providing a pointer to `s/scanf` will not be caught by the compiler, so its better to double check to avoid getting undefined behavior

- `sprintf` is similar to `s/scanf` but it gathers tokens and forms a string (the opposite of `sscanf` and `scanf`)

- Reading and wiring to files can be accomplished in C by using the `stdlib` functions `fopen()` and `fclose` , those functions use a File pointer which points to a struct that have information about the file such as the location of the buffer holding file content , current character position where the pointer is pointing, weather the file is being read or written , if its the end of the file, if there is errors while accessing the file , `stdlib` includes this structure called `FILE` and is initialized by `fopen` function which returns a `FILE*` and takes the file name and access mode (Read "r" , write "w" , append "a" ,or binary "b") those access flags can be overloaded such as "rw" for reading and writing accesses

- When the command interpreter (the shell) runs a program, three files are open, with file descriptors 0, 1, and 2, called the standard input, the standard output, and the standard error. If a program reads 0 and writes 1 and 2, it can do input and output without worrying about opening files

- If a file doesn't exist , it will be created , reading a non-existing file will error and `Fopen` will return NULL 

- Opening an existing file for writing will cause old content in file to be deleted, but appending will leave the old content and append new content 

- `getc` function returns the next character to the file pointer and EOF at the end
- `putc` function writes a character to the file

- the input stream `stdin` , output steam `stdout` , error stream `stderr` are all file pointers defined in the `stdlib` library , and they can be manipulated as files that's why `getchar` calls `getc` and `putchar` calls `putc` with the proper streams pre defined of course
  `#define putchar(c) putc(c, stdout)`

- `fscanf` and `fprintf` maybe used and are similar to `scanf` and `printf` functions except they take a file pointer as a 1st argument
  
- `fclose` frees the file and cleans up , its necessary for use after opening files  
- `ferror` returns a non-zero if an error occurred on the stream specified
- `exit` terminates the program just like return in main calls exit 

- `fgets` similar to `getline | gets` , what's different is that it reads depending on size of the like which is maxline-1 as the line returned will have the '\0'
- `fputs` similar to `puts` writes a line to the file , the line given cant contain '\n'

- `system` function executes the given command string in the command line

- `malloc` and `calloc` allocate memory on the heap , those memory allocations are dynamically allocated 
	Dynamic allocation means:
	- The size is determined at **runtime**, and allocation is done at runtime
	- The memory remains allocated until explicitly freed with `free`
	- The programmer manually manages its lifetime (it doesn't get freed automatically when out of scope , sort of static , but it requires the programmer to free it when its no longer needed and it can be used anywhere in the code sort of like a global variable )

- `malloc` returns a void pointer to n bytes of uninitialized memory, or NULL if it fails
- `calloc` returns a void pointer to enough space for an array of n objects of specified size , or NULL if failed , the storage is initialized to 0 automatically, example: 
  `int* ptr = (int*)calloc(n , sizeof(int))`  allocate memory block of size (n\*sizeof(int))

- void pointer can be casted to the designated data type the storage should hold

- `free` frees the memory block allocated by `m/calloc` , this is very important to do 

- In most operating systems, dynamically allocated memory (the heap) is managed by a **memory allocator** inside the C runtime library — not directly by the OS for every allocation.

- The allocator manages the heap as a collection of memory **blocks**. Each block typically contains:
	- A **header (metadata)** that stores information such as:
	    - The size of the block
	    - Whether the block is free or allocated [^21]
	- The **payload** (the usable memory returned to the program)


- The allocator usually keeps track of unused memory using a structure called a **free list**.
	- The free list contains blocks that were previously allocated and then freed.
	
	- When the program requests memory (e.g., via `malloc`), the allocator searches the free list for a block large enough to satisfy the request

	- If a suitable block is found, it may:
	    - Use the entire block, or
	    - Split the block into two parts (one allocated, one remaining free).
	(this algorithm is called **first fit**)

- Free blocks often contain:

	- A header (metadata)[^21]
	- A pointer to the next free block
	- Sometimes a pointer to the previous free block (if it’s a doubly linked list)
	This forms a linked list of free blocks.

- There is another algorithm called **best fit** that Traverse the entire free list and select the **smallest block that is still large enough** (unlike the first fit which Traverse the free list and select the **first block that is large enough** to satisfy the request) and if the block is found its unlinked from the list and returned to the user , any residue remains on the free list ( if no big enough block is found , a large chuck is obtained from the operating system and linked to the free list )

- Freeing also causes a search of the free list, to find the proper place to insert the block being freed. If the block being freed is adjacent to a free block on either side, it is coalesced with it into a single bigger block, so storage does not become too fragmented.

- In malloc, the requested size in characters is rounded up to the proper number of header sized units; the block that will be allocated contains one more unit, for the header itself ,and this is the value recorded in the size field of the header. The pointer returned by malloc points at the free space, not at the header itself. The user can do anything with the space requested, but if anything is written outside of the allocated space the list is likely to be scrambled.

 - That's the END :) , to more books 🍵

---
---
# Footer Reference

## 3️⃣ Quick Reference Table

| Conversion Type                         | Automatic      | Manual (cast) | Example                                                    |
| --------------------------------------- | -------------- | ------------- | ---------------------------------------------------------- |
| int → float / double                    | ✅ legal        | ✅ legal       | int i=5; float f=i;                                        |
| float → int                             | ❌ implicit     | ✅ legal       | float f=3.14; int i=(int)f;                                |
| smaller int → larger int                | ✅ legal        | ✅ legal       | char c=10; int i=c;                                        |
| larger int → smaller int                | ❌ may truncate | ✅ legal       | int i=300; char c=(char)i;                                 |
| signed ↔ unsigned                       | ✅ legal        | ✅ legal       | int a=-1; unsigned int b=(unsigned int)a;                  |
| pointer → void*                         | ✅ legal        | ✅ legal       | int *p; void *v=p;                                         |
| void* → pointer type                    | ⚠ cast needed  | ✅ legal       | void *v; int *p=(int*)v;                                   |
| struct ↔ struct                         | ❌ illegal      | ❌ illegal     | struct A a; struct B b=a;                                  |
| function pointer ↔ function pointer     | ❌ illegal      | ⚠ risky       | void f(int); void (*fp)(float)=(void(*)(float))f;          |
| integer ↔ pointer                       | ❌ illegal      | ⚠ risky       | int i=(int)&x;                                             |
| function arguments to bigger data types | ✅ legal        | ✅ legal       | foo(double x){}<br>float x;<br>foo(x) // x is now a double |


---

[^1]: # C Operator Precedence (High → Low)
	
	1. `()` (function call), `[]` (array subscript), `.` (struct/union member), `->` (pointer member) – Left-to-right  
	2. Postfix: `++` `--` – Left-to-right  
	3. Unary: `+` `-` `!` `~` `++` `--` `*` (dereference) `&` (address-of), `sizeof`, type casts – Right-to-left  
	4. Multiplicative: `*` `/` `%` – Left-to-right  
	5. Additive: `+` `-` – Left-to-right  
	6. Shift: `<<` `>>` – Left-to-right  
	7. Relational: `<` `<=` `>` `>=` – Left-to-right  
	8. Equality: `==` `!=` – Left-to-right  
	9. Bitwise AND: `&` – Left-to-right  
	10. Bitwise XOR: `^` – Left-to-right  
	11. Bitwise OR: `|` – Left-to-right  
	12. Logical AND: `&&` – Left-to-right  
	13. Logical OR: `||` – Left-to-right  
	14. Ternary: `?:` – Right-to-left  
	15. Assignment: `=` `+=` `-=` `*=` `/=` `%=` `<<=` `>>=` `&=` `^=` `|=` – Right-to-left  
	16. Comma: `,` – Left-to-right
	

[^2]: higher Operator Precedence are grouped together

[^4]:       • If either operand is long double, convert the other to long double. 
	• Otherwise, if either operand is double, convert the other to double. • Otherwise, if either operand is float, convert the other to float. 
	• Otherwise, convert char and short to int. 
	• Then, if either operand is long, convert the other to long.

[^5]: please follow the conversions manual

[^6]: if x is 1 and y is 2, then x & y is zero while x && y is one.

[^7]: Parentheses are not necessary around the first expression of a conditional expression, since the precedence of ?: is very low, just above assignment. They are advisable anyway, however, since they make the condition part of the expression easier to see.

[^8]: a fix to do
	int x = 10;
	x = (x +1) % INT_MAX;

[^9]: thats why `for(int i = 0; i < 10; i++)` will print same result as `for(int i = 0; i < 10; ++i)` , because update is done after body is executed 

[^10]: Array sizes must be specified with the definition, but are optional with an extern declaration.

[^11]: Because `x` stays in memory.

[^12]:     \a - bell
	\b - backspace
	\f - formfeed
	\n - newline
	\r - carriage return
	\t - horizontal tab
	\v - vertical tab
	\\\ - backslash
	\\? - question mark
	\\' - single quote
	\\" - double qutoe
	\ooo - octal
	\xhh - hexadecimal 
	\0 - Zero ( null terminator )

[^13]: \0 is the actual ZERO char with numeric value 0 but '0' isnt, and \0 is referred to as the NULL terminator

[^14]: ASCII upper case letters and lower case letters have a fixed distance apart, so distance between 'A' and 'a' is the same distance between 'C' and 'c'

[^15]: based on this info we can even change the array reference from arr\[i] to \*(arr + i), also &a\[i] same as a+i, fun fact thats the internal implementation from the compiler

[^16]: incrementing the `stringsPointers` will point to the next char pointers (we cant increment the array by itself as we said before so we can cast the array to char** ), incrementing `*stringsPointers` will point to the next char in the 1st element , this next example is a demo code
```c
int main() {
	char* arr[256]; //normal strings array
	arr[0] = "Hello";
	arr[1] = "World";
	arr[2] = "Bye";
	arr[3] = "World";

	printf("%s\t" "%s\t" "%s\t" "%s\t\n", arr[0], arr[1], arr[2], arr[3]); 

	char** arrPtr = (char**)arr; //pointer to char pointers
	arrPtr++; // increments the pointer to the next string (arr[0] -> arr[1])

	printf("%s\t" "%s\t" "%s\t" "%s\t\n", arrPtr[0], arrPtr[1], arrPtr[2], "Out of bound");

	arr[0]++; //increments the char pointer to the next charachter (string[0] -> string[1])
	printf("%s\t" "%s\t" "%s\t" "%s\t\n", arr[0], arr[1], arr[2], arr[3]);

}

/*

Output:
Hello   World   Bye     World
World   Bye     World   Out of bound
ello    World   Bye     World

*/
```

[^17]: ![[Pasted image 20260219193941.png]]

[^18]: These are most complex pointer / array declarations:
	#### Level 1 Read & Interpret
	- `int *p[10];` - > array of 10 pointers of type int
	- `int (*p)[10]` -> `p` is a pointer to an array of 10 integers
	- `int *(*p)[5]` -> `p` is a pointer to an **array of 5** elements that are **pointers to int**
	- `int (*p[5])[10]` -> array of 5 pointer, each pointer pointing to a 10 element int array
	- `int (*p)(int)` -> this declares a function pointer that takes an int and returns an int
	
	#### Level 2 Function Pointer Complexity
	- `int (*(*p)(int))[10]` -> `p` is a **pointer to a function that takes an int and returns a pointer to an array of 10 ints**
	- `void (*signal(int, void (*)(int)))(int)` -> `signal` is a **function that takes an int and a pointer to a function (int→void) and returns a pointer to a function takes int and returns void**
	- `char *(*(*a[N])())()` -> `a` is an **array of N pointers to functions returning pointers to functions returning char***
	- `int (*(*fp)(int, int))(int)` -> `fp` is a **pointer to a function that takes 2 ints and returns a pointer to a function taking an int and returning int**
	- `int (*(*p[5])(double))[3]` -> `p` is an **array of 5 pointers to functions that take a double and return a pointer to an array of 3 ints**

[^19]: With `typedef`:
```c
	typedef int (*func)(int, int);  
	  
	func f1;  
	func f2;  
	func f3;
```
Without `typedef`:

```c
int (*f1)(int, int);  
int (*f2)(int, int);  
int (*f3)(int, int);

//this is generally ugly and better to use one function pointer 
``` 

[^20]:     `%d` – Signed decimal integer  
	`%i` – Signed decimal integer (same as `%d` in `printf`)  
	`%u` – Unsigned decimal integer  
	`%o` – Unsigned octal (base 8)  
	`%x` – Unsigned hexadecimal (lowercase)  
	`%X` – Unsigned hexadecimal (uppercase)
	
	`%c` – Single character  
	`%s` – String (null-terminated char array)
	
	`%f` – Decimal floating point (default 6 decimal places)  
	`%lf` – Double (same as `%f` in `printf`, but required in `scanf`)  
	`%e` – Scientific notation (lowercase)  
	`%E` – Scientific notation (uppercase)  
	`%g` – Shortest representation of `%f` or `%e`  
	`%G` – Shortest representation (uppercase)
	`%p` – Memory address (pointer)

[^21]: ![[Pasted image 20260302025800.png]]
