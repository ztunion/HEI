# Homomorphic Encryption Interlingua (HEI)

# Why HEI

HE lets a system compute on data it cannot read, but it is typically 100 to 1,000,000 times slower than plaintext computation. Most work on speeding up HE makes each encrypted operation faster, through better algorithms, parallel code, or dedicated hardware.

HEI reduces how much encrypted work a task needs. Through the interlingua, it shrinks:

- **the input:** fewer encrypted operations and less encrypted data;
- **the model:** models trained on the interlingua can be smaller, with fewer layers and less bootstrapping;
- **the output:** a short interlingua answer, such as a few field values, cuts generation time for LLMs.

Because HEI works on what is computed, not on the HE scheme or the hardware, its gain adds to theirs.

## How it works

```
 CLIENT TRUST BOUNDARY                                 HE PROCESSING SYSTEM
+-------------------------------+                     (never sees plaintext)
|                               |
|  Input                        |
|    |                          |
|    v                          |
|  Compile to interlingua       |
|    |                          |
|    v                          |   ciphertext   +-----------------------------+
|  Encrypt  --------------------+--------------->|                             |
|                               |                |  Homomorphically encrypted  |
|                               |                |  processing (AI inference)  |
|  Decrypt  <-------------------+----------------|                             |
|    |                          |   ciphertext   +-----------------------------+
|    v                          |
|  Decompile                    |
|    |                          |
|    v                          |
|  Output                       |
|                               |
+-------------------------------+
```

HEI does not prescribe how input is translated into an interlingua and back, the interlingua format, or how it is encoded for HE. These depend on the task.

## Three kinds of models

| | Full-text model | Retrained full-text model | Interlingua model |
|---|---|---|---|
| What it is | An existing model, used as is | An existing architecture, retrained on the interlingua | A model designed for interlingua input |
| Source of speedup | Shorter input | Shorter input and a minimal output | Shorter input, a minimal output and a smaller model |
| Estimated speedup (author's) | 5–20× for long inputs; about 2–4× for classic BERT | Same, plus about 10× less LLM generation time when output drops from 100 to 10 tokens | About 10–100×, depending on how much smaller the model is |

All figures are estimates and have not yet been measured.

## Example: EVA-BERT

EVA-BERT applies HEI to vulnerability analysis. A client checks its software bill of materials (SBOM) against vulnerability data held by a provider, without revealing its inventory. The provider's BERT-based model runs under HE on interlingua records of SBOM entries and CVEs, and returns encrypted risk scores that only the client can decrypt.

## Documents

- [Introduction to HEI](<Introduction to HEI.md>): what HE is, how HEI improves on it, and open questions.
- [EVA-BERT white paper](EVA-BERT.md): encrypted vulnerability analysis with HEI.

## Status

HEI is a concept with estimated, not measured, performance. The next step is an end-to-end prototype with benchmarks against HE on the full input. ZTUnion is seeking partners.

## Feedback

Questions, critiques and challenges are welcome. Open an issue or contact michael@ztunion.com.

#
© 2026 ZTUnion LLC. All rights reserved.
