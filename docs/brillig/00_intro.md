# Intro to Brillig

BRILLIG - Bytecode Runtime Interface for Liberated Language Implementation... G

The G stands for whatever you like, like [Gabagool](https://en.wikipedia.org/wiki/Gabagool).

Brillig is a general virtual machine architecture for usage with an NP complete circuit language that aims to incorporate unconstrained or non-deterministic functionality. For example, a language which compiles down to ACIR can integrate unconstrained functions into its circuits by also compiling down to Brillig bytecode.

## Why we need Brillig

Zero-knowledge (ZK) domain-specific languages (DSL) enable developers to generate ZK proofs from their programs by compiling code down to the constraints of an NP-complete language (such as R1CS or PLONKish languages). However, the hard bounds of a constraint system can be very limiting to the functionality of a ZK DSL, and integrating a general VM is very useful for the following reasons:

1. Unconstrained execution

Enabling a circuit language to perform unconstrained execution is a powerful tool. Said another way, unconstrained execution lets developers generate witnesses from code that does not generate any constraints. Being able to execute logic outside of a circuit is critical for both circuit performance and constructing proofs on information that is external to a circuit.

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

Fetching information from somewhere external to a circuit can also be used to enable developers to improve circuit efficiency.

For example, we may have a finite field which we want to transform to a byte array for use somewhere else in our circuit. To convert the field to a byte array in our circuit would require looping over the field's size in bytes and performing multiple bit operations. Such as in the pseudocode below, where `x` is a finite field:

```
for i in 0..FIELD_SIZE_IN_BYTES {
    byte_array[i] = (x >> (8 * i)) & 0xff;
}
```

There are a couple problems with this approach.

1. Bit operations in circuits are very inefficient and require lots of constraints.

2. Finite fields are not inherently ordered, so any finite field `x` that we would want to perform a bit operation upon would have to be range constrained to some integer value. This ultimately defeats the purpose of being able to decompose any Field element to a byte array.

Instead, we can write out arithmetic constraints inside of our circuit that ensure we have a valid byte array. For a 254-bit finite field `x` we can write out the following pseudocode:

```
assert(
    (byte_array[0]*2^0 + byte_array[1]*2^8 + ... + byte_array[31]*2^248) - x == 0
)
```

However, there is a problem with this pseudocode as it has been laid out so far. The statements above make sense from the point of view of the verifier, but the prover does not know what the values of `byte_array` are implicitly. `byte_array[0]` could be the full value of `x` while the rest of the byte array values are 0, or the array could be the valid byte array that we want. The prover must inject the correct values into the arithmetic constraint above. The prover can inject this information by performing the byte decomposition in an unconstrained environment.

A ZK DSL does not just prove computation but also proves that some computation was handled correctly. Thus, it is necessary that when we switch from performing some operation directly inside of a circuit to inside of an unconstrained environment, the appropriate constraints are still laid down elsewhere in the circuit. The note selection algorithm example at the top of this section follows the same methodology. We are not constraining the unconstrained execution, but rather its outputs.

Brillig provides a way to evaluate such unconstrained functions in a safe, general VM.

2. Attributing incorrectness

Brillig's bytecode, when compiling a ZK DSL into a program commitment, must differentiate between runtime errors and simply invalid proofs. This proves useful in environments that rely on hedged trust and are distributed, such as blockchains.

In such systems, sequenced proofs of shared public state have unique trust assumptions. Proofs of correctness are not constructed by the user or transactor, creating complications. Two scenarios must be distinguished when executing functions in shared public state:

1. When a user attempts to call a public function via a sequencer but inputs data that causes the function to revert and the contract to fail.

2. When a transaction is valid but the prover deliberately assigns incorrect values to the witnesses, leading to a failing proof for a legitimate transaction.

In private functions, where the user is also the prover, distinguishing between these scenarios isn't necessary. However, for public functions, it becomes vital to separate them to accurately identify potential malicious actors in hedged trust scenarios. In the first scenario, the responsibility lies with the user, whereas in the second scenario, it's with the prover.

To ensure the protocol always demands valid proofs of knowledge, a virtual machine (VM) becomes necessary. Here, a VM operates as a zero-knowledge circuit that interprets a program. The VM sequentially executes program instructions and utilizes a public failure flag. In case of VM execution reverting, the failure flag is set to one, and on success, it is set to zero. Regardless of the outcome, an honest user will always have a means to create a valid proof.

Any general VM could technically be used to accomplish this task, such as WASM or the EVM. However, most other general VMs were designed with other execution environments in mind. Proving time is expected to take up the majority of any ZK DSL program's execution, and Brillig's [architecture](01_architecture.md) was specifically designed with SNARK proving efficiency in mind.
