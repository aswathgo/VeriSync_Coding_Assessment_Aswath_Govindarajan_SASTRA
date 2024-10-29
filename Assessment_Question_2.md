# Question 2: ZK Implementation Challenge

## Section 1: Problem Definition

- The given problem is an example for a Zero Knowledge Proof Problem.
Zero-knowledge proof (ZKP) is a cryptographic method that allows a
prover to demonstrate to a verifier that the prover knows a specific
piece of information without revealing any important details about the
information itself.  
- In the given problem, the goal is for a prover to
demonstrate knowledge of a value x such that: "x^2 + x + 7 = 9" without
revealing the actual value of x.
- Now, let's define what is available
publicly, and what is to be kept privately:
- Public Information /
Input, that can be known to both the prover and the verifier nodes: The
equation "x^2 + x + 7" and the target 9
- Private Information / Input,
that can only be known to the prover: The exact value of "x" that would
satisfy the equation "x^2 + x + 7 = 9"

## Section 2: Protocol Selection

- I have decided to choose "Plonk" protocol for the given problem
- Efficiency: It uses a universal trusted setup, which allows for quick
proof generation without the need for extensive pre-processing.
- Security: PLONK is quantum-proof, and is established on complex
cryptographic primitives
- Ease of implementation: Plonk protocol allows
for more flexibility in circuit design and is easier to work with when
implementing ZK circuits, especially for learners like me.

## Section 3: Circuit Design

- Inorder to create a ZK Circuit, this equation has to be converted
into a sequence of arithmetic operations, that can be implemented in the
circuit and that can be checked by the circuit.
- Input Gate: Accept the private input x (this remains hidden from the verifier)
- First
Operation: Find x^2
- Second Operation: Once x^2 is obtained, add x^2 +
x
- Third Operation: Add 7 to the result of the 2nd Operation to get
"x^2 + x + 7"
- Final Check: Ensure that the result of the Third
Operation equals the Public Target value (9)
- This circuit can be
expressed as a series of checks to confirm if the final output matches
the expected public value.
- The circuit's final operation will output
True if the result equals 9; otherwise, it outputs False. This result is
publicly available to the verifier, who can then see that the equation
is satisfied without knowing the value of x.

## Section 4: Implementation in Circom 
- Circom is chosen because it is
specially optimised for zk-circuits. Circom simplifies the process
enabling the developer to focus on the logic of their circuit while
leveraging the underlying cryptographic efficiency.
- Code snippets are
attached as answers for the first two questions in this section
- The following is the code to immplement the ZK Circuit
```
pragma circom 2.0.0;
template Quadratics() {
    signal private input i;
    signal output o;
    signal i_squared;
    i_squared <== i * i;
    o <== i_squared + i + 7;
    o === 9;
}
component main = Quadratics();
```

- The above code is to be run in zkREPL, an online compiler of Circom.
- The version of Circom language being used is given in the first line of code
- The next line defines a template, a circuit, that goes by the name "Quadratics". Templates are like functions, they are reusable components
- The next two  lines define a private input signal named "i", that will not be revealed to the verifier and a public output signal "o" which will be the result of the computation that the prover would prove
- The mathematical operations are defined in the next 3 lines of code
- The next line adds a constraint that "o" must be equal to "9", which is the main condition of the circuit
- The last line will create an instance of the "Quadratics" template

### Proof Generation and verification using JavaScript:

```
const snarkjs = require("snarkjs");
const fs = require("fs");

async function generateProof() {
    const input = {
        "x": 1  
    };
    const { proof, publicSignals } = await snarkjs.groth16.fullProve(
        input,
        "quadratic_equation.wasm",
        "quadratic_equation.zkey"
    );
    console.log("Proof: ", proof);
    console.log("Public Signals: ", publicSignals);
    const vKey = JSON.parse(fs.readFileSync("verification_key.json"));
    const verified = await snarkjs.groth16.verify(vKey, publicSignals, proof);
    console.log("Verification result: ", verified);
}
generateProof().then(() => {
    console.log("Done!");
}).catch((err) => {
    console.error(err);
});
```
- Snark JS Library is used here, for the cryptographic operations
- An asynchronous function is defined to generate the proof and verify it
- Once the input is set, fullProve method is called from the SnarkJS library to generate a proof, and will return a "proof" and "publicSignals"
- The proof is then logged along with the publicSignal
- from a JSON file, a verification key is obtained and is parsed
- "verify" method from snarkJS is called to verify the validity of the proof
- it returns a Boolean value
- The last part calls the "generateProof" function and handles the promise it returns. If the function completes successfully, it logs "Done!" to the console; if there's an error, it logs the error message.
- The circuit is compiled with the following command in bash terminal:
```
circom quadratic_equation.circom --r1cs --wasm --sym
```
- Two keys: proof key and verification key, are generated with the next command:
```
snarkjs groth16 setup quadratic_equation.r1cs pot12_final.ptau quadratic_equation.zkey
snarkjs zkey export verificationkey quadratic_equation.zkey verification_key.json
```
- Finally, the Javascript code is to be run using node, in the terminal for the verification of the proofs
```
node generate_proof.js
```

##### Tradeoffs to be considered:
1. Proof Generation Time: PLONK is faster at
this, compared to Groth16 and others, but still the circuit's complexity
can affect the Proof Generation time
2. Proof Size: Typically the size
is small, thereby alleviating bandwidth concerns in certain
applications.
3. Verification Time: As PLONK uses a universal trusted
setup, Proof Verification is quick, which is useful in scenarios that
require quick validation in its applications.
