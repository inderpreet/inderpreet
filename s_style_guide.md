# My Style Guide

## Formatting

### Line Size

Please keep the length of source lines to 79 characters or less, for maximum readability in the widest range of environments. Statements longer than 80 columns will be broken into sensible chunks, unless exceeding 80 columns significantly increases readability and does not hide information. Descendants are always substantially shorter than the parent and are placed substantially to the right. The same applies to function headers with a long argument list. However, never break user-visible strings such as printk messages, because that breaks the ability to grep for them.

### Indendations

Indentations are 8 characters.

### Braces
put the opening brace last on the line, and put the closing brace first, thusly:

```C
if (x is true) {
        we do y
}
```

This applies to all non-function statement blocks (if, switch, for, while, do). E.g.:

```c
switch (action) {
case KOBJ_ADD:
        return "add";
case KOBJ_REMOVE:
        return "remove";
case KOBJ_CHANGE:
        return "change";
default:
        return NULL;
}
```

and

```C
if (x == y) {
        ..
} else if (x > y) {
        ...
} else {
        ....
}
```

However, there is one special case, namely functions: they have the opening brace at the beginning of the next line, thus:

```c
int function(int x)
{
        body of function
}
```

### Spaces

use a space after these keywords:

> if, switch, case, for, do, while

but not with sizeof, typeof, alignof, or __attribute__. E.g.,

```C
s = sizeof(struct file);
```

Do not add spaces around (inside) parenthesized expressions. This example is bad:

```C
s = sizeof( struct file );
```

When declaring pointer data or a function that returns a pointer type, the preferred use of * is adjacent to the data name or function name and not adjacent to the type name. Examples:

```C
char *linux_banner;
unsigned long long memparse(char *ptr, char **retptr);
char *match_strdup(substring_t *s);
```

Use one space around (on each side of) most binary and ternary operators, such as any of these:

=  +  -  <  >  *  /  %  |  &  ^  <=  >=  ==  !=  ?  :

but no space after unary operators:

&  *  +  -  ~  !  sizeof  typeof  alignof  __attribute__  defined

no space before the postfix increment & decrement unary operators:

++  --

no space after the prefix increment & decrement unary operators:

++  --

and no space around the . and -> structure member operators.

Do not leave trailing whitespace at the ends of lines. Some editors with smart indentation will insert whitespace at the beginning of new lines as appropriate, so you can start typing the next line of code right away. However, some such editors do not remove the whitespace if you end up not putting a line of code there, such as if you leave a blank line. As a result, you end up with lines containing trailing whitespace.

Git will warn you about patches that introduce trailing whitespace, and can optionally strip the trailing whitespace for you; however, if applying a series of patches, this may make later patches in the series fail by changing their context lines.

## Naming

GLOBAL variables (to be used only if you **really** need them) need to have descriptive names, as do global functions. If you have a function that counts the number of active users, you should call that `count_active_users()` or similar, you should **not** call it `cntusr()`.

LOCAL variable names should be short, and to the point. If you have some random integer loop counter, it should probably be called `i`. Calling it `loop_counter` is non-productive, if there is no chance of it being mis-understood. Similarly, `tmp` can be just about any type of variable that is used to hold a temporary value.

## Typedefs

It’s a **mistake** to use typedef for structures and pointers.

```c
struct my_structure *a;     // preferd way of using structures
```

u8/u16/u32 are perfectly fine typedefs

In general, a pointer, or a struct that has elements that can reasonably be directly accessed should **never** be a typedef.

## Functions

Functions should be short and sweet, and do just one thing. They should fit on one or two screenfuls of text (the ISO/ANSI screen size is 80x24, as we all know), and do one thing and do that well.

A measure of the function is the number of local variables. They shouldn’t exceed 5-10, or you’re doing something wrong. Re-think the function, and split it into smaller pieces. A human brain can generally easily keep track of about 7 different things, anything more and it gets confused. You know you’re brilliant, but maybe you’d like to understand what you did 2 weeks from now.

In source files, separate functions with one blank line.

## Function return values

Functions should return a value based on the action performed. If the function is to return success or fail values, then it should be in the following nomenclature.

0 = Success
1 = Failed
... other numbers are error codes corresponding to error types. Define them globally in an errors.h file 

## Conditional Compilation

Avoid using #ifdef and #if macros as it makes reading code harder.

Instead, use such conditionals in a header file defining functions for use in those .c files, providing no-op stub versions in the #else case, and then call those functions unconditionally from .c files. The compiler will avoid generating any code for the stub calls, producing identical results, but the logic will remain easy to follow.

Prefer to compile out entire functions, rather than portions of functions or portions of expressions. Rather than putting an ifdef in an expression, factor out part or all of the expression into a separate helper function and apply the conditional to that function.

If you have a function or variable which may potentially go unused in a particular configuration, and the compiler would warn about its definition going unused, mark the definition as __maybe_unused rather than wrapping it in a preprocessor conditional. (However, if a function or variable _always_ goes unused, delete it.)

At the end of any non-trivial #if or #ifdef block (more than a few lines), place a comment after the #endif on the same line, noting the conditional expression used.

## Centralized exiting of functions

Albeit deprecated by some people, the equivalent of the goto statement is used frequently by compilers in form of the unconditional jump instruction.

The goto statement comes in handy when a function exits from multiple locations and some common work such as cleanup has to be done. If there is no cleanup needed then just return directly.

Choose label names which say what the goto does or why the goto exists. An example of a good name could be `out_free_buffer:` if the goto frees `buffer`. Avoid using GW-BASIC names like `err1:` and `err2:`, as you would have to renumber them if you ever add or remove exit paths, and they make correctness difficult to verify anyway.

The rationale for using gotos is:

- unconditional statements are easier to understand and follow
- nesting is reduced
- errors by not updating individual exit points when making modifications are prevented
- saves the compiler work to optimize redundant code away ;)

```c
int fun(int a)
{
        int result = 0;
        char *buffer;

        buffer = kmalloc(SIZE, GFP_KERNEL);
        if (!buffer)
                return -ENOMEM;

        if (condition1) {
                while (loop1) {
                        ...
                }
                result = 1;
                goto out_free_buffer;
        }
        ...
out_free_buffer:
        kfree(buffer);
        return result;
}
```

A common type of bug to be aware of is `one err bugs` which look like this:

```c
err:
        kfree(foo->bar);
        kfree(foo);
        return ret;
```

The bug in this code is that on some exit paths `foo` is NULL. Normally the fix for this is to split it up into two error labels `err_free_bar:` and `err_free_foo:`:

```c
err_free_bar:
       kfree(foo->bar);
err_free_foo:
       kfree(foo);
       return ret;
```

Ideally you should simulate errors to test all exit paths.

## Headers

Header files should be self-contained (compile on their own) and end in `.h`. Non-header files that are meant for inclusion should end in `.inc` and be used sparingly.

All header files should be self-contained. Users and refactoring tools should not have to adhere to special conditions to include the header. Specifically, a header should have [header guards](https://google.github.io/styleguide/cppguide.html#The__define_Guard) and include all other headers it needs.

### The #define Guard

All header files should have `#define` guards to prevent multiple inclusion. The format of the symbol name should be `_<PROJECT>___<PATH>___<FILE>__H_`.

To guarantee uniqueness, they should be based on the full path in a project's source tree. For example, the file `foo/src/bar/baz.h` in project `foo` should have the following guard:

```C
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_

```

## Include What You Use

If a source or header file refers to a symbol defined elsewhere, the file should directly include a header file which properly intends to provide a declaration or definition of that symbol. It should not include header files for any other reason.

Do not rely on transitive inclusions. This allows people to remove no-longer-needed #include statements from their headers without breaking clients. This also applies to related headers - foo.cc should include bar.h if it uses a symbol from it even if foo.h includes bar.h.

## Inline Functions

Define functions inline only when they are small, say, 10 lines or fewer.

You can declare functions in a way that allows the compiler to expand them inline rather than calling them through the usual function call mechanism.

Inlining a function can generate more efficient object code, as long as the inlined function is small. Feel free to inline accessors and mutators, and other short, performance-critical functions.

Overuse of inlining can actually make programs slower. Depending on a function's size, inlining it can cause the code size to increase or decrease. Inlining a very small accessor function will usually decrease code size while inlining a very large function can dramatically increase code size. On modern processors smaller code usually runs faster due to better use of the instruction cache.

A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

It is important to know that functions are not always inlined even if they are declared as such; for example, virtual and recursive functions are not normally inlined. Usually recursive functions should not be inline. The main reason for making a virtual function inline is to place its definition in the class, either for convenience or to document its behavior, e.g., for accessors and mutators.

## Names and Order of Includes

Include headers in the following order: Related header, C system headers, C++ standard library headers, other libraries' headers, your project's headers.

All of a project's header files should be listed as descendants of the project's source directory without use of UNIX directory aliases . (the current directory) or .. (the parent directory). For example, awesome-project/src/base/logging.h should be included as:

```C
#include "base/logging.h"
```

Headers should only be included using an angle-bracketed path if the library requires you to do so. In particular, the following headers require angle brackets:

1. C and C++ standard library headers (e.g. <stdlib.h> and <string>).
2. POSIX, Linux, and Windows system headers (e.g. <unistd.h> and <windows.h>).
3. In rare cases, third_party libraries (e.g. <Python.h>).

In dir/foo.cc or dir/foo_test.cc, whose main purpose is to implement or test the stuff in dir2/foo2.h, order your includes as follows:

1. dir2/foo2.h.
2. A blank line
3. C system headers, and any other headers in angle brackets with the .h extension, e.g., <unistd.h>, <stdlib.h>, <Python.h>.
4. A blank line
5. C++ standard library headers (without file extension), e.g., <algorithm>, <cstddef>.
6. A blank line
7. Other libraries' .h files.
8. A blank line
9. Your project's .h files.

Separate each non-empty group with one blank line.

With the preferred ordering, if the related header dir2/foo2.h omits any necessary includes, the build of dir/foo.cc or dir/foo_test.cc will break. Thus, this rule ensures that build breaks show up first for the people working on these files, not for innocent people in other packages.

dir/foo.cc and dir2/foo2.h are usually in the same directory (e.g., base/basictypes_test.cc and base/basictypes.h), but may sometimes be in different directories too.

Note that the C headers such as stddef.h are essentially interchangeable with their C++ counterparts (cstddef). Either style is acceptable, but prefer consistency with existing code.

Within each section the includes should be ordered alphabetically. Note that older code might not conform to this rule and should be fixed when convenient.

For example, the includes in awesome-project/src/foo/internal/fooserver.cc might look like this:

```C
#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "base/basictypes.h"
#include "foo/server/bar.h"
#include "third_party/absl/flags/flag.h"
```

Exception:

Sometimes, system-specific code needs conditional includes. Such code can put conditional includes after other includes. Of course, keep your system-specific code small and localized. Example:

```C
#include "foo/public/fooserver.h"

#include "base/port.h"  // For LANG_CXX11.

#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
```

## Local Variables

Place a function's variables in the narrowest scope possible, and initialize variables in the declaration.

C++ allows you to declare variables anywhere in a function. We encourage you to declare them in a scope as local as possible, and as close to the first use as possible. This makes it easier for the reader to find the declaration and see what type the variable is and what it was initialized to. In particular, initialization should be used instead of declaration and assignment, e.g.,:

```C
int i;
i = f();      // Bad -- initialization separate from declaration.

int i = f();  // Good -- declaration has initialization.

int jobs = NumJobs();
// More code...
f(jobs);      // Bad -- declaration separate from use.

int jobs = NumJobs();
f(jobs);      // Good -- declaration immediately (or closely) followed by use.

std::vector<int> v;
v.push_back(1);  // Prefer initializing using brace initialization.
v.push_back(2);

std::vector<int> v = {1, 2};  // Good -- v starts initialized.

```

Variables needed for if, while and for statements should normally be declared within those statements, so that such variables are confined to those scopes. E.g.:

```C
while (const char* p = strchr(str, '/')) str = p + 1;
```

There is one caveat: if the variable is an object, its constructor is invoked every time it enters scope and is created, and its destructor is invoked every time it goes out of scope.
```C
// Inefficient implementation:
for (int i = 0; i < 1000000; ++i) {
  Foo f;  // My ctor and dtor get called 1000000 times each.
  f.DoSomething(i);
}
```

It may be more efficient to declare such a variable used in a loop outside that loop:
```C
Foo f;  // My ctor and dtor get called once each.
for (int i = 0; i < 1000000; ++i) {
  f.DoSomething(i);
}
```

## Commenting

Comments are good, but there is also a danger of over-commenting. NEVER try to explain HOW your code works in a comment: it’s much better to write the code so that the **working** is obvious, and it’s a waste of time to explain badly written code.

Generally, you want your comments to tell WHAT your code does, not HOW. Also, try to avoid putting comments inside a function body: if the function is so complex that you need to separately comment parts of it, you should probably go back to chapter 6 for a while. You can make small comments to note or warn about something particularly clever (or ugly), but try to avoid excess. Instead, put the comments at the head of the function, telling people what it does, and possibly WHY it does it.

```C
/*
 * This is the preferred style for multi-line
 * comments in the Linux kernel source code.
 * Please use it consistently.
 *
 * Description:  A column of asterisks on the left side,
 * with beginning and ending almost-blank lines.
 */
 ```

Here is a sample Header file with comments

```C
/*****************************************************************************
* File Name: mbr3_api.h
*
* Version 1.00
*
* Author: ip_v1
* 
* Description:
*   This file contains the code to access CY8CMBR3xxx Touch chips
*   
*****************************************************************************/
#ifndef MBR3_API_
#define MBR3_API_

/* Provide C++ Compatibility */
#ifdef __cplusplus
extern "C" {
#endif

/******************************************************************************
* Included Headers
******************************************************************************/
#include <stdint.h>

/******************************************************************************
* Defines : Key Assignment Constants
*****************************************************************************/    
#define K_UP            (1)

    
/*****************************************************************************
*   Function Prototypes
*****************************************************************************/
     
uint8_t  mbr3_init(uint8_t);
uint8_t mbr3_check_for_device(uint8_t slave_address);
void dev_mbr3_test1(void);
uint8_t mbr3_get_button_stat(uint8_t slave_address, uint8_t *data_ptr);
    
#ifdef __cplusplus
}
#endif

#endif

/****************************End of File*************************************/
/
 
```

The corresponding Source File would be:

```C
/****************************************************************************
 * File Name: mbr3_api.c
 *
 * Version 1.00
 *
 * Author: ip_v1
 * 
 * Description:
 *   This file contains the code to access CY8CMBR3xxx Touch chips
 *   
 ****************************************************************************/


/******************************************************************************
 * Included headers
 ******************************************************************************/
#include "/src/touch_chip/mbr3_api.h"

#include "src/inlude/cy8cmbr3xxx_registers.h"

#include "src/system/i2c.h"
#include "src/system/delay.h"
#include "src/system/appconfig.h"


/******************************************************************************
 * Defines : BSP MACROS
 ******************************************************************************/
#define I2C1_SDA_IS_INPUT		IO_SDA1_SetDigitalInput()	//0=input
#define I2C1_SDA_IS_OUTPUT		IO_SDA1_SetDigitalOutput()	//1=output
#define I2C1_SDA_IN      		IO_SDA1_GetValue()
#define I2C1_SDA_HIGH	  		IO_SDA1_SetHigh()
#define I2C1_SDA_LOW	  		IO_SDA1_SetLow()
#define I2C1_SCL_HIGH	  		IO_SCL1_SetHigh()
#define I2C1_SCL_LOW	  		IO_SCL1_SetLow()

#define I2C_CLK_DELAY 			DelayUs(2)
#define I2C_HALFCLK_DELAY		DelayUs(1)


/******************************************************************************
 *   Data Constants
 ******************************************************************************/
const unsigned char CY8CMBR3116_config[128] = {
    0x03u, 0xFFu, 0x03u, 0xFFu, 0x00u, 0x00u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x80u, 0x80u, 0x7Fu, 0x7Fu,
    0x7Fu, 0x7Fu, 0x7Fu, 0x7Fu, 0x80u, 0x80u, 0x80u, 0x80u,
    0x96u, 0x96u, 0x80u, 0x80u, 0x03u, 0x00u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x80u,
    0x05u, 0x00u, 0x00u, 0x02u, 0x00u, 0x02u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x1Eu, 0x1Eu, 0x00u,
    0x00u, 0x1Eu, 0x1Eu, 0x00u, 0x00u, 0x00u, 0x01u, 0x01u,
    0x00u, 0xFFu, 0xFFu, 0xFFu, 0xFFu, 0xFFu, 0xFFu, 0xFFu,
    0xFFu, 0x00u, 0x00u, 0x00u, 0x10u, 0x03u, 0x01u, 0x40u,
    0x00u, 0x37u, 0x06u, 0x00u, 0x00u, 0x1Eu, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u,
    0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x00u, 0x3Fu, 0xEAu
};

/*****************************************************************************
 *   Extern Global Variables
 ******************************************************************************/
extern unsigned char guc_I2C_Port;

/*****************************************************************************
 *   Global Variables
 ******************************************************************************/
volatile uint8_t button_data[2];

/******************************************************************************
 *   Function Code
 ******************************************************************************/

/******************************************************************************
 * Function Name: mbr3_init
 ******************************************************************************
 *
 * Summary:
 *  This function is used to initialize the MBR3 Chip
 *  
 *
 * Parameters:
 *  uint8_t slave_address:
 *   This is the slave address of the chip- default is 0x37
 *   The function will check for register FAMILY_ID at 0x8f
 *     and the expected value will be 154 or 0x9a
 *
 * Return:
 *  0 if successful and
 *  1 if unsuccessful
 *
 *******************************************************************************/
uint8_t mbr3_init(uint8_t slave_address) 
{
    uint8_t i, j, _err = 0, _retries = 255;

    I2C_SetSlave(slave_address, 1);

    /*
     * Send address and get confirmation
     */
    do {
        I2C_Start();
        I2C_Write(slave_address);
        if (I2C_Ack() == 1) {
            _err = 1;
            _retries--;
        } else {
            _err = 0;
            _retries = 0;
        }
    } while (_retries > 0);

    /*
     *  Write register address
     */
    if (_err == 0) {
        I2C_Write(0x00);
        if (I2C_Ack() == 1) {
            _err = 2;
        }
    }

    /*
     * Write 128 bytes of config data
     */
    if (_err == 0) {
        i = CY8CMBR3xxx_CONFIG_DATA_LENGTH_WITH_CRC;
        j = 0;
        do {
            I2C_Write(CY8CMBR3116_config[j]);
            if (I2C_Ack() == 1) {
                _err = 3;
                break;
            }
            i--;
            j++;
        } while (i > 0); // while length
    }

    /*
     * Send Stop
     */
    I2C_Stop();

    return _err;
}

/*******************************************************************************
 * Function Name: mbr3_check_for_device
 ********************************************************************************
 *
 * Summary:
 *  This function is used to verify if the MBR3 chip is working on the I2C Bus
 *  
 *
 * Parameters:
 *  uint8_t slave_address:
 *   This is the slave address of the chip- default is 0x37
 *   The function will check for register FAMILY_ID at 0x8f
 *     and the expected value will be 154 or 0x9a
 *
 * Return:
 *  0 if successful and
 *  1 if unsuccessful
 *
 *******************************************************************************/
uint8_t mbr3_check_for_device(uint8_t slave_address) 
{
    uint8_t ret, data;
    I2C_SetSlave(slave_address, 1);
    ret = I2C_GetReg(0x8f, &data);
    if (data == CY8CMBR3xxx_DEFAULT_FAMILY_ID)
        return 0;
    else
        return 1;
}

/*******************************************************************************
 * Function Name: mbr3_get_button_stat
 ********************************************************************************
 *
 * Summary:
 *  This function is used to get button status
 *  
 *
 * Parameters:
 *  uint8_t slave_address:
 *   This is the slave address of the chip- default is 0x37
 *   The function will check for register FAMILY_ID at 0x8f
 *     and the expected value will be 154 or 0x9a
 * 
 *  uint8_t &data:
 *      Address of variable to get button status register
 *
 * Return:
 *  0 if successful and
 *  1 if unsuccessful
 *
 *******************************************************************************/
uint8_t mbr3_get_button_stat(uint8_t slave_address, uint8_t *data_ptr) 
{
    uint8_t _err;
    
    I2C_SetSlave(slave_address, 1);
    _err = I2C_GetDualReg(CY8CMBR3xxx_BUTTON_STAT, data_ptr);
//    _err = I2C_GetReg(0xaa, &data[0]);
    return _err;
}

/*******************************************************************************
 * Function Name: dev_mbr3_test1
 ********************************************************************************
 *
 * Summary:
 *  This function is used to test mbr3 in development
 *  
 *******************************************************************************/
void dev_mbr3_test1(void) 
{
    uint8_t temp=0;
    mbr3_init(CY8CMBR3xxx_DEFAULT_ADDRESS);
    while (1) {
        temp = mbr3_check_for_device(CY8CMBR3xxx_DEFAULT_ADDRESS);
        
        if (temp == 0) {
            mbr3_get_button_stat(CY8CMBR3xxx_DEFAULT_ADDRESS, &button_data[0]);
        }
        DelayS(1);
        I2C1_SCL_HIGH;
        I2C1_SDA_HIGH;
    }
}

/****************************End of File***************************************/
```


# Reference
Linux Coding Style Guide
- https://www.kernel.org/doc/html/v4.10/process/coding-style.html
Google Style Guide for C++
- https://google.github.io/styleguide/cppguide.html
The C Programming Language, Second Edition by Brian W. Kernighan and Dennis M. Ritchie. Prentice Hall, Inc., 1988. ISBN 0-13-110362-8 (paperback), 0-13-110370-9 (hardback).

The Practice of Programming by Brian W. Kernighan and Rob Pike. Addison-Wesley, Inc., 1999. ISBN 0-201-61586-X.

GNU manuals - where in compliance with K&R and this text - for cpp, gcc, gcc internals and indent, all available from [http://www.gnu.org/manual/](http://www.gnu.org/manual/)

WG14 is the international standardization working group for the programming language C, URL: [http://www.open-std.org/JTC1/SC22/WG14/](http://www.open-std.org/JTC1/SC22/WG14/)

Kernel process/coding-style.rst, by [greg@kroah.com](mailto:greg%40kroah.com) at OLS 2002: [http://www.kroah.com/linux/talks/ols_2002_kernel_codingstyle_talk/html/](http://www.kroah.com/linux/talks/ols_2002_kernel_codingstyle_talk/html/)
