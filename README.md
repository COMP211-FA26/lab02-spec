# Tar Heel Sort - `thsort`

## Abstract

In this lab you will implement `thsort`, a program that reads lines from standard input and prints them back in sorted order.

The `sort` program is a classic standard utility. Its purpose is to read lines from standard input (`stdin`), sort the lines, and prints them back out to standard output (`stdout`). It offers many different options for the *_type_* of sort it performs (lexicographic, numeric, etc). Your implementation will sort in a naive *_ascending_*, ASCII order, for now.

The purpose of this lab is to practice working with pointers, dynamic memory allocation, and applying knowledge acquired from library documentation.

## Setup

### Starter Repository

Accept the following classroom assignment: [https://classroom50.org/COMP211-FA26/comp211-fa26/assignments/lab-2/accept](https://classroom50.org/COMP211-FA26/comp211-fa26/assignments/lab-2/accept)

With your project repository open, click the green "Clone or Download" button. Copy the HTTPS URL. Back in a learncli container, clone this url with the `git clone <URL>` subcommand. Replace <URL> with your copied URL. (Right click to paste in Windows.) After cloning, change your working directory to the cloned repository.

## Demo

Before you begin, you should try the standard `sort` program:

```bash
learncli$ sort
the
quick
brown
fox
<CTRL-D>
brown
fox
quick
the
```

The primary use case for `sort` is *_not_* for interactive use (as demonstrated above), but rather to sort data piped in from another program or redirected from a file source. Let's create a file we can use as an example. Create a file called `lines-1.txt`.

Write the following to the file. 

```bash   
j
c
b
d
h
g
e
a
f
i
```

Next, run the following command. Remember in the shell, placing a `<` symbol before a filename, after a program, is a special syntax for *_redirecting_* standard input (`stdin`) to be read from a *_file_* rather than interactively from the keyboard. 

```plaintext
learncli$ sort < lines-1.txt
a
b
c
d
e
f
g
h
i
j
```

## Requirements

Now that we understand what `sort` does, let's get into what you'll be doing in the lab. By the end of the lab, you will write a program that mirrors `sort`. 

Your program must able to read arbitrarily long lines of input, as well as an arbitrary number of lines of input, and sort those lines according to `strcmp`'s lexocographic ordering of string data. 

Your program must store its data *_on the heap_* and specifically *_not_* in static memory. Your program must not leak any memory and must not perform any invalid heap accesses of memory outside its lifetime. Your program must print "out of memory" to `stderr` and exit immediately if *_any_* memory allocation call fails.

## Permitted Library Functions

You are encouraged to read the documentation of each of the library functions below before beginning. You may not need to use all of these functions, but it's likely you would find it beneficial to use most of them. For this lab you are only allowed to use functions from the following list of libraries. Any use of a library function outside of these will result in a 20% penalty subtracted from your autograding score.

The names of the functions below are links to their official documentation.

 - stdio.h

    - [printf](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/printf.html) - print to standard output

    - [fprintf](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/fprintf.html) - print to a specific file descriptor (`stderr` for error messages)

    - [getchar](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/getchar.html) - for reading input one byte at a time

    - [fgets](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/fgets.html) - for reading input blocks of bytes at a time

 - stdlib.h

    - [malloc](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/malloc.html) - memory allocation

    - [calloc](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/calloc.html) - contiguous allocation with 0-initialization

    - [realloc](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/realloc.html) - reallocate and resize a dynamic memory block

    - [free](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/free.html) - freeing dynamically allocated memory

    - [qsort](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/qsort.html) - a standard-library sorting function

    - [exit](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/exit.html) - for exiting the program prematurely

 - string.h

    - [strchr](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/strchr.html) - find a character in a string

    - [strcmp](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/strcmp.html) - compare two strings lexicographically

    - [strcpy](https\://pubs.opengroup.org/onlinepubs/9699919799/functions/strcpy.html) - copy a string and return a pointer to the beginning of the destination string.

## Part 0. Understanding `fgets` and Buffer Sizes.

Let's start by getting familiar with the `fgets` function and how buffer sizes affect input handling in C.

Open `part0/thsort-p0.c`, compile it, and run it to observe how it behaves. 

The program prompts the user to enter a string. It then stores the input in a fixed-size array on the stack and prints it back. The array where you store the input is called a buffer. Notice that the input string must not exceed 127 characters (plus one for the null terminator), as the buffer size is limited to 128 bytes.


- `fgets` reads a line of input from a given stream (`stdin` in this case) and stores it in the buffer you provide.

- The second argument (`sizeof(buffer)`) specifies the size of the buffer. `fgets` reads at most `size - 1` characters, leaving room to append the null terminator (`\0`). 

- If the input fits in the buffer, it’s stored as a null-terminated string.

- If the input is longer than the buffer, only part of it is read, and the rest remains in the input stream to be consumed later.

- `fgets` returns the same pointer you passed in (`buffer`) on success, and `NULL` on failure (e.g., if end-of-file is reached or an error occurs).

### Try This

Reduce the buffer size by changing `LINE_BUFFER_SIZE` to something small, like 10. Recompile and run the program again. Enter a long string — what do you notice?

****Before continuing, reset the buffer size to 128.****

### Note on Arrays and Pointers

When you pass an array (like `buffer`) to a function, it decays into a pointer to its first element. So in this case:

```
fgets(buffer, sizeof(buffer), stdin);
```

`buffer` is treated as a `char *`, pointing to the start of the array. This is why `fgets` can write into the memory you allocated with `char buffer[LINE_BUFFER_SIZE];`.

### Compiling and Running Part 0

Make sure you are in the root directory of your lab repository (the directory containing your Makefile).

1. Compile Part 0

```
make part0/thsort-p0
```

This tells `make` to compile `part0/thsort-p0.c` and create an executable named `thsort-p0` inside the part0 directory.

2. Run Part 0

Once compilation succeeds, run:

```
./part0/thsort-p0
```

You should see a prompt asking you to enter a string. For example:

```
learncli$ ./part0/thsort-p0
Enter a string: hello
You entered: hello
```


3. Make changes and recompile

If you modify `part0/thsort-p0.c`, run the same make command again:
`make part0/thsort-p0`


`make` will automatically recompile your program if the source file has changed.
Remember: `make` compiles your program, but it does not run it. You must execute the compiled program separately using `./part0/thsort-p0`.


## Part 1. Storing the buffer on the heap.

Open `part1/thsort-p1.c`. This is the same program as `part0`.

Now, modify the program such that it stores the buffer on the heap instead of the stack. Remember to check that `malloc` was successful. Your program must print "out of memory" to `stderr` and exit immediately if *_any_* memory allocation call fails.

Once you've made the change, test your modified program using various inputs. Make sure it still reads and prints the string correctly.

****Common issue:**** You may be encountering an error like this:

```bash
learncli$ make part1/thsort-p1
learncli$ ./part1/thsort-p1 
Enter a string: hello there
You entered: hello tlearncli$
```

If so, double-check that the size you're passing to `fgets` matches the size of the allocated buffer. 


### Valgrind

Before moving on, make sure your program doesn’t have any memory leaks by running it through Valgrind:

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./part1/thsort-p1
```

Look at the heap summary at the end of the output. It should include the line:

```bash
All heap blocks were freed -- no leaks are possible
```

If you see any memory still in use or not freed, double-check that you're freeing everything you've allocated with `malloc`.

### Running leak-check using Make

Your Makefile already includes a `leak-check` target that allows you to run Valgrind on any part of the lab.

Open your `Makefile` to view it:

```
leak-check:
ifeq ($(FILE),)
	@echo "Usage: make leak-check FILE=partN/thsort-pN"
else
	$(MAKE) $(FILE)
	valgrind --leak-check=full --show-leak-kinds=all \
	    --errors-for-leak-kinds=all ./$(FILE)
endif
```

Notice that this rule uses a variable named `FILE` to specify which executable you want to check.

For example, to check Part 1, run:

`make leak-check FILE=part1/thsort-p1`

When you run this command:

`make` checks whether `part1/thsort-p1` needs to be compiled. If the source file has changed, it recompiles the program.

Valgrind runs `./part1/thsort-p1` and checks for memory errors and leaks.

After the program finishes, Valgrind displays a summary of its findings.

If your program frees all allocated memory, you should see a message similar to:

```
All heap blocks were freed -- no leaks are possible
```

You can use the same target to check other parts of the lab by changing the value of `FILE`.

Note: Unlike the compilation targets, `leak-check` actually runs your program. If your program expects input from the keyboard, you will need to provide that input and press `Ctrl-D` to indicate `EOF` when appropriate.

## Part 2. Reading Lines Until EOF

So far, your program reads a single line of input, prints it, and then exits. In this step, you'll modify the program so that it continues reading and printing lines until it reaches EOF (end-of-file).

Take a look at the updated `main` function. Notice that it now contains a loop that repeatedly calls `readline`. The loop continues as long as `readline` returns a valid pointer and stops when `readline` returns `NULL`, indicating that there is no more input to read.

To make this work, you'll implement the `readline` function. This function will do much of the same work that `main` did in Part 1: reading a line of input and storing it in memory. The difference is that we're now **encapsulating this functionality in a separate function so that we can call it repeatedly.**

Open `part2/thsort-p2.c` and complete the implementation of `readline` according to the following requirements (again, this is very similar to `main` in part 1):

- Allocate memory on the heap to store a single line of input.

    - If any memory allocation fails, print `"out of memory"` to `stderr` and exit immediately.

- Use `fgets` to read one line from `stdin`.

- Return a pointer to the line that was read, or `NULL` if there is no more input to read.

- Make sure any memory allocated by readline is properly freed when it is no longer needed.

Before moving on, compile and test your program. Verify that it correctly reads and prints multiple lines of input and stops when it reaches EOF. 

### Testing your Implementation

To test your program properly, you will create:

- Input files (`.in`) that simulate user input

- Expected output files (`.out`) that contain exactly what your program should print



1. Create the following directory structure :

```text
.
...
├── part2/
│   ├── tests/
│   │   ├── blank-line.in
│   │   ├── blank-line.out
│   │   ├── multiple-lines.in
│   │   ├── multiple-lines.out
│   │   ├── single-line-basic.in
│   │   ├── single-line-basic.out
│   │   ├── whitespace-preserved.in
│   │   └── whitespace-preserved.out
│   └── thsort-p2.c
└── run_tests.sh
```

You will create the contents of each `.in` and `.out` file.

2. `single-line-basic` test

`single-line-basic.in`

```bash
hello

```

Make sure that there are no extra spaces and that there is a newline after `hello`.

This simulates a user typing

```bash

hello<Enter>

```

`single-line-basic.out`

```bash
Enter a string: You entered: hello
Enter a string: 
```

Why Does the Output Look Like This?

- Your `main()` function prints:

    `printf("Enter a string: ");`

    before reading input.

- When the input comes from a file:

    - The program prints the prompt:

        `Enter a string:`

- It reads `hello\n` from the file.

- It prints:

    `You entered: hello`

- The loop runs again and prints another prompt:

    `Enter a string:`

- This time `readline()` returns NULL because it reaches EOF.

- That final prompt still appears because it is printed before the program checks for EOF.

Now, run your test:

```bash
learncli$ ./part2/thsort-p2 < part2/tests/single-line-basic.in > part2/tests/single-line-basic.actual
learncli$ diff part2/tests/single-line-basic.out part2/tests/single-line-basic.actual 
```

2. `multiple-lines` test

`multiple-lines.in`

```bash
apple
banana
carrot

```

`multiple-lines.out`

```bash
Enter a string: You entered: apple
Enter a string: You entered: banana
Enter a string: You entered: carrot
Enter a string: 
```

Now, run the test! 

3. `blank-line` test

This test should verify

- `readline` still returns a valid string when the line is just `\n`.

- Your program does not skip empty lines.

`blank-line.in`

```bash
hello

there

```

`blank-line.out`

```bash
Enter a string: You entered: hello
Enter a string: You entered: 
Enter a string: You entered: there
Enter a string: 
```

Make sure that there is a space after the last "Enter a string".

Run your test! 

4. `whitespace-preserved` test

This test should verify

- Leading spaces in the input remain in the output.

- Trailing spaces in the input remain in the output.

`whitespace-preserved.in`

```bash
  leading space
trailing space  
  both  

```

`whitespace-preserved.out`

```bash
Enter a string: You entered:   leading space
Enter a string: You entered: trailing space  
Enter a string: You entered:   both  
Enter a string: 
```

Make sure that there is a space after the last "Enter a string".

Run your test!

### Automating your Tests

Open `run_tests.sh` in the repository root. Make it executable:

```bash
chmod +x run_tests.sh
```

To run the Part 2 tests from the repository root:

```bash
make part2/thsort-p2
./run_tests.sh part2
```

### Valgrind

Once your program works as expected, check for memory leaks:

```bash
make leak-check FILE=part2/thsort-p2
```

Check the heap summary for `All heap blocks were freed -- no leaks are possible`.

## Part 3: Storing Multiple Lines

In Part 2, your program repeatedly read a line of input, printed it, and then moved on to the next line. In other words, **each line was printed immediately after it was read**.

In this part, you'll change that behavior. Instead of printing each line as soon as it is read, your program will **read and store all the lines first, then print them after reaching `EOF`** (end-of-file).

### Storing Multiple Lines

We need a way to keep track of multiple lines at the same time.

To do this, we'll use an **array of pointers**, where each element points to a different line stored on the heap.

For now, you can assume that the input contains at most 5 lines. In a later part, we'll remove this restriction so the program can handle any number of lines.


### Implementing the Functions

Open `part3/thsort-p3.c` and take a look at `main`. Notice that we've separated the program into three main steps:

1. **`readlines`** reads all the lines from `stdin` and stores pointers to them in an array.
2. **`printlines`** prints the stored lines after all the input has been read.
3. **`freelines`** frees the dynamically allocated memory before the program exits.

You'll implement each of these functions.

**Step 1: `readline`**

Copy your implementation of `readline` from Part 2. You won't need to change this function because it still has the same job: read a single line and return a pointer to it.

**Step 2: `readlines`**

Implement `readlines` so that it:

- Allocates space on the heap for an array of up to `MAX_NUM_LINES` pointers.
- Repeatedly calls `readline` to read one line at a time until reaching `EOF`.
- Stores each pointer returned by `readline` in the array.
- Updates `num_lines` to reflect how many lines were actually read.
- Returns a pointer to the array of pointers.

Notice that `readlines` takes an `int* num_lines` parameter. This allows the function to update the `num_lines` variable in `main`, so the other functions know how many lines were stored.

**Relationship between readline and readlines**

Each call to `readline` allocates memory for one string and returns a pointer to it. The `readlines` function collects these pointers in a separate dynamically allocated array.

**Step 3: `printlines`**

Implement `printlines` so that it:

- Iterates through the array of pointers.
- Prints each of the stored lines in the order they were read.

Remember that `printlines` should not read any new input. All the lines have already been stored by `readlines`.

**Step 4: `freelines`**

Implement `freelines` so that it frees all the memory allocated by the program.

Remember that each line was allocated separately on the heap, and the array of pointers was also allocated on the heap. **You'll need to free both the individual lines and the array itself.**

As in the previous part, if any memory allocation fails, print `"out of memory"` to `stderr` and exit immediately.

### Testing your implementation

1. Add the following files under `part3/`:

```text
part3/
├── tests/
│   ├── three-lines.in
│   ├── three-lines.out
│   ├── includes-empty-line.in
│   ├── includes-empty-line.out
│   ├── five-lines.in
│   └── five-lines.out
└── thsort-p3.c
```

2. `three-lines`

`three-lines.in`

```bash
red
green
blue

```

`three-lines.out`

```bash
red
green
blue

```

3. `includes-empty-line`.

Implement this test on your own. This test checks that an empty line is preserved and printed.

4. `five-lines`

Inplement this test on your own. This test confirms handling up to 5 lines.

**The TAs will be checking that you have implemented these tests!**

### Running your tests

To run the Part 3 tests from the repository root:

```bash
make part3/thsort-p3
./run_tests.sh part3
```


### Valgrind

Once your program works as expected, check for memory leaks:

```bash
make leak-check FILE=part3/thsort-p3
```

## Part 4. Sorting the Strings

In Part 3, your program read and stored multiple lines of input, then printed them in the order they were entered. In this part, you'll modify your program to sort the lines alphabetically before printing them.

Open `part4/thsort-p4.c` and copy your implementation from Part 3.

To sort the lines, you'll use the C standard library function `qsort`. You'll need to:

1. Write a comparison function called cmp that determines the order of two strings.

2. Call `qsort` to sort the array of pointers before printing the lines.

### Sorting with qsort

`qsort` is a general-purpose sorting function provided by C's standard library (`stdlib.h`). It can sort arrays containing any type of data, including integers, strings, and structs.

Here's its function signature:

```
void qsort(void *base, size_t nitems, size_t size,
           int (*compar)(const void *, const void *));
```

`qsort` takes four arguments:

- `base`: A pointer to the beginning of the array to sort.

- `nitems`: The number of elements in the array.

- `size`: The size of each element in bytes.

- `compar`: A pointer to a function that compares two elements and determines their order.

The first three arguments describe the array. The fourth tells `qsort` how to compare its elements.

### Function Pointers

You've already worked with pointers to variables, which store memory addresses. In C, you can also have pointers to functions, which store the addresses of functions.

A function pointer allows you to pass a function as an argument to another function. The receiving function can then call it.

This is exactly what `qsort` does! Since `qsort` can sort arrays of any type, it needs you to provide a function that knows how to compare the elements in your particular array.

Notice the fourth parameter in the `qsort` signature:

`int (*compar)(const void *, const void *)`

This declares compar as a pointer to a function that:

Takes two `const void *` arguments.

Returns an `int`.

When you call `qsort`, you'll pass it the name of your comparison function, `cmp`. In this context, the function name provides a pointer to that function.

### Writing a Comparison Function
Since we're sorting strings alphabetically, we'll use `strcmp` from `string.h` to compare them.

`strcmp` returns:

- A negative value if the first string is "less than" the second
- Zero if they’re equal
- A positive value if the first is "greater than" the second

Add the following comparison function to your program:

```C
int cmp(const void *a, const void *b) {
    const char *str1 = *(const char **)a;
    const char *str2 = *(const char **)b;
    return strcmp(str1, str2);
}
```

**Why do we need `char **`?**

Remember that your array contains pointers to strings (`char *`), rather than the strings themselves.

When `qsort` calls `cmp`, it passes pointers to two elements of the array. Since each element is already a `char *`, the arguments point to `char *` values, giving us an additional level of indirection.

We cast each argument to `const char **` and dereference it to obtain the string pointer that we can pass to `strcmp`.

The `const` qualifiers indicate that the comparison function should not modify the strings or the array elements being compared.

### Calling qsort

Now that you have a comparison function, you're ready to sort your array.

Add a call to `qsort` in main, after `readlines` and before `printlines`.

For your call, think about what each argument should be:

- What variable points to the beginning of your array?

- How many lines did you read?

- What is the size of each element in your array? 

- What is the name of your comparison function?

Once you've added the call, your program should read all the lines, sort them alphabetically, and then print them in sorted order.

### Testing Your Program

Before moving on, compile and test your program. Try entering lines in reverse alphabetical order and verify that they are printed in alphabetical order.

Once your program works as expected, run it through Valgrind to check for memory leaks.

## Part 5. Reading an Arbitrary Number of Lines

So far, your program can only store a fixed number of lines. In this part, you'll modify it to handle any number of input lines by dynamically resizing the array of pointers as needed.

Open `part5/thsort-p5.c` and copy your implementation from Part 4.

Your current implementation allocates enough space for `MAX_NUM_LINES` pointers. Instead of limiting the number of lines your program can read, you'll now use `realloc` to grow the array whenever it runs out of space.

How you grow the array is up to you. For example, you could double its capacity each time it fills up, increase its capacity by 50%, or use another strategy.

Keep in mind that you're resizing the array of pointers, not the individual lines. Each line should still be allocated separately by readline.

As before, if any memory allocation fails, print `"out of memory"` to `stderr` and exit immediately.

### Testing
Before moving on, test your program with inputs that exceed `MAX_NUM_LINES` to verify that the array grows correctly and all lines are preserved. Here is an example test to get you started:

Input: `tests/part5/tests/simple.in`

~~~bash
pear
apple
banana
kiwi
grape
orange
fig
date
plum
apricot
blueberry
cherry

~~~

Output: `tests/part5/tests/simple.out`

~~~bash
apple
apricot
banana
blueberry
cherry
date
fig
grape
kiwi
orange
pear
plum

~~~

Next, you’ll write your own tests. This is a critical skill to develop as a computer scientist. Writing code is only half the job; the other half is verifying that it works correctly, reliably, and under edge cases you may not have initially considered.

**The TAs will check that you have implemented tests covering the following cases:**

- duplicates_present
    - Include repeated identical lines; output should contain all duplicates.
- empty_input
    - No lines at all (immediate EOF). Output should be empty; program must not crash.
- many_lines_stress
    - Like 1,000+ lines; checks repeated realloc correctness.


### Check for Memory Leaks

Once your program works as expected, ensure that it doesn’t have any memory leaks by running your test cases through Valgrind. 

## Part 6. Reading Lines of Arbitrary Length

So far, your program can handle any number of input lines, but each line is still limited to a fixed length (`LINE_BUFFER_SIZE`). In this final part, you'll modify your program to handle lines of any length by dynamically resizing the memory allocated for each line.

Open `part6/thsort-p6.c` and copy your implementation from Part 5.

Just as you did with the array of pointers in Part 5, you'll now use `realloc` to grow the buffer for an individual line whenever it runs out of space.

Start by allocating enough memory for a line of `LINE_BUFFER_SIZE` bytes. As you read input, expand the buffer as needed until you've read the entire line (a line ends with `\n` or `EOF`).

Remember that C strings must end with a null terminator (`\0`), so make sure you always allocate enough space for it.

**You should only need to modify `readline` for this part.** Because you've separated your program into functions, you can change how individual lines are read without changing how the program stores, prints, or frees them.

As before, if any memory allocation fails, print `"out of memory"` to `stderr` and exit immediately.


### Testing

A good way to test your implementation is to temporarily reduce `LINE_BUFFER_SIZE` to a small value, such as 8 bytes. This makes it easier to test whether your program correctly grows its buffer using `realloc`, without having to type very long lines of input.

Remember that a buffer of 8 bytes can store at most 7 characters plus the null terminator (`\0`). If the input includes a newline character (`\n`), that newline must also fit in the buffer. For example, a line containing 6 characters followed by `\n` fits exactly in 8 bytes, but a line containing 7 characters followed by `\n` requires 9 bytes.

Try the following test cases with `LINE_BUFFER_SIZE` set to 8.

1. Short line: A line that fits in the initial buffer without resizing

`dog`

2. Exact fit: Six characters, a newline, and `\0` use exactly 8 bytes

`abcdef`

3. Just over capacity: Seven characters, a newline, and `\0` require 9 bytes, forcing a resize

`abcdefg`

4. Multiple resizes: A long line that requires the buffer to grow multiple times

`abcdefghijklmnopqrstuvwxyz`

5. Multiple lines: A mixture of short and long lines, ensuring each line is stored correctly

```
cat
abcdefghijklmnop
dog
```

Once this is working, change `LINE_BUFFER_SIZE back` to 128 and try the test cases provided for you in `part6/tests`.

### Common Error

**Missing trailing newline:** Not all input files end with `\n`. Make sure your program processes the final line when it encounters `EOF`, even if there is no trailing newline. However, don't create an extra empty line if the file already ends with `\n`.

### Check for memory leaks

Make sure your program doesn’t have any memory leaks by running your test cases through Valgrind.
