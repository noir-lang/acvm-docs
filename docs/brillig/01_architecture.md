# Brillig Architecture

Design principles

---

The focus of Brillig is on simplicity, safety, correctness, and SNARK-proving efficiency.

## Finite Field VM

The Brillig VM operates over finite fields and supports up to a 128-bit integer representation. The finite field Brillig supports is generalizable and based upon the field ACIR supports (where [Arkworks](https://github.com/arkworks-rs/algebra/tree/master/ff) is used for the finite field interface). The ACIR fields currently supported are the prime fields of the BN254 curve and the BLs12-381 curve.

All integers ultimately translate to the finite field on which Brillig is based. For example, when a BinaryIntOp is used, the field is first cast to a fixed bitsize integer that can be accommodated within the field, prior to performing operations. Certain operations, such as ordered comparison or signed division, only make sense in the context of BinaryIntOp. The exact maximum integer value the field can store should not be relied upon; however, it is assured to be capable of packing 3 32-bit integers.

The decision to have Brillig operate over finite fields simplifies SNARK proving and optimizes for efficiency, as ultimately all data types translate to a finite field, which is the native data type for SNARKs.

## Bytecode Structure and Function

Brillig bytecode acts as an alternate compilation target for a domain-specific circuit language. It will be explored in more detail in another section of this document. The bytecode is available both standalone and as part of ACIR in the form of the Brillig opcode. It primarily operates on fixed register indices but also includes dedicated memory access operations. Designed as a minimal register machine, it supports conditional jumps, a lightweight callstack, and can access a flat memory array of field element cells, thereby fulfilling the core requirements of a low-level VM.

The `MAX_REGISTERS` is 2\*\*16.

## Interfacing with Provers

During execution, when a backend integrated with ACVM encounters the Brillig opcode, it is expected to trust the outputs of the opcode. In the ACIR context, the Brillig opcode is intended for unconstrained functions, hence the assumption of not needing to prove execution. However, the Brillig opcode might be processed by a prover backend for reasons outlined in the "Attributing incorrectness" section. The specifics of this interaction reflect the inherent trust requirements and complexity of operating within a blockchain environment. In this case, a zkVM approach would be needed to prove execution.

## Execution Model

The Brillig VM is a low-level register-based machine. It uses a program counter to step through its readonly bytecode data and consists of a callstack with just return addresses, in addition to register and memory spaces filled with field elements. Control flow operations are limited to a non-conditional jump, conditional jumps (checking if a register is zero), and a call operation that manipulates the callstack. There's also a foreign call instruction that allows any non-Brillig input to enter the system.

The VM accepts four initialization parameters.

1. An initial list of registers, represented by a list of field elements
2. The initial memory, represented by a list of field elements
3. The bytecode to be executed
4. A list of foreign call results. Each result is a list of field elements

The VM then processes each opcode according to their [specification](02_opcode_listing.md).

If the VM reaches a foreign call instruction, it will first fetch the call's input according to the register information inside the instruction. Through an internal counter and the foreign call results supplied to the VM, the VM will determine whether the outputs have been resolved. If they have not been resolved, the VM will then pause and return a status containing the foreign call inputs. The caller of the VM should interpret the call information returned and update the Brillig process. Execution can then continue as normal. While technically the foreign call result is considered part of the VM's input along with bytecode and initial registers, it is practically an input to the program.

Error and Exception Handling

---

Failed asserts, represented by the TRAP opcode, during the execution of an unconstrained function result in an error in the Brillig opcode, accompanied by data detailing the failure. In a hedged trust blockchain environment, a prover might still want to generate a 'valid' proof of an error result so that incorrectness can be correctly attributed. This emphasizes the importance of handling errors and exceptions within the context of a blockchain-based VM.

## Conclusion

The Brillig VM provides a flexible and efficient environment for executing unconstrained code in a circuit environment. By thoughtfully integrating with snark provers and blockchain technology and paying careful attention to error handling, Brillig fulfills a key role within the Noir programming ecosystem.
