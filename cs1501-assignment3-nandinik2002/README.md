# CS 1501 – Algorithm Implementation – Assignment #3

Due: Friday, March 22nd @ 11:59pm on Gradescope

Late submission deadline: Sunday, March 24th @11:59pm with 10% penalty per late day

You should submit the Java files `DLBCodeBook.java`, `ArrayCodeBook`, and `LZW.java` to GradeScope (the link is on Canvas). You must also submit a writeup named `a3.md` and an Assignment Information Sheet `InfoSheet.md` as described below.


## OVERVIEW
 
**Purpose:** This assignment will help you understand the LZW compression algorithm. 

_Goal 1_: Use adaptive codeword width to allow the codebook size to increase beyond the `4096` entries in the textbook's implementation.

_Goal 2_: Allow the user to reset the codebook after its maximum size is reached to allow LZW to learn new patterns. 

## PROCEDURE

1.	Thoroughly review the description and explanation of the LZW compression algorithm as discussed in lectures and the Sedgewick textbook.

2.	Read over and make sure you understand the code provided in Lab 4 in the file `LZWmod.java`.

3.	There are two fundamental problems with Lab 4 code as given: 

    - The code uses a fixed length and a relatively small codeword size (12 bits).  With this limit, the program will run out of codewords relatively quickly and not handle large files (e.g., large archive files) well.

    - When all code words are used, the program continues to use the existing codebook for the remainder of the compression.  This may be okay if the rest of the file to be compressed is similar to what has already been compressed, but it may not be.

4. In this assignment, we will modify Lab 4's LZW code to correct these problems (somewhat).  

    - First, the code from Lab 4 has been restructured to separate the LZW algorithm from the codebook data structures. Two interfaces have been created, namely `CompressionCodeBookInterface` and `ExpansionCodeBookInterface`, to model the functions provided by the codebook for LZW compression and expansion, respectively. An implementation that uses a DLB Trie is provided for the `CompressionCodeBookInterface`, and one that uses an array for `ExpansionCodeBookInterface` has also been created. The DLB implementation is in `DLBCodeBook.java`, and the array implementation is in `ArrayCodeBook.java`. The main advantage of that separation is keeping the LZW algorithm unmodified (mostly) as we correct the problems mentioned above. Although the provided code has the same problems, **we can correct them with a minimal change to `LZW.java`**.

    Please carefully review the provided code, convincing yourself precisely what each function and statement within each function accomplishes. 
 
    -  In the first modification, the `DLBCodeBook` and `ArrayCodeBook` will be modified so that the LZW algorithm has a varying number of bits for the codewords, as discussed in the adaptive-codeword-size idea in class.  The codeword size should vary from 9 to 16 bits and increment when all codewords have been used.  This requires a slight modification to the classes. Still, you must understand precisely what the program is doing at each step to do this successfully (so you can keep the compress and decompress processes **in sync**).  Compression and expansion are in sync when expansion reads each codeword with the correct size written by Compression. You will need to make the following modifications:
      - Modify the public `add` methods to implement the logic for incrementing codeword size
      - Modify `getCodewordWidth()` in `ArrayCodeBook` to return the **effective** codeword size to sync the expansion with the compression.
      - Modify `LZW.java` so the codebooks are initialized with the correct minimum and maximum codeword sizes.
    
    Once you get the program to work, thoroughly test it to ensure it is correct. If the algorithm is correct, the expanded file should be identical to the original one. Later, this file gives hints about the variable-length code word implementation.

    As a partial solution to the codebook filling issue, you will allow the user to reset the codebook via a command-line argument. See more details in the command-line arguments examples below, but the argument `r` will cause the codebook to reset once all (16-bit) codewords have been used, and the argument `n` (for "do nothing") will cause the codebook to stay as is.
      As discussed in the lecture, the reset option erases and resets the entire codebook and starts rebuilding it from scratch.  **Keep compression and expansion in sync.**
      You must add code to reset the codebook to the public `add` methods of `DLBCodeBook` and `ArrayCodeBook`. Calling the private `initialize` method should help you do this.  
      - You will also need to modify `getCodewordWidth()` in `ArrayCodeBook` to return the effective codeword size to keep the expansion in sync with compression.
      Since a file may now be compressed with or without resetting the codebook, your program must be able to discern this fact to decompress correctly.  This is done by writing a 1-bit flag at the beginning of the compressed file. Then, before decompression, your program will read this flag and determine whether or not to reset the codebook when running out of codewords.
      - Note the `flushIfFull` boolean variable inside `LZW.java` and how it is used when calling `add` and `getCodewordWidth`. Consider setting this variable based on the command-line compression arguments and the bit flag read from the compressed file in expansion.

5. The provided `LZW.java` already has a command-line argument to choose compression or decompression. Modify the program so that a command-line argument allows the user to choose how to act when all codewords have been used for compression. This extra argument should be an `n` for "do nothing" or `r` for "reset."  Note that these arguments are only used during compression; for decompression, the algorithm should automatically detect which technique was used during compression.

6. File input and output is supplied using the standard redirect operators for standard I/O: Use "<" to redirect the input from a file and use ">" to redirect the output to a file. For example, if you wish to compress the file `code.txt` into the file `code.lzw`, resetting the codebook when you run out of codewords, you would enter at the prompt:

```shell
$ java LZW - r < code.txt > code.lzw
```
**Note that the input redirection operator (<) doesn't work with PowerShell under Windows.**

To prevent headaches (especially during debugging), you should not replace the original file with the new one (i.e., leave the original file unchanged). Thus, make sure you use a different name for the output file.  If you want to decompress the `code.lzw` file, you might enter the following command at the Windows command prompt or Unix shell.

```shell
$ java LZW + < code.lzw > code.rec
```

The file `code.rec` should now be identical to the file `code.txt` (You can confirm that using `diff` in Linux/MacOS and `fc /b` in Windows).  Note that in the decompression command, there is no flag for what to do when the codebook fills – this should be obtained from the front of the compressed file itself (which, again, requires only a single bit).

6.	Once your `LZW.java` program works, you should analyze its performance.  Several files to use for testing are provided in this repository. Specifically, you will compare the performance of 4 different implementations:

  - The provided implementation (same as Lab 4) that uses 12-bit codewords before any modifications. 
  - Your modified implementation with codeword size going from 9 bits to 16 bits as explained above without codebook reset.
  - Your modified implementation with codeword size going from 9 bits to 16 bits as explained above with codebook reset.
  - The predefined Unix `compress` program (which also uses the LZW algorithm).  If you have a Mac or Linux machine, you can run this version directly on your computer.   If you have a Windows machine, you can use the version of `compress.exe` in this repository (obtained originally from http://unxutils.sourceforge.net/ ).  To run `compress.exe`, you need to open a command-line prompt and run it from there. To decompress with this program, use the flag `-d`.

Run all programs on all of the files, and for each file, record the original size, compressed size, and compression ratio (original size / compressed size).

7.	Write a short paper, named `a3.md` using [Github Markdown syntax](https://guides.github.com/features/mastering-markdown/), that discusses each of the following:
  - How all four of the LZW variation programs compared to each other (via their compression ratios) for each file.  Where there was a difference between them, explain (or speculate) why.  To support your assertions, include a **table** showing all of the results of your tests (original sizes, compressed sizes, and compression ratios for each algorithm).
  - For all algorithms, indicate which of the test files gave the best and worst compression ratios and speculate as to why this was the case. If any files did not compress at all or compressed very poorly (or even expanded), speculate as to why.

## Debugging

Carefully trace what your code is doing as you modify it.  You only have to write a few lines of code for this program, but it could still require a substantial amount of time to get it to work correctly.  The trickiest parts occur when the codeword size increases and when the codebook is reset.  It is vital that these changes be made consistently during both compress and decompress.  One idea for debugging is to have an extra output file for each of the `compress()` and `expand()` methods to output any debug messages, as explained below.

Since we use `LZW.java` by redirecting its output, it won't be possible to use `System.out.println()` to print debug messages to the console. Instead, you should use `System.err.println()`, which uses the standard error stream, which is still connected to the console. If you want to save the debug messages to a file (for example, to have compression and expansion debug messages side by side), you can redirect the standard error stream as in the following commands:

`java LZW - < input.txt > input.lzw 2> debug-compress.txt`

`java LZW + < input.lzw > input.rec 2> debug-expand.txt`

Then, you can open the files `debug-compress.txt` and `debug-expand.txt` side by side for comparison. 

For debugging, I recommend that you print the values of the codeword size, the written/read codeword value, the corresponding string, and the (codeword, string) pair added to the codebook at each step. Printing these out the iterations just before and after a codeword width change or reset is done can help you a lot to debug your code efficiently.

  
## EXTRA CREDIT

If you want to try some extra credit on this assignment, you can seamlessly implement the reset so that the user does not have to specify whether or not to reset the codebook.  This would involve some type of monitoring of the compression ratio once the codewords are all used, and a reset would occur only when the compression ratio degrades to some level (you may have to do some trial and error to find a good value for the reset trigger).

## SUBMISSION REQUIREMENTS
You must submit to Gradescope the following files:

1. Your `LZW.java`, `DLBCodeBook.java`, and `ArrayCodeBook.java` 
2. Assignment Information Sheet.
3. The writeup, named `a3.md`.

The idea from your submission is that your TA (or the autograder or both) can compile and run your programs from the command line WITHOUT ANY additional files or changes, so be sure to test it thoroughly before submitting it. If the TA cannot compile or run your submitted code, it will be graded as if the program does not work.
If you cannot get the programs working as given, indicate any changes you made and why on your Assignment Information Sheet. You will lose some credit for not getting it to work properly, but getting the main programs to work with modifications is better than not getting them to work at all.  A template for the Assignment Information Sheet can be found in this repository. You do not have to use this template, but your sheet should contain the same information.  

**Note**: If you use an IDE, such as NetBeans, Eclipse, or IntelliJ, to develop your programs, make sure they will compile and run on the command line before submitting – this may require some modifications to your program (such as removing some package information). 

## RUBRICS
__*Please note that if an autograder is available, its score will be used as guidance for the TA, not as an official final score*__.

Please also note that the autograder rubrics are the definitive rubrics for the assignment. The TA will use the rubrics below to assign partial credit if your code scores < 60% of the autograder score. If your code is manually graded for partial credit, the maximum score you can get for the auto-graded part is 60% of the autograder score.

Item|Points
----|------
`DLBCodeBook` modified correctly for adaptive codeword size|	20 points
`ArrayCodeBook` modified correctly for adaptive codeword size|	20 points
`LZW.java` modified correctly for adaptive codeword size|	5 points
`LZW.java` modified correctly for reset|	10 points
`DLBCodeBook` modified correctly for reset|	10 points
`ArrayCodeBook` modified correctly for reset|	10 points
Write-up|	15 points
Comments and coding style|	5 points
Assignment Information Sheet|	5 points
Extra Credit|	10 points
