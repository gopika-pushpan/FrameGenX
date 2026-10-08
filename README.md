# FrameGenX

## A Modular Retrieval-Augmented and Self-Reflective Framework for Text-to-Image Generation

FrameGenX is a modular text-to-image (T2I) generation framework designed to improve **visual grounding, structural consistency, knowledge incorporation, and parameter-efficient adaptation** in diffusion-based image generation.

The framework combines:

- **Prompt Parsing**
- **RealRAG-based Visual Retrieval**
- **Context Fusion**
- **SDXL-based Generation**
- **ControlNet-based Structural Conditioning**
- **LoRA-based Parameter-Efficient Adaptation**
- **CLIP-based Semantic Evaluation**
- **VQA-based Logical Verification**
- **SRRL-based Self-Reflective Refinement**

Unlike a monolithic image generation pipeline, FrameGenX separates generation into independently replaceable stages, allowing individual components to be evaluated, replaced, and ablated.

---

## Table of Contents

- [Overview](#overview)
- [Research Objectives](#research-objectives)
- [Problem Statement](#problem-statement)
- [Proposed Architecture](#proposed-architecture)
- [Core Modules](#core-modules)
- [Pipeline](#pipeline)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Notebook Pipeline](#notebook-pipeline)
- [Evaluation Metrics](#evaluation-metrics)
- [Experimental Results](#experimental-results)
- [Ablation Study](#ablation-study)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Research & Publication](#research--publication)
- [References](#references)
- [Acknowledgements](#acknowledgements)

---

# Overview

Modern text-to-image models can produce highly realistic images, but they still face several important challenges:

1. **Knowledge Cutoff**
   - Generative models have knowledge limited by their training data.
   - Newly emerging entities, events, styles, or real-world concepts may not be represented accurately.

2. **Structural Hallucination**
   - Diffusion models may produce visually plausible but logically incorrect structures.
   - Examples include incorrect object counts, spatial relationships, object interactions, or physically inconsistent scenes.

3. **Adaptation Bottleneck**
   - Full fine-tuning of large diffusion models requires substantial computational resources.
   - This makes domain or style adaptation difficult for researchers working with limited GPU resources.

4. **Lack of Self-Correction**
   - Conventional generation pipelines generally return an image without systematically evaluating whether the generated image satisfies the original prompt.

FrameGenX addresses these limitations by combining **retrieval, structural conditioning, parameter-efficient adaptation, semantic evaluation, and iterative refinement**.

---

# Research Objectives

FrameGenX was designed around four primary objectives.

### 1. Bridge the Knowledge Gap

Retrieve relevant real-world visual references at generation time rather than relying entirely on the knowledge encoded in the pretrained model.

### 2. Reduce Structural Hallucinations

Use retrieved context, structural conditioning, CLIP evaluation, and VQA-based verification to improve consistency between the generated image and the requested scene.

### 3. Enable Parameter-Efficient Adaptation

Use **LoRA with rank `r = 8`** to adapt SDXL without requiring full model fine-tuning.

### 4. Build an Open and Modular Architecture

Design the system as independently replaceable modules so that individual components can be evaluated through modular experimentation and ablation studies.

---

# Problem Statement

Existing text-to-image systems can generate high-quality images but may struggle with:

- Real-world knowledge that appeared after model training
- Multi-object compositional prompts
- Spatial relationships
- Object consistency
- Fine-grained structural constraints
- Domain-specific adaptation
- Logical verification of generated content

FrameGenX addresses these problems through a hybrid pipeline combining **RealRAG, SDXL, ControlNet, LoRA, CLIP, VQA, and SRRL**.

---

# Proposed Architecture

FrameGenX follows a modular five-stage architecture:

```text
                USER PROMPT
                     │
                     ▼
            ┌─────────────────┐
            │  Prompt Parsing │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────┐
            │ RealRAG Retrieval│
            └────────┬────────┘
                     │
             Reference Images
              + Embeddings
                     │
                     ▼
            ┌─────────────────┐
            │  Context Fusion │
            └────────┬────────┘
                     │
          Fused Text + Visual
            Context + Depth
                     │
                     ▼
        ┌─────────────────────────┐
        │    SDXL + LoRA          │
        │ Generation + Adaptation │
        └────────────┬────────────┘
                     │
                Base Image
                     │
                     ▼
            ┌─────────────────┐
            │ SRRL Refinement │
            └────────┬────────┘
                     │
              CLIP + VQA Check
                     │
             ┌───────┴───────┐
             │               │
            PASS             FAIL
             │               │
             ▼               ▼
        FINAL IMAGE     Corrective Prompt
                             │
                             └──► Refinement Loop
