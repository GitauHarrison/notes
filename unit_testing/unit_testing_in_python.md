# Unit Testing in Python

A unit test in Python essentially does two things (1) It runs a fraction or a part of an application and (2) It is verifies that that part works correctly. Let us imagine that we have a very simple script that returns a prime number when called.

```python
def prime_number(n):
    return 11
```

## Test Runners

This is unrealistically simple, but anyways, let us go ahead and test the outcome. If we want to test this piece of code, the first thing we need to do is get a test runner. A test runner is a tool that is used to run or execute tests and export results. There are two main test runners in Python. The first is the built-in [unittest](https://docs.python.org/3/library/unittest.html) package from Python, and then there is [pytest](https://docs.pytest.org/en/7.2.x/). You will see me implement a hybrid solution, utilizing both packages. The two aren't vastly different. In fact, _pytest_ is a "enhanced unittest" in that:

- It maintains the object-oriented approach using the `TestCase` class from `unittest` to structure and organize the test units
- Assertions are written using the `assert` statement, though _pytest's_ assertions are more verbose when there is a failure.

## Testing

Before we can even think of testing this script, we will need to create a file to save this work. We can create a sample file called `prime_numbers.py` then modify our code slightly to use the `print()` statement before running it on the terminal. 

```python
# prime_numbers.py

def prime_number(n):
    # return 11
    print(11)

prime_number(11)


# On the terminal
$ python3 prime_numbers.py

# Output 
11
```

A prime number is a number that is divisible only by the integer `1` and itself. In other words, a prime number's factors are `1` and itself. Example prime numbers include 2, 3 5, and 7. To properly assess if a number is prime, we can refactor our code as follows:

```python
# prime_numbers.py

def prime_number(num):
    # Check if number is greater than 1
    if num > 1:
        # Check for factors from number greater than 1
        for var in range(2, num):
            # If number is divisible, then it is not a prime number
            if (num % var) == 0:
                print(num, " is not a prime number")
                # Break out of the loop if number is not prime
                break
        # If prime number found
        else:
            print(num, " is a prime number")
    # If a number not greater than 1 is used, then it is not prime
    else:
        print(num, " is not  a prime number")

prime_number(2)

```

## Writing a Test Case

The method above does not use a test runner. The test runner, especially _pytest_, looks for any file that has the word "test" in its filename before executing the test. So, we need to create a new file with the name `test_prime_number.py`.

```python
$ touch test_prime_number.py
```

With the file created, we can update it with the following:

```python
# test_prime_number.py

import unittest
from prime_number import prime_number


class TestPrimeNumber(unittest.TestCase):
    def test_prime_number(self):
        num1 = 9
        num2 = 11
        assert prime_number(num1) == False
        assert prime_number(num2) == True
```

> A testcase is created by subclassing `unittest.TestCase`. The sole test above has been defined with method whose name start with the letters test. This naming convention informs the test runner about which methods represent tests.<br>
> The crux of each test method, in our case we only have one test method, is the call to `assert` to check for an expected result.

This test assumes that the function `prime_number()` returns either `True` or `False`. Our current function uses the print statement and does not return anything. So, I will make some modification to the code to ensure it returns either boolean.

```python
# prime_numbers.py

def prime_number(num):
    # Check if number is greater than 1
    if n > 1:
        # Check for factors from number greater than 1 (i.e 2)
        for var in range(2, n):
            # If number is divisible, then it is not a prime number
            if (num % var) == 0:
                return False
        # If prime number found
        return True
    # If a number not greater than 1 is used, then it is not prime
    else:
        print(num, " is not  a prime number")
        return False

prime_number(2)

```

Besides using the `return` statement, I have refactored the code to get rid of the `else` part of the statement returning `True`. This is so because the condition inside the `for` loop no longer breaks out of the iteration. All code blocks inside it were de-indented.


## Running A Test

To run the test above, we need to install `pytest` in an active virtual environment.

```python
(venv)$ pip3 install pytest
```

It is assumed that both `prime_number.py` and `test_prime_number.py` files are in the same directory. On your terminal, run the `pytest` command.

```python
(venv)$ pytest


# Output

======================================= test session starts ====================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item

test_prime_number.py .                                                                    [100%]

======================================= 1 passed in 0.03s  =====================================
```

The `pytest` command intelligently knows where to look for to get the test file. As mentioned above, it assumes any file with the word "test" in it, either in the current directory or subdirectories, containes unit tests.

The test passes, as you can see above. But what happends when there is a failure? To deliberately make the test fail, let us update our test file as follows:

```python
# test_prime_number.py

import unittest
from prime_number import prime_number


class TestPrimeNumber(unittest.TestCase):
    def test_prime_number(self):
        num = 11
        assert prime_number(num) == False

```

On our terminal, we can run the `pytest` command again.

```python
(venv)$ pytest

# Output

================================== test session starts ============================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item                                             

test_prime_number.py F                                                                       [100%]

========================================= FAILURES ================================================
_____________________________________ TestPrimeNumber.test_prime_number ___________________________

self = <test_prime_number.TestPrimeNumber testMethod=test_prime_number>

    def test_prime_number(self):
        n = 11
>       assert prime_number(n) == False
E       assert True == False
E        +  where True = prime_number(11)

test_prime_number.py:8: AssertionError
----------------------------------------- Captured stdout call ------------------------------------
11  is a prime number
===================================== short test summary info =====================================
FAILED test_prime_number.py::TestPrimeNumber::test_prime_number - assert True == False
===================================== 1 failed in 0.06s ===========================================
```

Pytest does a very good job in helping one figure out exactly where the test failed, and provides the expected and actual results for the failed assertion. Above, you can see that the line `assert prime_number(11)` expected `False` though our script returned `True`. This failed test was an experiment, so make sure to return the passing condition.


## Test Coverage

How can we tell that the test above is good enough? Naturally, we will use our own judgement to determine how much automated tests we need to implement as insurance against possible failures in the future. Thankfully, there is a tool that we can use to help give a better picture of the scope of our tests. This tool is called [coverage](https://coverage.readthedocs.io/en/7.0.0b1/). It excels at noting which part of the code has been executed, then analyzes the source to identify which code could have been executed but was not.

There is a plugin for `pytest` called [pytest-cov](https://pytest-cov.readthedocs.io/en/latest/) that supports code coverage in a test run. We need to install it in an active virtual environment too.

```python
(venv)$ pip3 install pytest-cov
```

To implement a test run with code coverage for `prime_number` module, we will run this command on the terminal:

```python
(venv)$ pytest --cov=prime_number

# Output
=================================== test session starts ==================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item                         

test_prime_number.py .                                                              [100%]

-------------- coverage: platform linux, python 3.8.10-final-0 --------------
Name              Stmts   Miss  Cover
-------------------------------------
prime_number.py      11      2    82%
-------------------------------------
TOTAL                11      2    82%


=================================== 1 passed in 0.07s ===================================
```

It is recommend to limit the code coverage to the application module or package which is passed as an argument tot he `--cov` option. It there is no restriction, then the code coverage will apply to all Python processes, making the output very noisy.

Note that our report has a resounding 82%. However, where is the remaining 18%? Does this mean it is possible we have not written some test for our module?

The `pytest-cov` plugin can generate the final report in several formats. The one I have shown above is the most basic format, called `term`, because it is printed in the terminal. A variant to show missing code coverage is `term-missing`. It shows what lines of code were not covered.

```python
(venv)$ pytest --cov=prime_number --cov-report=term-missing

# Output
=================================== test session starts ==================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item       

test_prime_number.py .                                                              [100%]

-------------- coverage: platform linux, python 3.8.10-final-0 --------------
Name              Stmts   Miss  Cover   Missing
-----------------------------------------------
prime_number.py      11      2    82%   17-18
-----------------------------------------------
TOTAL                11      2    82%


=================================== 1 passed in 0.07s ===================================
```

The `term-missing` shows the list of line numbers that did not execute during the test. In the case above, line 17 - 18 in the `prime_number()` function was not covered. This two lines fall under the `else` part of the conditional statement checking for a number greater than 1.

```python
# prime_number.py

def prime_number(n):
    # Check if number is greater than 1
    if n > 1:
        # ...
    else:
        # This code block was not covered by the test
        print(n, " is not  a prime number")
        return False

prime_number(2)

```

Thankfully, our code coverage treats the lines with conditional statements as needing double coverage to account for the two possible outcomes. This accounting is referred to as _branch coverage_ and is enabled by the option `--cov-branch`.

```python
(venv)$ pytest --cov=prime_number --cov-report=term-missing --cov-branch

# Output
=================================== test session starts ==================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item       

test_prime_number.py .                                                              [100%]

-------------- coverage: platform linux, python 3.8.10-final-0 --------------
---------- coverage: platform linux, python 3.8.10-final-0 -----------
Name              Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------
prime_number.py      11      2      6      1    82%   17-18
-------------------------------------------------------------
TOTAL                11      2      6      1    82%


=================================== 1 passed in 0.07s ===================================
```

To ensure that our code gets a 100% coverage, I will modify `prime_number()` to factor in if the input value is greater than 1.

```python
# test_prime_number.py

import unittest
from prime_number import prime_number


class TestPrimeNumber(unittest.TestCase):
    def test_prime_number(self):
        num1 = 9
        num2 = 11
        assert prime_number(num1) == False
        assert prime_number(num1 < 1) == False
        assert prime_number(num2) == True
        assert prime_number(num2 < 1) == False

```

Above, I have included a simple condition within the `prime_number()` function to check if the input number is actually creater than 1. To test the coverage, I can re-run the earlier command.

```python
(venv)$ pytest --cov=prime_number --cov-report=term-missing --cov-branch

# Output
=================================== test session starts ==================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item       

test_prime_number.py .                                                              [100%]

-------------- coverage: platform linux, python 3.8.10-final-0 --------------
Name              Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------
prime_number.py      11      0      6      0   100%
-------------------------------------------------------------
TOTAL                11      0      6      0   100%


=================================== 1 passed in 0.07s ===================================
```

## Code Coverage Exception

If at any one point you think that a piece of code does not need to be tested, it is possible to make such code lines as exceptions, and make them counted as covered. They will not appear in coverage reports as missing. This can be done using the text `pragma: no cover` to the line or lines in question. 

```python
# prime_number.py

def prime_number(n):
    if n > 1:
        for var in range(2, n):
            if (n % var) == 0:
                print(n, " is not a prime number")
                return False
        print(n, " is a prime number")
        return True
    else:                                        # pragma: no cover
        print(n, " is not  a prime number")      # pragma: no cover
        return False                             # pragma: no cover

prime_number(2)

```

I will modify the test module to exclude the assertions that check if the input value is greater than 1.

```python
# test_prime_number.py

import unittest
from prime_number import prime_number


class TestPrimeNumber(unittest.TestCase):
    def test_prime_number(self):
        num1 = 9
        num2 = 11
        assert prime_number(num1) == False
        assert prime_number(num2) == True

```

In the terminal, I can re-run the coverage command:

```python
(venv)$ pytest --cov=prime_number --cov-report=term-mission --cov-branch

# Output
=================================== test session starts ==================================
platform linux -- Python 3.8.10, pytest-7.2.0, pluggy-1.0.0
rootdir: /home/harry/software_development/python/flask/current_projects/notes/unit_testing
plugins: cov-4.0.0
collected 1 item       

test_prime_number.py .                                                              [100%]

-------------- coverage: platform linux, python 3.8.10-final-0 --------------
Name              Stmts   Miss Branch BrPart  Cover   Missing
-------------------------------------------------------------
prime_number.py       9      0      4      0   100%
-------------------------------------------------------------
TOTAL                 9      0      4      0   100%


=================================== 1 passed in 0.07s ===================================
```

Notice that the statements under coverage have now reduced from 11 to 9. The code coverage is 100%. If your script includes `print()` statements, you have the liberty to decide if you want them included into the tests.