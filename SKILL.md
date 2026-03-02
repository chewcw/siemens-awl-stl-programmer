---
name: siemens-awl-stl-programmer
description: Generate Siemens S7-300/S7-400 Statement List (STL/AWL) programs compatible with Step7 Classic. Use this skill whenever the user asks to write, generate, create, or modify any PLC logic, function blocks, organization blocks, data blocks, or control programs in STL/AWL format. Also trigger for any questions about Siemens S7 memory addressing, timers, counters, accumulators, or block structure. Always use this skill even if the user just mentions "STL", "AWL", "Step7", "S7-300", "S7-400", "PLC block", "OB", "FB", "FC", or "DB".
---

# Siemens S7 STL (AWL) Programming Skill

---

## Overview

Generate **Siemens S7-300 / S7-400 Statement List (STL / AWL)** programs compatible with **Step7 Classic**.

Core model knowledge:
- Accumulator-based execution (ACCU1 / ACCU2)
- RLO (Result of Logic Operation) for boolean logic
- S5 timers, counters, INT and REAL arithmetic
- OB / FB / FC / DB block hierarchy
- Standard scan cycle model (Read → Execute → Write)

For full instruction syntax rules, refer to `reference/STL_Core_Instructions.md`.

---

## CRITICAL: Always Match These Exact Block Styles

The examples below are the **required formatting reference**. Every block you generate must follow these patterns exactly — indentation, semicolons, comment style, CALL syntax, VAR sections, and NETWORK/TITLE structure.

---

### Organization Block (OB)

```
ORGANIZATION_BLOCK "OB1"
TITLE = Main Program Cycle
{ S7_Optimized_Access := 'TRUE' }
VERSION : 0.1

VAR_TEMP
      RawAnalog : Int;
      ScaledValue : Real;
      HI : Real;
      LOW : Real;
      OUTPUT : Real;
      StartPB : Bool;
      StopPB : Bool;
      Overload : Bool;
      Temperature : Real;
      MotorOutput : Bool;
      AlarmOutput : Bool;
END_VAR

BEGIN

NETWORK
TITLE = Read Analog Input
// Read raw analog value from peripheral input word
      L     PIW256;            // Peripheral Input Word
      T     #RawAnalog;

NETWORK
TITLE = Scale analog value
// Call FC to scale raw analog value to engineering units
      CALL "Analog Scaling"
      (  IN                          := #RawAnalog ,
         HI_LIM                      := #HI ,
         LO_LIM                      := #LOW ,
         OUT                         := #OUTPUT
      );

NETWORK
TITLE = Call Function Block with Scaled Value
// Call FB with inputs and map outputs to peripherals
      CALL "Motor Control", "Motor Control_DB"
      (  StartPB                     := #StartPB ,
         StopPB                      := #StopPB ,
         Overload                    := #Overload ,
         Temperature                 := #Temperature ,
         MotorOutput                 := #MotorOutput ,
         AlarmOutput                 := #AlarmOutput
      );

END_ORGANIZATION_BLOCK
```

---

### Function (FC) — Stateless

```
FUNCTION "Analog Scaling" : Void
{ S7_Optimized_Access := 'TRUE' }
NAME : '1'
VERSION : 0.1
   VAR_INPUT
      IN : Int;
      HI_LIM : Real;
      LO_LIM : Real;
   END_VAR

   VAR_OUTPUT
      OUT : Real;
   END_VAR

   VAR_TEMP
      TempReal : Real;
   END_VAR


BEGIN
NETWORK
TITLE = Convert INT to REAL
//Convert integer input to real for arithmetic
      L #IN;
      ITD;
      DTR;
      T #TempReal;

NETWORK
TITLE = Normalize to 0-27648
//Divide by full-scale analog range
      L #TempReal;
      L 27648.0;
      /R;
      T #TempReal;

NETWORK
TITLE = Scale to HI_LIM-LO_LIM
//Apply engineering unit scaling
      L #HI_LIM;
      L #LO_LIM;
      -R;
      L #TempReal;
      *R;
      L #LO_LIM;
      +R;
      T #OUT;

END_FUNCTION
```

---

### Function Block (FB) — Stateful

```
FUNCTION_BLOCK "Motor Control"
{ S7_Optimized_Access := 'TRUE' }
NAME : '1'
VERSION : 0.1

VAR_INPUT
    StartPB      : BOOL ;
    StopPB       : BOOL ;
    Overload     : BOOL ;
    Temperature  : REAL ;
END_VAR

VAR_OUTPUT
    MotorOutput  : BOOL ;
    AlarmOutput  : BOOL ;
END_VAR

VAR
    StartEdge    : BOOL ;
    RunLatch     : BOOL ;
    HighTemp     : BOOL ;
    StartDelay   : TON ;
END_VAR

BEGIN

NETWORK
TITLE = Rising Edge Detection for Start
// Detect rising edge of StartPB to initiate start sequence
      A     #StartPB;
      FP    #StartEdge;
      =     #StartEdge;

NETWORK
TITLE = Start Latch Logic
// Motor starts if StartPB pressed, no stop signal, no overload
      A     #StartEdge;
      AN    #StopPB;
      AN    #Overload;
      S     #RunLatch;

NETWORK
TITLE = Stop and Overload Logic
// Motor stops on StopPB or overload condition
      A     #StopPB;
      R     #RunLatch;

      A     #Overload;
      R     #RunLatch;

NETWORK
TITLE = Start Delay Timer (3 seconds)
// Introduce delay before motor output activates
      A     #RunLatch;
      L     S5T#3S;
      T    #StartDelay.PT;

NETWORK
TITLE = Motor Output Logic
// Activate motor output after start delay elapses
      A     #StartDelay.Q;
      =     #MotorOutput;

NETWORK
TITLE = High Temperature Alarm (>80 deg)
// Set HighTemp flag if temperature exceeds 80 degrees
      L     #Temperature;
      L     80.0;
      >R;
      =     #HighTemp;

NETWORK
TITLE = Alarm Logic
// Activate alarm on overload or high temperature
      A     #HighTemp;
      O     #Overload;
      =     #AlarmOutput;

END_FUNCTION_BLOCK
```

---

### FB Instance Data Block

```
DATA_BLOCK "Motor Control_DB"
{ S7_Optimized_Access := 'TRUE' }
NAME : '1'
VERSION : 0.1
NON_RETAIN
"Motor Control"

BEGIN

END_DATA_BLOCK
```

---

### Global Data Block (DB)

```
DATA_BLOCK "Data_block_1"
{ S7_Optimized_Access := 'TRUE' }
NAME : '1'
VERSION : 0.1
NON_RETAIN
   VAR
      TagName1 : Bool;
      TagName2 : Byte;
      Motor1 { S7_SetPoint := 'False'} : "UDT_MOTOR";
   END_VAR

BEGIN

END_DATA_BLOCK
```

---

### User-Defined Data Type (UDT)

```
TYPE "UDT_MOTOR"
VERSION : 0.1
   STRUCT
      RUN_FEEDBACK : Bool;
      SPEED : Real;
   END_STRUCT;

END_TYPE
```

---

## Formatting Rules (Non-Negotiable)

These rules are derived from the example blocks above. Follow them without exception.

**Block header:**
- Use quoted symbolic names: `FUNCTION_BLOCK "Motor Control"` — not numeric references like `FB 20`
- Always include `{ S7_Optimized_Access := 'TRUE' }` on the line immediately after the block keyword line
- Always include `VERSION : 0.1`

**VAR sections:**
- Each variable on its own line ending with `;`
- Align variable names and data types for readability
- Data types in UPPERCASE: BOOL, INT, REAL, BYTE, TON, etc.

**Networks:**
- Every logical group gets its own `NETWORK` + `TITLE = ...` header
- Add a `// comment` line directly below TITLE to describe the network's purpose
- Instructions are indented with 6 spaces
- Every instruction line ends with `;`
- Leave a blank line between NETWORKs

**CALL syntax:**
- FC call: `CALL "FC Name"` with parameters in aligned `( param := value ,` format
- FB call: `CALL "FB Name", "Instance DB Name"` with same parameter format
- Align `:=` using spaces so all assignments line up vertically
- Each parameter on its own line; last parameter has no trailing comma

**Comments:**
- Use `//` for inline and network description comments
- Do not use `(*` ... `*)` block comment style

**Semicolons:**
- Every instruction line ends with `;`
- Every VAR declaration ends with `;`
- Every STRUCT member ends with `;`

---

## Code Generation Constraints

MUST:
- Use symbolic quoted names (e.g., `"Motor Control"`) for all block references
- Generate complete, compilable blocks — no stubs or placeholders
- Separate every logical group into its own NETWORK with TITLE and comment
- Use uppercase STL mnemonics (A, AN, O, =, S, R, L, T, FP, FN, ITD, DTR, etc.)
- Prefix all local/static variables with `#` (e.g., `#StartPB`, `#RunLatch`)
- Use S5 time format for timers (e.g., `S5T#3S`, `S5T#500MS`)
- Always generate the matching instance DB when generating an FB

MUST NOT:
- Mix LAD, SCL, or TIA Portal syntax
- Use numeric-only block references (FB 20, FC 5) unless the user explicitly requires them
- Invent unsupported mnemonics
- Write pseudocode or placeholder logic
- Assume S7-1200/1500 features unless explicitly requested

---

## Block Hierarchy

```
OB (organization block — main scan cycle entry point)
 ├── CALL "FC Name"              → stateless logic, no instance DB
 └── CALL "FB Name", "DB Name"  → stateful logic, requires instance DB
```

Global DBs and UDTs are standalone — defined independently and referenced by name.

---

## Industrial Context

Generated examples must resemble real industrial use cases:
- Motor start/stop with interlocks and fault latching
- Alarm conditions (temperature, overload, pressure)
- Timer-based delays and sequencing
- Analog input scaling (0–27648 raw to engineering units)
- Counter-based batch control
- Conveyor, pump, or valve control

Avoid toy, abstract, or academic examples.