---
title: UMIE model support and integration
source_commit: b18082b3abdae22b98cc96228c4102575681ff67
watches:
  - specs
  - crates/umie-spec
  - crates/umie-model
  - crates/umie-metal
  - crates/umie-cuda
  - crates/umie-asr
  - crates/umie-embedding
  - crates/umie-voxcpm
  - crates/umie-acestep
  - crates/umie-separate
  - docs/sdk.md
  - docs/reference/model-catalog.md
  - benchmarks/results
---

# UMIE: one resident model, modular behavior

UMIE—the Universal Modular Inference Engine—is a Rust inference engine for the fungOS ecosystem. A base model stays in memory while validated behavior assets change its operation. Lua recipes describe each model; Rust loads checkpoints, manages memory and runs inference.

UMIE has native Metal and CUDA paths, a Rust SDK, C ABI and serving interfaces. Model coverage is a set of explicit operations and encodings, rather than a blanket promise that every model in a family works. The engine is in active development. Model weights and their upstream licenses are separate from UMIE's Apache-2.0 software license.

## What “support” means

A recipe names a model and maps its tensors. A schema check validates that plan. Checkpoint admission then checks actual tensor names, dtypes and derived shapes. A backend must implement the required operators and encodings. Finally, real checkpoint execution and semantic parity need their own evidence.

The tables distinguish recipe validation from checkpoint execution. Benchmark results apply to the recorded model, encoding, backend and code revision. Configurations without execution records remain unverified.

## Model families and execution evidence

| Models | Architecture / purpose | Evidence in source | Limits |
|---|---|---|---|
| Qwen2.5 1.5B Instruct | Dense causal decoder; text and constrained decisions | Recorded Metal BF16/Q8 and CUDA BF16/Q4/Q8 greedy generation; native RLCD records on both backends | Tested configurations are listed; other Qwen2.5 sizes require separate validation |
| Llama 3.2 1B Instruct | Standard dense causal attention | Checked-in model recipe and contract fixtures | Inventory establishes schema validation, not checkpoint generation parity |
| Granite 4.1 8B | Standard attention with model-specific scaling | Recorded CUDA Q4/Q8 generation; Metal/CUDA primitive records | Primitive execution is narrower than whole-model generation |
| Qwen3 8B / 30B-A3B; Qwen3.5 0.8B / 4B | Dense, MoE and hybrid-attention recipes | Typed recipe resolution | Consult operators and materialization admission before execution; family recognition is not full coverage |
| Ornith 1.5 9B / 35B-A3B | Hybrid decoder recipes | Historical CUDA greedy generation records, including native NVFP4 on the 35B recipe | Older README WIP notes describe earlier scope; prefer operation records at explicit revisions |
| Swift-Qwen3.8 27B; Qwen3.8-Flash-Next | Hybrid decoder recipes | Historical CUDA greedy generation records, with NVFP4 paths | Results apply to the recorded backend, revision and encoding |
| GPC-1; Granite 3.0 1B-A400M; Nemotron 3 Nano 30B-A3B | Additional decoder targets | Model recipes, typed family validation and subsystem source | Recipe presence does not establish generation parity |
| Qwen3-VL 8B text | Text decoder extraction | Explicit text-only recipe | Does not imply image input or full vision-language execution |
| Qwen2.5 3B/7B/14B/32B; Coder 7B/14B/32B; Qwen3 4B/14B/32B; Granite 3.1 2B/8B | Incubator recipes | Pinned upstream metadata and fixture/schema tests | Catalog explicitly declares non-production, schema-validated/unverified; no loading or generation claim |

Recorded greedy generation operations are separated from embeddings, RMSNorm, RoPE, linear kernels and decoder-stack measurements in the generated evidence table below. CPU RLCD reduction is an algorithmic operation, not a CPU implementation of every whole model.

## Audio and structured policy models

| Model / subsystem | Implemented boundary | Evidence and scope |
|---|---|---|
| Mel-band Roformer Deux; BS-Roformer SW 6-stem | Typed separation recipes, separation and streaming modules | Existing stream parity and transport tests; resident Metal test is hardware dependent. Schema checks alone do not establish output quality |
| ACE-Step 1.5 standard-turbo | Audio-generation recipe and `umie-acestep` backend ABI | Asset validation and execution module implemented; end-to-end audio quality and hardware validation pending |
| VoxCPM2 behavior assets | Speech-generation plan, voice behavior as a loadable asset, `umie-voxcpm` execution boundary | Base and behavior identities are distinct; custom voice recipes are not a claim about arbitrary voice packages |
| Nemotron 3.5 ASR streaming | Native FastConformer/RNN-T frontend, encoder and transducer runtime in `umie-asr` | Experimental implementation outside the Lua catalog; hardware validation pending |
| Discogs-EffNet | Audio embeddings compatible with Essentia's Discogs-EffNet pipeline in `umie-embedding` | Pinned assets and frontend identifiers; audio embeddings are distinct from text sentence embeddings |
| Taiga S1 | Typed-option policy assembled from generic encoder operations | Recipe requires admitted checkpoint configuration and shape facts; default model-catalog resolution cannot supply those facts |
| SAM Audio small TV | Prompt-aware audio separation recipe | Recipe/plan boundary exists; Roformer geometry or coverage should not be inferred for SAM Audio |

Audio and speech-recognition models require their own input formats, preprocessing, output rates and checkpoints. Image diffusion models are a separate development effort and are not covered by this matrix.

## Behavior and data boundaries

- **Model recipes:** instruction-bounded, I/O-free Lua resolves tensor mappings and materialization policy. A recipe is not authenticated just because its sandbox accepts it; admission/signature policy belongs to callers.
- **Resident base:** tensor materialization is owned by Rust. Adapters, constrained decision plans and other admitted assets have explicit identities.
- **RLCD:** bounded schema-constrained choices are scored from model logits; tokenizer continuation boundaries are owned by `umie-tokenizer`.
- **Cache identity:** inputs that change tensors or KV semantics must participate in identity. A matching model name alone is insufficient.
- **Checkpoints:** SafeTensors metadata and shard indices are validated before execution. A readable GGUF checkpoint is not automatically executable; its native encoding must have a backend path.

## Integration entry points

For embedding, begin with the repository's [SDK guide](https://github.com/FuturePresentLabs/umie/blob/b18082b3abdae22b98cc96228c4102575681ff67/docs/sdk.md). It documents Rust, C ABI ownership and streaming contracts. Use that contract for handles and buffer release rather than reproducing internal layouts.

The serving layer exposes health, model discovery and metrics:

```text
GET /health
GET /v1/models
GET /metrics
```

Wait for model readiness, then test the operation your client needs. The [Rust HTTP SDK reference](https://github.com/FuturePresentLabs/umie/blob/b18082b3abdae22b98cc96228c4102575681ff67/docs/sdk-rust-http.md) documents the client interface. Repository access may be required.

## Regenerate the inventory

UMIE owns the catalog and its provenance. From its checked-out repository:

```sh
scripts/catalog-models
scripts/catalog-models --check
scripts/catalog-models --site-source /path/to/fungos-web/docs-source/umie.md
scripts/catalog-models --site-source /path/to/fungos-web/docs-source/umie.md --check
cargo test -p umie-spec --example model_catalog
cargo test -p umie-spec --test incubator_catalog
```

The deterministic Rust example resolves recipes using the existing sandbox, emits JSON plus Markdown, and checks generated-file drift. It records per-backend resolution errors explicitly. Historical benchmark JSON contributes operation, encoding and the report's last committed revision/date; those dates are commit dates, not inferred measurement timestamps.

The outputs are `docs/reference/model-recipes.json` and `docs/reference/model-support.md`. The [catalog reference](https://github.com/FuturePresentLabs/umie/blob/b18082b3abdae22b98cc96228c4102575681ff67/docs/reference/model-catalog.md) explains evidence levels and limitations. A reviewer must update this page's watched code baseline after examining changed source; regeneration never automatically blesses new behavior.

<!-- umie-model-catalog:start -->

# UMIE model recipe matrix

Generated by `scripts/catalog-models`. A schema pass is **not** working model support. Both columns use synthetic host facts and default recipe inputs; no weights, device, latency, quality, or parity are tested. Execution remains unverified by this catalog. Recipes requiring admitted checkpoint context can be unresolved here without being unsupported by their owning subsystem.

| Model / recipe | Architecture family | Modality | Metal schema | CUDA schema | Hardware execution |
|---|---|---|---|---|---|
| ACE-Step/Ace-Step1.5-standard-turbo (`specs/ace_step_1_5_turbo.lua`) | ace_step_1_5 | audio generation | schema validated | schema validated | Not tested by catalog |
| becruily/mel-band-roformer-deux (`specs/bs_roformer_becruily_deux.lua`) | mel_band_roformer | audio separation | schema validated | schema validated | Not tested by catalog |
| jarredou/bs-roformer-sw-6stem (`specs/bs_roformer_sw_6stem.lua`) | bs_roformer | audio separation | schema validated | schema validated | Not tested by catalog |
| harshatheg/GPC-1 (`specs/gpc1.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| ibm-granite/granite-3.0-1b-a400m (`specs/granite_3_0_1b_a400m.lua`) | granite_moe | text decoder | schema validated | schema validated | Not tested by catalog |
| ibm-granite/granite-4.1-8b (`specs/granite_4_1_8b.lua`) | granite | text decoder | schema validated | schema validated | Not tested by catalog |
| ibm-granite/granite-3.1-2b-instruct (`specs/incubator/granite_3_1_2b_instruct.lua`) | granite | text decoder | schema validated | schema validated | Not tested by catalog |
| ibm-granite/granite-3.1-8b-instruct (`specs/incubator/granite_3_1_8b_instruct.lua`) | granite | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-14B-Instruct (`specs/incubator/qwen2_5_14b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-32B-Instruct (`specs/incubator/qwen2_5_32b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-3B-Instruct (`specs/incubator/qwen2_5_3b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-7B-Instruct (`specs/incubator/qwen2_5_7b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-Coder-14B-Instruct (`specs/incubator/qwen2_5_coder_14b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-Coder-32B-Instruct (`specs/incubator/qwen2_5_coder_32b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-Coder-7B-Instruct (`specs/incubator/qwen2_5_coder_7b_instruct.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3-14B (`specs/incubator/qwen3_14b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3-32B (`specs/incubator/qwen3_32b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3-4B (`specs/incubator/qwen3_4b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| meta-llama/Llama-3.2-1B-Instruct (`specs/llama_3_2_1b.lua`) | llama | text decoder | schema validated | schema validated | Not tested by catalog |
| nvidia/NVIDIA-Nemotron-3-Nano-30B-A3B-NVFP4 (`specs/nemotron_3_nano_30b_a3b.lua`) | nemotron_h | text decoder | schema validated | schema validated | Not tested by catalog |
| ornith-ai/Ornith-1.5-9B (`specs/ornith_1_5_9b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| ornith-ai/Ornith-1.5-35B-A3B (`specs/ornith_35b_a3b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen2.5-1.5B-Instruct (`specs/qwen2_5_1_5b.lua`) | qwen2 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3-30B-A3B (`specs/qwen3_30b_a3b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3.5-0.8B (`specs/qwen3_5_0_8b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3.5-4B (`specs/qwen3_5_4b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3-8B (`specs/qwen3_8b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen-Image-2.1/text_encoder (`specs/qwen3_vl_8b_text.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| Qwen/Qwen3.8-Flash-Next (`specs/qwen4_exp_flash_next.lua`) | qwen4_exp | text decoder | schema validated | schema validated | Not tested by catalog |
| facebook/sam-audio-small-tv (`specs/sam_audio_small_tv.lua`) | sam_audio | audio separation | schema validated | schema validated | Not tested by catalog |
| ukisai/Swift-Qwen3.8-27b (`specs/swift_qwen3_8_27b.lua`) | qwen3_5 | text decoder | schema validated | schema validated | Not tested by catalog |
| taiga_s1 (`specs/taiga_s1.lua`) | requires admitted asset context | unresolved | not resolved | not resolved | Not tested by catalog |
| FuturePresentLabs/tts-jarvis (`specs/voxcpm2_jarvis.lua`) | voxcpm2 | speech synthesis | schema validated | schema validated | Not tested by catalog |

JSON includes resolver errors and evidence. Incubator upstream revisions and metadata hashes are maintained in `specs/incubator/catalog.json`; the catalog's tests compare them to fixtures. `training_policy_reference.lua` is a training policy, not a model, and is excluded. A resolved decoder family can still be rejected by model compilation or a backend with missing operators.

## Recorded runtime operations

Derived from checked-in benchmark reports, separately from schema resolution. These are historical operation records, not tests performed by this command or a guarantee of correctness. Date and revision refer to the report's last committed change, not a measurement timestamp. Missing recorded evidence means unknown, not unsupported.

| Model | Backend | Recorded operations / encodings | Report revision/date |
|---|---|---|---|
| ibm-granite/granite-4.1-8b | CUDA | linear-resident / bf16 | 38c34d4205b5a8c29c6b240dc434f3c8e4011e1e 2026-09-20 |
| ibm-granite/granite-4.1-8b | CUDA | rms-norm / bf16 | 62503159c98a10ebca7db3b3b59dd08994fe22c7 2026-09-20 |
| ibm-granite/granite-4.1-8b | CUDA | generate-greedy / q4_block32 | b1f724b774a9bd5db3efc48a2bfcd457f703690e 2026-09-20 |
| ibm-granite/granite-4.1-8b | CUDA | generate-greedy / q8_block32 | 0fb2025236e6641856537f2ec35c91c8358e8f9c 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | decoder-stack / bf16 | 15e408eb39fe567da8a8f2595e71f31bd1bf6b93 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | token-embed-rms-norm / bf16 | ea06ee5166673729b2487e814a7a427f6d4eaa49 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | generate-greedy / bf16 | be18cfcb011bddc9b171ea184af8dd18dd2d5e94 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | generate-greedy / q4_block32 | be18cfcb011bddc9b171ea184af8dd18dd2d5e94 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | generate-greedy / q8_block32 | be18cfcb011bddc9b171ea184af8dd18dd2d5e94 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | rlcd-native / bf16 | 0048c87336e1a2d5f28ca1c92bd7e87856f57e75 2026-09-20 |
| ornith-ai/Ornith-1.5-9B | CUDA | generate-greedy / q4_block32 | a042d2ede0a01997b9c24acad92ad4875484ef32 2026-09-22 |
| ibm-granite/granite-4.1-8b | CUDA | rope-resident / f32-activation | e2f2ebe87d2d9f226dcd9e12be1d3913efdcf477 2026-09-20 |
| ornith-ai/Ornith-1.5-9B | CUDA | generate-greedy / bf16 | a042d2ede0a01997b9c24acad92ad4875484ef32 2026-09-22 |
| ornith-ai/Ornith-1.5-9B | CUDA | rms-norm-resident / bf16 | e2f2ebe87d2d9f226dcd9e12be1d3913efdcf477 2026-09-20 |
| ornith-ai/Ornith-1.5-9B | CUDA | rms-norm / bf16 | 62503159c98a10ebca7db3b3b59dd08994fe22c7 2026-09-20 |
| ornith-ai/Ornith-1.5-9B | CUDA | rope-resident / f32-activation | e2f2ebe87d2d9f226dcd9e12be1d3913efdcf477 2026-09-20 |
| ornith-ai/Ornith-1.5-35B-A3B | CUDA | generate-greedy / native_nvfp4 | 30f246d759440d284400c9342fc1e1f271eee693 2026-09-26 |
| ukisai/Swift-Qwen3.8-27b | CUDA | generate-greedy / native_nvfp4 | fc5b91b7f35b50828b11bd41df0d1ed3a9fe5958 2026-09-22 |
| ukisai/Swift-Qwen3.8-27b | CUDA | generate-greedy / q4_block32 | a042d2ede0a01997b9c24acad92ad4875484ef32 2026-09-22 |
| Qwen/Qwen2.5-1.5B-Instruct | CUDA | rope-resident / f32-activation | 1d3f949b0a6ef7bbcf862653e4c1d88e54a3723d 2026-10-02 |
| Qwen/Qwen2.5-1.5B-Instruct | Metal | rlcd-native / bf16 | fc38d541ddd5eecd65748a11bdc4ed0ba129e0b4 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | Metal | generate-greedy / bf16 | e8d18016f659d1effb30cf317dbf77c7701aede1 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | Metal | generate-greedy / q8_block32 | e8d18016f659d1effb30cf317dbf77c7701aede1 2026-09-20 |
| ibm-granite/granite-4.1-8b | Metal | rope-resident / f32-activation | e2f2ebe87d2d9f226dcd9e12be1d3913efdcf477 2026-09-20 |
| ornith-ai/Ornith-1.5-9B | Metal | rope-resident / f32-activation | e2f2ebe87d2d9f226dcd9e12be1d3913efdcf477 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | Metal | decoder-stack / bf16 | a3a7d9449dca5c57eb7b6a03bb9cc147822421d7 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | Metal | linear-resident / bf16 | 38c34d4205b5a8c29c6b240dc434f3c8e4011e1e 2026-09-20 |
| Qwen/Qwen2.5-1.5B-Instruct | Metal | token-embed-rms-norm / bf16 | f79e045d7641fd01b2212ec5b982d05780d055ae 2026-09-20 |
| qwen/qwen2.5-1.5b-instruct-rlcd | CPU | rlcd-reduce / log-f32 | aba1499ccc0320364a6fa5f923a889debb10d0f0 2026-09-20 |
| Qwen/Qwen3.8-Flash-Next | CUDA | generate-greedy / native_nvfp4 | f03a993500d95de439cb051afb4472cbfb4c8aba 2026-09-27 |
<!-- umie-model-catalog:end -->
