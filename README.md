# OmniPersist

**Persistent Dynamic World Modeling via Coupled Panoramic Evolution**

OmniPersist is a single-stage video diffusion framework for camera-controlled world modeling. It jointly generates perspective video and a temporally sparse 360° panoramic stream so that moving subjects can continue evolving outside the camera's field of view and remain consistent when the camera returns.

## Overview

Existing perspective video models often lose track of moving subjects during camera revisits: subjects may change appearance, disappear, or return with implausible motion. OmniPersist maintains a panoramic representation of scene evolution alongside the perspective video within a shared diffusion transformer.

Given a text prompt, an observed perspective image or video prefix, and a camera trajectory with known intrinsics, the model predicts both streams in a single denoising process. It requires no initial panoramic image, auxiliary depth input, or auxiliary generative model at inference.

## Key Features

- **Coupled panoramic evolution:** Dense perspective video and sparse panoramic states share one video diffusion backbone, providing context for content outside the current viewport.
- **Geometric Anchoring (GeoAnch):** Explicit geometric correspondences support feature exchange between perspective and panoramic views at synchronized timestamps.
- **Temporal Compensation (TemComp):** Local feature aggregation across neighboring timestamps bridges the different frame rates of the two streams.
- **Flexible camera conditioning:** The framework supports latent-level conditioning with CameraCtrl and attention-level conditioning with UCPE.
- **Adjustable panoramic density:** A sampling interval controls the trade-off between panoramic prediction cost and generation quality.

## Method

OmniPersist uses **Wan2.2-TI2V-5B** as its backbone. Perspective and panoramic tokens are jointly processed by a shared diffusion transformer adapted with rank-32 LoRA. GeoAnch and TemComp connect the streams after each transformer block.

Visual conditions for both streams are derived from the observed perspective prefix. Camera poses and intrinsics reproject observed content onto the panoramic canvas, with masks identifying known regions. The panoramic stream runs at a lower frame rate while maintaining a shared timeline with the perspective stream.

Training jointly optimizes a flow-matching objective for both views. Panoramic targets use circular latent encoding and latitude-dependent loss weighting. At inference, the panoramic stream can remain in latent space when only perspective video output is needed.

## Tasks and Benchmark

The paper evaluates two tasks, each targeting an 81-frame video:

| Task | Observed perspective input | Target video |
| --- | --- | --- |
| Image-to-video (I2V) | 1 frame | 81 frames |
| Video continuation | 21 frames | 81 frames |

The paired panoramic–perspective dataset is constructed from real-world 360-1M panoramic footage:

- Approximately **125K** high-quality clips, each with **81 frames at 16 fps**.
- Perspective resolution: **832 × 480**; panoramic resolution: **1024 × 512**.
- Four trajectory types: object tracking, single revisit, double revisit, and approximately 360° camera rotation.
- **300 evaluation videos**, with their source footage excluded from the training split.

Evaluation covers 14 metrics across visual quality, camera controllability, background revisit fidelity, and dynamic subject persistence.

## Selected Results

The following results are reported in the manuscript at panoramic sampling interval **s = 4**. Each baseline uses the same dataset, Wan2.2-TI2V-5B backbone, and LoRA configuration as its OmniPersist counterpart.

### Image-to-video

| Method | FID ↓ | FVD ↓ | Rotation error ↓ | Re-entry identity ↑ |
| --- | ---: | ---: | ---: | ---: |
| CameraCtrl | 23.60 | 288.5 | 13.086 | 0.2325 |
| CameraCtrl + OmniPersist | **22.18** | **262.3** | **5.556** | **0.3355** |
| UCPE | 25.33 | 333.7 | 8.127 | 0.2097 |
| UCPE + OmniPersist | **21.89** | **272.7** | **4.976** | **0.3694** |

### Video continuation

| Method | FVD ↓ | Background PSNR ↑ | Re-entry identity ↑ | Subject trajectory error ↓ |
| --- | ---: | ---: | ---: | ---: |
| CameraCtrl | 227.5 | 13.43 | 0.2494 | 0.5161 |
| CameraCtrl + OmniPersist | **194.9** | **15.22** | **0.3771** | **0.4289** |
| UCPE | 264.6 | 14.55 | 0.2778 | 0.5255 |
| UCPE + OmniPersist | **210.3** | **15.72** | **0.3849** | **0.4245** |

Re-entry identity measures foreground DINOv2 feature similarity when subjects return to view. Background PSNR is measured on intersected static-background masks at revisit timestamps. Arrows indicate whether higher or lower values are better; bold values indicate improvements over the corresponding baseline.

Both OmniPersist variants improve all 14 evaluated metrics over their respective perspective-only baselines in both tasks at s = 4.

## Repository Status

This repository currently contains the project README. Training and inference code, model checkpoints, dataset preparation tools, and evaluation scripts are not included in this initial commit.

## Limitations

Training currently relies on panoramic video data. Real-time streaming and long-horizon video generation remain future directions.
