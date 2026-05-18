
- \!#\[path] - provides the path of the interpreter for the script we want to execute
- Variable = value - this is how to declare and initialize variables in bash
- $Variable - this is how we use variables in bash
- read Variable - reads value from user (input) and saves it in variable
- positional argument - $1 , $2 , $3 ...
- Piping ( | ) - forward of a command to the other 
- output redirection (> , >>) - used to log the output of a command to a file (> replaces text , >> appends text)
- < , << - are used to pass an object to a command rather than using positional arguments 
	`cat << EOF` will wait for input till we write EOF

```
if [Expr]; then
	do something
elif []; then  
	do something
else
	do something
fi
```
- this is an if condition in bash
  logical == is =
  logical || is |
  logical && is &

- ${Variable,,} will ignore the upper and lower cases in logical operations
```
case Variable in 
	Case | Case)
		do something
		;;
	Case)
		do something
		;;
	*)
		default option
		;;
esac
```
- this is a case statement in bash

- ARRAY=(item1 item2 item3)
- ARRAY\[@] - means all items in the array
- ARRAY\[0] - 1st item in array

```
for item in ${ARRAY[@]}; do
	do something with $item;
done
```
- this is a for loop in bash

- Variable=$(command) - catches output of command and saves it in variable

```
myFunc(){
	echo $1
}

myFunc test // calls the function and prints test
```
- this is function in bash

- local variable - the local keyword avoids a function from overriding global variables
