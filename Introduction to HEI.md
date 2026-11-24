# **Introduction to Homomorphic Encryption Interlingua (HEI)**

© 2026 ZTUnion LLC. All rights reserved.

`michael@ztunion.com`

HEI is an approach to making homomorphic encryption (HE) computation practical for various AI tasks by using an interlingua, a short task-scoped representation of the input. In this approach, the client (the entity that sends the request) compiles input into an interlingua and encrypts it. An HE processing system then works on the ciphertext without ever seeing plaintext. Finally, the client decrypts the result, decompiles it, and releases the output.

# **What is homomorphic encryption**

Homomorphic encryption (HE) lets a system compute on encrypted data without decrypting it. The decrypted result matches, or closely approximates, what the same computation on plaintext would produce \[1\]. Fully homomorphic encryption (FHE) supports any computation \[2\]. Other forms support only certain operations, or a limited number of them.

## **Advantages over other privacy-enhancing technologies (PETs)**

* Trusted execution environments (TEEs) require trust in the processing system's supply chain and its operator. HE keeps data confidential without it \[3\].

* Multi-party computation needs several parties computing together, or a client that stays online through rounds of interaction \[4\]. HE needs neither.

* Differential privacy typically adds noise to outputs to protect individuals in aggregate results \[5\]. HE protects the inputs themselves.

* Common FHE schemes are lattice-based and considered to be resistant to quantum attacks \[6\].

## **Disadvantages**

* Overall, HE is typically 100 to 1,000,000 times slower than plaintext computation, depending on the workload and hardware acceleration \[7–10\]. Ciphertexts are also much larger than the data they carry \[9\], and the evaluation keys the HE processing system needs can reach several gigabytes \[11\].

  * Each operation adds noise to the ciphertext, and multiplication adds the most. Clearing it requires *bootstrapping*, a step that refreshes the ciphertext and is often the most expensive one.

* HE protects confidentiality, not integrity. An HE processing system can return a wrong or tampered result, and the client cannot detect it from the ciphertext alone.

* HE is hard to use. Each use case needs its own settings ("parameters"), such as the security level, the precision and the multiplicative depth: how many multiplications can run in sequence before bootstrapping. It also needs its own arrangement of data in ciphertexts ("packing") to keep HE processing efficient. The HE compute code must be designed around both. Because the data stays encrypted, the code also cannot branch on data values.

# **What is HEI**

Instead of encrypting the full input, the client compiles it into an interlingua that keeps only the fields the task needs. The HE processing system computes on the encrypted interlingua and returns an encrypted result, also in interlingua form, such as field values or a label. The client decrypts the result and decompiles it into the output.

```
 CLIENT TRUST BOUNDARY                                HE PROCESSING SYSTEM
+-------------------------------+                    (never sees plaintext)
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
|    |                          |                
|    v                          |
|  Decompile                    |
|    |                          |
|    v                          |
|  Output                       |
|                               |
+-------------------------------+
```

## **Interlingua in other fields**

An interlingua is a neutral intermediate representation. Content is converted into it so it can be processed independently of its original form. The idea is well established:

* Machine translation: interlingua-based systems translate source text into a language-neutral meaning representation, then generate the target language from it. This avoids building a separate translator for every language pair \[12\].

* Compilers: source code is lowered into an intermediate representation, such as LLVM IR, used by many compilers. Optimization and code generation work on that representation instead of on each source language \[13\].

* Semantic representation: formats such as Abstract Meaning Representation (AMR) capture who did what to whom in a sentence, independent of its wording \[14\]. Grammatical Framework uses a shared abstract syntax as an interlingua across languages \[15\].

HEI applies the same idea to encrypted computation. As an approach, it does not define how input is translated into an interlingua and back, the interlingua representation, or how the interlingua is encoded for HE. These depend on the task and can use existing methods. Like a compiler IR, the interlingua can be where the encrypted work is planned and optimized. Here the optimization target is HE cost, and the representation is limited to what the task needs.

## **How HEI differs from embedding vectors**

An embedding vector compresses the whole input into numbers whose meaning is fixed by the model that produced them. Much of the original text can be recovered from it \[16\], and it cannot be read or audited directly. An interlingua holds only the fields the task needs. Its schema can be read and approved, and its result decompiles back to a clear answer. Embeddings support similarity search and learned classification, but not decisions on exact field values. Vectors remain the better fit for true similarity tasks.

# **How HEI improves on HE**

There are three main approaches to speeding up HE: 1\) better algorithms, including HE-friendly model designs \[17\], 2\) parallel code and optimizations for CPUs and GPUs \[7\], and 3\) dedicated hardware such as ASICs \[11\]. All three make each encrypted operation faster.

HEI takes a different approach. Instead of making each encrypted operation faster, it reduces how much encrypted work the task needs. Through the interlingua, HEI reduces the input, the model's complexity, such as its number of layers, and the output. Studies show that the information well-defined tasks need, for example in clinical notes, can be captured in structured, named fields, such as diagnosis, medication and dosage \[18–20\]. Prompt-compression research shows how small that information can be: in some benchmarks, the task-relevant part of a prompt was about 20 times shorter than the full text, with little loss of performance \[21\]. Because HEI works on what is computed, not on the HE scheme or the hardware, it complements the other three approaches, and its gain largely holds as they improve.

The client compiles the interlingua before encryption. Computation runs on an HE processing system operated by a provider, typically on accelerated hardware such as GPUs, because the model may be too demanding to run on the client, or proprietary and not shared with clients.

The interlingua speeds up HE in three ways:

First, a shorter input means fewer encrypted operations \[9\]. For input processing, this gain is expected to be much greater on ciphertext than on plaintext, where shorter prompts give only modest latency gains \[22\], because HE computation scales more closely with input length \[9\].

Second, models trained on the interlingua are expected to be smaller, with fewer layers and parameters, because small task-specific models often match much larger general models on well-defined tasks \[23, 24\]. Fewer layers mean fewer multiplications in sequence, so these models need less bootstrapping \[7, 9\].

Third, for LLMs, a short interlingua output cuts generation time, since each generated token passes through the whole model.

Additionally, the amount of encrypted data the HE processing system must hold and move grows with the input and output length and with the model's size and depth, since deeper computation needs larger ciphertexts. By reducing all of these, the interlingua also reduces memory use and data movement \[9\].

## **Encrypted training and fine-tuning**

Encrypted training on full text has been demonstrated only on a limited scale. A 2025 study fine-tuned just two transformer layers of a BERT-style model under HE \[17\], compared with 12 layers in BERT-base and 32 in Llama-3-8B. Training is costly under HE because it repeats the model's computation over many records and many passes. Once weights are updated with encrypted data, they become encrypted too, so encrypted data must be multiplied by encrypted weights. That multiplication takes longer than multiplying encrypted data by plaintext weights, from several times to about 30 times longer in published benchmarks, depending on the scheme and parameters \[25\].

The interlingua reduces this cost in two ways:

First, it shortens each training record, both the input and the expected output, so every pass does less work \[9\], and the saving repeats across the whole training run. A retrained full-text model gets only this benefit, because it keeps all its layers.

Second, an interlingua model, which takes the interlingua fields as input, is expected to have fewer layers and parameters \[23, 24\]. Fewer parameters mean fewer encrypted weights to multiply, and fewer layers mean fewer multiplications in sequence and less bootstrapping in every pass \[7, 9\].

For these reasons, the author expects encrypted training and fine-tuning of small interlingua models to become far more feasible than training on full text.

## **Encrypted inference**

Encrypted inference on full text has been shown for BERT \[10\] and for Llama-3-8B \[26\]. HEI can be applied to three kinds of models, and in all three, the model's inference must be refactored to run under HE:

* **Full-text model**: a conventional model trained on full text. It keeps its trained weights and takes the interlingua as its input or prompt. The gain comes from the shorter input and, for LLMs, also from a shorter output if the model is told to respond in a specific interlingua format.

* **Retrained full-text model**: a full-text model retrained on the interlingua. It keeps the same architecture, so it processes input about as fast as the full-text model. Because it has learned the interlingua, it reads it more accurately and responds in interlingua format reliably, without being told. The gain comes from the shorter input and a minimal output, such as field values only.

* **Interlingua model**: a model designed specifically for interlingua input and output, instead of full text. Because the compile step has already extracted the fields, it is expected to be smaller and shallower \[23, 24\]. The gain comes from the shorter input, a minimal output and a smaller model.

The table below compares the three models. Input and output length are given in tokens, which are words or parts of words.

| Speedup | Full-text model | Retrained full-text model | Interlingua model |
| ----- | ----- | ----- | ----- |
| Source | Shorter input; for LLMs, also a shorter output when told to respond in interlingua format | Shorter input and a minimal output | Shorter input, a minimal output and a smaller model |
| Estimate (author's) | 5–20× for long inputs; about 2–4× for classic BERT | Same as the full-text model, plus, for LLMs, about 10× less generation time when output drops from 100 to 10 tokens | About 10–100×, depending on how much smaller the model is |

The estimates are based on published GPU timings for HE \[9, 10\] and assume the interlingua is up to about 20 times shorter than the full text \[21\]. The full-text-model figures are for input processing: long inputs are about 1,000+ tokens, classic BERT inputs are capped at 512 tokens, and LLMs with long outputs gain less. The retrained-model figure for LLMs assumes generation time is proportional to the number of output tokens, so cutting the output from 100 tokens to 10 cuts generation time about tenfold. The interlingua-model figure assumes HE cost scales with model size.

# **Use cases**

* **Vulnerability analysis**: assess software and hardware bills of materials with a provider's model, using fields such as component, version and vendor, against vulnerability records such as CVEs. The bills of materials stay encrypted.

* **Customer support triage**: classify and route support tickets by issue type and priority with a provider's model, which never sees customer data.

* **KYC risk scoring**: score customer risk with a provider's model from compiled fields, such as customer type, country and product, keeping customer data encrypted.

* **Out-of-band policy analysis**: have a provider's model review encrypted records in batches against the provider's policies and return encrypted findings, which the client can use to refine its own policies.

* **Regulated workflows**: process regulated data, such as patient records for clinical coding and billing, with a provider's model. The client compiles the needed fields, such as diagnoses and procedures, and the provider returns encrypted codes without seeing the data.

* **Encrypted training**: train small task-specific models on encrypted interlingua fields, out-of-band, without exposing the data to the provider. The data owner holds the key and can decrypt the trained weights if needed.

# **To-dos**

These are the open questions and concerns about HEI, with how each will be checked. The list is tentative and will evolve, and reader input is welcome.

* **Interlingua design and task fit**: Which tasks reduce well to named fields, and which, such as tasks that depend on exact wording or tone, need the full input? How much work does it take to define the fields for a task and tailor them to each model, and can they be reused? Check: design an interlingua for at least two tasks and two models, record the effort, and test whether it keeps what each task needs.

* **Interlingua output**: Can LLMs answer reliably with a short interlingua output, and how much time does it save? Check: compare telling a model to respond in interlingua format with training it on interlingua output, on accuracy, format-failure rate and latency.

* **Training data**: Where does the data come from to train retrained full-text and interlingua models? Check: generate training data by compiling existing labeled inputs and converting their labels to interlingua outputs.

* **Encrypted training**: How much does the interlingua reduce the cost of encrypted training and fine-tuning compared with training on full text? Check: measure the time, memory and data size of training an interlingua model under HE, and the accuracy of the resulting model, and compare them with training on full text, measured at small scale or estimated.

* **Speedup measurements**: Do the speedup estimates for the three kinds of models hold? Check: benchmark HE on the full input against HE on the interlingua for BERT and LLMs such as Llama, measuring accuracy, latency, throughput and data size.

# **Conclusion**

HEI is an approach to making homomorphic encryption practical by encrypting a short interlingua of what the task needs, instead of the full input.

HEI's value is that it attacks HE's main barrier, cost, by reducing the work to be done. A shorter input means fewer encrypted operations and less encrypted data. Models trained on the interlingua are expected to be smaller and shallower, so they need less bootstrapping, and a short interlingua output cuts generation time for LLMs. Because HEI works on what is computed, not on the HE scheme or the hardware, its gain adds to that of faster algorithms, parallel code and dedicated hardware.

HEI is also expected to make encrypted training of task-specific models far more feasible. It fits best on well-defined tasks, such as classification, scoring and policy checks, where the input reduces to named fields.

The performance figures in this document are the author's estimates. The next step is to measure them. That means building an end-to-end prototype, starting with vulnerability analysis on software and hardware bills of materials, then benchmarking it against HE on the full input and working through the open questions in "To-dos". ZTUnion is seeking partners.

# **References**

1. J. H. Cheon, A. Kim, M. Kim, and Y. Song, "Homomorphic Encryption for Arithmetic of Approximate Numbers," ASIACRYPT 2017\.

2. C. Gentry, "Fully Homomorphic Encryption Using Ideal Lattices," STOC 2009\.

3. J. Chuang, A. Seto, N. Berrios, S. van Schaik, C. Garman, and D. Genkin, "TEE.fail: Breaking Trusted Execution Environments via DDR5 Memory Bus Interposition," IEEE S\&P 2026\. [https\://tee.fail](https://tee.fail)

4. A. C. Yao, "How to Generate and Exchange Secrets," FOCS 1986\.

5. C. Dwork, F. McSherry, K. Nissim, and A. Smith, "Calibrating Noise to Sensitivity in Private Data Analysis," TCC 2006\.

6. M. Albrecht et al., "Homomorphic Encryption Security Standard," HomomorphicEncryption.org, Toronto, Canada, 2018\.

7. W. Choi, H. Yu, J. Kim, H. Ji, J. Park, and J. H. Ahn, "Theodosian: A Deep Dive into Memory-Hierarchy-Centric FHE Acceleration," arXiv:2512.18345, 2025\.

8. L. Daksha, S. Guzelhan, K. Shivdikar et al., "FHECore: Rethinking GPU Microarchitecture for Fully Homomorphic Encryption," arXiv:2602.22229, 2026\.

9. Z. Gong, R. Ran, F. Yao, and W. Wen, "AEGIS: Scaling Long-Sequence Homomorphic Encrypted Transformer Inference via Hybrid Parallelism on Multi-GPU Systems," arXiv:2604.03425, 2026\.

10. J. Zhang, X. Yang, L. He et al., "Secure Transformer Inference Made Non-interactive," NDSS 2025\.

11. J. Kim, G. Lee, S. Kim, G. Sohn, J. Kim, M. Rhu, and J. H. Ahn, "ARK: Fully Homomorphic Encryption Accelerator with Runtime Data Generation and Inter-Operation Key Reuse," MICRO 2022\. arXiv:2205.00922.

12. W. J. Hutchins and H. L. Somers, An Introduction to Machine Translation, Academic Press, 1992\.

13. C. Lattner and V. Adve, "LLVM: A Compilation Framework for Lifelong Program Analysis & Transformation," CGO 2004\.

14. L. Banarescu et al., "Abstract Meaning Representation for Sembanking," Proceedings of the 7th Linguistic Annotation Workshop and Interoperability with Discourse (LAW), 2013\.

15. A. Ranta, Grammatical Framework: Programming with Multilingual Grammars, CSLI Publications, 2011\.

16. J. X. Morris, V. Kuleshov, V. Shmatikov, and A. M. Rush, "Text Embeddings Reveal (Almost) As Much As Text," EMNLP 2023\.

17. D. Rho, T. Kim, M. Park, J. W. Kim, H. Chae, E. K. Ryu, and J. H. Cheon, "Encryption-Friendly LLM Architecture," ICLR 2025\.

18. S. J. Chen, M. V. Maddali, C. Langlotz, C. Bluethgen, J. Chen, and R. Raj, "Evaluation of Large Language Models for Structured Data Extraction From Interstitial Lung Disease Clinical Notes: Comparative Study," J Med Internet Res 2026;28:e90547.

19. A. M. Garcia-Carmona, M.-L. Prieto, E. Puertas, and J.-J. Beunza, "Leveraging Large Language Models for Accurate Retrieval of Patient Information From Medical Reports: Systematic Evaluation Study," JMIR AI 2025;4:e68776.

20. J. Huang, D. M. Yang, R. Rong et al., "A critical assessment of using ChatGPT for extracting structured data from clinical notes," npj Digital Medicine 7, 106, 2024\.

21. H. Jiang, Q. Wu, C.-Y. Lin, Y. Yang, and L. Qiu, "LLMLingua: Compressing Prompts for Accelerated Inference of Large Language Models," EMNLP 2023\.

22. C. Kummer, L. Jurkschat, M. Färber, and S. Vahdati, "Prompt Compression in the Wild: Measuring Latency, Rate Adherence, and Quality for Faster LLM Inference," arXiv:2604.02985, 2026\.

23. M. J. J. Bucher and M. Martini, "Fine-Tuned 'Small' LLMs (Still) Significantly Outperform Zero-Shot Generative AI Models in Text Classification," arXiv:2406.08660, 2024\.

24. C.-Y. Hsieh, C.-L. Li, C.-K. Yeh, H. Nakhost, Y. Fujii, A. Ratner, R. Krishna, C.-Y. Lee, and T. Pfister, "Distilling Step-by-Step\! Outperforming Larger Language Models with Less Training Data and Smaller Model Sizes," Findings of ACL 2023\.

25. F. Boemer, A. Costache, R. Cammarota, and C. Wierzynski, "nGraph-HE2: A High-Throughput Framework for Neural Network Inference on Encrypted Data," WAHC 2019\. arXiv:1908.04172.

26. J. Park, S. Park, J. H. Park, J. H. Ahn, J. H. Cheon, G. Hanrot, J. W. Kim, M. Park, and D. Stehlé, "Scaling up FHE-based Privacy-Preserving ML: Higher Throughput, Longer Inputs for LLama-3-8B," arXiv:2601.18511, 2026\.
