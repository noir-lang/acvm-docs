# Opcode Listing
## BinaryFieldOp

This opcode carries out a binary operation on the contents of the `lhs` and `rhs` registers and stores the result in the `destination` register.

```yaml

BinaryFieldOp {
        destination: MemoryAddress,
        op: BinaryFieldOp,
        lhs: MemoryAddress,
        rhs: MemoryAddress,
    }
```


**Wire format example:** 

```json

{
  "BinaryFieldOp": {
    "destination": "Relative(3)",
    "op": "Add",
    "lhs": "Relative(1)",
    "rhs": "Relative(2)"
  }
}
```


## BinaryIntOp

This opcode carries out a binary operation on the `bit_size`-sized integers in the `lhs` and `rhs` registers and stores the result in the `destination` register.

```yaml

    BinaryIntOp {
        destination: MemoryAddress,
        op: BinaryIntOp,
        bit_size: IntegerBitSize,
        lhs: MemoryAddress,
        rhs: MemoryAddress,
    }
```



**Wire format example:** 

```json
{
  "BinaryIntOp": {
    "destination": "Relative(2)",
    "op": "LessThanEquals",
    "bit_size": "U32",
    "lhs": "Relative(1)",
    "rhs": "Relative(0)"
  }
}
```
## Not
  This opcode performs a bitwise NOT operation, of `bit_size` size, on the value in the `source` address and stores the result in the `destination`.

```yaml
    Not {
        destination: MemoryAddress,
        source: MemoryAddress,
        bit_size: IntegerBitSize,
    }
```

**Wire format example:** 

```json
{
  "Not": {
    "destination": "Relative(2)",
    "source": "Relative(1)",
    "bit_size": "Integer(U32)",
  }
}
```

## Cast
  This opcode casts (truncate) the value in the `source` address to `bit_size` bits and stores the result in the `destination`.

```yaml
    Cast {
        destination: MemoryAddress,
        source: MemoryAddress,
        bit_size: BitSize,
    }
```
**Wire format example:** 

```json
{
  "Cast": {
    "destination": "Direct(32836)",
    "source": "Direct(32836)",
    "bit_size": "Integer(U32)"
  }
}
```
## JumpIfNot

This opcode sets the program counter to the value of `location` if the value at `condition` register is zero.

```yaml

    JumpIfNot {
        condition: MemoryAddress,
        location: Label,
    }
```


**Wire format example:** 

```yaml

{
  "JumpIfNot": {
    "condition": Relative(4),
    "location": 10
  }
}
```


## JumpIf

This opcode sets the program counter to the value of `location` if the value at `condition` register is non-zero.

```yaml

JumpIf {
        condition: MemoryAddress,
        location: Label,
    }
```



**Wire format example:** 

```yaml

{
  "JumpIf": {
    "condition": Relative(1),
    "location": 10
  }
}
```


## Jump

This opcode sets the program counter to the value of `location`.

```css

Jump {
        location: Label,
    }
```



**Wire format example:** 

```yaml

{
  "Jump": {
    "location": 100
  }
}
```

## CallDataCopy
This opcode copies `size_address` bytes at the `offset_address` position in the calldata, into the `destination` address.

```yaml
  CalldataCopy {
        destination_address: MemoryAddress,
        size_address: MemoryAddress,
        offset_address: MemoryAddress,
  },
```

**Wire format example:** 

```yaml

{
  "CalldataCopy": {
    destination_address: Direct(32836),
    size_address: Relative(2),
    offset_address: Relative(3) 
  }
}
```

## Call

This opcode sets the program counter to the value of `location` and pushes the next address to the call stack.

```yaml

Call {
        location: Label,
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
        destination: MemoryAddress,
        bit_size: BitSize,
        value: F,
    }
```


**Wire format example:** 

```yaml

{
  "Const": {
    destination: Direct(10),
    bit_size: Integer(U32)
    value: 32838
  }
}
```
## Indirect Const
This opcode stores the `value` at the address pointed inside `destination_pointer`.
``` yaml
  IndirectConst {
        destination_pointer: MemoryAddress,
        bit_size: BitSize,
        value: F,
  }
```
**Wire format example:** 

```yaml

{
  "IndirectConst": {
    destination: Direct(10),
    bit_size: Integer(U64)
    value: 17843811134343075018
  }
}
```
## Return

This opcode sets the program counter to the value at the top of the call stack, removing the top element from the call stack.

These should generally be emitted when a Brillig function wants to exit, unless the Brillig function is the entry-point. A Brillig entry-point should end with a Stop operation.

From a static analysis point of view, Return is the only opcode that can jump to a non-constant program instruction. It aims to be enough to make recursive function calls without having the undesirable effects of arbitrary jumps (see for example discussions such as https://github.com/ethereum/aleth/issues/3404). This ensures that Brillig bytecode can be statically analyzed without a path-narrowing data analysis.


```yaml

Return
```



**Wire format example:** 

```json

"Return"
```

- ## ForeignCall

This opcode is used to get data from an external source. It works similarly to a low-level syscall in C-like programming architectures, interfacing with the outer system in a way that interrupts the program and updates its memory or registers. From the semantics of execution and proving, it is as if we had an input that we stored that was simply copied when we hit the foreign call. A simulation pass may indeed store these results, and a later prover simply would consider the opcode as a copy of these results to memory or registers.

ForeignCall is used whenever the outer system would better handle data, or in the case of a VM, possibly a constraint unknown to Brillig.
It is meant to be used generally for any case that an external system needs to interleave with a Noir program. This varies from printing, to setting data in a database. It requires a function name that is interpreted by the simulator context, a memory address for the input data and the output data, to store the result, a. It can reference either array/vector (i.e. pointers with a length) or a value. 

```yaml

ForeignCall {
    function: "print",
    destination: [],
    destination_value_types: [],
    input: [MemoryAddress(Relative(1)), HeapArray(HeapArray { pointer: Relative(4), size: 16 }), HeapArray(HeapArray { pointer: Relative(5), size: 72 }), MemoryAddress(Relative(3))],
    input_value_types: [Simple(Integer(U1)), Array { value_types: [Simple(Integer(U8))], size: 16 }, Array { value_types: [Simple(Integer(U8))], size: 72 }, Simple(Integer(U1))]
}
```


**Wire format example:** 
Using value registers
```json
{
  "ForeignCall": {
    "function": "read_contract_tree",
    "destination": [{
      "RegisterIndex": 1
    }],
    "input": [{
      "RegisterIndex": 0
    }]
  }
}
```
or, using memory pointer registers

```json
{
  "ForeignCall": {
    "function": "matrix_2x2_transpose",
    "destination": [{
      "HeapArray": [
        1,
        4
      ]
    }],
    "input": [{
      "HeapArray": [
        0,
        4
      ]
    }]
  }
}
```

## Mov

The `Mov` opcode is used to move the content in the `source` address to the `destination` address.

```yaml

Mov {
    destination: MemoryAddress,
    source: MemoryAddress,
}
```

**Wire format example:** 

```yaml

{
  "Mov": {
    "destination": Direct(2),
    "source": Direct(1)
  }
}
```


## Conditional Mov

The `Conditional Mov` opcode is used to move the content in the `source_a` or `source_b` address to the `destination` address, depending on the value of the `condition` value.

```yaml

Mov {
    destination: MemoryAddress,
    source_a: MemoryAddress,
    source_b: MemoryAddress,
    condition: MemoryAddress,
}
```

**Wire format example:** 

```yaml

{
  "ConditionalMov": {
    "destination": Direct(4),
    source_a: Direct(1),
    source_b: Direct(2),
    condition: Direct(3)
  }
}
```

## Load

The `Load` opcode retrieves data from memory at the address specified by the source_pointer and stores it in the destination register.

```yaml

Load {
    destination: MemoryAddress,
    source_pointer: MemoryAddress,
}
```



**Wire format example:** 

```yaml

{
  "Load": {
    "destination": Relative(1),
    "source_pointer": Relative(2)
  }
}
```


## Store

The `Store` opcode takes the content of the source register and stores it in the memory location specified by the destination_pointer.

```yaml

Store {
    destination_pointer: MemoryAddress,
    source: MemoryAddress
}
```



**Wire format example:** 

```yaml

{
  "Store": {
    "destination_pointer": Relative(2),
    "source": Relative(3)
  }
}
```

## BlackBox

BlackBox opcodes are used to call some functions that are natively executed by the Brillig VM. They correspond to ACIR blackbox functions.

```yaml
    BlackBox(BlackBoxOp),
```

The BlackBoxOp varies depending on the operation.  For instance Keccakf1600 will perform Keccak permutation.
```yaml
Keccakf1600 { input: HeapArray, output: HeapArray },
```


**Wire format example:** 

```yaml

{
  BlackBox(AES128Encrypt { 
    inputs: HeapVector { pointer: Relative(5), size: Relative(6) },
    iv: HeapArray { pointer: Relative(7), size: 16 },
    key: HeapArray { pointer: Relative(8), size: 16 },
    outputs: HeapVector { pointer: Relative(9), size: Relative(10) }
  })
}
```

## Trap

The `Trap` opcode is used to interrupt the normal flow of execution if a runtime error occurs.
It returns the data specified in the `revert_data` vector.



```yaml

"Trap": {
        revert_data: HeapVector,
    }
```

**Wire format example:** 

```yaml

{
  "Trap": {
    revert_data: HeapVector { pointer: Direct(1), size: Direct(2) } 
  }
}
```

## Stop

The `Stop` opcode is used to halt the execution of the program. It returns the data specified in the `return_data` vector.

It should be used to exit from the outer Brillig program, while most of the Brillig program should use the "Return" opcode. 
There is no particular requirement on a Brillig program about what states they are allowed to "Stop" in, however, 
any system using Brillig may impose additional semantics on the final Brilig memory and registers. 
In this case, a bytecode emitter should make sure that the calling convention of the Brillig program is followed before Stop'ing.

**Wire format example:** 

```yaml

"Stop": {
        return_data: HeapVector { pointer: Relative(2), size: Relative(3) } ,
    }
```
