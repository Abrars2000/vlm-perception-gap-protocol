# Scope v4 — Option B (frozen VLM checkpoints, exploratory)

Supersedes scope.md v3. Change record is at the end of this file.

Status: Phase 1 load and forward tests complete on Kaggle (Tesla T4, transformers 4.52.4, torch 2.11.0+cu130). Predictor values and gap results have NOT been computed. Model replacement below was made before any predictor or gap result existed.

## 1. Framing

- Three already-pretrained VLM checkpoints are used as-is. No task-specific training, fine-tuning, or weight updates.
- P1 and P2 are computed from a fixed caption corpus and frozen encoder features/labels, before VLM answer scoring.
- A, B and C are diagnostic/evaluation measurements on the same controlled image set:
  - A = encoder linear-probe accuracy (pooled final vision-encoder features, before the projector).
  - B = projected-token linear-probe accuracy (pooled visual tokens after the projector, before the language model).
  - C = VLM answer accuracy (exact match after deterministic parsing to canonical labels).
  - Gap = A − C; projector loss = A − B; readout loss = B − C.
- Class-balanced test set; identical image-level train/test split for all three measurements.

## 2. Attributes (4, unchanged from v3)

| Attribute | Values | Counterfactual rule |
|---|---|---|
| Color | red, green, blue, yellow | Change object colour only |
| Count | 1, 2, 3, 4 | Change object count only |
| Relative position | left-of, right-of, above, below | Change the relation only |
| Existence | present, absent | Add/remove the target object only |

Object-noun lexicon: COCO-80 category names. The counterfactual image set is to be generated and frozen; manifest (rendering method, seeds, object lists, image count, split) is written before the confirmatory run.

## 3. Models (3, exploratory) — v4

| Model | Hugging Face ID | Pinned revision | Precision (Phase 1) | Peak VRAM (T4) | Phase 1 result |
|---|---|---|---|---|---|
| Qwen2.5-VL-3B-Instruct | Qwen/Qwen2.5-VL-3B-Instruct | 243fd99abe513d2a02a98274ea34c07e8f961b0f | fp16 | 8.13 GB | load pass, forward pass ("Red") |
| Molmo-7B-D-0924 | allenai/Molmo-7B-D-0924 | cab33fb7f1a40091911f81165f8481920621948f | 4-bit nf4 | 14.17 GB | load pass, forward pass ("Red.") |
| LLaVA-1.5-7B-hf | llava-hf/llava-1.5-7b-hf | b234b804b114d9e37bb655e11cbbb5f5e971b7a9 | 4-bit nf4 | 4.89 GB | load pass, forward pass ("Red") |

**Dropped:** mPLUG-Owl2 LLaMA2 7B (`MAGAer13/mplug-owl2-llama2-7b`, revision 7d115a29e0ddc402eaa54a4326a2eaf415256351). Revision SHA matched, but `model_type mplug_owl2` is not recognised by transformers 4.52.4 and the load error indicates the checkpoint needs code not available in transformers 4.52.4 (cause not further investigated). Dropped before any predictor or gap result was computed.

**Known implementation notes (must be recorded with every result):**
- Molmo: 47 vision-encoder LayerNorm modules cast to fp16 after 4-bit load, to fix a dtype mismatch in the remote code. Results from Molmo are conditional on this workaround.
- Molmo peak VRAM (14.17 GB) is close to the T4 usable limit (~14.56 GB). Batch size 1 is required; OOM is possible with longer prompts.
- Qwen was first tested under transformers 5.20.2 (peak 4.31 GB, answer "Red"); the final Phase 1 record is the 4.52.4 run (peak 8.13 GB, answer "Red").
- Processor choice (fast vs slow image processor) must be fixed and logged before the full run.

**Model count implication:** 3 models. LOMO therefore has 3 folds with 2 training models each. This remains exploratory (see section 6).

## 4. Caption corpus (for P1)

- MS-COCO Captions 2017, train split: `captions_train2017.json`.
- SHA-256 verified in Phase 1 on 3 Nov 2026: `4b62086319480e0739ef390d04084515defb9c213ff13605a036061e33314317` — **MATCH**.
- Caption text only; COCO images are not used as the counterfactual set.

## 5. Predictors

Primary: **P1** (language prior, log-frequency and PMI on the caption corpus) and **P2** (encoder-feature separability, Fisher ratio and silhouette on pre-projector features). Both z-scored on the training portion of each fold.

Sensitivity only: **P3** (projector bandwidth: visual-token count and retention ratio). Not part of the primary law and not used to support any primary claim.

Formulas unchanged from v3.

## 6. Primary predictive law and validation

Gap_am = β0 + β1·P1_a + β2·P2_am + u_m + v_a + ε_am

- LOAO: 4 folds (one held-out attribute each, 3 models in training).
- LOMO: 3 folds (one held-out model each, 2 models in training). Reported as exploratory.
- Pooled out-of-fold R² with 95% crossed cluster-bootstrap CI, LOAO and LOMO separately.
- Small-scope warning: 4 attributes × 3 models = 12 cells; 4 attribute clusters and 3 model clusters. Intervals will be unstable.

### 6.1 P2–A confound check (required)

Diagnostic: compare C ~ A with C ~ A + P2 on held-out data. Report both and the P2–A correlation.

## 7. Sensitivity analysis (P3, not confirmatory)

Two mutually exclusive specifications (with-P3 with model dummies absent; without-P3 with model dummies), compared under identical LOAO and LOMO folds. Reported as descriptive only. See `p3_sensitivity` in the simulation package.

## 8. Pilot protocol — pipeline check only

Pilot is a software/data-integrity check. No predictor value, gap, ΔR², projector loss or readout loss is computed, displayed or logged during pilot. Output is PIPELINE CHECK: PASSED or FAILED with a pass/fail checklist.

## 9. Compute plan

Kaggle GPU (Tesla T4, 16 GB), approximately 30 hours/week. Process checkpoints sequentially; cache frozen encoder and projected features. Record GPU, VRAM, precision, library versions and runtime for every run.

## 10. Change record (v3 → v4)

| # | Change | Reason | Made before any predictor/gap result? |
|---|---|---|---|
| 1 | mPLUG-Owl2 dropped | `mplug_owl2` model_type not recognised by transformers 4.52.4; needs authors' code | Yes |
| 2 | LLaVA-1.5-7B-hf added (pinned `b234b804…`) | Replacement with native transformers support, load and forward verified | Yes |
| 3 | Molmo precision fixed to 4-bit nf4, LayerNorm fp16 workaround recorded | fp16 on T4 ran out of memory; dtype mismatch in remote code | Yes |
| 4 | Qwen precision fixed to fp16, transformers 4.52.4 | Final environment; earlier run under 5.20.2 recorded in notes | Yes |
| 5 | COCO captions SHA-256 verified (MATCH) | Phase 1 check | Yes |
| 6 | Model count unchanged at 3 (one model replaced by another) | Replacement, not reduction | Yes |

Phase 1 log: `phase1_load_log.json` (saved in the Kaggle working directory).
