# Brillig Example

For the following Noir:

```
fn main() {
    let mut a = 10;
    let mut b = 5;
    let mut c = 0;
    
    c = a + b;
    
    if c <= 15 {
        a = a * 2;
    } else {
        b = b * 2;
    }
    
    print(a, b, c);
}
```

We may output the following Brillig:

```
[
    {
        "Const": {
            "destination": 0,
            "value": { "inner": "10" }
        }
    },
    {
        "Const": {
            "destination": 1,
            "value": { "inner": "5" }
        }
    },
    {
        "Const": {
            "destination": 2,
            "value": { "inner": "0" }
        }
    },
    {
        "BinaryFieldOp": {
            "destination": 2,
            "op": "Add",
            "lhs": 0,
            "rhs": 1
        }
    },
    {
        "BinaryIntOp": {
            "destination": 3,
            "op": "LessThanEquals",
            "bit_size": 32,
            "lhs": 2,
            "rhs": 4
        }
    },
    {
        "JumpIfNot": {
            "condition": 3,
            "location": 8
        }
    },
    {
        "BinaryFieldOp": {
            "destination": 0,
            "op": "Multiply",
            "lhs": 0,
            "rhs": 5
        }
    },
    {
        "Jump": {
            "location": 10
        }
    },
    {
        "BinaryFieldOp": {
            "destination": 1,
            "op": "Multiply",
            "lhs": 1,
            "rhs": 5
        }
    },
    {
        "ForeignCall": {
            "function": "print",
            "destination": {
                "RegisterIndex": 6 // Arbitrary, not used
            },
            "input": {
                "RegisterIndex": 0,
                "RegisterIndex": 1,
                "RegisterIndex": 2
            }
        }
    },
    {
        "Stop": {}
    }
]
```