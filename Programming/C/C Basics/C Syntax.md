
## Const

Const Types are typically in all caps ( ie. const int VARONE = 5) to benefit readablity

const must be initialized and declared in the same statement

## Operators

C divides the operators into the following groups:

- Arithmetic operators
- Assignment operators
- Comparison operators
- Logical operators
- Bitwise operators


Arithmetic Operators

Arithmetic operators are used to perform common mathematical operations.

| Operator | Name           | Description                            | Example | Try it |
| -------- | -------------- | -------------------------------------- | ------- | ------ |
| +        | Addition       | Adds together two values               | x + y   |        |
| -        | Subtraction    | Subtracts one value from another       | x - y   |        |
| *        | Multiplication | Multiplies two values                  | x * y   |        |
| /        | Division       | Divides one value by another           | x / y   |        |
| %        | Modulus        | Returns the division remainder         | x % y   |        |
| ++       | Increment      | Increases the value of a variable by 1 | ++x     |        |
| --       | Decrement      | Decreases the value of a variable by 1 | --x     |        |

">>" Operator is the right shift operator 
	- right shifting my n divides the number by 2^n

#include <stdio.h>

int main() {
    unsigned int x = 64;  // Binary: 0100 0000
    printf("x >> 1 = %u\n", x >> 1);  // Divide by 2, Output: 32
    printf("x >> 2 = %u\n", x >> 2);  // Divide by 4, Output: 16
    printf("x >> 3 = %u\n", x >> 3);  // Divide by 8, Output: 8
    return 0;
}

## Booleans

bool is not a built in data type so you must import the following header file to use it: 

"#include <stdbool.h>"

Boolean values are returned as integers 
1 or any number that is not 0 represents true
0 represents false

## Conditions and If Statements

- Use `if` to specify a block of code to be executed, if a specified condition is `true`
- Use `else` to specify a block of code to be executed, if the same condition is `false`
- Use `else if` to specify a new condition to test, if the first condition is `false`
- Use `switch` to specify many alternative blocks of code to be executed

if (_condition_) {  
  _// block of code to be executed if the condition is true_  
}

### Switch Statements 
	switch (_expression_) {  
	  case x:  
	    _// code block_  
	    break;  
	  case y:  
	    _// code block_  
	    break;  
	  default: // Code for "else" (none of the cases match) 
	  break;
	}



## Ternary Operators

- else if 
		_variable_ = (_condition_) ? _expressionTrue_ : _expressionFalse_;

In C, the term **"ternary operator"** specifically refers to the conditional operator (`? :`), as it is the only operator in the language that takes **three operands**. By definition, a "ternary operator" must have three parts.

### The Conditional Operator (`? :`)
This is the only ternary operator in C, and its syntax and usage were explained above.

### Other Operators with Similar Behavior
While there are no other ternary operators in C, there are some operators or constructs that might appear similar or serve comparable purposes under specific circumstances:

1. **Logical OR (`||`) and AND (`&&`)**  
   These are binary (two-operand) operators but can be used in chained conditions:
   ```c
   int result = condition1 || condition2;
   ```
   However, they don't allow an "else" outcome like the ternary operator.

2. **`if-else` Statement**  
   The `if-else` statement is not an operator, but it provides a more explicit alternative for conditional logic:
   ```c
   if (condition) {
       result = value_if_true;
   } else {
       result = value_if_false;
   }
   ```

3. **Compound Operators (Comma Operator)**  
   The comma operator (` , `) evaluates multiple expressions in sequence and returns the last value. While it isn't a ternary operator, it can sometimes be used in clever ways:
   ```c
   int x = (expression1, expression2, result);
   ```

### Conclusion
In C, the conditional operator (`? :`) is the **only ternary operator**. If you're looking for alternative approaches to perform similar operations, constructs like `if-else` or logical operators may help, but they don't qualify as ternary.


## Loops

### While Loop

The `while` loop loops through a block of code as long as a specified condition is `true`:

**Syntax**

while (_condition_) {  
  _// code block to be executed_  
}

### Do/While Loop

The `do/while` loop is a variant of the `while` loop. This loop will execute the code block once, before checking if the condition is true, then it will repeat the loop as long as the condition is true.

**Syntax**

do {  
  _// code block to be executed_}  
while (_condition_);

### For Loop

When you know exactly how many times you want to loop through a block of code, use the `for` loop instead of a `while` loop:

**Syntax**

for (_expression 1_; _expression 2_; _expression 3_) {  
  _// code block to be executed_  
}

- the first expression is only executed once at the start of the for loop
- the third expression updates the loop variable each time the code block is executed

### Break and Continue Statements

You have already seen the `break` statement used in an earlier chapter of this tutorial. It was used to "jump out" of a [`switch`](https://www.w3schools.com/c/c_switch.php) statement.

The `break` statement can also be used to jump out of a **loop**

The `continue` statement breaks one iteration (in the loop), if a specified condition occurs, and continues with the next iteration in the loop.

## Arrays

Arrays are used to store multiple values in a single variable, instead of declaring separate variables for each value.

To create an array, define the data type (like `int`) and specify the name of the array followed by **square brackets []**.

To insert values to it, use a comma-separated list inside curly braces, and make sure all values are of the same data type:

int myNumbers[] = {25, 50, 75, 100};

To access an array element, refer to its **index number**.

Array indexes start with **0**: [0] is the first element. [1] is the second element, etc.

This statement accesses the value of the **first element [0]** in `myNumbers`:

### Example

int myNumbers[] = {25, 50, 75, 100};  
printf("%d", myNumbers[0]);  
  
// Outputs 25

- **It is important to note that all elements in an array must be of the same data type.**

### Array Size

int myNumbers[] = {10, 25, 50, 75, 100};  
printf("%lu", sizeof(myNumbers)); // Prints 20

Why did the result show `20` instead of `5`, when the array contains 5 elements?

- **It is because the `sizeof` operator returns the size of a type in bytes.**

You learned from the [Data Types chapter](https://www.w3schools.com/c/c_data_types.php) that an `int` type is usually 4 bytes, so from the example above, 4 x 5 (_4 bytes x 5 elements_) = **20 bytes**.

Knowing the memory size of an array is great when you are working with larger programs that require good memory management.