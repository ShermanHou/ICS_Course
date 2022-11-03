# A Tour of Computer Systems
## informations in computer systems
First of all, let us learn about how we express informations in computer systems:
- all stored as `bits`, and organized by 'chunks' called `bytes`(continuous 8 bits)
- and bits are distinguished also by their `context`

So how to 'translate' units of any expressions into 'bits'?
Here we have our common `ASCLL Standard` to do it, and the files consist of matched bits under ASCLL Standard are called `text files` and other files are called `binary files` 
## translation of the program files
We have finished typing in a C program file(called `sourse file`), we want to run it on our computer, how do our computer systems realize this?

Overall, the sourse file can't be understood by computer systems, because all the computers know are `0` and `1` :), but the text of sourse file are high-level C language which can and only can be easily understand by humans, so we need other prewriten programs,called `compiler driver`('gcc' as an example), to translate the 'human code language' of sourse file into `machine language`.After the translation, we store the final `executable object file`(of course, it's binary file) on the disk.

Let's go deeper and look at the physes of translation:
### Ⅰ preprocessing physe
As we all know, when we write C program, we always 'include' some `header files` which are contained in other C programs(normally they have `.h` suffix) to supply our program.In this physe, the `preprocessor(cpp)` will read the header files which we include and insert them directly into our text file(with `.i` suffix), as a longer C program.

### Ⅱ compilation physe
In the physe, our 'text file' gets into a low-level language form 'text file'(with `.s` suffix).This kind of language is a textual expression of the final machine language, which is called `assembly language`. The `compiler(cc1)` helps us to do this translation. What's important about 'assembly language' is providing a common output for all kind of high-level program language(such as C,Python) and for all kind of compiler.So it's really important to learn it and the compilation process to optimize our program by obtaining a more efficient binary file(machine language) to execute by our computer system nicely and quickly.

### Ⅲ assembly physe
Assembly physe carries out the key translation which turn text file into binary object file(with `.o` suffix), in other words, translating 'assembly language' into real 'machine language' which can be executed directly :).The translation is done by `assembler(as)`. Then, assembler package it in a form known as  `relocatable object program`. 

### Ⅳ linking physe
The 'relocatable object program' contains all the translated instructions that we write in the sourse file and in head files which we include, but actually, the head files which come along with the compiler normally just contain the declarations of functions, instead of the contents of them(some functions used nearly for certain are even not in head files, such as 'printf').Hense, the head file' real effect is to tell us what functions it has declared and the concrete contents will be linked in the future linking physe as separate precompiled binary object files('.o'). These files are part of  `standard C library` which is provided by every C compiler.This kind of linking between binary files is called `merge`, and the `linker(ld)` handle it. What we get eventually is called `executable object file`, and it's stored on disk, and is ready to be loaded into memory and executed by the computer system. 
## hardware organization of systems for running programs
We have learned about the translation processes of a sourse file by compilation system, now, let us figure out how the system executes the executable object file. This process is based on the hardware organazation of systems.
Firstly, we should keep an important concept in mind: the smaller our storage is, the faster it will work. So we need a smaller place to store the data related to the program to be executed, this temporary storage device is called `main memory`. In fact, it holds both 'the program' and 'the data it manipulates'. Logically, it is organized as a linear array of bytes, each item with unique array index(address) starting at zero, and physically. We will explore further about the main memory which consists of a collection of `dynamic random access memory(DRAM)`.
Another extremely important part of hardware organization is `processor`. Also the main memory is smaller than disk, but the  