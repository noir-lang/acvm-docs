# Brillig Example

For the following Noir:

```
fn main() {
    let mut a = 10;
    let mut b = 5;
    let mut c = 0;

    c = a + b;

    if c <= 15 {
        a = a * b;
    } else {
        b = a + b;
    }

    print(a, b, c);
}
```

One possible Brillig output would be:

```
[
    { // location = 0
        "Const": {
            "destination": 0,
            "value": { "inner": "10" }
        }
    },
    { // location = 1
        "Const": {
            "destination": 1,
            "value": { "inner": "5" }
        }
    },
    { // location = 2
        "Const": {
            "destination": 2,
            "value": { "inner": "0" }
        }
    },
    { // location = 3
        "Const": {
            "destination": 3,
            "value": { "inner": "15" }
        }
    },
    { // location = 4
        "BinaryIntOp": {
            "destination": 2,
            "op": "Add",
            "bit_size": 32,
            "lhs": 0,
            "rhs": 1
        }
    },
    { // location = 5
        "BinaryIntOp": {
            "destination": 4,
            "op": "LessThanEquals",
            "bit_size": 32,
            "lhs": 2,
            "rhs": 3
        }
    },
    { // location = 6
        "JumpIfNot": {
            "condition": 4,
            "location": 9
        }
    },
    { // location = 7
        "BinaryFieldOp": {
            "destination": 0,
            "op": "Multiply",
            "lhs": 0,
            "rhs": 1
        }
    },
    { // location = 8
        "Jump": {
            "location": 10
        }
    },
    { // location = 9
        "BinaryFieldOp": {
            "destination": 1,
            "op": "Add",
            "lhs": 0,
            "rhs": 1
        }
    },
    { // location = 10
        "ForeignCall": {
            "function": "print",
            "destination": [], // No output
            "input": [
                {"RegisterIndex": 0},
                {"RegisterIndex": 1},
                {"RegisterIndex": 2}
            ]
        }
    },
    { // location = 11
        "Stop": {}
    }
]
```

The execution and interpretation of the program would be as follows:

1. The three `Const` instructions load values into reg0, reg1, reg2, and reg3. These are variables a, b and c, and a temporary equal to 15
2. At location 4, let reg2 (c) equal reg0 plus reg1 (a + b)
3. Let reg4 (a temporary value) equal reg2 LessThanEquals reg3 (c <= 15)
4. If reg4 is 0 (so c > 15), jump to location 9, where we set b = a + b, then go to location 10. Otherwise, if c <= 15, we fall through to location 7, set a = a \* b, and then go to location 10.
5. At location 10, we queue up inputs to a foreign call from reg0, reg1, and reg2 (variables a, b, and c). This interrupts execution, calls to the outer system, and then returns to Brillig execution. If this had outputs, they might be written to registers and memory.
6. We finally reach the final location where we `Stop`. If this were a function to be called by another Brillig function, we would say `Return`.
