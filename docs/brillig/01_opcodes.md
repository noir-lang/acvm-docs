# Opcode Specifications
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
    "op": {
      "Cmp": "Lte"
    },
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

This opcode sets the program counter to the value of `location` if the value at `condition` register is non-zero.

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
        location: Label,
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

```markdown

Return
```



**Wire format example:** 

```json

"Return"
```


## ForeignCall

This opcode executes a foreign function defined by `function`, stores the result in the `destination` register or memory, and passes `input` as arguments to the function.

```markdown

ForeignCall
```

- ## ForeignCall

This opcode is used to get data from an external source. It requires a function name that is interpreted by the simulator context, a destination register to store the result, and an input register containing the input data. These can be either memory registers (i.e. pointers with a length) or value registers.

```yaml

ForeignCall {
    function: String,
    destination: RegisterValueOrArray,
    input: RegisterValueOrArray,
}
```



**Wire format example:** 
Using value registers
```json
{
  "ForeignCall": {
    "function": "read_contract_tree",
    "destination": {
      "RegisterIndex": 1
    },
    "input": {
      "RegisterIndex": 0
    }
  }
}
```
or, using memory pointer registers

```json
{
  "ForeignCall": {
    "function": "matrix_2x2_transpose",
    "destination": {
      "HeapArray": [
        1,
        4
      ]
    },
    "input": {
      "HeapArray": [
        0,
        4
      ]
    }
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

**Wire format example:** 

```json

"Stop"
```
