# Human-Intent-Preserving Residual Control for Robotic Grasping

**Shiwei Chen**  
Master's Thesis Research

**Research areas:** sEMG · Dexterous Manipulation · Residual Reinforcement Learning · Shared Autonomy · Human–Robot Interaction

This project develops a two-stage framework for adaptive robotic grasping: a causal temporal decoder maps multichannel surface electromyography (sEMG) to a compact grasp representation, and a residual reinforcement-learning controller introduces bounded task-level corrections around that human-derived command. Contact, interaction-force, robotic-hand, and object-motion feedback support grasp adaptation without replacing the human-provided control baseline.

> This repository is a public research showcase for an ongoing Master's thesis. Source code, human-recorded datasets, model checkpoints, detailed experiment configurations, and complete numerical artifacts remain private.

## Key Contributions

- A two-stage sEMG-driven grasp-control framework combining causal human-intention decoding with residual reinforcement learning.
- A human-command-preserving formulation in which the learned policy applies only bounded task-level corrections around the explicit human-derived baseline.
- Contact-aware grasp adaptation informed by robotic-hand state, object motion, contact information, and interaction-force feedback.
- Round-disjoint evaluation on complete held-out recorded trajectories, avoiding random temporal mixing of neighboring samples between training and evaluation.

## Human sEMG Interface

Human forearm activity is measured with a wearable multichannel sEMG interface. These non-invasive muscle signals provide the input to the causal intention-decoding stage and establish the human-generated reference for downstream grasp control.

<p align="center">
  <img src="Assets/SemgSensor.jpg" width="42%" alt="Wearable multichannel sEMG interface positioned on the forearm">
</p>

<p align="center"><sub>Wearable forearm sEMG interface used to acquire muscle activity for intention decoding.</sub></p>

## System Architecture

The framework separates signal-level intention decoding from task-level adaptation. A causal temporal model produces the baseline grasp representation from sEMG, while the residual policy uses simulated hand–object feedback to compute a constrained correction before actuation of the Hannes robotic hand.

<p align="center">
  <img src="Assets/主框架图新.png" width="100%" alt="Two-stage architecture for causal sEMG intention decoding and residual robotic grasp control">
</p>

<p align="center"><sub>Two-stage architecture: human-derived grasp decoding followed by bounded, contact-aware residual control.</sub></p>

```text
Human input
→ causal intention decoding
→ baseline grasp command
→ bounded residual correction
→ robotic-hand actuation
→ contact and motion feedback
```

## Demonstrated Manipulation Task

The MuJoCo-based simulation evaluates a complete contact-rich manipulation sequence: approach, grasp establishment, lift, transfer, placement, and release. The task therefore tests sustained hand–object interaction across multiple phases rather than an isolated finger-closing action.

<p align="center">
  <img src="Assets/抓握任务全程流程图.png" width="100%" alt="Complete simulated manipulation sequence from approach to release">
</p>

<p align="center"><sub>Complete simulated sequence with the Hannes robotic hand: approach, grasp, lift, transfer, placement, and release.</sub></p>

## Recorded-Motion Residual-Control Study

A complementary Unity/ROS workflow provides recorded human motion references for simulation-based residual-control experiments. The spatial hand trajectory is preserved and replayed unchanged; the learned controller modifies only the grasp-related component in response to contact and dynamics feedback. This separation retains the demonstrated motion path while allowing adaptive grasp correction.

<p align="center">
  <img src="Assets/Fig_Framework.png" width="100%" alt="Unity ROS recorded-motion and residual grasp-control framework">
</p>

<p align="center"><sub>Recorded-motion study separating unchanged spatial replay from adaptive residual correction of the grasp command.</sub></p>

## Evaluation Protocol

Evaluation follows a leave-one-round-out, round-disjoint design. One complete recording round is held out for evaluation, the remaining rounds are used for training, and the process is repeated across rounds. Neighboring temporal samples from the same recording are therefore not randomly divided between training and test sets.

<p align="center">
  <img src="Assets/Fig_loro_gesture.png" width="100%" alt="Manipulation phases and leave-one-round-out evaluation protocol">
</p>

<p align="center"><sub>Representative manipulation phases and the round-disjoint leave-one-round-out evaluation design.</sub></p>

## Representative Results

The following figures provide selected representative evidence from the current research. Complete numerical artifacts, extended analyses, and unpublished per-round results remain part of the thesis and associated research outputs.

### sEMG Intention-Decoding Evaluation

Round-level held-out evaluation compares the causal temporal decoder with an MLP reference model. The figure reports decoding measures for each held-out fold and descriptive cross-round summaries, making variation across recording rounds visible without implying statistical significance.

<p align="center">
  <img src="Assets/Cross-Validation+指标.png" width="100%" alt="Round-level sEMG decoding evaluation for the causal temporal decoder and MLP reference model">
</p>

<p align="center"><sub>Round-disjoint sEMG intention-decoding evaluation with held-out-fold results and descriptive cross-round summaries.</sub></p>

### Residual Grasp-Control Evaluation

The held-out comparison visualizes stable-hold time proportion, unsafe interaction time, residual-control magnitude, and tangential-to-normal interaction-force balance. Residual magnitude quantifies how strongly the autonomous controller departs from the human-derived baseline; stable-hold measurements describe temporal behavior rather than object-level task-success probability.

<p align="center">
  <img src="Assets/Fig_Overall.png" width="100%" alt="Held-out comparison of the baseline residual TD3 and proposed residual controller">
</p>

<p align="center"><sub>Descriptive held-out comparison of the recorded baseline, residual TD3, and the behavior-regularized residual controller.</sub></p>

### Contact-Force and Spectral Analysis

Complementary simulation-based analyses examine normal force, tangential force, and the tangential-to-normal force ratio during valid contact. Time-domain force behavior and Welch spectral content provide additional views of contact interaction and control smoothness across the evaluated controllers.

<p align="center">
  <img src="Assets/Fig.Friction_Spectrum.png" width="100%" alt="Contact-force friction-ratio and Welch spectral analysis">
</p>

<p align="center"><sub>Representative contact-force and friction-ratio behavior, cross-round force summaries, and Welch spectral analysis.</sub></p>

## Research Scope

| Area | Role in the project |
| --- | --- |
| Human intention decoding | Causal mapping from multichannel sEMG to a compact grasp representation |
| Dexterous manipulation | Contact-rich grasping and object transfer with the Hannes robotic hand |
| Residual reinforcement learning | Bounded task-level correction around the human-derived baseline |
| Human-centered / shared control | Preservation of human intent while adding feedback-driven assistance |
| Generalization evaluation | Assessment on complete recorded rounds held out from training |

## Public Release Scope

**Included**

- High-level system and experimental design
- Selected task and evaluation figures
- Representative descriptive results

**Not publicly released**

- Human-recorded datasets and participant-related information
- Training, preprocessing, and evaluation source code
- Model checkpoints and internal experiment configurations
- Complete numerical results and unpublished ablations

## Research Status

This is ongoing Master's thesis research and the current public showcase is simulation-focused. The full methodology, implementation, statistical analysis, and supporting research artifacts remain part of the thesis and associated research outputs.

## Citation

This repository presents ongoing Master's thesis research by **Shiwei Chen**.

Formal thesis and publication citations will be added after the associated work becomes publicly available. Please contact the author before redistributing unpublished figures or research material.
