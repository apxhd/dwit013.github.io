

Chapter 10 – Debugging
======================

Debugging
=========

 This chapter covers some tools and techniques for finding the root cause of bugs in your program to help you fix bugs faster and with less effort.

To paraphrase an old joke among programmers, “Writing code accounts for 90 percent of programming. Debugging code accounts for the other 90 percent.”

Your computer will do only what you tell it to do; it won’t read your mind and do what you _intended_ it to do. Even professional programmers create bugs all the time, so don’t feel discouraged if your program has a problem.

Fortunately, there are a few tools and techniques to identify what exactly your code is doing and where it’s going wrong. First, you will look at logging and assertions, two features that can help you detect bugs early. In general, the earlier you catch bugs, the easier they will be to fix.

Second, you will look at how to use the debugger. The debugger is a feature of IDLE that executes a program one instruction at a time, giving you a chance to inspect the values in variables while your code runs, and track how the values change over the course of your program. This is much slower than running the program at full speed, but it is helpful to see the actual values in a program while it runs, rather than deducing what the values might be from the source code.

Raising Exceptions
==================

Python raises an exception whenever it tries to execute invalid code. In [Chapter 3](/chapter3 "Chapter 3. Functions"), you read about how to handle Python’s exceptions with `try` and `except` statements so that your program can recover from exceptions that you anticipated. But you can also raise your own exceptions in your code. Raising an exception is a way of saying, “Stop running the code in this function and move the program execution to the `except` statement.”

Exceptions are raised with a `raise` statement. In code, a `raise` statement consists of the following:

*   The `raise` keyword
    
*   A call to the `Exception()` function
    
*   A string with a helpful error message passed to the `Exception()` function
    

For example, enter the following into the interactive shell:

\>>> **raise Exception('This is the error message.')**
Traceback (most recent call last):
  File "<pyshell#191>", line 1, in <module>
    raise Exception('This is the error message.')
Exception: This is the error message.

If there are no `try` and `except` statements covering the `raise` statement that raised the exception, the program simply crashes and displays the exception’s error message.

Often it’s the code that calls the function, not the function itself, that knows how to handle an exception. So you will commonly see a `raise` statement inside a function and the `try` and `except` statements in the code calling the function. For example, open a new file editor window, enter the following code, and save the program as _boxPrint.py_:

   def boxPrint(symbol, width, height):
       if len(symbol) != 1:
❶         raise Exception('Symbol must be a single character string.')
       if width <= 2:
❷         raise Exception('Width must be greater than 2.')
       if height <= 2:
❸         raise Exception('Height must be greater than 2.')
       print(symbol \* width)
       for i in range(height - 2):
           print(symbol + (' ' \* (width - 2)) + symbol)
       print(symbol \* width)

   for sym, w, h in (('\*', 4, 4), ('O', 20, 5), ('x', 1, 3), ('ZZ', 3, 3)):
       try:
           boxPrint(sym, w, h)
❹     except Exception as err:
❺         print('An exception happened: ' + str(err))

Here we’ve defined a `boxPrint()` function that takes a character, a width, and a height, and uses the character to make a little picture of a box with that width and height. This box shape is printed to the screen.

Say we want the character to be a single character, and the width and height to be greater than 2. We add `if` statements to raise exceptions if these requirements aren’t satisfied. Later, when we call `boxPrint()` with various arguments, our `try`/`except` will handle invalid arguments.

This program uses the `except Exception as err` form of the `except` statement ❹. If an `Exception` object is returned from `boxPrint()` ❶❷❸, this `except` statement will store it in a variable named `err`. The `Exception` object can then be converted to a string by passing it to `str()` to produce a user-friendly error message ❺. When you run this _boxPrint.py_, the output will look like this:

\*\*\*\*
\*  \*
\*  \*
\*\*\*\*
OOOOOOOOOOOOOOOOOOOO
O                  O
O                  O
O                  O
OOOOOOOOOOOOOOOOOOOO
An exception happened: Width must be greater than 2.
An exception happened: Symbol must be a single character string.

Using the `try` and `except` statements, you can handle errors more gracefully instead of letting the entire program crash.

Getting the Traceback as a String
=================================

When Python encounters an error, it produces a treasure trove of error information called the _traceback_. The traceback includes the error message, the line number of the line that caused the error, and the sequence of the function calls that led to the error. This sequence of calls is called the _call stack_.

Open a new file editor window in IDLE, enter the following program, and save it as _errorExample.py_:

def spam():
    bacon()
def bacon():
    raise Exception('This is the error message.')

spam()

When you run _errorExample.py_, the output will look like this:

Traceback (most recent call last):
  File "errorExample.py", line 7, in <module>
    spam()
  File "errorExample.py", line 2, in spam
    bacon()
  File "errorExample.py", line 5, in bacon
    raise Exception('This is the error message.')
Exception: This is the error message.

From the traceback, you can see that the error happened on line 5, in the `bacon()` function. This particular call to `bacon()` came from line 2, in the `spam()` function, which in turn was called on line 7. In programs where functions can be called from multiple places, the call stack can help you determine which call led to the error.

The traceback is displayed by Python whenever a raised exception goes unhandled. But you can also obtain it as a string by calling `traceback.format_exc()`. This function is useful if you want the information from an exception’s traceback but also want an `except` statement to gracefully handle the exception. You will need to import Python’s `traceback` module before calling this function.

For example, instead of crashing your program right when an exception occurs, you can write the traceback information to a log file and keep your program running. You can look at the log file later, when you’re ready to debug your program. Enter the following into the interactive shell:

\>>> **import traceback**
>>> **try:**
         **raise Exception('This is the error message.')**
**except:**
         **errorFile = open('errorInfo.txt', 'w')**
         **errorFile.write(traceback.format\_exc())**
         **errorFile.close()**
         **print('The traceback info was written to errorInfo.txt.')**

116
The traceback info was written to errorInfo.txt.

The `116` is the return value from the `write()` method, since 116 characters were written to the file. The traceback text was written to _errorInfo.txt_.

Traceback (most recent call last):
  File "<pyshell#28>", line 2, in <module>
Exception: This is the error message.

Assertions
==========

An _assertion_ is a sanity check to make sure your code isn’t doing something obviously wrong. These sanity checks are performed by `assert` statements. If the sanity check fails, then an `AssertionError` exception is raised. In code, an `assert` statement consists of the following:

*   The `assert` keyword
    
*   A condition (that is, an expression that evaluates to `True` or `False`)
    
*   A comma
    
*   A string to display when the condition is `False`
    

For example, enter the following into the interactive shell:

\>>> **podBayDoorStatus = 'open'**
>>> **assert podBayDoorStatus == 'open', 'The pod bay doors need to be "open".'**
>>> **podBayDoorStatus = 'I\\'m sorry, Dave. I\\'m afraid I can\\'t do that.'**
>>> **assert podBayDoorStatus == 'open', 'The pod bay doors need to be "open".'**
Traceback (most recent call last):
  File "<pyshell#10>", line 1, in <module>
    assert podBayDoorStatus == 'open', 'The pod bay doors need to be "open".'
AssertionError: The pod bay doors need to be "open".

Here we’ve set `podBayDoorStatus` to `'open'`, so from now on, we fully expect the value of this variable to be `'open'`. In a program that uses this variable, we might have written a lot of code under the assumption that the value is `'open'`—code that depends on its being `'open'` in order to work as we expect. So we add an assertion to make sure we’re right to assume `podBayDoorStatus` is `'open'`. Here, we include the message `'The pod bay doors need to be "open".'` so it’ll be easy to see what’s wrong if the assertion fails.

Later, say we make the obvious mistake of assigning `podBayDoorStatus` another value, but don’t notice it among many lines of code. The assertion catches this mistake and clearly tells us what’s wrong.

In plain English, an `assert` statement says, “I assert that this condition holds true, and if not, there is a bug somewhere in the program.” Unlike exceptions, your code should _not_ handle `assert` statements with `try` and `except`; if an `assert` fails, your program _should_ crash. By failing fast like this, you shorten the time between the original cause of the bug and when you first notice the bug. This will reduce the amount of code you will have to check before finding the code that’s causing the bug.

Assertions are for programmer errors, not user errors. For errors that can be recovered from (such as a file not being found or the user entering invalid data), raise an exception instead of detecting it with an `assert` statement.

Using an Assertion in a Traffic Light Simulation
------------------------------------------------

Say you’re building a traffic light simulation program. The data structure representing the stoplights at an intersection is a dictionary with keys `'ns'` and `'ew'`, for the stoplights facing north-south and east-west, respectively. The values at these keys will be one of the strings `'green'`, `' yellow'`, or `'red'`. The code would look something like this:

market\_2nd = {'ns': 'green', 'ew': 'red'}
mission\_16th = {'ns': 'red', 'ew': 'green'}

These two variables will be for the intersections of Market Street and 2nd Street, and Mission Street and 16th Street. To start the project, you want to write a `switchLights()` function, which will take an intersection dictionary as an argument and switch the lights.

At first, you might think that `switchLights()` should simply switch each light to the next color in the sequence: Any `'green'` values should change to `'yellow'`, `'yellow'` values should change to `'red'`, and `'red'` values should change to `'green'`. The code to implement this idea might look like this:

def switchLights(stoplight):
    for key in stoplight.keys():
        if stoplight\[key\] == 'green':
            stoplight\[key\] = 'yellow'
        elif stoplight\[key\] == 'yellow':
            stoplight\[key\] = 'red'
        elif stoplight\[key\] == 'red':
            stoplight\[key\] = 'green'

switchLights(market\_2nd)

You may already see the problem with this code, but let’s pretend you wrote the rest of the simulation code, thousands of lines long, without noticing it. When you finally do run the simulation, the program doesn’t crash—but your virtual cars do!

Since you’ve already written the rest of the program, you have no idea where the bug could be. Maybe it’s in the code simulating the cars or in the code simulating the virtual drivers. It could take hours to trace the bug back to the `switchLights()` function.

But if while writing `switchLights()` you had added an assertion to check that _at least one of the lights is always red_, you might have included the following at the bottom of the function:

assert 'red' in stoplight.values(), 'Neither light is red! ' + str(stoplight)

With this assertion in place, your program would crash with this error message:

   Traceback (most recent call last):
     File "carSim.py", line 14, in <module>
       switchLights(market\_2nd)
     File "carSim.py", line 13, in switchLights
       assert 'red' in stoplight.values(), 'Neither light is red! ' + str(stoplight)
❶ AssertionError: Neither light is red! {'ns': 'yellow', 'ew': 'green'}

The important line here is the `AssertionError` ❶. While your program crashing is not ideal, it immediately points out that a sanity check failed: Neither direction of traffic has a red light, meaning that traffic could be going both ways. By failing fast early in the program’s execution, you can save yourself a lot of future debugging effort.

Disabling Assertions
--------------------

Assertions can be disabled by passing the `-O` option when running Python. This is good for when you have finished writing and testing your program and don’t want it to be slowed down by performing sanity checks (although most of the time `assert` statements do not cause a noticeable speed difference). Assertions are for development, not the final product. By the time you hand off your program to someone else to run, it should be free of bugs and not require the sanity checks. See Appendix B for details about how to launch your probably-not-insane programs with the `-O` option.

Logging
=======

If you’ve ever put a `print()` statement in your code to output some variable’s value while your program is running, you’ve used a form of _logging_ to debug your code. Logging is a great way to understand what’s happening in your program and in what order its happening. Python’s `logging` module makes it easy to create a record of custom messages that you write. These log messages will describe when the program execution has reached the logging function call and list any variables you have specified at that point in time. On the other hand, a missing log message indicates a part of the code was skipped and never executed.

Using the logging Module
------------------------

To enable the `logging` module to display log messages on your screen as your program runs, copy the following to the top of your program (but under the `#! python` shebang line):

import logging
logging.basicConfig(level=logging.DEBUG, format=' %(asctime)s - %(levelname)s
- %(message)s')

You don’t need to worry too much about how this works, but basically, when Python logs an event, it creates a `LogRecord` object that holds information about that event. The `logging` module’s `basicConfig()` function lets you specify what details about the `LogRecord` object you want to see and how you want those details displayed.

Say you wrote a function to calculate the _factorial_ of a number. In mathematics, factorial 4 is 1 × 2 × 3 × 4, or 24. Factorial 7 is 1 × 2 × 3 × 4 × 5 × 6 × 7, or 5,040. Open a new file editor window and enter the following code. It has a bug in it, but you will also enter several log messages to help yourself figure out what is going wrong. Save the program as _factorialLog.py_.

import logging
logging.basicConfig(level=logging.DEBUG, format=' %(asctime)s - %(levelname)s
- %(message)s')
logging.debug('Start of program')

def factorial(n):
    logging.debug('Start of factorial(%s)' % (n))
    total = 1
    for i in range(n + 1):
        total \*= i
        logging.debug('i is ' + str(i) + ', total is ' + str(total))
    logging.debug('End of factorial(%s)' % (n))
    return total

print(factorial(5))
logging.debug('End of program')

Here, we use the `logging.debug()` function when we want to print log information. This `debug()` function will call `basicConfig()`, and a line of information will be printed. This information will be in the format we specified in `basicConfig()` and will include the messages we passed to `debug()`. The `print(factorial(5))` call is part of the original program, so the result is displayed even if logging messages are disabled.

The output of this program looks like this:

2015-05-23 16:20:12,664 - DEBUG - Start of program
2015-05-23 16:20:12,664 - DEBUG - Start of factorial(5)
2015-05-23 16:20:12,665 - DEBUG - i is 0, total is 0
2015-05-23 16:20:12,668 - DEBUG - i is 1, total is 0
2015-05-23 16:20:12,670 - DEBUG - i is 2, total is 0
2015-05-23 16:20:12,673 - DEBUG - i is 3, total is 0
2015-05-23 16:20:12,675 - DEBUG - i is 4, total is 0
2015-05-23 16:20:12,678 - DEBUG - i is 5, total is 0
2015-05-23 16:20:12,680 - DEBUG - End of factorial(5)
0
2015-05-23 16:20:12,684 - DEBUG - End of program

The `factorial()` function is returning `0` as the factorial of `5`, which isn’t right. The `for` loop should be multiplying the value in `total` by the numbers from `1` to `5`. But the log messages displayed by `logging.debug()` show that the `i` variable is starting at `0` instead of `1`. Since zero times anything is zero, the rest of the iterations also have the wrong value for `total`. Logging messages provide a trail of breadcrumbs that can help you figure out when things started to go wrong.

Change the `for i in range(n + 1):` line to `for i in range(`**`1,`** `n + 1):`, and run the program again. The output will look like this:

2015-05-23 17:13:40,650 - DEBUG - Start of program
2015-05-23 17:13:40,651 - DEBUG - Start of factorial(5)
2015-05-23 17:13:40,651 - DEBUG - i is 1, total is 1
2015-05-23 17:13:40,654 - DEBUG - i is 2, total is 2
2015-05-23 17:13:40,656 - DEBUG - i is 3, total is 6
2015-05-23 17:13:40,659 - DEBUG - i is 4, total is 24
2015-05-23 17:13:40,661 - DEBUG - i is 5, total is 120
2015-05-23 17:13:40,661 - DEBUG - End of factorial(5)
120
2015-05-23 17:13:40,666 - DEBUG - End of program

The `factorial(5)` call correctly returns `120`. The log messages showed what was going on inside the loop, which led straight to the bug.

You can see that the `logging.debug()` calls printed out not just the strings passed to them but also a timestamp and the word _DEBUG_.

Don’t Debug with print()
------------------------

Typing `import logging` and `logging.basicConfig(level=logging.DEBUG, format= '%(asctime)s - %(levelname)s - %(message)s')` is somewhat unwieldy. You may want to use `print()` calls instead, but don’t give in to this temptation! Once you’re done debugging, you’ll end up spending a lot of time removing `print()` calls from your code for each log message. You might even accidentally remove some `print()` calls that were being used for nonlog messages. The nice thing about log messages is that you’re free to fill your program with as many as you like, and you can always disable them later by adding a single `logging.disable(logging.CRITICAL)` call. Unlike `print()`, the `logging` module makes it easy to switch between showing and hiding log messages.

Log messages are intended for the programmer, not the user. The user won’t care about the contents of some dictionary value you need to see to help with debugging; use a log message for something like that. For messages that the user will want to see, like _File not found_ or _Invalid input, please enter a number_, you should use a `print()` call. You don’t want to deprive the user of useful information after you’ve disabled log messages.

Logging Levels
--------------

_Logging levels_ provide a way to categorize your log messages by importance. There are five logging levels, described in [Table 10-1](#calibre_link-17 "Table 10-1. Logging Levels in Python") from least to most important. Messages can be logged at each level using a different logging function.

Table 10-1. Logging Levels in Python

  

Level

Logging Function

Description

DEBUG

`logging.debug()`

The lowest level. Used for small details. Usually you care about these messages only when diagnosing problems.

INFO

`logging.info()`

Used to record information on general events in your program or confirm that things are working at their point in the program.

WARNING

`logging.warning()`

Used to indicate a potential problem that doesn’t prevent the program from working but might do so in the future.

ERROR

`logging.error()`

Used to record an error that caused the program to fail to do something.

CRITICAL

`logging.critical()`

The highest level. Used to indicate a fatal error that has caused or is about to cause the program to stop running entirely.

Your logging message is passed as a string to these functions. The logging levels are suggestions. Ultimately, it is up to you to decide which category your log message falls into. Enter the following into the interactive shell:

\>>> **import logging**
>>> **logging.basicConfig(level=logging.DEBUG, format=' %(asctime)s -**
**%(levelname)s - %(message)s')**
>>> **logging.debug('Some debugging details.')**
2015-05-18 19:04:26,901 - DEBUG - Some debugging details.
>>> **logging.info('The logging module is working.')**
2015-05-18 19:04:35,569 - INFO - The logging module is working.
>>> **logging.warning('An error message is about to be logged.')**
2015-05-18 19:04:56,843 - WARNING - An error message is about to be logged.
>>> **logging.error('An error has occurred.')**
2015-05-18 19:05:07,737 - ERROR - An error has occurred.
>>> **logging.critical('The program is unable to recover!')**
2015-05-18 19:05:45,794 - CRITICAL - The program is unable to recover!

The benefit of logging levels is that you can change what priority of logging message you want to see. Passing `logging.DEBUG` to the `basicConfig()` function’s `level` keyword argument will show messages from all the logging levels (DEBUG being the lowest level). But after developing your program some more, you may be interested only in errors. In that case, you can set `basicConfig()`’s `level` argument to `logging.ERROR`. This will show only ERROR and CRITICAL messages and skip the DEBUG, INFO, and WARNING messages.

Disabling Logging
-----------------

After you’ve debugged your program, you probably don’t want all these log messages cluttering the screen. The `logging.disable()` function disables these so that you don’t have to go into your program and remove all the logging calls by hand. You simply pass `logging.disable()` a logging level, and it will suppress all log messages at that level or lower. So if you want to disable logging entirely, just add `logging.disable(logging.CRITICAL)` to your program. For example, enter the following into the interactive shell:

\>>> **import logging**
>>> **logging.basicConfig(level=logging.INFO, format=' %(asctime)s -**
**%(levelname)s - %(message)s')**
>>> **logging.critical('Critical error! Critical error!')**
2015-05-22 11:10:48,054 - CRITICAL - Critical error! Critical error!
>>> **logging.disable(logging.CRITICAL)**
>>> **logging.critical('Critical error! Critical error!')**
>>> **logging.error('Error! Error!')**

Since `logging.disable()` will disable all messages after it, you will probably want to add it near the `import logging` line of code in your program. This way, you can easily find it to comment out or uncomment that call to enable or disable logging messages as needed.

Logging to a File
-----------------

Instead of displaying the log messages to the screen, you can write them to a text file. The `logging.basicConfig()` function takes a `filename` keyword argument, like so:

import logging
logging.basicConfig(**filename='myProgramLog.txt'**, level=logging.DEBUG, format='
%(asctime)s - %(levelname)s - %(message)s')

The log messages will be saved to _myProgramLog.txt_. While logging messages are helpful, they can clutter your screen and make it hard to read the program’s output. Writing the logging messages to a file will keep your screen clear and store the messages so you can read them after running the program. You can open this text file in any text editor, such as Notepad or TextEdit.

IDLE’s Debugger
===============

The _debugger_ is a feature of IDLE that allows you to execute your program one line at a time. The debugger will run a single line of code and then wait for you to tell it to continue. By running your program “under the debugger” like this, you can take as much time as you want to examine the values in the variables at any given point during the program’s lifetime. This is a valuable tool for tracking down bugs.

To enable IDLE’s debugger, click **Debug**▸**Debugger** in the interactive shell window. This will bring up the Debug Control window, which looks like [Figure 10-1](#calibre_link-18 "Figure 10-1. The Debug Control window").

When the Debug Control window appears, select all four of the **Stack**, **Locals**, **Source**, and **Globals** checkboxes so that the window shows the full set of debug information. While the Debug Control window is displayed, any time you run a program from the file editor, the debugger will pause execution before the first instruction and display the following:

*   The line of code that is about to be executed
    
*   A list of all local variables and their values
    
*   A list of all global variables and their values
    

![The Debug Control window](/images/000020.jpg)

Figure 10-1. The Debug Control window

You’ll notice that in the list of global variables there are several variables you haven’t defined, such as `__builtins__`, `__doc__`, `__file__`, and so on. These are variables that Python automatically sets whenever it runs a program. The meaning of these variables is beyond the scope of this book, and you can comfortably ignore them.

The program will stay paused until you press one of the five buttons in the Debug Control window: Go, Step, Over, Out, or Quit.

Go
--

Clicking the Go button will cause the program to execute normally until it terminates or reaches a _breakpoint_. (Breakpoints are described later in this chapter.) If you are done debugging and want the program to continue normally, click the **Go** button.

Step
----

Clicking the Step button will cause the debugger to execute the next line of code and then pause again. The Debug Control window’s list of global and local variables will be updated if their values change. If the next line of code is a function call, the debugger will “step into” that function and jump to the first line of code of that function.

Over
----

Clicking the Over button will execute the next line of code, similar to the Step button. However, if the next line of code is a function call, the Over button will “step over” the code in the function. The function’s code will be executed at full speed, and the debugger will pause as soon as the function call returns. For example, if the next line of code is a `print()` call, you don’t really care about code inside the built-in `print()` function; you just want the string you pass it printed to the screen. For this reason, using the Over button is more common than the Step button.

Out
---

Clicking the Out button will cause the debugger to execute lines of code at full speed until it returns from the current function. If you have stepped into a function call with the Step button and now simply want to keep executing instructions until you get back out, click the **Out** button to “step out” of the current function call.

Quit
----

If you want to stop debugging entirely and not bother to continue executing the rest of the program, click the **Quit** button. The Quit button will immediately terminate the program. If you want to run your program normally again, select **Debug**▸**Debugger** again to disable the debugger.

Debugging a Number Adding Program
---------------------------------

Open a new file editor window and enter the following code:

print('Enter the first number to add:')
first = input()
print('Enter the second number to add:')
second = input()
print('Enter the third number to add:')
third = input()
print('The sum is ' + first + second + third)

Save it as _buggyAddingProgram.py_ and run it first without the debugger enabled. The program will output something like this:

Enter the first number to add:
**5**
Enter the second number to add:
**3**
Enter the third number to add:
**42**
The sum is 5342

The program hasn’t crashed, but the sum is obviously wrong. Let’s enable the Debug Control window and run it again, this time under the debugger.

When you press F5 or select **Run**▸**Run Module** (with **Debug**▸**Debugger** enabled and all four checkboxes on the Debug Control window checked), the program starts in a paused state on line 1. The debugger will always pause on the line of code it is about to execute. The Debug Control window will look like [Figure 10-2](#calibre_link-19 "Figure 10-2. The Debug Control window when the program first starts under the debugger").

![The Debug Control window when the program first starts under the debugger](/images/000063.jpg)

Figure 10-2. The Debug Control window when the program first starts under the debugger

Click the **Over** button once to execute the first `print()` call. You should use Over instead of Step here, since you don’t want to step into the code for the `print()` function. The Debug Control window will update to line 2, and line 2 in the file editor window will be highlighted, as shown in [Figure 10-3](#calibre_link-20 "Figure 10-3. The Debug Control window after clicking Over"). This shows you where the program execution currently is.

![The Debug Control window after clicking Over](/images/000068.jpg)

Figure 10-3. The Debug Control window after clicking Over

Click **Over** again to execute the `input()` function call, and the buttons in the Debug Control window will disable themselves while IDLE waits for you to type something for the `input()` call into the interactive shell window. Enter **`5`** and press Return. The Debug Control window buttons will be reenabled.

Keep clicking **Over**, entering **`3`** and **`42`** as the next two numbers, until the debugger is on line 7, the final `print()` call in the program. The Debug Control window should look like [Figure 10-4](#calibre_link-21 "Figure 10-4. The Debug Control window on the last line. The variables are set to strings, causing the bug."). You can see in the Globals section that the first, second, and third variables are set to string values `'5'`, `'3'`, and `'42'` instead of integer values `5`, `3`, and `42`. When the last line is executed, these strings are concatenated instead of added together, causing the bug.

![The Debug Control window on the last line. The variables are set to strings, causing the bug.](/images/000000.jpg)

Figure 10-4. The Debug Control window on the last line. The variables are set to strings, causing the bug.

Stepping through the program with the debugger is helpful but can also be slow. Often you’ll want the program to run normally until it reaches a certain line of code. You can configure the debugger to do this with breakpoints.

Breakpoints
-----------

A _breakpoint_ can be set on a specific line of code and forces the debugger to pause whenever the program execution reaches that line. Open a new file editor window and enter the following program, which simulates flipping a coin 1,000 times. Save it as _coinFlip.py_.

   import random
   heads = 0
   for i in range(1, 1001):
❶     if random.randint(0, 1) == 1:
           heads = heads + 1
       if i == 500:
❷         print('Halfway done!')
   print('Heads came up ' + str(heads) + ' times.')

The `random.randint(0, 1)` call ❶ will return `0` half of the time and `1` the other half of the time. This can be used to simulate a 50/50 coin flip where `1` represents heads. When you run this program without the debugger, it quickly outputs something like the following:

Halfway done!
Heads came up 490 times.

If you ran this program under the debugger, you would have to click the Over button thousands of times before the program terminated. If you were interested in the value of `heads` at the halfway point of the program’s execution, when 500 of 1000 coin flips have been completed, you could instead just set a breakpoint on the line `print('Halfway done!')` ❷. To set a breakpoint, right-click the line in the file editor and select **Set Breakpoint**, as shown in [Figure 10-5](#calibre_link-22 "Figure 10-5. Setting a breakpoint").

![Setting a breakpoint](/images/000001.jpg)

Figure 10-5. Setting a breakpoint

You don’t want to set a breakpoint on the `if` statement line, since the `if` statement is executed on every single iteration through the loop. By setting the breakpoint on the code in the `if` statement, the debugger breaks only when the execution enters the `if` clause.

The line with the breakpoint will be highlighted in yellow in the file editor. When you run the program under the debugger, it will start in a paused state at the first line, as usual. But if you click Go, the program will run at full speed until it reaches the line with the breakpoint set on it. You can then click Go, Over, Step, or Out to continue as normal.

If you want to remove a breakpoint, right-click the line in the file editor and select **Clear Breakpoint** from the menu. The yellow highlighting will go away, and the debugger will not break on that line in the future.

Summary
=======

Assertions, exceptions, logging, and the debugger are all valuable tools to find and prevent bugs in your program. Assertions with the Python `assert` statement are a good way to implement “sanity checks” that give you an early warning when a necessary condition doesn’t hold true. Assertions are only for errors that the program shouldn’t try to recover from and should fail fast. Otherwise, you should raise an exception.

An exception can be caught and handled by the `try` and `except` statements. The `logging` module is a good way to look into your code while it’s running and is much more convenient to use than the `print()` function because of its different logging levels and ability to log to a text file.

The debugger lets you step through your program one line at a time. Alternatively, you can run your program at normal speed and have the debugger pause execution whenever it reaches a line with a breakpoint set. Using the debugger, you can see the state of any variable’s value at any point during the program’s lifetime.

These debugging tools and techniques will help you write programs that work. Accidentally introducing bugs into your code is a fact of life, no matter how many years of coding experience you have.

Practice Questions
==================

  

  

Q:

1\. Write an `assert` statement that triggers an `AssertionError` if the variable `spam` is an integer less than `10`.

Q:

2\. Write an `assert` statement that triggers an `AssertionError` if the variables `eggs` and `bacon` contain strings that are the same as each other, even if their cases are different (that is, `'hello'` and `'hello'` are considered the same, and `'goodbye'` and `'GOODbye'` are also considered the same).

Q:

3\. Write an `assert` statement that _always_ triggers an `AssertionError`.

Q:

4\. What are the two lines that your program must have in order to be able to call `logging.debug()`?

Q:

5\. What are the two lines that your program must have in order to have `logging.debug()` send a logging message to a file named _programLog.txt_?

Q:

6\. What are the five logging levels?

Q:

7\. What line of code can you add to disable all logging messages in your program?

Q:

8\. Why is using logging messages better than using `print()` to display the same message?

Q:

9\. What are the differences between the Step, Over, and Out buttons in the Debug Control window?

Q:

10\. After you click Go in the Debug Control window, when will the debugger stop?

Q:

11\. What is a breakpoint?

Q:

12\. How do you set a breakpoint on a line of code in IDLE?

Practice Project
================

For practice, write a program that does the following.

Debugging Coin Toss
-------------------

The following program is meant to be a simple coin toss guessing game. The player gets two guesses (it’s an easy game). However, the program has several bugs in it. Run through the program a few times to find the bugs that keep the program from working correctly.

import random
guess = ''
while guess not in ('heads', 'tails'):
    print('Guess the coin toss! Enter heads or tails:')
    guess = input()
toss = random.randint(0, 1) # 0 is tails, 1 is heads
if toss == guess:
    print('You got it!')
else:
    print('Nope! Guess again!')
    guesss = input()
    if toss == guess:
       print('You got it!')
    else:
        print('Nope. You are really bad at this game.')

[![](/images/prev.png)](/chapter9)[![](/images/toc.png)](/#toc)[![](/images/next.png)](/chapter11)

Support the author by purchasing the print/ebook bundle from [No Starch Press](https://www.nostarch.com/automatestuff) or separately on [Amazon](http://www.amazon.com/gp/product/1593275994/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1593275994&linkCode=as2&tag=playwithpyth-20&linkId=HDM7V3T6RHC5VVN4).

[![Automate the Boring Stuff book cover thumbnail](/images/automate_small_cover.png)](http://www.amazon.com/gp/product/1593275994/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1593275994&linkCode=as2&tag=playwithpyth-20&linkId=HDM7V3T6RHC5VVN4)

Read the author's other Creative Commons licensed Python books.

[![Coding with Minecraft book cover thumbnail](/images/cover_codingwithminecraft_thumb.png)](https://turtleappstore.com/book) [![Cracking Codes with Python book cover thumbnail](/images/cover_crackingcodes_thumb.png)](https://inventwithpython.com/cracking/) [![Invent with Python book cover thumbnail](/images/cover_invent4th_thumb.png)](https://inventwithpython.com/invent4thed) [![Scratch Programming Playground book cover thumbnail](/images/cover_scratchprogrammingplayground_thumb.jpg)](https://inventwithscratch.com/book/) [![Making Games with Python and Pygame book cover thumbnail](/images/cover_makinggames_thumb.png)](https://inventwithpython.com/pygame/)
