## Architecture Fundamentals
---

### 1.2.1 CPU, ISA, and Assembly

Overview, we'll discuss the:
	- CPU 
	- Instructions
	- Registers
	- Machine Code
	- Assembly Language
	- Memory
	- The Stack
	- and more


The ` CPU ` is the central processing unit of the device, it is in charge of executing machine code. 

The ` Machine Code ` is the set of instructions that a CPU processes. 

Each ` Instruction ` is a primitive command that executes a specific operation of data. Such as move data, changes the execution flow of the program, performs arithmetic or logical operations, and more.


CPU Instructions are represented in ` hexadecimal ` (HEX).

Therefore, the same instructions are translated into Assembly language (` ASM `).
	 - The two most popular are NASM (Netwide Assembler) and MASM (Microsoft Macro Assembler)


IMPORTANT:
	- Each CPU has its own instruction set architecture (` ISA `). The ISA is the set of instructions that a programmer must understand and use to write a program correctly for that specifc CPU and machine. 
	- ISA contains memory, registers, instructions, etc.


The most common ISA is the ` x86 ` instruction set, originating from the Intel 8086.

The x86 acronym identifies 32-bit processors, while x64 ( aka ` x86_64 ` or ` AMD64 ` ).


The number of bits, 32 or 64, refers to the width of the CPU registers.

Each CPU has a fixed set of registes that are accessed when required. Registers are temporary values used by the CPU to get and store data.


### 1.2.2 Registers

This course refers to a specific group of registers: The General Purpose Registers ( ` GPRs ` ).

The table below explains some of the naming coventions:

```
----------------------------------------------------------------------------------------
 x86 Naming Convention |      Name      |  Purpose 
----------------------------------------------------------------------------------------
         EAX           |   Accumulator  | Used in Arithmetic operation.
----------------------------------------------------------------------------------------
         ECX           |    Counter     | Used in shift/rotate instruction and loops
----------------------------------------------------------------------------------------
         EDX           |     Data       | Used in Arithmetic operation and I/O
----------------------------------------------------------------------------------------
         EBX           |     Base       | Used as a pointer in data
----------------------------------------------------------------------------------------
         ESP           |  Stack Pointer | Pointer to the top of the stack
----------------------------------------------------------------------------------------
         EBP           |  Base Pointer  | Pointer to the base of the stack
----------------------------------------------------------------------------------------
         ESI           |  Source Index  | Used as a pointer to a source in stream 
	    			   |		        | operation
----------------------------------------------------------------------------------------
         EDI           |  Destination   | Used as a pointer to a destination in stream 
                       |                | operation
----------------------------------------------------------------------------------------
```


The naming convention of the old 8-bit CPU had 16-bit registers divided into two parts:
	- a low byte, identified by an L at the end of the name
	- a high byte, identified by an H at the end of the name


The 16-bit naming convention combines the L and the H, and replaces it with an X. While the Stack Pointer, Base Pointer, Source, and Destination registers it simply remove the L.

In the 32-bit representation, the register acronym is prefixed with an E, meaning extended. Whereas, in the 64-bit representation, the E is replaced with an R. 

The below table summarizing the naming conventions 

```
----------------------------------------------------------------------------------------
Register    |    Accumulator    |     Counter     |      Data      |     Base        |
----------------------------------------------------------------------------------------
64-bit      |       RAX         |       RCX       |      RDX       |      RBX        |
----------------------------------------------------------------------------------------
32-bit      |    |     EAX      |    |     ECX    |    |    EDX    |    |    EBX     |  
----------------------------------------------------------------------------------------
16-bit      |        |    AX    |       |   CX    |      |   DX    |       |   BX    |
----------------------------------------------------------------------------------------
8-bit       |        | AH | AL  |       | CH | CL |      | DH | DL |       | BH | BL |
----------------------------------------------------------------------------------------
```

```
----------------------------------------------------------------------------------------
Register    |   Stack Pointer   |   Base Pointer   |    Source     |  Destination    |
----------------------------------------------------------------------------------------
64-bit      |       RSP         |       RBP       |      RSI       |      RDI        |
----------------------------------------------------------------------------------------
32-bit      |    |     ESP      |    |     EBP    |    |    ESI    |    |    EDI     |  
----------------------------------------------------------------------------------------
16-bit      |        |    SP    |       |    BP   |       |   SI   |       |   DI    |
----------------------------------------------------------------------------------------
8-bit       |        |    | SPL |       |   | BPL |       |  | SIL |       |   | DIL |
----------------------------------------------------------------------------------------
```

In addition to all these registers, the EIP (x86 naming convention) is important.

The Instruction Pointer ( ` EIP ` ) controls the program execution by storing a pointer to the address of the next instruction to be executed.


### 1.2.3 Process Memory

When a process runs, it is typically organized in the memory as shown below.

```
		                    Lower Memory Address
		                             0
		                  ______________________
		                  |                    |
	            Instructions  |       .text        |
	                          |____________________|
	                          |                    |
	    Initialized Variable  |       .data        |
	                          |____________________|
	                          |                    |
	  Uninitialized Variable  |        BSS         |    
	                          |____________________|
	                          |                    |
	                          |        Heap        |
	                          |____________________|
	                          |         |          |
	                          |         |          |
	                          |         V          |
	                          |                    |
	                          |         ∧          |
	                          |         |          |
	                          |_________|__________|
	                          |                    |
	                          |       Stack        |
	                          |____________________|
	
	                               0xFFFFFFFF
	                          Higher Memory Address
```

This process is divided up into four regions: Text, Data, the Heap, and the Stack.

The ` Text ` region, or instruction segment, is fixed by the program and contains the program code. This region is read-only since the program shouldn't change during execution. 

The ` Data ` region, is divided into initalized data and unitializied data. Initialized data includes items such as static and global declared variables that are pre-defined and can be modified. 

The uninitialized data, named Block Started by Symbol (` BSS `), also starts variables that are initialized by zero or do not have explicit initialization (` static int t` ).

Next is the ` Heap `, which starts after the ` BSS ` section. During the execution, the program can request more space in the memory via `brk` and `sbrk` system calls, used by `malloc`, `realoc`, and `free`. So the size of the data region can be extended. 

The last region is the `stack`. 


### 1.2.4 The Stack

The Stack is a Last-in-First-out ( `LIFO` ) block of memory. It is located in the higher part of the memory. It is kinda like an array used for saving a functon's return addresses, passing function arguments, and storing local arguments.

The purpose of the ESP register ( Stack Pointer ) is to identify the top of the stack, and it is modified each time a value is pushed in ( `PUSH` ) or popped out ( `POP` ).

The stack grows downwards, from the from the higher memory address to the lower memory address. 

When they were making it, they decided that the `Heap` start from the Lower Addresses and the `Stack` would start from the Higher Address.

```
	                   ________________________________
	                   |                              |
	               0   | Heap --->         <--- Stack | 0xFFFFFFFF
	 Lower Addresses   |                              | Higher Addresses
	                   |______________________________|
```

The most fundamental operations in the LIFO structure are `PUSH` and `POP`.


### 1.2.4.1 PUSH Instruction

#### `PUSH E` Example

PUSH Instruction:
	- `PUSH E`

PUSH Process:
	- A `PUSH` process is executed, and the `ESP` register is modified 

Starting Value:
	- The `ESP` points to the top of the stack.


PROCESS:
	- A `PUSH` instruction subtracts 4 (in 32-bit) or 8 (in 64-bit) from the `ESP` and writes the data to the memory address in the `ESP`, and then updates the `ESP` to the top of the stack. 
	- Remember that the stack grows backwards. Therefore the `PUSH` subtracts 4 or 8, in order to point to a lower memory location on the stack. 
	- If we do not subtract it, the `PUSH` operation will overwrite the current location pointed by `ESP`(the top) and we would lose data.

Simple words: Since it goes from a higher memory address to a lower one, the top-most item in the stack gets data put on top of it. Getting close to the lower memory addresses.


```
                               
	                              _______
	                             /       |
	                            /        V
	                           /   _____________
	                          /    |     E     |  ESP - 4
	     _____________       /     |-----------|
	ESP  |     A     |      /      |     A     |
	     |-----------|     /       |-----------|
	     |     B     |   PUSH (E)  |     B     |
             |-----------|             |-----------|
	     |     C     |             |     C     |
	     |-----------|             |-----------|
	     |     D     |             |     D     |
	     ‾‾‾‾‾‾‾‾‾‾‾‾‾             ‾‾‾‾‾‾‾‾‾‾‾‾‾

```


#### A more detailed explanation of `PUSH`:

Starting Value:
	- `ESP` points to the following address: `0x0028FF80`

Process:
	- The program executes the instruction `PUSH 1`. `ESP` decreases by 4, becoming `0x0028FF7C`, and the value `1` will be pushed in the stack.
 
Ending Value:
	- `ESP` points to the following memory address: `0x0028FF7C`


```
                               
	      PUSH 1 -------------------------------\
	        |                                   |
                |	                            | 		
	        |         _____________             V             _____________
	ESP = 0x0028FF80  |    ...    |     ESP = 0x0028FF7C  ->  |  00000001 |
	                  |-----------|                           |-----------|
	                  |  ..data.. |                           |  ..data.. |
		          |-----------|                           |-----------|
	                  |  ..data.. |                           |  ..data.. |
	                  |-----------|                           |-----------|
	                  |  ..data.. |                           |  ..data.. |
	                  ‾‾‾‾‾‾‾‾‾‾‾‾‾                           ‾‾‾‾‾‾‾‾‾‾‾‾‾
```



### 1.2.4.1 POP Instruction

#### `POP E` Example

POP Process:
	- A `POP` is executed, and the `ESP` register is modified.

Starting Value:
	- The `ESP` points to the top of the stack. ( Previous ESP + 4 )

Process:
	- The `POP` operation is the opposite of `PUSH`, and it retrieves the data from the top of the stack.
	- Therefore the data contained at the address of the `ESP` is retrieved and stored (usually in another register).
	- After the `POP` operation, the `ESP` value is added by 4 (x86) or 8 (x64).
		- Cause it the top is the lower memory address so it goes back down to the higher memory address.


Ending Value:
	- The `ESP` points to the top of the stack

```

	     _____________           
	ESP  |     E     | ---> POP (E)         
	     |-----------|                 _____________
	     |     A     |                 |     A     | ESP + 4
	     |-----------|                 |-----------|
	     |     B     |                 |     B     |
	     |-----------|                 |-----------|
	     |     C     |                 |     C     |
	     |-----------|                 |-----------|
	     |     D     |                 |     D     |
	     ‾‾‾‾‾‾‾‾‾‾‾‾‾                 ‾‾‾‾‾‾‾‾‾‾‾‾‾

```


#### A more detailed explanation of `POP`:


Starting Value:
	- After the `PUSH 1`, the `ESP` points to the memory address: `0x0028FF7C`

Process:
	- The program executed the reverse instruction, `POP EAX`.
	- The value (00000001) of the `ESP` ( `0x0028FF7C`, the top of the stack ), will be popped out from the stack and copied over to the `EAX` register. 
	- Then, the `ESP` will be updated by adding 4, becoming `0x0027FF80`.

Ending Value: 
	- `ESP` points to the original value 

```
                               
	      POP EAX-------------------------------\
	        |                                   |
                |	 	                    | 		
	        |         _____________             |             _____________
  ESP = 0x0028FF7C ->     |  00000001 |             |             |  00000001 |
	                  |-----------|             V             |-----------|
	                  |  ..data.. |     ESP = 0x0028FF80  ->  |  ..data.. |
	                  |-----------|                           |-----------|
	                  |  ..data.. |                           |  ..data.. |
	                  |-----------|                           |-----------|
	                  |  ..data.. |                           |  ..data.. |
	                  ‾‾‾‾‾‾‾‾‾‾‾‾‾                           ‾‾‾‾‾‾‾‾‾‾‾‾‾
```



### 1.2.4.3 Procedures and Functions

- It is important to note that if a value is popped out, it will still stay in the stack until another value has overwritten it.

### 1.2.4.4 Stack Frames

- **Functions** contain two important components:
	- The prologue: This prepares the stack to be used, similar to putting a bookmark in a book.
	- The epilogue: When the function completed, the epilogue returns the stack to the prologue settings.

The stack consists of logical *Stack Frames* (portions/areas of the Stack), that are `PUSH`ed when calling a function and `POP`ed when returning a value. 


- When a subroutine, such as a function or procedure, is started, a stack frame is created and assigned to the current `ESP` location. This allows the subroutine to operate independently in its own location in the stack.

- When the subroutine ends, two things happen:
	1. The program receives the parameters passed from the subroutine.
	2. The Instruction Pointer (`EIP`) is reset to the location at the time of initial call.

- So the Stack Frame keeps track of the location where each subroutine should return the control when it terminates. 


Stack Frames have three main operations:
	1. When the function is called, the arguments (in brackets) need to be evaluated.
	2. The control flow jumps to the body of the function, and the program executes its code. 
	3. Once the function ends, a return statement is encounterd, the program returns to the function call.


#### C Example
```C
int b() { // function b
	return 0;
}

int a() { // function a
	b();
	return 0;
}

int main() { // main function
	a();
	return 0;
}
```

Step 1: 
	- The first entry point is `main()`
	- The first stack frame that needs to be pushed to the Stack is the `main()` stack frame. 
	- Once initialized, the stack pointer is set to the top of the stack and a new `main()` stack frame is created.

The stack then looks like this:

```
	Lower Memory
	 Addresses
	     ∧
	     |     
	     |     ______________
	     |     |            |
	     |     --------------
	     |     | Frame for  |
	     |     |   main()   |
	     |     --------------
	     |     |            |
	     |     --------------
	     |
	     |
	Higher Memory
	 Addresses

```

Step 2:
	- Once inside `main()`, the first instruction that executes is a call to the function `a()`
	- Now a new stack frame for `a()` is created at the top of the stack.

```
	Lower Memory
	 Addresses                     --------------
	     ∧                         |            |
	     |                         --------------
	     |     ______________      | Frame for  | 
	     |     |            |      |    a()     | 
	     |     --------------      --------------
	     |     | Frame for  |      | Frame for  |
	     |     |   main()   |      |   main()   |
	     |     --------------      --------------
	     |     |            |      |            |
	     |     --------------      --------------
	     |                        main() calls a()
	     |
	Higher Memory
	 Addresses

```

Step 3:
	- Once the function`a()` starts, the first instructions set is a call to the `b()` function. 
	- Here, the stack pointer is set, and a new stack frame for `b()` will be pushed to the stop of the stack.

```
                                                           ______________
                                                           |            |
	Lower Memory                                       --------------
	 Addresses                     ______________      | Frame for  |  
	     ∧                         |            |      |    b()     |
	     |                         --------------      --------------
	     |     ______________      | Frame for  |      | Frame for  | 
	     |     |            |      |    a()     |      |    a()     | 
	     |     --------------      --------------      --------------
	     |     | Frame for  |      | Frame for  |      | Frame for  |
	     |     |   main()   |      |   main()   |      |   main()   |
	     |     --------------      --------------      --------------
	     |     |            |      |            |      |            |
	     |     --------------      --------------      --------------
	     |                        main() calls a()     a() calls b()  
	     |
	Higher Memory
	 Addresses

```

Step 4:
	- The function `b()` does nothing and just returns. 
	- When the function completes, the stack pointer is moved up to its previous location, and the program returns to the stack frame of `a()` and continues on with the next function

```
                                                        ______________
                                                        |            |
	Lower Memory                                    --------------
	 Addresses                   ______________     | Frame for  |      ______________ 
	     ∧                       |            |     |    b()     |      |            |
	     |                       --------------     --------------      --------------
	     |    ______________     | Frame for  |     | Frame for  |      | Frame for  | 
	     |    |            |     |    a()     |     |    a()     |      |    a()     | 
	     |    --------------     --------------     --------------      --------------
	     |    | Frame for  |     | Frame for  |     | Frame for  |      | Frame for  |
	     |    |   main()   |     |   main()   |     |   main()   |      |   main()   |
	     |    --------------     --------------     --------------      --------------
	     |    |            |     |            |     |            |      |            |
	     |    --------------     --------------     --------------      --------------
	     |                       main() calls a()   a() calls b()       return from b()
	     |
	Higher Memory
	 Addresses

```

Step 5:
	- The next instruction executed is the return statement contained in `a()`
	- The `a()` stack frame is popped, the stack pointer is reset, and it is returned to the `main()` stack frame.

```
						______________
						|            |
Lower Memory                                    --------------
 Addresses                   ______________     | Frame for  |    ______________  
	 ∧                   |            |     |    b()     |    |            |  __________
	 |                   --------------     --------------    --------------  |        |
	 |  ______________   | Frame for  |     | Frame for  |    | Frame for  |  ----------
	 |  |            |   |    a()     |     |    a()     |    |    a()     |  | Frame  |
	 |  --------------   --------------     --------------    --------------  |  for   |
	 |  | Frame for  |   | Frame for  |     | Frame for  |    | Frame for  |  | main() |
	 |  |   main()   |   |   main()   |     |   main()   |    |   main()   |  |        |
	 |  --------------   --------------     --------------    --------------  |--------|
	 |  |            |   |            |     |            |    |            |  |        |
	 |  --------------   --------------     --------------    --------------  ----------
	 |                   main() calls a()   a() calls b()     return from b()  ret. a()
	 |                             
Higher Memory
 Addresses

```



#### C Example for Stack Frames

```C
void functest(int a, int b, int c) {
	int test1 = 55;
	int test2 = 56;
}
int  main(int argc, char *argv[]) {
	int x = 11;
	int y = 12;
	int z = 13;
	functest(30,31,32);
	return 0;
}
```

Step 1:
	- When the program starts, the function `main()` parameters ( `argc`, `argv` ) will be pushed onto the stack from left to right. 

```
	Lower Memory
	 Addresses
	     ∧
	     |     ______________  
	     |     |            |  __
	     |     |    argc    |    \
	     |     |            |     \
	     |     --------------       Parameters of main()
	     |     |            |     /
	     |     |    argv    |    /
	     |     |            |  --
	     |     --------------
	     |     |    ...     |
	     |     --------------
	     |
	Higher Memory
	 Addresses

```

Step 2:
	- `CALL` the function `main()`. Then, the processor `PUSH`es the content of the `EIP` to the stack and points to the first byte after the `CALL` instruction.

Step 3:
	- The caller (the instruction that executes the function calls - which is the OS in this case) loses its control, and the callee (the function that is being called - which is the main function) takes control. 

```
	Lower Memory
	 Addresses
	     ∧
	     |
	     |     ______________
	     |     |            |    
	     |     |  old EIP   |  <-- Return address from main()
	     |     |            |      (The next instruction to 
	     |     --------------       execute once we return 
	     |     |            |       from main)
	     |     |    argc    | 
	     |     |            | 
	     |     -------------- 
	     |     |            | 
	     |     |    argv    |  
	     |     |            |  
	     |     --------------
	     |     |    ...     |
	     |     --------------
	     |
	Higher Memory
	 Addresses

```


Step 4: 
	- Now we're in the `main()` function, and a new stack needs to be created. 
	- The stack frame is defined by the `EBP` (Base Pointer) and the `ESP` (Stack Pointer)


```
	Lower Memory
	 Addresses           /--- Contains the base pointer of the stack
	     ∧               |
	     |               V
	     |     ______________
	     |     |            |  <--- At this time, both EBP and ESP
	     |     |  old EBP   |       points to this address in the
	     |     |            |       memory
	     |     -------------- 
	     |     |            | \   
	     |     |  old EIP   |  \
	     |     |            |   \ 
	     |     --------------    \
	     |     |            |     \
	     |     |    argc    |       
	     |     |            |       Old Stack Frame
	     |     --------------     
	     |     |            |     /
	     |     |    argv    |    /
	     |     |            |   /
	     |     --------------  /
	     |     |    ...     | /
	     |     -------------- 
	     |
	Higher Memory
	 Addresses

```



### 1.2.4.5 Prologue

The previous step is known as a *prologue*: it is a sequence of intstructions that take place at the beginning of a function. This will occur for all functions. Once the callee gets control, it will execute the following instructions.

```
1  push ebp
2  mov ebp, esp
3  sub esp, X    // X is a number
```

The first instruction: `1  push ebp`, saves the old base pointer onto the stack so it can be restored later when the function returns. 

`EBP` is currently pointing to the location of the top of the previous stack frame.


The second instruction: `2  mov ebp, esp`, copies the value of the stack pointer (ESP) into the base pointer (EBP). 

Like: ebp = esp

This then creates a new stack frame on top of the Stack.
	- The base of the new stack frame is on top of the old stack frame


The third instruction: `3  sub esp, X`, moves the stack pointer (top of the stack) by decreasing its value. This is necessary to make space for local variables.

The below diagram illustrates what happens once the prologue is complete.

```
	Lower Memory
	 Addresses
	     ∧
	     |     |            |  EBP - X
	     |     --------------
	     |     |            |
	     |     |            |  EBP - 8 
	     |     |            |
	     |     --------------
	     |     |            |
	     |     |            |  EBP - 4
	     |     |            |
	     |     --------------
	     |     |            |
	     |     |  old EBP   |  ESP + 4
	     |     |(caller EBP)|
	     |     |            |
	     |     --------------
	     |     |            |    
	     |     |  old EIP   |  ESP + 4
	     |     |            | 
	     |     -------------- 
	     |     |            | 
	     |     |    argc    |  ESP + 8 
	     |     |            | 
	     |     -------------- 
	     |     |            | 
	     |     |    argv    |  ...
	     |     |            |  
	     |     --------------
	     |     |    ...     |
	     |     --------------
	     |
	Higher Memory
	 Addresses

```


Since the `main()` function contains other variables, and a function call, the actual stack frame is bigger.

Once the prologue ends, the stack frame for `main()` is complete, and the local variables are copied to the stack. Since `ESP` is not pointing to the memory address right after `EBP`, we cannot push the `PUSH` operation, since `PUSH`stores the value at the top of the stack. 


The instructions after prologue are usually like the following:

```
MOV DWORD PTR SS:[ESP+Y], 0B
```

This instruction means:
	- Move the value `0B` ( hex of 11 - first local variable ) into the memory address location pointed at `ESP+Y`. 
	- Note that `Y` is a number and `ESP+Y` points to a memory address between `ESP` and `EBP`.

This process will repeat through all the variables and look like the following:

```
	Lower Memory
	 Addresses
	     ∧
	     |
	     |     ______________ <--- ESP
	     |     |            |
	     |     |   13 (y)   | EBP - 12
	     |     |            |  
	     |     --------------
	     |     |            |
	     |     |   12 (z)   |  EBP - 8 
	     |     |            |
	     |     --------------
	     |     |            |
	     |     |   11 (x)   |  EBP - 4
	     |     |            |
	     |     --------------
	     |     |            |
	     |     |  old EBP   |  ESP + 4
	     |     |(caller EBP)|
	     |     |            |
	     |     --------------
	     |     |            |    
	     |     |  old EIP   |  ESP + 4
	     |     |            | 
	     |     -------------- 
	     |     |            | 
	     |     |    argc    |  ESP + 8 
	     |     |            | 
	     |     -------------- 
	     |     |            | 
	     |     |    argv    |  ...
	     |     |            |  
	     |     --------------
	     |     |    ...     |
	     |     --------------
	     |
	Higher Memory
	 Addresses

```


I'm gonna skip over Endianness cause I'm tired of making notes.

### 1.2.6 NOPs

The No Operation Instruction (NOP) is an assembly language instruction that does nothing. When a program encounters a `NOP`, it will skip to the next instruction. 
 
In Intel x86 CPUs, `NOP` instructions are represented with the hex value of `0x90`


#### NOP-Sled 

The NOP-Sled is a technique that is used during the exploitation of Buffer Overflows. Its only purpose is to fill a large (or small) portion of the stack with NOPs. 

This will allow us to slide down to the instruction we really want to execute, which is usually after the NOP-sled. 

That's it for these slides.

