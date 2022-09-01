# Getting Started with Binary Numbers Using Python

![Binary numbers](/images/binary_in_python/binary_numbers.jpeg)

You have probably heard of binary. Binary is prefixed with bi, which means two. So what exactly is it? This article explains what binary is by first comparing it to another number system before breaking down the core concepts of the binary number system.

### Structure of the article

1. [Understanding base-10](#understanding-base-10)
2. [Binary in the context of computers](#binary-in-the-context-of-computers)
3. [Binary numbering system](#binary-numbering-system)
4. [Binary logical operators](#binary-logical-operators)
5. [Converting integers to binary](#converting-integers-to-binary)
6. [Simple task](#simple-task)

## Understanding base-10

Many, if not all, of us, have counted numbers from 0 to 9. They are used to define the base-10 numbering system. Put in a simpler way, base-10 is the way we assign place values to numerals depending on their positions. The places or positions are based on the power of 10. Each number position is 10 times the value of the number to its right. For example, the number 234 on a base-10 would be broken down as follows:

```python
2 = 2*10 ** 2
3 = 3*10 ** 1
4 = 4*10 ** 0
```

Anthropologists believe that the base-10 numbering system came about because humans have 10 fingers. Though in the context of computers, they have just "one" finger because they are electrical machines. Let me explain further.

We interact with electricity in daily life, for example when turning the lights on or off. The switch has two states: ON or OFF. We say the switch is on if electricity is following through it, and it is off if it is not flowing through it.

## Binary in the context of computers

The brain of a computer is its CPU (central processing unit). In very simple terms, the CPU is nothing more than a series of electric switches connected. One switch produces two states, that is ON and OFF. If you have two switches, then the number of states generated is four:

```python
ON-ON
ON-OFF
OFF-ON
OFF-OFF
```

If the number of switches is three, then the states derived will total eight.

```python
ON-ON
ON-OFF
OFF-ON
OFF-OFF
ON-ON
ON-OFF
OFF-ON
OFF-OFF
```

In other words, `n` switches can count `2**n` values. If you have 10 switches, then the number of values will be `2**10`.

Most modern computers are said to be _32-bit processors_. What this means is that the internal counters are made up of 32 switches each and can count up to `2**32` (~4.3 billion if you do the Math).

## Binary Numbering System

The binary numbering system works like the decimal (or base-10) system. With base-10, the first 10 numerals are 0, 1, 2, 3, 4, 5, 6, 7, 8, 9. After 9, tens are introduced. For example, the number 10 is `1 * 10**1`. 90 such values are available in the tens position. In the hundreds, a number such as 121 is equivalent to `1*10**2 + 2*10**1 + 1*10*0`.

Binary numbers are usually represented using 0 and 1. These values correspond to the states of a switch, as discussed earlier, where 0 means OFF and 1 means ON of the internal hardware components of a computer.
Typically, to represent a number in binary, it takes more than a single 0 and a single 1. Let us take the binary number 10011010010 as an example. What digit does it represent?

```python
1 = 1*2**10
0 = 0*2**9
0 = 0*2**8
1 = 1*2**7
1 = 1*2**6
0 = 0*2**5
1 = 1*2**4
0 = 0*2**3
0 = 0*2**2
1 = 1*2**1
0 = 0*2**0
```

Adding all these operations together, we will get:

```python
$ python3

>>> 1*2**10 + 0*2**9 + 0*2**8 +1*2**7 + 1*2**6 + 0*2**5 + 1*2**4 + 0*2**3 + 0*2**2 + 1*2**1 + 0*2**0

# Output
1234
```

In other words, the binary 10011010010 represents the number 1234. If you find using the operator `**` cumbersome to represent power, you can alternatively use `1 * pow(2, 10)`.

Usually, binary numbers are preferred to be limited to a _byte_, where __one byte is equivalent to 8 bits__. There is no technical reason for this although, from a hardware design perspective, it is much easier when the number of bits is a power of 2.

## Binary Logical Operators

Just like decimals, binary numbers can be added, subtracted, multiplied, and divided. Every computing device that exists has a dedicated circuit called an Arithmetic Logical Unit whose sole purpose is to take input in binary form and output the result of the operations on those numbers.

### The AND Operator

The AND operator is a logical operator in Python, denoted by the ampersand sign (that is &). It is used to return TRUE if both statements are true.

```python
$ python3

>>> x = 1
>>> x < 5 and x > 0

# Output
True
```

The value of x is indeed greater than 0 but less than 5.

At the bit level, the AND operator will return 1 if both bits are 1. If either bit is 0, the AND operator will return 0.

_x is ON and Y is ON returns 1. If x is ON and y is OFF, it will return OFF. If x is OFF and y is ON, it will return OFF. (remember that ON is represented by 1 while OFF is represented by 0)_

```python
# Example operation
   
   10101101 (173)
&  01101011 (107)
   ________
   00101001 (41)
```

In the example shown above, we are matching the bits using the AND operator.

## The OR Operator

The OR operator is denoted by the pipe sign (that is |) _bitwise_. It is used to return true if either of the operands is true.

```python
$ python3

>>> x = 5
>>> x < 5 or x == 5

# Output
True
```

_In the context of bits, if x is ON OR y is OFF, then x OR y is ON. If both x OR y is ON, then x OR y is ON. If both x OR y is OFF, then x OR y is OFF._

```python
# Example operation
   
   10101101 (173)
|  01101011 (107)
   ________
   11101111 (239)
```

Notice that there is no relationship between the input and output.

### The NOT Operator

This operator reverses an outcome. In an instance where the outcome is 0, then the NOT operator will inverse it and it will become 1. The reverse is also true: 1 becomes 0.

```python
$ python3

>>> x = 5
>>> x != 5

# Output
False
```
In the example above we know that x is equivalent to 5. The comparison operator != inverses or negates the result.

### The XOR Operator

Also known as the Exclusive OR (XOR), this operation will return 0 if both bits are 1. It is denoted by the caret sign (that is ^)

```python
# Example operation
   
   10101101 (173)
^  01101011 (107)
   ________
   11000110 (198)
```

## Converting Integers to Binary

Now that we have an in-depth understanding of what binary is, let us look at how we can convert an integer into its equivalent binary representation. We have already seen how to do so in earlier sections. For example, the number 1234 would be 10011010010. We arrived at this answer by doing the following:

```python
1*2**10 + 0*2**9 + 0*2**8 +1*2**7 + 1*2**6 + 0*2**5 + 1*2**4 + 0*2**3 + 0*2**2 + 1*2**1 + 0*2**0
```

So, how did we get the binary numbers in the first place? There are a few ways we can do the conversion in Python:
- `bin()`
- `format()`
- f-strings or string formatting

### Using the `bin()` function

The _bin_ is short for binary. It is used to convert an integer into a binary string.

```python
$ python3

>>> bin(1234)

# Output
'0b10011010010'
```
This function typically prepends _0b_. It accepts negative integers too.

```python
$ python3

>>> bin(-1234)

# Output
'-0b10011010010'
```

With the addition of the negative sign at the beginning of the binary number, we know that the integer is negative.

### Using the `format()` function

This function takes a value plus the format spec, which in this case is "b".

```python
$ python3

>>> my_negative_number = -1234
>>> my_positive_pos_number = 1234

>>> format(my_negative_number, 'b')
'-10011010010'

>>> format(my_positive_pos_number, 'b')
'10011010010'
```

Typically you do not get the _0b_ added to the beginning of the output as is the case with `bin()`.

### String formatting and f-strings

When you format a string, you have the freedom to define what options you want to be included in the output.

```python
$ python3

>>> '{0:b}'.format(1234)
'10011010010'
```

The 0 will be dynamically replaced by the parameter in `format()` . The `b` defines how you want the conversion to happen, in this case, it will be binary.

A more modern way of formatting strings as of Python3.6 is the use of "formatted string literals," also known as _f-strings_. Not only are they more readable, more concise, and less prone to error than other ways of formatting, but they are also faster!

```python
$ python3

>>> f'{0:b}'
'10011010010'
```
## Simple Task

Now that you know how to convert an integer into its binary representation using Python, try to create a function that takes an integer as input, converts it to binary, and prints out the number of occurrences of the value 0.
