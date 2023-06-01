# Intro to Brillig
BRILLIG - Bytecode Runtime Interface for Liberated Language Implementation... G

The G stands for whatever you like, like [Gabagool](https://en.wikipedia.org/wiki/Gabagool).

Noir can compile to bytecode for Brillig, a general VM architecture for use with Noir.
It is used for functions that are unconstrained, non-deterministic, or otherwise need a VM. 

Why we need Brillig
---

Noir is a zero-knowledge language that specializes in zero-knowledge proofs that flexibly can compile code to constraints. Despite its specialization, Noir still requires a general VM for the following reasons:

1) Unconstrained execution

In Noir, not all execution situations are fully constrained.
For example, multiple encrypted notes are a common way to represent single private values.
When notes need to be presented for a proof, the notes selected need to be verified, but the exact choice is not fully constrained. 
The note selection algorithm does not need a proof of execution, just the note outputs suffice to constrain the system.
We refer to these discretionary choice functions as unconstrained functions. 
```
// Say we need to reveal and nullify certain notes to pass this check
// How can we formalize this partially constrained choice?
if get_balance() > min_amount {
    // ... perform some action ...
}
```

Brillig provides a way to evaluate such unconstrained functions in a safe, general VM.

2) Attributing incorrectness

Brillig's bytecode is used when compiling Noir into a program commitment must differentiate between runtime errors and simply invalid proofs. This proves useful in environments that rely on hedged trust and are distributed, such as blockchains.

In such systems, sequenced proofs of shared public state have unique trust assumptions. Proofs of correctness are not constructed by the user/transactor, creating complications. Two scenarios must be distinguished when executing functions on shared public state:

1. When a user attempts to call a public function via a sequencer but inputs data that causes the function to revert and the contract to fail.
2. When a transaction is valid, but the prover deliberately assigns incorrect values to the witnesses, leading to a failing proof for a legitimate transaction.

In private functions, where the user is also the prover, distinguishing between these scenarios isn't necessary. However, for public functions, it becomes vital to separate them to accurately identify potential malicious actors in hedged trust scenarios. In the first scenario, the responsibility lies with the user, whereas, in the second scenario, it's with the prover.

To ensure the protocol always demands valid proofs of knowledge, a virtual machine (VM) becomes necessary. Here, a VM operates as a zero-knowledge circuit that interprets a program. The VM sequentially executes program instructions and utilizes a public failure flag. In case of VM execution reverting, the failure flag is set to one, and on success, it is set to zero. Regardless of the outcome, an honest user will always have a means to create a valid proof.

Design principles
--- 

The focus of Brillig is on simplicity, safety, correctness and SNARK proving efficiency. Proving time is expected to take the majority of any realistic Noir program. 
