# Doing Complex Code Generation in Rust

### Introduction

Now that "simple" code generation is completed, now we can move to complex code generation,
i.e. code with branching control flow that jumps all over the place. In the previous Phase,
code execution was simple, only occuring in a linear fashion. Now, code that can now perform 
loops and branch to different locations with many complex execution paths.

For Phase 4, you will be doing code generation for the following control flow statements:
* While loops
* If and If/Else Statements
* Break and Continue
* Nested While Loops

### Reviewing Structured Programming

In computer hardware, there is no such instruction corresponds to a "while" loop, and there is
no such instruction that corresponds to an "if" statement. Instead, these high-level language
constructs are translated into "jump" and "branch" statements. Plain old "jump" and "branch" assembly 
language statements are then executed on the hardware. There is no such thing as a "while" loop
in hardware, this is a human software construct to help humans model control flow in a structured
way.

To generate IR that does “While loops” and “If statements”, translate the behavioral structures into their corresponding
conditional branch, labels, and unconditional jump statements. One should be able to nest loops inside loops, and nest if 
statements within if statements. A user program should be able to define multiple loops within a program. In an if/else statement, if the if
condition results in true, the if code block should be executed. And if the if condition results in
false, the else code block should be executed.

### Interpreter

You can include the interpreter found in `compiler.rs` as part of your project. You do **not** need to make
any modifications to the interpreter. You can make any change you want to the existing interpreter code.
The interpreter code as found in `compiler.rs` should be sufficient enough to complete Phase 3 and 4. **This is
the same exact interpreter found in Phase 3.**

### IR Syntax and Semantics

There are 4 relevant instructions for doing branching and jumping. They are as follows:

| Instruction               | Description                                                                      |
|---------------------------|----------------------------------------------------------------------------------|
| %label                    | declares a label '%label'. Used in branching code                                |
| %jmp  %label              | jumps to '%label' unconditionally                                                |
| %branch_if var, %label    | jumps to '%label' if var is 1. Does nothing if var is 0                          |
| %branch_ifn var, %label   | jumps to '%label' if var is 0. Does nothing if var is 1                          |

Use `%label` to declare a point that someone can jump to. `%jmp` can be used to jump to the `%label`.

By generating the following code, you can create an infinite loop:

```
%func main() 
%label
%out 0
%jmp %label
%endfunc
```

When running the IR, it should continue to print 0 until infinity.

Use `%branch_if` to jump to the `%label` only if the `var` is 1. `%branch_if` will do nothing if
`var` is 0.

`%branch_ifn` is the opposite: it only jumps to the `%label` if the `var` is 0. `%branch_ifn` will do
nothing if the `var` is 1.

Here is the entire instruction set for the IR, if you need a refresher of what the instructions are.

| Instruction               | Description                                                                      |
|---------------------------|----------------------------------------------------------------------------------|
| %func func(%int a, %int b)| declares a function named 'function' with parameters a and b in that order       |
| %endfunc                  | closes the existing function                                                     |
| %int  variable            | declares a 32 bit integer value named 'variable'                                 |
| %int [] array, 32         | declares an array of 32 bit integers of length 32                                |
| %mov  dest, src1          | dest = src1                                                                      |
| %mov  [array + i], src1   | array[i] = src1                                                                  |
| %mov  dest, [array + i]   | dest = array[i]                                                                  |
| %add  dest, src1, src2    | dest = src1 +  src2                                                              |
| %sub  dest, src1, src2    | dest = src1 -  src2                                                              |
| %mult dest, src1, src2    | dest = src1 *  src2                                                              |
| %div  dest, src1, src2    | dest = src1 /  src2                                                              |
| %mod  dest, src1, src2    | dest = src1 %  src2                                                              |
| %lt   dest, src1, src2    | dest = src1 <  src2                                                              |  
| %le   dest, src1, src2    | dest = src1 <= src2                                                              |
| %neq  dest, src1, src2    | dest = src1 != src2                                                              |
| %eq   dest, src1, src2    | dest = src1 == src2                                                              |
| %gt   dest, src1, src2    | dest = src1 >  src2                                                              |
| %ge   dest, src1, src2    | dest = src1 >= src2                                                              |
| %out  value               | prints out the value to standard output                                          |
| %in   value               | store an integer from standard input into 'value'                                |
| %call dest, func(a,b)     | calls a function 'func' with parameters (a,b). Stores the return value in 'dest' |
| %ret  value               | return 'value' from the function.                                                |
| %label                    | declares a label '%label'. Used in branching code                                |
| %jmp  %label              | jumps to '%label' unconditionally                                                |
| %branch_if var, %label    | jumps to '%label' if var is 1. Does nothing if var is 0                          |
| %branch_ifn var, %label   | jumps to '%label' if var is 0. Does nothing if var is 1                          |

### Generated Example IR Code

Here are some examples of possible generated IR outputs. One can generate any IR code for the given code, as
long as the generated IR functions in the same way. **Any IR generated is acceptable, as long as it outputs
the same numbers**.

#### loop

Given a simple loop `loop.min`:
```
func main() {
    int i;
    i = 0;
    while i < 10 {
        print(i);
        i = i + 1;
    }
}
```
You can generate the follow IR. Note that `%jmp %loopbegin1` is used to jump to the beginning of the loop,
marked `%loopbegin1`. We break out of the loop using `%branch_ifn _temp, %endloop1` to break out of the loop.
```
%func main()
%int i
%mov i, 0
%loopbegin1
%int _temp1
%lt _temp1, i, 10
%branch_ifn _temp1, %endloop1
%out i
%int _temp2
%add _temp2, i, 1
%mov i, _temp2
%jmp %loopbegin1
%endloop1
%endfunc
```
The output of the `loop.tt` IR should be:
```
0
1
2
3
4
5
6
7
8
9
```
---
Given the following `if.tt` program:
```
func main() {
    int a;
    int b;
    int c;

    
    a = 100;
    b = 50;
    if a < b {
        c = 0;
    } else {
        c = 1;
    }

    # Should print out '1'.
    print(c);



    a = 100;
    b = 50;
    if a >= b {
        c = 0;
    } else {
        c = 1;
    }

    # Should print out '0'
    print(c);
}
```

The following IR can be generated:
```
%func main()
%int a
%int b
%int c
%mov a, 100
%mov b, 50
%int _temp1
%lt _temp1, a, b
%branch_if _temp1, %iftrue1
%jmp %else1
%iftrue1
%mov c, 0
%jmp %endif1
%else1
%mov c, 1
%endif1
%out c
%mov a, 100
%mov b, 50
%int _temp2
%ge _temp2, a, b
%branch_if _temp2, %iftrue2
%jmp %else2
%iftrue2
%mov c, 0
%jmp %endif2
%else2
%mov c, 1
%endif2
%out c
%endfunc
```

The output of `if.tt` is the following:
```
1
0
```


---
Given the following `nested_loop.tt` program:
```
func main() {
    int i;
    int j;
    i = 0;
    while i < 2 {
        j = 0;
        while j < 3 {
            print(j);
            j = j + 1;
        }
        i = i + 1;
    }
}
```

For `nested_loop.tt`, the following IR is valid:
```
%func main()
%int i
%int j
%mov i, 0
%loopbegin1
%int _temp1
%lt _temp1, i, 2
%branch_ifn _temp1, %endloop1
%mov j, 0
%loopbegin2
%int _temp2
%lt _temp2, j, 3
%branch_ifn _temp2, %endloop2
%out j
%int _temp3
%add _temp3, j, 1
%mov j, _temp3
%jmp %loopbegin2
%endloop2
%int _temp4
%add _temp4, i, 1
%mov i, _temp4
%jmp %loopbegin1
%endloop1
%endfunc
```

The output of the `nested_loop.tt` IR should be:
```
0
1
2
0
1
2
```
---

Given the following `break.tt` high level language code:

```
func main() {
    int i;
    i = 0;
    while i < 10 {
        if i >= 4 {
            break;
        }
        print(i);
        i = i + 1;
    }
}
```

You can generate the following IR. The IR uses `%jmp %endloop1` to implement a break statement.

```
%func main()
%int i
%mov i, 0
%loopbegin1
%int _temp1
%lt _temp1, i, 10
%branch_ifn _temp1, %endloop1
%int _temp2
%ge _temp2, i, 4
%branch_if _temp2, %iftrue1
%jmp %endif1
%iftrue1
%jmp %endloop1
%endif1
%out i
%int _temp3
%add _temp3, i, 1
%mov i, _temp3
%jmp %loopbegin1
%endloop1
%endfunc
```

The IR code when executed should produce the following result:

```
0
1
2
3
```

