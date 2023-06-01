# Brillig Architecture


Finite Field VM
---
The Brillig VM operates on finite fields using a 128 bit integer representation, matching the native representation in Noir and its backends. This decision simplifies SNARK proving and optimizes for efficiency. The exact maximum integer value the field can store should not be relied upon, however, it is assured to be capable of packing 3 32-bit integers. When a BinaryIntOp is used, the field is first cast to a fixed bitsize integer that can be accommodated within the field, prior to performing operations. Certain operations, such as ordered comparison or signed division, only make sense in the context of BinaryIntOp.

Bytecode Structure and Function
---
The bytecode, an alternate compilation target of Noir code, is explored in more detail in another section of this document. The bytecode is available both standalone and as part of ACIR in the form of the Brillig opcode. It primarily operates on fixed register indices but also includes dedicated memory access operations. Designed as a minimal register machine, it supports conditional jumps, a lightweight callstack, and can access a flat memory array of field element cells, thereby fulfilling the core requirements of a low-level VM.

Interfacing with Provers
---
During execution, when a normal Noir backend encounters the Brillig opcode, it is expected to trust the outputs of the opcode. In the ACIR context, the Brillig opcode is intended for unconstrained functions, hence the assumption of not needing to prove execution. However, the Brillig opcode might be processed by a prover backend for reasons outlined in the "Attributing incorrectness" section. The specifics of this interaction reflect the inherent trust requirements and complexity of operating within a blockchain environment. In this case, a zkVM approach would be needed to prove execution.

Execution Model
---
The Brillig VM is a low-level register-based machine. It uses a program counter to step through its readonly bytecode data and consists of a callstack with just return addresses, in addition to register and memory spaces filled with field elements. Control flow operations are limited to a non-conditional jump, conditional jumps (checking if a register is zero), and a call operation that manipulates the callstack. There's also a foreign call instruction that allows any non-Noir input to enter the system. While technically the foreign call result is considered part of the VM's input along with bytecode and initial registers, it is practically an input to the program. Running the VM in real-world scenarios will cause it to pause, at which point the foreign call result can be provided, and execution can continue.

Error and Exception Handling
--- 
Failed asserts, represented by the TRAP opcode, during the execution of an unconstrained function, result in an error in the Brillig opcode, accompanied by data detailing the failure. In a hedged trust blockchain environment, a prover might still want to generate a 'valid' proof of an error result so that incorrectness can be correctly attributed. This emphasizes the importance of handling errors and exceptions within the context of a blockchain-based VM.

Conclusion
---
In summary, the Brillig VM provides a flexible and efficient environment for executing Noir code, especially in unconstrained situations. By thoughtfully integrating with snark provers and blockchain technology, and with careful attention to error handling, Brillig fulfills a key role within the Noir programming ecosystem.