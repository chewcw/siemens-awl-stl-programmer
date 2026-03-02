# 🧠 System Instruction: Siemens S7 STL (AWL) Core Instruction Reference

Follow these rules strictly:

* STL is accumulator-based (ACCU1 / ACCU2).
* Boolean instructions modify the RLO (Result of Logic Operation).
* Instructions execute sequentially.
* Logic flows left-to-right within a network.
* Always use valid Siemens STL mnemonics.
* Use uppercase mnemonics.
* Keep industrial structure clear and deterministic.
* Assume standard scan cycle model (Read → Execute → Write).

---

# 1. Boolean Logic Instructions

These operate on the RLO (Result of Logic Operation).

Valid mnemonics:

```
A     // AND
AN    // AND NOT
O     // OR
ON    // OR NOT
=     // Assign RLO
S     // Set (latch)
R     // Reset (unlatch)
```

Example:

```awl
A     I0.0      // AND
AN    I0.1      // AND NOT
O     I0.2      // OR
ON    I0.3      // OR NOT
=     Q0.0      // Assign RLO to operand
S     M0.0      // Set (latch)
R     M0.0      // Reset (unlatch)
```

### Execution Notes

* Logic flows left to right.
* `A` loads operand into RLO (or ANDs with existing RLO).
* `=` writes RLO into the specified operand.
* `S` and `R` are retentive operations.
* RLO is built sequentially.

---

# 2. Edge Detection

Used for detecting signal transitions.

```
FP    // Rising edge
FN    // Falling edge
```

Example:

```awl
A     I0.0
FP    M0.1      // Rising edge detection
FN    M0.2      // Falling edge detection
```

### Notes

* `FP` = Flank Positive
* `FN` = Flank Negative
* Requires memory bit to store previous state.
* Used for pushbutton detection, pulse counting, event triggering.

---

# 3. Timers (S5 Style Timers)

Classic Siemens timer format.

Time literals:

```
S5T#3S
S5T#500MS
```

Example:

```awl
L     S5T#5S
SE    T1        // On-delay timer
SA    T2        // Off-delay timer
```

### Common Timer Instructions

| Instruction | Meaning     |
| ----------- | ----------- |
| `SE`        | On-delay    |
| `SA`        | Off-delay   |
| `SD`        | Pulse timer |


Rules:

* Timers use S5 time format (e.g., `S5T#3S`, `S5T#500MS`).
* Preset must be loaded before use.
* `CU` increments on rising edge.

---

# 4. Counter Instructions

```awl
L     C#10      // Load preset value
CU    C1        // Count up
CD    C1        // Count down
```

### Notes

* `CU` increments counter on rising edge.
* `CD` decrements counter.
* Counter preset must be loaded before use.

---

# 5. Arithmetic Instructions

### Integer Math

```awl
L     MW10
L     MW12
+I
T     MW14
```

| Instruction | Meaning          |
| ----------- | ---------------- |
| `+I`        | Add Integer      |
| `-I`        | Subtract Integer |
| `*I`        | Multiply Integer |
| `/I`        | Divide Integer   |

---

### Real Math

```awl
L     MD10
L     MD14
+R
T     MD18
```

| Instruction | Meaning       |
| ----------- | ------------- |
| `+R`        | Add Real      |
| `-R`        | Subtract Real |
| `*R`        | Multiply Real |
| `/R`        | Divide Real   |

---

# 6. Compare Instructions

### Integer Compare

```awl
L     MW10
L     100
>I
=     Q0.1
```

### Real Compare

```awl
L     MD10
L     80.0
>R
=     M0.0
```

| Instruction | Meaning             |
| ----------- | ------------------- |
| `>I`        | Greater than (INT)  |
| `<I`        | Less than (INT)     |
| `==I`       | Equal (INT)         |
| `>R`        | Greater than (REAL) |
| `<R`        | Less than (REAL)    |
| `==R`       | Equal (REAL)        |

Comparison result affects RLO.

---

# 7. Data Movement

```awl
L     MW10      // Load into ACCU1
T     MW12      // Transfer ACCU1
```

| Instruction | Meaning     |
| ----------- | ----------- |
| `L`         | Load        |
| `T`         | Transfer    |
| `ITD`       | INT → DINT  |
| `DTR`       | DINT → REAL |

---

# 8. Accumulator Model

STL uses two accumulators:

* ACCU1
* ACCU2

Example:

```awl
L     10
L     20
+I
```

Execution:

1. ACCU1 = 10
2. ACCU1 = 20, ACCU2 = 10
3. Result = 30 in ACCU1

---

# 9. RLO (Result of Logic Operation)

Boolean logic builds on RLO.

Example:

```awl
A     I0.0
AN    I0.1
=     Q0.0
```

Flow:

* Load `I0.0` into RLO
* AND NOT `I0.1`
* Assign result to `Q0.0`

---

# 10. PLC Scan Model Assumption

Always assume:

1. Inputs are read.
2. OB1 executes sequentially.
3. Outputs are written.
4. Next scan begins.

Do not generate event-driven code.
Do not assume parallel execution.

---

# 11. Output Constraints for AI

When generating STL:

* Use correct Siemens mnemonics.
* Do not mix with Ladder or SCL.
* Avoid pseudocode.
* Respect accumulator logic.
* Maintain deterministic execution order.
* Keep networks logically separated.

# 12. Block Structures & Calling Syntax Reference
(for STEP 7 Classic – S7-300 / S7-400)

## Block Types Overview

| Block Type | Purpose                              | Has Memory? | Requires Instance DB?  | Typical Use Case                     | Execution Entry Point? |
|------------|--------------------------------------|-------------|------------------------|--------------------------------------|------------------------|
| OB         | Organization Block                   | No          | No                     | Cyclic, startup, interrupt, error    | Yes (called by OS)     |
| FC         | Function (stateless subroutine)      | No          | No                     | Calculations, logic utilities        | No                     |
| FB         | Function Block (stateful)            | Yes         | Yes                    | Timers, counters, state machines     | No                     |
| DB         | Data Block (global or instance)      | Yes         | —                     | Store variables, arrays, UDTs        | No                     |
| UDT        | User-Defined Data Type               | —          | —                     | Reusable struct template             | No                     |

## Block Definition Syntax Examples

* Refer to `../example_blocks/OrganizationBlock.awl` for OB1 structure
* Refer to `../example_blocks/Function.awl` for FC pattern
* Refer to `../example_blocks/FunctionBlock.awl` for FB structure
* Refer to `../example_blocks/FunctionBlockInstanceDB.awl` for relevent FB's instance data block format
* Refer to `../example_blocks/DataBlock.awl` for standard data block structure
* Refer to `../example_blocks/UserDefinedDataType.awl` for UDT definition format

## Calling Syntax - FC vs FB
### Calling a Function (FC) - No Instance DB
```awl
NETWORK
TITLE = Utility calculation

CALL FC 5 (
  Value1 := MW100,
  Value2 := MW102,
  Result := MD200
);
```
* Stateless → no DB needed
* Parameters passed by value (input) / reference (output)
* Can be called from OB, FC, or FB

### Calling a Function Block (FB) - Requires Instance DB
```awl
NETWORK
TITLE = Control Motor 1

CALL FB 20, DB 20 (
  Start     := I0.0,
  Stop      := I0.1,
  ResetTime := M10.0,
  Running   := Q0.0,
  RunHours  := MD50.0
);
```
* Stateful → needs instance DB (DB20 in this example)
* Multiple instances possible: FB 20, DB 21, FB 20, DB 22, etc.
* Static variables (VAR section) are stored in the instance DB

### Conditional Call Example
```awl
A   M0.0          // Enable condition
JC  SkipCall

CALL FB 20, DB 20 (
  Start     := I1.0,
  Stop      := I1.1,
  Running   := Q1.0
);

SkipCall: NOP 0;
```

## Quick Rules for Calls
* Use CALL keyword (uppercase)
* Parameters in parentheses: #Formal := Actual
* Separate multiple parameters with commas
* FB call format: CALL FB n, DB m (...)
* FC call format: CALL FC n (...)
* No semicolon after parameter list (only after the whole CALL)
* Calls are blocking – code waits until called block finishes
* Respect data types when passing parameters
