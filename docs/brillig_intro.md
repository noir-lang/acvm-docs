BRILLIG - Bytecode Runtime Interface for Liberated Language Implementation... G

The G stands for whatever you like, like [Gabagool](https://en.wikipedia.org/wiki/Gabagool).

Brillig is a general VM architecture for use with Noir.
It is used for functions that are unconstrained, non-deterministic, or otherwise need a VM. 

Why we need Brillig

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

The Brillig bytecode is used when compiling Noir into a program commitment over such a virtual machine circuit that can differentiate errors from incorrect proofs. This is applicable to hedged trust, distributed environments like blockchains.

In such systems, sequenced proofs of shared public state have unique trust assumptions. Proofs of correctness are not constructed by the user/transactor, creating complications. Two scenarios must be distinguished when executing functions on shared public state:

1. A user wants to call a public function through a sequencer but provides inputs that cause the function to revert and the contract to fail.
2. A transaction is valid, but the prover intentionally assigns incorrect values to the witnesses and creates a failing proof for a legitimate transaction.

For private functions, the user is the prover, so we don't need to distinguish between the two scenarios. However, for public functions, it is crucial to differentiate between them to correctly identify potential bad actors in hedged trust scenarios. The responsibility lies with the user in scenario one and with the prover in scenario two.

A virtual machine (VM) is needed to ensure that the protocol always requires valid proofs of knowledge. In this context, a VM is a zero-knowledge circuit that interprets a program. The VM executes program instructions sequentially and uses a public failure flag. If the VM execution reverts, the failure flag is set to one, and if it succeeds, the flag is set to zero. Regardless of the outcome, an honest user will always have a way to create a valid proof.

Design principles
TODO move to footnote
The focus of Brillig is on safety, correctness and SNARK proving efficiency. Proving time is expected to take the majority of any realistic Noir program. Future optimization work for pure execution (e.g., transpiling to WASM) can be considered in the future. However, for this to be justified, there would first need to be use cases that are both complicated AND have a constraint-solving cost that is smaller than the running time. This is considered unlikely/niche.
