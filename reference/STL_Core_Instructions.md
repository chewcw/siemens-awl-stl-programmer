# Siemens S7 STL (AWL) Core Instruction Reference

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
      A     I0.0;      // AND
      AN    I0.1;      // AND NOT
      O     I0.2;      // OR
      ON    I0.3;      // OR NOT
      =     Q0.0;      // Assign RLO to operand
      S     M0.0;      // Set (latch)
      R     M0.0;      // Reset (unlatch)
```

### Execution Notes

* Logic flows left to right.
* `A` loads operand into RLO (or ANDs with existing RLO).
* `=` writes RLO into the specified operand.
* `S` and `R` are retentive — they hold state across scans.
* RLO is built sequentially within a network.

---

# 2. Edge Detection

Used for detecting signal transitions.

```
FP    // Rising edge (Flank Positive)
FN    // Falling edge (Flank Negative)
```

Example:

```awl
      A     I0.0;
      FP    M0.1;      // Rising edge — M0.1 stores previous state
      =     M0.2;      // M0.2 is true for one scan on rising edge
```

### Notes

* `FP` and `FN` require a dedicated memory bit to store the previous state.
* The memory bit must not be reused elsewhere.
* Used for pushbutton detection, pulse counting, event triggering.
* The result is true for exactly one scan cycle on the detected edge.

---

# 3. Timers

Two timer styles are available. Choose based on whether state must persist in an instance DB.

---

## 3a. S5 Timers (classic, for use in FC or standalone logic)

Time literal format: `S5T#3S`, `S5T#500MS`, `S5T#1M30S`

| Instruction | Meaning                        |
|-------------|-------------------------------|
| `SP`        | Pulse timer                   |
| `SE`        | On-delay (non-retentive)      |
| `SD`        | Retentive on-delay            |
| `SA`        | Off-delay                     |
| `SS`        | Retentive pulse timer         |

Example:

```awl
      A     I0.0;
      L     S5T#5S;
      SE    T1;         // Start on-delay timer T1 for 5 seconds

      A     T1;         // Check if timer has elapsed
      =     Q0.0;
```

### Notes

* Always precede the timer instruction with `A <condition>` and `L S5T#...`.
* Check timer status with `A T<n>` after the timer instruction.
* S5 timers are global — their state is not stored in an instance DB.

---

## 3b. IEC Timers — TON, TOF, TP (for use inside FB, stored in instance DB)

IEC timers are declared in the FB's `VAR` section as `TON`, `TOF`, or `TP` type. Their state is stored automatically in the instance DB.

| Type  | Meaning        |
|-------|---------------|
| `TON` | On-delay       |
| `TOF` | Off-delay      |
| `TP`  | Pulse timer    |

Accessing IEC timer members:

```
#TimerName.IN    // Input (enable signal)
#TimerName.PT    // Preset time
#TimerName.Q     // Output (elapsed flag)
#TimerName.ET    // Elapsed time
```

Example (inside FB):

```awl
NETWORK
TITLE = Start Delay Timer (3 seconds)
// Load preset and transfer to timer input — timer runs while RunLatch is set
      A     #RunLatch;
      L     S5T#3S;
      T     #StartDelay.PT;

NETWORK
TITLE = Motor Output After Delay
// Activate output once timer has elapsed
      A     #StartDelay.Q;
      =     #MotorOutput;
```

### When to use which:

* Use **S5 timers** (`SE`, `SP`, etc.) in FCs or when no instance DB is available.
* Use **IEC timers** (`TON`, `TOF`, `TP`) inside FBs where state is stored in the instance DB.

---

# 4. Counter Instructions

```awl
      L     C#10;      // Load preset value
      CU    C1;        // Count up on rising edge
      CD    C1;        // Count down on rising edge
      R     C1;        // Reset counter
      A     C1;        // Check if counter value > 0
```

| Instruction | Meaning                          |
|-------------|----------------------------------|
| `CU`        | Count up (on rising edge)        |
| `CD`        | Count down (on rising edge)      |
| `R`         | Reset counter to 0               |
| `A Cn`      | RLO = 1 if counter value > 0     |

### Notes

* Load preset with `L C#<value>` before the counter instruction.
* `CU` and `CD` increment/decrement on rising edge only.
* Counter value can be loaded into ACCU1 with `L C<n>`.

---

# 5. Arithmetic Instructions

### Integer Math

```awl
      L     MW10;
      L     MW12;
      +I;
      T     MW14;
```

| Instruction | Meaning          |
|-------------|-----------------|
| `+I`        | Add Integer      |
| `-I`        | Subtract Integer |
| `*I`        | Multiply Integer |
| `/I`        | Divide Integer   |

---

### Real Math

```awl
      L     MD10;
      L     MD14;
      +R;
      T     MD18;
```

| Instruction | Meaning       |
|-------------|--------------|
| `+R`        | Add Real      |
| `-R`        | Subtract Real |
| `*R`        | Multiply Real |
| `/R`        | Divide Real   |

---

# 6. Compare Instructions

Comparison result sets or clears the RLO, which can then drive `=`, `S`, `R`, or jump instructions.

### Integer Compare

```awl
      L     MW10;
      L     100;
      >I;
      =     Q0.1;
```

### Real Compare

```awl
      L     MD10;
      L     80.0;
      >R;
      =     M0.0;
```

| Instruction | Meaning              |
|-------------|---------------------|
| `>I`        | Greater than (INT)   |
| `<I`        | Less than (INT)      |
| `==I`       | Equal (INT)          |
| `>=I`       | Greater or equal (INT)|
| `<=I`       | Less or equal (INT)  |
| `<>I`       | Not equal (INT)      |
| `>R`        | Greater than (REAL)  |
| `<R`        | Less than (REAL)     |
| `==R`       | Equal (REAL)         |
| `>=R`       | Greater or equal (REAL)|
| `<=R`       | Less or equal (REAL) |
| `<>R`       | Not equal (REAL)     |

---

# 7. Data Movement

```awl
      L     MW10;      // Load MW10 into ACCU1
      T     MW12;      // Transfer ACCU1 to MW12
```

| Instruction | Meaning              |
|-------------|---------------------|
| `L`         | Load into ACCU1      |
| `T`         | Transfer ACCU1 to operand |
| `ITD`       | INT → DINT (widen)   |
| `DTR`       | DINT → REAL (convert)|

### Type Conversion Chain (INT → REAL)

```awl
      L     #IN;       // Load INT
      ITD;             // Convert INT to DINT
      DTR;             // Convert DINT to REAL
      T     #TempReal; // Store as REAL
```

---

# 8. Accumulator Model

STL uses two accumulators (ACCU1, ACCU2). Each `L` instruction shifts ACCU1 into ACCU2 and loads the new value into ACCU1. Arithmetic operates on ACCU1 and ACCU2, result goes into ACCU1.

Example:

```awl
      L     10;    // ACCU1 = 10
      L     20;    // ACCU2 = 10, ACCU1 = 20
      +I;          // ACCU1 = 30 (ACCU2 + ACCU1... wait, it's ACCU2 op ACCU1)
      T     MW0;   // MW0 = 30
```

### Accumulator arithmetic order:
- For `+I`, `-I`, `*I`, `/I`: result = ACCU2 op ACCU1
- For `-I`: result = ACCU2 - ACCU1 (order matters — load dividend/minuend first)
- For `/I`: result = ACCU2 / ACCU1 (load dividend first, then divisor)

---

# 9. RLO (Result of Logic Operation)

Boolean logic builds the RLO sequentially within a network.

Example:

```awl
      A     I0.0;      // RLO = I0.0
      AN    I0.1;      // RLO = RLO AND NOT I0.1
      =     Q0.0;      // Q0.0 = RLO
```

* RLO is reset to 1 at the start of each new boolean sequence.
* `=` reads RLO without clearing it — can assign to multiple outputs.
* `S` and `R` are conditional on RLO but do not clear it.

---

# 10. Jump Instructions

Used for conditional branching within a network or block. Labels are defined with `<label>: NOP 0;`.

| Instruction | Meaning                              |
|-------------|-------------------------------------|
| `JC`        | Jump if RLO = 1                     |
| `JCN`       | Jump if RLO = 0                     |
| `JU`        | Unconditional jump                  |
| `JZ`        | Jump if result = 0 (after arithmetic)|
| `JP`        | Jump if result > 0                  |
| `JN`        | Jump if result ≠ 0                  |

Example — conditional FC call:

```awl
      A     M0.0;         // Enable condition
      JCN   SkipCall;     // Skip if condition false

      CALL "My FC"
      (  Input1 := MW10 ,
         Output1 := MW12
      );

SkipCall: NOP 0;          // Jump target — no operation
```

### Notes

* Labels must be unique within the block.
* `NOP 0` is a no-operation instruction used as a jump target.
* Avoid overusing jumps — prefer structured network separation where possible.

---

# 11. CALL Instruction

Used to invoke FCs and FBs from OBs, FCs, or FBs.

| Syntax | Usage |
|--------|-------|
| `CALL "FC Name" ( ... )` | Call a Function (no instance DB) |
| `CALL "FB Name", "DB Name" ( ... )` | Call a Function Block with instance DB |

### FC Call Example

```awl
      CALL "Analog Scaling"
      (  IN                          := #RawAnalog ,
         HI_LIM                      := #HI ,
         LO_LIM                      := #LOW ,
         OUT                         := #OUTPUT
      );
```

### FB Call Example

```awl
      CALL "Motor Control", "Motor Control_DB"
      (  StartPB                     := #StartPB ,
         StopPB                      := #StopPB ,
         Overload                    := #Overload ,
         Temperature                 := #Temperature ,
         MotorOutput                 := #MotorOutput ,
         AlarmOutput                 := #AlarmOutput
      );
```

### Rules

* Use quoted symbolic names — not numeric references (`FC 5`, `FB 20`) unless required.
* Align `:=` assignments vertically using spaces.
* Each parameter on its own line; last parameter has no trailing comma.
* CALL is blocking — execution resumes only after the called block completes.
* FB calls require a dedicated instance DB — one DB per FB instance.

---

# 12. PLC Scan Model Assumption

Always assume:

1. Inputs are read at the start of each scan.
2. OB1 executes sequentially, network by network.
3. Outputs are written at the end of each scan.
4. Next scan begins immediately.

Do not generate event-driven code.
Do not assume parallel execution.

---

# 13. Output Constraints for AI

When generating STL:

* Use correct Siemens mnemonics — no invented instructions.
* Do not mix with Ladder (LAD) or Structured Control Language (SCL).
* Do not mix TIA Portal syntax (S7-1200/1500 style) unless explicitly requested.
* Avoid pseudocode.
* Respect accumulator load order for arithmetic (especially subtraction and division).
* Maintain deterministic execution order within each network.
* Keep networks logically separated — one purpose per network.
* Every instruction line ends with `;`.
* Block examples and formatting rules are defined in SKILL.md — refer there for complete templates.