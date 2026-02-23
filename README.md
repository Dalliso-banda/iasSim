# IASSim — Princeton IAS Machine Web Emulator

A fully interactive, browser-based emulator of the Princeton IAS (Institute for Advanced Study) machine — the pioneering Von Neumann architecture computer designed in 1946. Built as a teaching tool for computer science students, it runs entirely in a single HTML file with no dependencies or installation required.

---

## What Is the IAS Machine?

The IAS machine was designed by Arthur Burks, Herman Goldstine, and John von Neumann at Princeton in 1946. It was one of the first computers to store both instructions and data in the same memory (the stored-program concept), making it the foundational model for virtually all modern computers. Understanding it gives students a direct window into the origins of computer architecture.

---

## Features

### Full IDE
- **Code editor** with line numbers, tab support, and syntax-aware layout
- **Two language modes** — IAS Assembly and the high-level JVNTRAN language
- **Four preloaded example programs** accessible from the dropdown

### Assembler
- Two-pass assembler with full label resolution
- Supports all 22 IAS opcodes
- Pseudoinstructions: `.data` (define a 40-bit constant) and `.empty` (fill an unused half-word)
- Inline comments with `;`
- Clear error messages with line numbers

### JVNTRAN Compiler
JVNTRAN is a minimal high-level language that compiles down to IAS assembly. It is designed to show students what a high-level language for this machine might have looked like.

Supported constructs:
- `var name = value` — variable declaration with optional initializer
- `name = expr` — assignment
- `+`, `-`, `*`, `/` — arithmetic (two operands)
- `while (cond) { }` — loop
- `if (cond) { } else { }` — conditional
- Conditions: `x > 0`, `x > y`, `x < y`, `x >= y`

### Debugger
- **Step Forward** — execute one instruction and inspect state
- **Step Backward** — rewind to any previous machine state (full history snapshots)
- **Run** — execute up to 100,000 instructions continuously
- **Stop** — interrupt a running program at any point
- **Reset** — restore memory and registers to post-assemble state

### CPU Register Panel
All 7 IAS registers displayed live with change-flash animation:

| Register | Width | Description |
|---|---|---|
| AC | 40-bit | Accumulator — primary arithmetic register |
| AR | 40-bit | Arithmetic Register — secondary / multiply-divide |
| CC | 12-bit | Control Counter — the program counter |
| CR | 20-bit | Control Register — currently executing instruction |
| FTR | 8-bit | Function Table Register — current opcode |
| MAR | 12-bit | Memory Address Register — current operand address |
| SR | 40-bit | Selectron Register — value read from / written to memory |

### Memory Viewer
- Displays all non-zero memory words
- Shows word address, hex value, and decoded instruction pair
- Highlights the currently executing word in cyan
- Auto-scrolls to the active instruction during stepping

### Console
- Assembly results and error messages
- Halt notifications with final AC and AR values
- Run statistics (step count)

---

## Getting Started

### Running the Emulator

No installation needed. Open `iassim.html` in any modern web browser:

```
File → Open → iassim.html
```

Works in Chrome, Firefox, Safari, and Edge. Works offline. Works on mobile.

### Your First Program

1. The editor opens with the **Sum 1..N** example preloaded
2. Click **▶ Assemble & Load** (or press `F5`)
3. Switch to the **Memory** tab to see loaded words
4. Click **Step ▶** (or press `F8`) to execute one instruction at a time
5. Watch the registers update — AC will count down as the loop runs
6. Click **Run ▶▶** (or press `F9`) to run to completion
7. Check the console — it will show the final value of AC

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `F5` | Assemble & Load |
| `F8` | Step Forward |
| `F7` | Step Backward |
| `F9` | Run |
| `F2` | Reset |
| `Tab` | Insert 4 spaces in editor |

---

## IAS Instruction Set Reference

The IAS machine has 22 instructions. Each 20-bit instruction consists of an 8-bit opcode and a 12-bit memory address operand. Two instructions are packed into each 40-bit memory word (left and right).

| Opcode | Mnemonic | Description |
|---|---|---|
| 0 | `halt` | Stop execution |
| 1 | `S(x)->Ac+` | Load mem[x] into AC |
| 2 | `S(x)->Ac-` | Load negative of mem[x] into AC |
| 3 | `S(x)->AcM` | Load absolute value of mem[x] into AC |
| 4 | `S(x)->Ac-M` | Load negative absolute value of mem[x] into AC |
| 5 | `S(x)->Ah+` | Add mem[x] to AC |
| 6 | `S(x)->Ah-` | Subtract mem[x] from AC |
| 7 | `S(X)->AhM` | Add absolute value of mem[x] to AC |
| 8 | `S(X)->Ah-M` | Subtract absolute value of mem[x] from AC |
| 9 | `S(x)->R` | Load mem[x] into AR |
| 10 | `R->A` | Copy AR to AC |
| 11 | `S(x)*R->A` | Multiply mem[x] × AR; high 40 bits → AC, low 40 bits → AR |
| 12 | `A/S(x)->R` | Divide AC by mem[x]; quotient → AR, remainder → AC |
| 13 | `Cu->S(x)` | Unconditional jump to left instruction at address x |
| 14 | `Cu'->S(x)` | Unconditional jump to right instruction at address x |
| 15 | `Cc->S(x)` | If AC ≥ 0, jump to left instruction at x |
| 16 | `Cc'->S(x)` | If AC ≥ 0, jump to right instruction at x |
| 17 | `At->S(x)` | Store AC to mem[x] |
| 18 | `Ap->S(x)` | Patch address field of left instruction at mem[x] with low 12 bits of AC |
| 19 | `Ap'->S(x)` | Patch address field of right instruction at mem[x] with low 12 bits of AC |
| 20 | `L` | Shift AC left 1 bit |
| 21 | `R` | Shift AC right 1 bit (arithmetic — sign bit preserved) |

> **Note on opcodes 18–19:** These are used for *self-modifying code*, which is how the IAS machine implements array indexing. The program modifies its own instruction operands at runtime to step through memory locations.

---

## Assembly Language

```asm
; Semicolons begin comments
; Labels end with a colon

label:  S(x)->Ac+  varname   ; load mem[varname] into AC
        S(x)->Ah+  other     ; AC = AC + mem[other]
        At->S(x)   result    ; mem[result] = AC
        Cc->S(x)   label     ; if AC >= 0, jump to label
        Cu->S(x)   label     ; unconditional jump
        halt                 ; stop the machine

        .empty               ; fills one unused 20-bit half-word with 0

result: .data 0              ; defines a 40-bit word initialized to 0
count:  .data 10             ; initialized to 10
```

**Memory layout:** Instructions are packed two per word. The assembler automatically handles left/right placement. `.data` always starts on a new word boundary. `.empty` fills the current half-word slot.

**Labels** resolve to word addresses (0–4095). When used as jump targets, the assembler uses the word address. Left vs. right instruction is controlled by whether you use `Cu->S(x)` (left) or `Cu'->S(x)` (right).

---

## JVNTRAN Language

```
// Line comments with //

var x = 10          // declare variable, optionally with initial value
var y = 0
var one = 1

// Arithmetic assignment (two operands only)
y = x + y
y = x - y
y = x * y
y = x / y

// While loop
while (x > 0) {
  y = y + x
  x = x - one
}

// If / else
if (y > 0) {
  y = y - one
} else {
  y = y + one
}
```

Supported conditions: `x > 0`, `x > y`, `x < y`, `x >= y`

JVNTRAN compiles to IAS assembly internally. You can switch to ASM mode to see the compiled output before it is assembled and loaded. This is useful for understanding how high-level constructs map to machine instructions.

---

## Architecture Details

| Property | Value |
|---|---|
| Memory | 4,096 words |
| Word size | 40 bits |
| Instructions per word | 2 (left + right) |
| Opcode size | 8 bits |
| Operand size | 12 bits (addresses 0–4095) |
| Integer representation | Two's complement |
| Number range | −549,755,813,888 to 549,755,813,887 |

---

## Example Programs

### Sum 1 to N (Assembly)
Computes 0 + 1 + 2 + ... + n. The result is stored in `sum`. Uses a conditional branch to loop and decrement `n`.

### Sum 1 to N (JVNTRAN)
The same algorithm written in JVNTRAN. Compare the two to see how the compiler generates assembly.

### Fibonacci (Assembly)
Computes the 8th Fibonacci number (21) by iterating through the sequence using two variables tracking consecutive terms.

### Max of Two (JVNTRAN)
Demonstrates the `if/else` construct by finding the larger of two values `a` and `b`.

---

## Limitations

- No I/O instructions — all data must be initialized in memory via `.data` before execution (faithful to the original machine)
- JVNTRAN arithmetic expressions support exactly two operands (e.g., `a + b`, not `a + b + c`)
- No floating-point support — all arithmetic is integer only
- Self-modifying code (opcodes 18–19) is supported at the machine level but not abstracted in JVNTRAN
- Run limit: 100,000 instructions per "Run" to prevent infinite loop lockups

---

## Browser Compatibility

| Browser | Status |
|---|---|
| Chrome 80+ | ✅ Fully supported |
| Firefox 75+ | ✅ Fully supported |
| Safari 14+ | ✅ Fully supported |
| Edge 80+ | ✅ Fully supported |
| Mobile Chrome/Safari | ✅ Responsive layout |

Requires BigInt support (available in all modern browsers since 2020). No internet connection required after the font loads — fonts are loaded from Google Fonts on first open and cached by the browser.

---

## Credits

Based on the original **IASSim** Java application by **Barry Fagin** (US Air Force Academy) and **Dale Skrien** (Colby College), which emulates the machine described in:

> Burks, A.W., Goldstine, H.H., and von Neumann, J. (1946). *Preliminary Discussion of the Logical Design of an Electronic Computing Instrument.* Institute for Advanced Study, Princeton, New Jersey.

The original paper is available at: https://hdl.handle.net/2027.42/3972

This web port preserves the original machine's instruction set and architecture faithfully, adds a JVNTRAN compiler, and delivers the full IDE experience in a single HTML file accessible to any student with a web browser.

---

## License

This emulator is provided free for educational use. The IAS architecture and original IASSim concept are credited to Barry Fagin, Dale Skrien, and the original authors of the 1946 Princeton report.
