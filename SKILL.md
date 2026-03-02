---
name: siemens-awl-stl-programmer
description: Guide for generating Siemens S7-300 / S7-400 Statement List (STL / AWL) programs compatible with Step7 Classic. Use when creating PLC logic in STL format, ensuring valid syntax and adherence to Siemens programming conventions.
---

## Siemens S7 STL (AWL) Programming

---

## Overview

This skill is about generating **Siemens S7-300 / S7-400 Statement List (STL / AWL)** programs compatible with **Step7 Classic**.

The agent understands:

* Accumulator-based execution model (ACCU1 / ACCU2)
* RLO (Result of Logic Operation)
* Boolean logic flow
* S5 timers
* Counters
* Arithmetic (INT and REAL)
* Compare operations
* OB / FB / FC structure
* Scan-cycle execution model

All generated code must be valid Siemens STL syntax.

---

## File References

When generating complete programs, refer to `reference/STL_Core_Instructions.md` for instruction syntax rules

If generating a full project, maintain this hierarchy:

```
OB1
 ├── FC (stateless logic)
 ├── FB (stateful logic) with instance DB
 └── DB (data block)
```

---

## Code Generation Constraints

The AI must:

* Use uppercase STL mnemonics
* Avoid pseudocode
* Avoid mixing with LAD or SCL
* Maintain deterministic order
* Separate logic using NETWORK blocks
* Use valid Siemens memory addressing (I, Q, M, DB, MW, MD, etc.)

The AI must NOT:

* Invent unsupported mnemonics
* Mix TIA Portal SCL syntax
* Assume S7-1200/1500 features unless requested
* Generate non-accumulator-style arithmetic

---

## Industrial Context Assumption

Generated examples should resemble:

* Motor start/stop logic
* Interlocks
* Alarm conditions
* Timer-based delays
* Analog scaling
* Counter-based batching
* Conveyor or pump control

Avoid toy or abstract programming examples.

---

## Expansion Capability

This skill may be extended to support:

* Indirect addressing (AR1 / AR2)
* FIFO implementation
* State machine in pure STL
* Communication handling
* Migration comparison (STL vs LAD vs SCL)
* TIA Portal compatibility differences
