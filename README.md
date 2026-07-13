# ST3PH-FORTH // Interactive WebAssembly-Styled IDE & Runtime

[![License: MIT](https://shields.io)](https://github.com/ST3PH-X/interactive-forth-wasm-ide/blob/main/LICENSE)
[![Live Demo](https://shields.io)](https://st3ph-x.github.io/interactive-forth-wasm-ide/)

An ultra-lightweight, zero-dependency, Turing-complete **Forth Development Environment** and virtual machine engine written in pure vanilla JavaScript. This platform emulates low-level hardware interactions, physical linear memory routing, and hardware I/O interrupts. 

To demonstrate absolute **Turing-completeness**, a complete bare-metal **Snake Game** was developed entirely within the system's raw Forth dialect, обсчитывая физику векторов и сдвиг массивов исключительно через манипуляции с ячейками памяти.

---

## 🎓 Academic Value as a Computer Science Tool

In modern high-level engineering education, students often lose touch with bare-metal computation constraints. This platform serves as a powerful **conceptual bridge** between modern web engineering and low-level computer architecture classes (such as MIT 6.004 / Stanford CS107).

By restricting students to programming strictly in the native Forth environment—without touching the underlying JavaScript engine—the system delivers critical educational value:
- **Destroys High-Level Laziness**: Strips away variables, objects, and garbage-collected abstractions. Students must track memory addresses and manage scope manually using the data stack.
- **Embedded Systems Emulation**: Mentally clones the constraints of microcontrollers and real-time kernels where storage is measured in bytes and parameters are piped directly through processor registers.
- **Algorithmic Discipline**: Writing complex code on top of a naked stack structure forces students to produce highly optimized, concise loops. Spaghetti routines simply fail to compile.

---

<details>
<summary>💻 System Architecture & Low-Level Specifications</summary>
<br>

The runtime emulates a hardware processing unit with a dedicated execution loop and two isolated data structures:

### 1. The Dual-Stack Topology
- **Data Stack (Parameter Stack)**: Handles all active calculation contexts, logic comparisons, and function parameters using Reverse Polish Notation (RPN / Postfix). Operates strictly on a Last-In, First-Out (LIFO) model.
- **Return Stack (Control Stack)**: Completely isolated from arithmetic words. It holds subroutine callback pointers and tracks execution contexts during definite nesting loops.

### 2. Linear Memory Map (WASM-Inspired)
Instead of relying on JavaScript objects, the engine instantiates an internal raw array buffer (`Uint32Array(8192)`). Addresses are mapped directly to physical integer coordinates:
- `0x0000 - 0x03E7` (Addresses 0 - 999): High-performance Framebuffer (160x120 monochromatic/color map layout).
- `0x03E8 - 0x07CF` (Addresses 1000 - 1999): Dynamic Game State & Hardware Registers (Head X/Y, Direction, Delays).
- `0x07D0 - 0x0FFF` (Addresses 2000 - 4095): Cyclical Array Vectors Buffer (Tail coordinates management array).

### 3. Asynchronous I/O Intercept Loops
The JavaScript loop hooks window event listeners to trap keyboard interrupts. When a key is targeted, it populates a virtual hardware buffer register. Forth utilizes polling words (`key?` and `key`) to extract, process, and immediately flush the input stream without blocking execution blocks.

</details>

<details>
<summary>📋 Core Instruction Set & Stack Notation Dictionary</summary>
<br>

All operations within the engine are documented below. Stack notation diagrams follow the standard Forth convention: `( input_elements -- output_elements )`, where the rightmost item represents the top of the stack (TOS).

<table style="width: 100%; border-collapse: collapse; margin-top: 10px;">
    <thead>
        <tr style="border-bottom: 2px solid #242435;">
            <th style="padding: 10px; text-align: left; background: #161622; color: #9d4edd;">Category</th>
            <th style="padding: 10px; text-align: left; background: #161622; color: #9d4edd;">Word</th>
            <th style="padding: 10px; text-align: left; background: #161622; color: #9d4edd;">Stack Diagram</th>
            <th style="padding: 10px; text-align: left; background: #161622; color: #9d4edd;">Operational Description</th>
        </tr>
    </thead>
    <tbody>
        <!-- Stack Operations -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Stack Ops</td>
            <td style="padding: 8px;"><code>dup</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n -- n n )</code></td>
            <td style="padding: 8px;">Duplicates the top element of the active data stack.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Stack Ops</td>
            <td style="padding: 8px;"><code>drop</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n -- )</code></td>
            <td style="padding: 8px;">Discards the top element from the stack.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Stack Ops</td>
            <td style="padding: 8px;"><code>swap</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n1 n2 -- n2 n1 )</code></td>
            <td style="padding: 8px;">Reverses the positions of the top two data stack elements.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Stack Ops</td>
            <td style="padding: 8px;"><code>over</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n1 n2 -- n1 n2 n1 )</code></td>
            <td style="padding: 8px;">Copies the second stack element onto the top of the stack.</td>
        </tr>
        <!-- Arithmetic -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Math</td>
            <td style="padding: 8px;"><code>+</code>, <code>-</code>, <code>*</code>, <code>/</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n1 n2 -- res )</code></td>
            <td style="padding: 8px;">Executes binary arithmetic on signed 32-bit integers. Div by zero drops an execution halt block.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Math</td>
            <td style="padding: 8px;"><code>mod</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n1 n2 -- rem )</code></td>
            <td style="padding: 8px;">Returns the remainder of integer division (modulo arithmetic).</td>
        </tr>
        <!-- Comparison -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Logic Ops</td>
            <td style="padding: 8px;"><code>&lt;</code>, <code>&gt;</code>, <code>=</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( n1 n2 -- flag )</code></td>
            <td style="padding: 8px;">Evaluates logic. Pushes <code>-1</code> (True in Forth) if comparison is valid, or <code>0</code> (False).</td>
        </tr>
        <!-- Memory I/O -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Memory I/O</td>
            <td style="padding: 8px;"><code>@</code> (fetch)</td>
            <td style="padding: 8px; color: #00ffaa;"><code>( addr -- val )</code></td>
            <td style="padding: 8px;">Pointer dereference. Reads a 32-bit value directly from the specified index in linear memory array.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Memory I/O</td>
            <td style="padding: 8px;"><code>!</code> (store)</td>
            <td style="padding: 8px; color: #00ffaa;"><code>( val addr -- )</code></td>
            <td style="padding: 8px;">Writes a 32-bit integer literal value into the specified index destination inside linear memory.</td>
        </tr>
        <!-- Hardware I/O -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Hardware I/O</td>
            <td style="padding: 8px;"><code>key?</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( -- flag )</code></td>
            <td style="padding: 8px;">Asynchronous pipeline query. Returns <code>-1</code> if a hardware keyboard key is actively held down, else <code>0</code>.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Hardware I/O</td>
            <td style="padding: 8px;"><code>key</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( -- char )</code></td>
            <td style="padding: 8px;">Extracts the virtual keycode from the hardware latch buffer, pushes it to stack, and instantly flushes buffer.</td>
        </tr>
        <!-- Control Flow -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Flow Control</td>
            <td style="padding: 8px;"><code>if..else..then</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( flag -- )</code></td>
            <td style="padding: 8px;">Conditional branching statement block. Executes internal block if flag != 0. <code>then</code> marks end of construct.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Flow Control</td>
            <td style="padding: 8px;"><code>do ... loop</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( limit init -- )</code></td>
            <td style="padding: 8px;">Definite loop. Pushes counter states onto return stack and loops from init up to limit boundary index.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Flow Control</td>
            <td style="padding: 8px;"><code>i</code>, <code>j</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( -- index )</code></td>
            <td style="padding: 8px;">Runtime loop counters extraction. <code>i</code> pulls current inner loop iteration index. <code>j</code> pulls the outer loop index.</td>
        </tr>
        <!-- System and Graphics -->
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Graphics</td>
            <td style="padding: 8px;"><code>plot</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( x y color -- )</code></td>
            <td style="padding: 8px;">Hardware graphic bridge. Writes target color directly to (x, y) coordinates into frame display layout.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">Graphics</td>
            <td style="padding: 8px;"><code>cls</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( -- )</code></td>
            <td style="padding: 8px;">Wipes frame canvas buffer array back to default dark opaque processing space.</td>
        </tr>
        <tr style="border-bottom: 1px solid #242435;">
            <td style="padding: 8px; color: #64748b; font-weight: bold;">System</td>
            <td style="padding: 8px;"><code>exit</code></td>
            <td style="padding: 8px; color: #00ffaa;"><code>( -- )</code></td>
            <td style="padding: 8px;">Forces instantaneous crash termination out of the current nested token subroutine subroutine loop.</td>
        </tr>
    </tbody>
</table>

</details>

<details>
<summary>🎓 Academic Laboratory Coursework Syllabus (MIT / Stanford Architecture Spec)</summary>
<br>

This curriculum is designed to teach low-level algorithmic engineering, pointer routing, and memory optimization constraints strictly inside the **ST3PH-FORTH** execution environment. Students are forbidden from modifying the underlying JavaScript driver engine.

### Laboratory Assignment 1: Stack Manipulation Mechanics & Integer Physics
- **Objective**: Mastery of Reverse Polish Notation (RPN), stack tracking, and data overflow boundaries.
- **Task Description**: Build a custom word `quadratic ( a b c x -- result )` that solves the polynomial equation $f(x) = ax^2 + bx + c$ using only pure stack primitives (`dup`, `swap`, `over`, `drop`) without storing values in variables or memory slots.
- **Evaluation Criteria**: Minimum number of data mutations. The stack must be completely clean upon subroutine exit except for the final integer output.

### Laboratory Assignment 2: Linear Memory Address Pointer Routing & Buffer Controls
- **Objective**: Comprehensive understanding of hardware multi-address arrays, vector allocations, and basic memory safety rules.
- **Task Description**: Write a program that treats linear memory slots from address `4000` to `4020` as a continuous vector data bank. Develop a routine that finds the largest signed integer inside this micro-allocated section and pushes its address index onto the parameter stack.
- **Evaluation Criteria**: Proper array edge collision processing. The routine must not leak reads or corrupt data boundaries inside peripheral display blocks.

### Laboratory Assignment 3: Direct Asynchronous Video Framebuffer Pipeline Generation
- **Objective**: Practical understanding of real-time polling loops, hardware interrupt processing, and geometric coordinates rendering.
- **Task Description**: Implement a high-performance custom graphic screensaver or complex interactive display block. Students must use definite `do ... loop` primitives to render a beautiful checkerboard pattern or procedural sine wave approximation on the active 160x120 screen grid layout.
- **Evaluation Criteria**: Efficiency of the internal execution loop. The Forth code must run smoothly inside the rendering cycle without choking the browser frame processing budget.

</details>

---

## ⚖️ License & Intellectual Property
Developed by **Stephaniia Bubnova 2026**. This project is officially distributed and licensed under the conditions of the open-source **MIT License**. For deep structural alignment details, review the repository [LICENSE](https://github.com/ST3PH-X/interactive-forth-wasm-ide/blob/main/LICENSE) mapping manifest.
