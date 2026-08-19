# Lab 3: Reverse Engineering

## 3.1. Reverse Engineering with GDB
GNU Debugger: a tool that allows you to see what is happening "inside" a program as it executes, or check why an error has occurred.  
Before using GDB it is useful to first run the program and check with:  
`strings` allows you to view embedded text/strings in binary files
`objdump` displays information about object files. 

Useful GDB commands:
info func : Prints out all the functions inside of the program.
disas <function name> : Print the assembly code and machine instruction number of a function.
b *<machine instruction address> : Pauses the program's execution at the machine instruction address and prints the program's state.
x/2x $rsp : Prints the first 2*4=8 bytes from the start of the stack ($rsp; on 32-bit binaries the stack pointer is called $esp instead)
r : Starts the program's execution from the very start.
c : Continue the program's execution to the next breakpoint or until completion.
si : Execute the next machine instruction and then print the state of the program.  

## 3.2. Ghidra
- Inspecting program strings can be a helpful starting point.
- Double clicking addresses takes you to where they are in the code and displays the "source" code in the Decompiler section
- Walkthrough a program by double-clicking on functions
- Rename variables or functions using 'L' or right clicking to make it more readable.
