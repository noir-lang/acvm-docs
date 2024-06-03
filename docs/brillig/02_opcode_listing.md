# Opcode Listing

## BinaryFieldOp

This opcode carries out a binary operation on the contents of the `lhs` and `rhs` registers and stores the result in the `destination` register.

```yaml

BinaryFieldOp {
        destination: RegisterIndex,
        op: BinaryFieldOp,
        lhs: RegisterIndex,
        rhs: RegisterIndex,
    }
```

**Wire format example:**

```json
{
  "BinaryFieldOp": {
    "destination": 0,
    "op": "Add",
    "lhs": 1,
    "rhs": 2
  }
}
```

## BinaryIntOp

This opcode carries out a binary operation on the `bit_size`-sized integers in the `lhs` and `rhs` registers and stores the result in the `destination` register.

```yaml

BinaryIntOp {
        destination: RegisterIndex,
        op: BinaryIntOp,
        bit_size: u32,
        lhs: RegisterIndex,
        rhs: RegisterIndex,
    }
```

**Wire format example:**

```json
{
  "BinaryIntOp": {
    "destination": 2,
    "op": "LessThanEquals",
    "bit_size": 32,
    "lhs": 1,
    "rhs": 0
  }
}
```

## JumpIfNot

This opcode sets the program counter to the value of `location` if the value at `condition` register is zero.

```yaml

JumpIfNot {
    condition: RegisterIndex,
    location: Label,
}
```

**Wire format example:**

```json
{
  "JumpIfNot": {
    "condition": 3,
    "location": 10
  }
}
```

## JumpIf

This opcode sets the program counter to the value of `location` if the value at the `condition` register is non-zero.

```yaml

JumpIf {
        condition: RegisterIndex,
        location: Label,
    }
```

**Wire format example:**

```json
{
  "JumpIf": {
    "condition": 2,
    "location": 10
  }
}
```

## Jump

This opcode sets the program counter to the value of `location`.

```css
Jump {
  location: Label;
}
```

**Wire format example:**

```json
{
  "Jump": {
    "location": 100
  }
}
```

## Call

This opcode sets the program counter to the value of `location` and pushes the next address to the call stack.

```css
Call {
  location: Label;
}
```

**Wire format example:**

```json
{
  "Call": {
    "location": 4
  }
}
```

## Const

This opcode stores a `value` in the `destination` register.

```yaml

Const {
        destination: RegisterIndex,
        value: Value,
    }
```

**Wire format example:**

```json
{
  "Const": {
    "destination": 0,
    "value": {
      "inner": "0000000000000000000000000000000000000000000000000000000000000000"
    }
  }
}
```

## Return

This opcode sets the program counter to the value at the top of the call stack, removing the top element from the call stack.

These should generally be emitted when a Brillig function wants to exit, unless the Brillig function is the entry-point. A Brillig entry-point should end with a Stop operation.

From a static analysis point of view, Return is the only opcode that can jump to a non-constant program instruction. It aims to be enough to make recursive function calls without having the undesirable effects of arbitrary jumps (see for example discussions such as https://github.com/ethereum/aleth/issues/3404). This ensures that Brillig bytecode can be statically analyzed without a path-narrowing data analysis.

```markdown
Return
```

**Wire format example:**

```json
"Return"
```

- ## ForeignCall

This opcode is used to get data from an external source. It works similarly to a low-level syscall in C-like programming architectures, interfacing with the outer system in a way that interrupts the program and updates its memory or registers. From the semantics of execution and proving, it is as if we had an input that we stored that was simply copied when we hit the foreign call. A simulation pass may indeed store these results, and a later prover would simply consider the opcode as a copy of these results to memory or registers.

ForeignCall is used whenever the outer system would better handle data, or in the case of a VM, possibly a constraint unknown to Brillig.

It is meant to be used generally in any case where an external system needs to interleave with a Noir program. This varies from printing to setting data in a database. It requires a function name that is interpreted by the simulator context, a destination register to store the result, and an input register containing the input data. These can be either memory registers (i.e., pointers with a length) or value registers.

```yaml

ForeignCall {
    function: String,
    destination: Vec<RegisterValueOrArray>,
    input: Vec<RegisterValueOrArray>,
}
```

**Wire format example:**
Using value registers

```json
{
  "ForeignCall": {
    "function": "read_contract_tree",
    "destination": [
      {
        "RegisterIndex": 1
      }
    ],
    "input": [
      {
        "RegisterIndex": 0
      }
    ]
  }
}
```

or, using memory pointer registers

```json
{
  "ForeignCall": {
    "function": "matrix_2x2_transpose",
    "destination": [
      {
        "HeapArray": [1, 4]
      }
    ],
    "input": [
      {
        "HeapArray": [0, 4]
      }
    ]
  }
}
```

## Mov

The `Mov` opcode is used to move the content of the source register to the destination register.

```yaml

Mov {
    destination: RegisterIndex,
    source: RegisterIndex,
}
```

**Wire format example:**

```json
{
  "Mov": {
    "destination": 0,
    "source": 1
  }
}
```

## Load

The `Load` opcode retrieves data from memory at the address specified by the source_pointer and stores it in the destination register.

```yaml

Load {
    destination: RegisterIndex,
    source_pointer: RegisterIndex,
}
```

**Wire format example:**

```json
{
  "Load": {
    "destination": 0,
    "source_pointer": 1
  }
}
```

## Store

The `Store` opcode takes the content of the source register and stores it in the memory location specified by the destination_pointer.

```yaml

Store {
    destination_pointer: RegisterIndex,
    source: RegisterIndex,
}
```

**Wire format example:**

```json
{
  "Store": {
    "destination_pointer": 0,
    "source": 1
  }
}
```

## Trap

The `Trap` opcode is used to interrupt the normal flow of execution if a runtime error occurs. It does not require any parameters.

**Wire format example:**

```json
"Trap"
```

## Stop

The `Stop` opcode is used to halt the execution of the program. It does not require any parameters.

It should be used to exit from the outer Brillig program, while most of the Brillig program should use the "Return" opcode.

There is no particular requirement on a Brillig program about what states they are allowed to "Stop" in, however,

any system using Brillig may impose additional semantics on the final Brillig memory and registers.

In this case, a bytecode emitter should make sure that the calling convention of the Brillig program is followed before Stop'ing.

**Wire format example:**

```json
"Stop"
```
