# HybridCPU-v2 + SingNextOS — Community Help Wanted

I am looking for developers, researchers, hardware engineers, and systems people who would like to help move **HybridCPU-v2** and **SingNextOS** from a fairly detailed research architecture toward stronger compiler, emulator, hardware, tooling, and integration implementations.

* **HybridCPU-v2:** https://github.com/yuriyyak23/HybridCPU-v2
* **SingNextOS:** https://github.com/yuriyyak23/SingNextOS
* **Base HybridCPU research paper:** https://zenodo.org/records/20137443

## The idea

I believe the main architectural directions and most of the software/hardware platform specifications are now outlined well enough to start a broader implementation phase.

**HybridCPU-v2** is a research instruction-set emulator/runtime built around a fixed 8-slot VLIW carrier, 4-way SMT, heterogeneous typed lanes, runtime-owned legality, replay/evidence mechanisms, explicit retire/publication boundaries, stream/vector and MatrixTile execution, accelerator contours, and a versioned compiler/runtime contract.

**SingNextOS** is the OS/runtime side of the co-design: a capability-native, ownership-oriented managed operating-system research platform in C#/.NET. It explores typed services, managed isolation, explicit memory ownership, resource accounting, provider-neutral heterogeneous execution, virtualization, DMA/accelerators, SecureCompute, and CXL/fabric integration.

The two projects intentionally meet at a narrow boundary:

> SingNextOS owns semantic authority, ownership, resource truth, publication, and reclaim.
> HybridCPU owns execution legality and architectural retire.
> Compilers and schedulers may describe intent and policy, but must not silently become runtime authority.

## Why I think this is practical now

One of the motivations behind SingNextOS is that a modern .NET compiler toolchain gives us mechanisms that did not exist, or were far less mature, in earlier Managed OS experiments.

Roslyn analyzers, source generators, compiler profiles, deterministic builds, static admission passes, NativeAOT/JIT tooling, and generated typed protocols make it possible to explore many long-standing Managed OS ideas **without forking C# or inventing a new language**.

That does not eliminate the need for runtime checks. The compiler can prove or reject useful structural properties, generate protocol/sentry code, and transport typed metadata, while runtime legality and authority remain explicit and fail-closed.

This compiler/runtime boundary is one of the areas where I would especially value help from experienced compiler developers.

---

# Help wanted

The project is broad, but there are several relatively independent contribution tracks.

## 1. Compiler / Roslyn / lowering

**This is the highest-priority area.**

Relevant existing pieces include Roslyn architecture/security analyzers and generators in SingNextOS, static ManagedCap admission, HybridCPU typed-slot facts, compiler/runtime contract validation, and guarded lowering for heterogeneous execution.

Useful contributions include:

* improving Roslyn analyzers for kernel, SIP, driver, capability, ownership, and unsafe-code rules;
* extending source generators for typed SIP protocols and security sentries;
* making compiler diagnostics clearer and easier to act on;
* implementing or reviewing HybridCPU lowering and bundle construction;
* strengthening compiler/runtime conformance tests;
* using runtime telemetry as compiler optimization feedback without making telemetry an authority source;
* investigating JIT/NativeAOT integration for SingNextOS profiles;
* exploring profile-driven compilation for kernel, SIP/service, driver, and application components;
* designing safe lowering for VectorStream, MatrixTile, DSC and external-accelerator operations;
* building differential tests between compiler assumptions and runtime legality.

A central design constraint is:

```text
compiler metadata != runtime legality
```

The compiler may be stricter than the runtime, but stale or forged compiler metadata must never silently become the correctness authority.

## 2. FPGA / SpinalHDL

I would like help creating an FPGA implementation path and, specifically, translating the executable HybridCPU ISE model into **SpinalHDL** in a disciplined way.

The goal is not a mechanical line-by-line rewrite of C# into HDL. The ISE should remain the executable architectural reference, while the HDL implementation is checked against it.

A useful staged approach would be:

1. define the hardware-visible architectural contract;
2. build a C# ↔ HDL differential/co-simulation harness;
3. implement bundle transport/decode and one small closed execution contour;
4. add typed-lane admission and scheduling;
5. implement register/rename/commit and retire-visible state;
6. add replay/evidence behavior;
7. expand into Stream/Vector, MatrixTile, DMA/DSC and accelerator paths;
8. continuously compare HDL behavior against the ISE reference tests.

Contributors with experience in **SpinalHDL, Verilog/SystemVerilog, Verilator, FPGA toolchains, formal verification, VLIW/SMT pipelines, cache/DMA protocols, or accelerator interfaces** would be extremely valuable.

Even a small first milestone — for example a reproducible co-simulation harness and one verified execution lane — would materially move the project forward.

## 3. GUI debugger / monitor / architecture explorer

Both projects already expose architecture-relevant diagnostics and telemetry. I would like to turn that data into a useful interactive debugger/monitor instead of relying only on logs and test output.

Possible views:

* VLIW bundle and per-slot/lane visualization;
* Stage A / Stage B admission decisions;
* legality decisions and reject reasons;
* per-virtual-thread scheduling;
* pipeline and retire timeline;
* replay/template/certificate activity;
* register rename/commit state;
* Stream/MatrixTile/DSC/L7 activity;
* memory publication and fence events;
* capability lineage and Region ownership;
* resource reservation/settlement;
* SIP/session/invocation lifecycle;
* external-operation state;
* virtualization/CXL provider activity.

A good tool should make an important distinction visible:

```text
evidence != authority
completion != publication
mapping != ownership
```

The GUI(i have base WinForms projects) could become a common observability surface for the software ISE, tests, QEMU/provider models, and later FPGA hardware.


## 4. QEMU / virtualization / CXL

SingNextOS already has provider-neutral virtualization and CXL/fabric architecture, while HybridCPU-v2 has virtualization/SecureCompute work and explicit memory/DMA/accelerator boundaries.

I would like to add a **QEMU-based executable integration environment**, especially for CXL and virtualization work.

Possible work items:

* QEMU device/backend models for selected HybridCPU/SingNextOS provider contracts;
* a reference CXL.io / CXL.mem / CXL.cache environment;
* virtual MMIO/IRQ/DMA paths;
* memory-placement and visibility experiments;
* IOMMU/coherency experiments behind explicit feature gates;
* nested-domain/virtual-device test scenarios;
* fault injection and disconnect/reconciliation tests;
* trace bridging into the GUI monitor;
* CI scenarios that run the same semantic tests against host/model, QEMU, and eventually FPGA backends.

QEMU should be treated as an executable provider/backend, not as a source of SingNextOS authority. Likewise, emulated CXL coherence must not silently imply ownership, publication, or replay safety.

Experience with **QEMU internals, PCIe/CXL, IOMMU, device emulation, virtualization, memory models, or Linux/KVM test infrastructure** would be especially helpful.

## 5. Verification, tests, formal methods, and documentation

There is also a lot of useful work for contributors who do not want to own a large subsystem.

Examples:

* property-based tests for authority and lifecycle state machines;
* replay/rollback and retire-order tests;
* compiler/runtime differential tests;
* negative tests for forbidden or stale authority;
* concurrency/race tests;
* model-vs-provider conformance suites;
* ISE-vs-HDL differential testing;
* fuzzing decoders, descriptors, sideband metadata, and protocol state machines;
* formalization of small invariants;
* reproducible benchmarks;
* documentation review and architecture diagrams;
* small examples that demonstrate one feature end-to-end.

The projects intentionally use a conservative claim discipline: parser support, telemetry, fake backends, DTOs, or successful model execution are not treated as proof of production behavior.

---

# Good first contributions

You do **not** need to understand the whole architecture before helping.

Some reasonable first PRs could be:

* add a focused Roslyn analyzer rule with tests;
* improve one compiler diagnostic and its documentation;
* create a minimal ISE trace reader;
* build a small GUI timeline for retire/replay events;
* export a stable machine-readable trace from one subsystem;
* add a compiler/runtime agreement test;
* create a SpinalHDL skeleton plus C#/Verilator differential-test harness;
* model one QEMU MMIO/DMA device path;
* add a CXL provider test scenario;
* fuzz one parser or descriptor boundary;
* write a small architecture example and validate it against current code/tests.

If you are unsure where your background fits, please open a GitHub Discussion or Issue and describe what you are interested in: compiler, .NET runtime, OS design, FPGA, SpinalHDL, QEMU, CXL, GUI/debugging, verification, or documentation.

---

# Contribution principles

Please keep changes aligned with a few core rules:

* **Live code and executable tests outrank prose.**
* **Authority must remain with its defined owner.**
* **Compiler metadata, telemetry, identity, mapping, and provider receipts are not authority by themselves.**
* **Completion is not automatically publication.**
* **CXL coherence is not automatically ownership.**
* **Unsupported features should fail closed rather than silently degrade.**
* **New architectural claims should come with executable evidence.**
* **Hardware and emulator implementations should preserve the same externally visible semantic boundaries.**
* **Prefer small, reviewable milestones over large speculative rewrites.**

For architecture-changing work, a PR should ideally explain:

```text
What invariant is affected?
Who owns the authoritative state?
Where is the transition/linearization point?
What becomes externally visible, and when?
What happens on failure, cancellation, restart, or stale state?
What tests demonstrate the claim?
```

---

# Where to start reading

### HybridCPU-v2

Start with the repository README and then follow its current reading order into the WhiteBooks, operational semantics, validation baseline, evidence matrix, compiler/runtime contract, Stream/MatrixTile/DSC/L7 documentation, and virtualization/SecureCompute material.

https://github.com/yuriyyak23/HybridCPU-v2

### SingNextOS

Start with the repository README, the vNext architecture/authority/resource documents, SingCap-M material, HybridCPU integration whitebook, external requirements, Roslyn analyzers/generators, runtime, provider abstractions, and tests.

https://github.com/yuriyyak23/SingNextOS

Please use each repository's current `global.json`, build scripts, and validation instructions rather than assuming a particular SDK/toolchain version from this document.

---

# How to help

You can help by:

* opening design/implementation Issues;
* joining GitHub Discussions;
* reviewing architecture or code;
* submitting small focused PRs;
* implementing one of the tracks above;
* running tests on different systems;
* donating FPGA/CI/compute resources;
* helping with documentation and examples;
* connecting the project with compiler, FPGA, OS, virtualization, QEMU, or CXL developers.

I am particularly interested in collaborators who enjoy working across boundaries: **compiler ↔ runtime, OS ↔ architecture, emulator ↔ RTL, and semantic model ↔ real hardware**.

---

# Sponsorship

Code contributions are not the only useful form of support.

If you would like to support development financially:

**PayPal:** https://paypal.me/YAKGitHub

Tooling support is also welcome, especially:

* OpenAI API / Codex development credits or equivalent AI-assisted development budget;
* CI compute;
* FPGA boards or remote FPGA access;
* synthesis/simulation tooling;
* test hardware;
* cloud build/test capacity.

For security, please do **not** share personal account passwords or private credentials. Sponsored credits, organization/project access, vouchers, hardware access, or other properly scoped project resources are preferable.

---

# Current status

These are research projects, not production-ready general-purpose CPU/OS products.

The architecture is deliberately being developed with explicit implementation/evidence boundaries. Some contours are executable today, some are model/helper-level, and others are future-gated. Contributions that make one boundary more executable, testable, reproducible, or understandable are valuable.

I think the software and hardware direction is now sufficiently defined that a wider community can help turn the current architecture into stronger implementations.

If any part of this intersects with your interests, **please join the discussion, open an issue, or send a pull request. Even a narrow contribution is useful.**

Thank you.

<!--
**yuriyyak23/yuriyyak23** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
