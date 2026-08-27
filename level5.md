# MicroVM — Level 5

Your level 4 VM is already [brainfuck](https://en.wikipedia.org/wiki/Brainfuck). Brainfuck is a tiny esoteric programming language with only eight commands. The only difference: real brainfuck programs aren't arrays of numbers — they're strings like `++++++++[>+++++++++<-]>.`.

The mapping is:

| Character | Your opcode |
|-----------|------------|
| `+` | `INC` |
| `-` | `DEC` |
| `>` | `RIGHT` |
| `<` | `LEFT` |
| `.` | `PRINT` |
| `,` | `READ` |
| `[` | `JUMP_IF_ZERO` |
| `]` | `JUMP_IF_NOT_ZERO` |

All other characters are ignored (they're comments).

There's one problem: in your level 4 bytecode, each jump instruction includes an exact numeric target. In brainfuck source code, `[` doesn't say where `]` is — you have to **find the matching bracket**.

You need to solve the bracket matching problem. Here's the approach:

**Before running the program**, scan through the entire source string. Build a table that maps every `[` position to its matching `]` position, and vice versa.

```c
// pseudocode
int match[program_length];   // initialize entries to -1 if your language needs it

int depth = 0;
int opens[MAX_DEPTH];        // a stack! you know how these work now

for i = 0 to length-1:
    if program[i] == '[':
        if depth == MAX_DEPTH:
            error("bracket nesting too deep")

        opens[depth] = i
        depth += 1

    if program[i] == ']':
        if depth == 0:
            error("unmatched ]")

        depth -= 1
        int open = opens[depth]
        match[i] = open              // ] points back to [
        match[open] = i              // [ points forward to ]

if depth != 0:
    error("unmatched [")
```

Look at that — it's a stack, just like level 2. You're using a stack to compile the bracket pairs. `[` pushes its position, `]` pops the matching `[` and wires them together.

After this, your VM loop barely changes. Instead of reading a numeric jump target from `program[pc+1]`, you look it up:

```c
// Level 4:
case JUMP_IF_ZERO:
    if (tape[dp] == 0) pc = program[pc+1];
    else pc += 2;

// Level 5 (brainfuck):
case '[':
    if (tape[dp] == 0) pc = match[pc] + 1;   // skip to after the matching ]
    else pc += 1;                            // note: pc += 1, not 2. no operand byte anymore.
    break;

case ']':
    if (tape[dp] != 0) pc = match[pc] + 1;   // jump back to after the matching [
    else pc += 1;
    break;

// Any other character is a comment.
default:
    pc += 1;
    break;
```

At `]`, jumping to `match[pc]` instead of `match[pc] + 1` can also work if your loop immediately re-executes `[`. The version above jumps straight to the first command inside the loop.

**Real brainfuck programs to execute**

After implementing bracket matching, run these two real brainfuck source strings directly in your interpreter.

1. Multiply loop — prints `H` because `8 × 9 = 72`:

```brainfuck
++++++++[>+++++++++<-]>.
```

2. Hello World — prints `Hello World!` followed by a newline:

```brainfuck
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```

If both of these work, your brainfuck interpreter is done.

**How to debug it**

If they don't work, start smaller:

1. `++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++.` — that's 72 `+` signs then `.`. Should print `H`. If this doesn't work, your `INC`/`PRINT` is broken.

2. Try the multiply loop above. If the 72-plus program works but the multiply loop doesn't, your bracket matching or jumps are broken.

3. If the multiply loop works but Hello World doesn't, your bracket matcher probably fails on **nested** brackets. The Hello World program has `[` inside `[` — make sure your depth counter handles that.

---

That's the whole path. Five levels, each one adding exactly one concept:

1. Fetch-decode-execute
2. Stack
3. Control flow
4. Tape memory model
5. Source parsing (bracket matching)
