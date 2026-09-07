# Human-Intent-Preserving Residual Control for Robotic Grasping

**Shiwei Chen**  
Master's Thesis Research Demo

**Research areas:** sEMG · Dexterous Manipulation · Residual Reinforcement Learning · Shared Autonomy · Human–Robot Interaction

This Master's thesis investigates how human muscle activity can provide the primary command for dexterous robotic grasping while task-level learning improves physical execution. The proposed hierarchical framework first decodes multichannel surface electromyography (sEMG) into an executable low-dimensional grasp command, then applies bounded residual reinforcement learning (RL) around that human-derived baseline using hand, contact, force, and holding feedback.

The framework is evaluated progressively. It is first validated in a six-dimensional synergy-control setting in MuJoCo and is subsequently adapted to an existing one-dimensional PowerGrasp interface driven by real-time sEMG and human-operated motion recorded through Unity/ROS. These are successive validation levels of one research framework, not separate research projects.

> This repository is a public-facing research demo for an ongoing Master's thesis. It presents the research concept, experimental progression, and selected descriptive evidence; source code, human-recorded data, checkpoints, detailed configurations, and complete numerical artifacts remain private.

## Research Question and Central Idea

Supervised sEMG decoding can produce a continuous and interpretable grasp command, but prediction accuracy alone does not optimize physical interaction with an object. Grasp execution additionally depends on contact transitions, object dynamics, force distribution, and holding stability. Conversely, learning a complete dexterous-hand controller from scratch would weaken the direct connection between user intention and robot behavior.

The thesis therefore separates two functions within one coordinated controller:

1. **Signal-level intention decoding** generates the primary human-derived command.
2. **Task-level residual optimization** makes bounded local corrections using physical interaction feedback.

The baseline command remains recoverable at every control step: when the residual is zero, the system executes the original human-derived command.

## Main Contributions

- A two-stage hierarchical framework that separates continuous sEMG intention decoding from contact-aware task optimization while keeping both components in one control loop.
- A causal TCN decoder for converting eight-channel sEMG into a six-dimensional hand-synergy representation, evaluated against an MLP under round-level data separation.
- A low-dimensional residual TD3 controller that refines an executable baseline rather than generating the complete hand command, together with zero-residual regularization that discourages unnecessary intervention.
- A progressive validation strategy: algorithmic evaluation in six-dimensional synergy space followed by adaptation of the same residual-control principle to a real-sEMG-driven, one-dimensional PowerGrasp interface with complete-round held-out evaluation.

## Human sEMG Interface

Human forearm activity is acquired using a wearable multichannel surface-electromyography armband. The recorded muscle activity provides a non-invasive signal related to grasp intention and forms the human-side input to the control framework.

<p align="center">
  <img src="Assets/SemgSensor.jpg" width="42%" alt="Wearable multichannel sEMG armband positioned on the forearm">
</p>

<p align="center"><sub>Wearable forearm sEMG interface used to acquire muscle activity for continuous grasp-intention decoding.</sub></p>

## Two-Stage Control Framework

The overall method combines supervised decoding and residual task optimization. In the first stage, a causal temporal model estimates a low-dimensional synergy command from recent sEMG history. In the second stage, the decoded command is retained as the baseline while a residual policy uses simulated hand–object interaction feedback to produce a bounded correction.

The refined synergy is mapped to the actuators of the Hannes robotic hand and executed in MuJoCo. This structure gives the human-derived signal responsibility for the primary grasp pattern and gives RL limited authority to compensate for task-level interaction effects.

<p align="center">
  <img src="Assets/主框架图新.png" width="100%" alt="Two-stage sEMG decoding and bounded residual reinforcement-learning framework">
</p>

<p align="center"><sub>Overall thesis framework: causal sEMG-to-synergy decoding followed by bounded, contact-aware residual optimization.</sub></p>

```text
Human forearm sEMG
→ causal intention decoding
→ human-derived baseline command
→ bounded residual correction
→ Hannes hand actuation
→ contact, force, motion, and holding feedback
```

### Stage 1 — Causal sEMG Intention Decoding

The signal-decoding stage maps eight-channel sEMG sequences to a continuous six-dimensional hand-synergy representation. A Temporal Convolutional Network (TCN) models the temporal structure of the signal using causal operations, so the current output depends on present and preceding input rather than future samples. An MLP serves as the feedforward reference model.

Decoder evaluation is organized by recording round. Each fold assigns complete rounds to training, validation, and held-out testing rather than randomly distributing neighboring temporal samples across data splits. The comparison examines held-out prediction accuracy and variation across folds, after which the TCN is retained as the executable baseline decoder for residual-control validation.

<p align="center">
  <img src="Assets/Cross-Validation+指标.png" width="100%" alt="Round-level cross-validation and held-out TCN and MLP sEMG decoding results">
</p>

<p align="center"><sub>Stage 1 evidence: round-level TCN/MLP evaluation for six-dimensional sEMG-to-synergy decoding.</sub></p>

### Stage 2 — Bounded Task-Level Residual Optimization

The residual controller does not predict a replacement trajectory. It receives the current baseline together with robotic-hand state, object motion, contact information, interaction-force feedback, holding state, and recent control history. A TD3 Actor then generates a bounded correction in the low-dimensional control space.

Zero-residual regularization introduces a soft preference for minimal intervention. It does not force the policy to reproduce the baseline; instead, it encourages the controller to retain meaningful corrections when the task requires them and return toward the human-derived command when large corrections are unnecessary.

## Progressive Experimental Validation

The same methodological principle is examined at two complementary levels. The first establishes algorithmic feasibility in a structured multidimensional control space. The second adapts the framework to an existing laboratory interface with a different baseline model and a more constrained grasp command.

| Validation level | Baseline command | Residual action | Purpose |
| --- | --- | --- | --- |
| **Six-dimensional synergy validation** | TCN-decoded hand-synergy signal | Bounded correction of individual synergy components | Test signal-to-task integration and residual grasp optimization in MuJoCo |
| **One-dimensional interface adaptation** | PowerGrasp command decoded from real-time sEMG by the laboratory regression model | Bounded scalar correction of grasp intensity | Test compatibility with recorded human operation and held-out task rounds |

Because the two levels use different baseline models, action spaces, task configurations, and evaluation protocols, their numerical results are interpreted within each level rather than compared directly.

## Six-Dimensional Synergy-Control Validation

The initial validation is implemented in `SimMuJoCo_Sidegrasp`. The pretrained TCN supplies a six-dimensional synergy baseline, and the residual policy modifies individual synergy components before the refined command is mapped to the Hannes hand. This setting tests whether task feedback can improve physical grasp execution while preserving the main temporal structure of the decoded intention.

### Contact-Rich Manipulation Task

The simulated task contains a complete manipulation sequence: approach, grasp establishment, lift, transfer, placement, and release. The controller must therefore operate across contact transitions, stable holding, object transport, and intentional release rather than performing only an isolated closing motion.

<p align="center">
  <img src="Assets/抓握任务全程流程图.png" width="100%" alt="Complete MuJoCo manipulation sequence from approach to release">
</p>

<p align="center"><sub>Six-dimensional validation task in MuJoCo: approach, grasp, lift, transfer, placement, and release.</sub></p>

This level provides algorithmic validation of the complete sEMG–TCN–residual-RL pipeline. Its residual-control evaluation uses the available six-dimensional motion rounds and is not presented as the later complete-round LORO generalization experiment.

## One-Dimensional PowerGrasp Adaptation

The subsequent `SimUnity` stage adapts the same residual-control principle to the existing Hannes PowerGrasp laboratory interface. Here, real-time eight-channel sEMG is processed by the laboratory regression model to generate a continuous one-dimensional grasp-intention signal. A motion-tracked handheld controller independently determines the position and orientation of the virtual hand in Unity.

The recorded signals are synchronized and reconstructed in MuJoCo. The participant-generated spatial hand trajectory and wrist commands are retained as external inputs, while the object evolves according to MuJoCo dynamics and hand–object contact. Residual RL modifies only the scalar grasp intensity; it does not alter the recorded spatial path.

### Physical–Virtual Experimental Workflow

The one-dimensional reference is converted into a baseline PowerGrasp command and mapped to coordinated Hannes finger actuation. The residual controller then applies a bounded scalar adjustment using fingertip force, contact, hand-state, holding-state, and recent-control information.

<p align="center">
  <img src="Assets/Fig_Framework.png" width="100%" alt="Real-sEMG-driven Unity ROS and MuJoCo one-dimensional residual-control workflow">
</p>

<p align="center"><sub>System-adaptation workflow: sEMG-derived PowerGrasp baseline, unchanged human-operated spatial replay, scalar residual control, and MuJoCo contact feedback.</sub></p>

```text
Real-time sEMG → laboratory regression model → baseline PowerGrasp command
                                               + bounded scalar residual
Human-operated Unity trajectory ─────────────→ unchanged spatial replay
                                               ↓
                                      Hannes hand in MuJoCo
```

### Complete-Round Held-Out Evaluation

The one-dimensional controller is evaluated using an eight-fold leave-one-round-out (LORO) protocol. For each fold, one complete recorded task round is reserved for final evaluation and the remaining rounds are used for policy training. The held-out round is excluded from policy optimization and checkpoint selection, avoiding random temporal mixing between neighboring samples from the same task sequence.

All compared controllers use the same recorded hand trajectory, wrist inputs, initial object condition, MuJoCo model, and time alignment for a given held-out round. The comparison therefore isolates the effect of the one-dimensional residual grasp correction.

<p align="center">
  <img src="Assets/Fig_loro_gesture.png" width="100%" alt="Manipulation phases and eight-fold leave-one-round-out evaluation protocol">
</p>

<p align="center"><sub>One-dimensional system adaptation: representative task phases and complete-round LORO evaluation.</sub></p>

## Representative Evidence

The following figures report selected descriptive evidence from the thesis. They summarize the one-dimensional SimUnity evaluation and should not be combined numerically with the earlier six-dimensional validation.

### Held-Out Grasp-Control Comparison

Three control conditions are compared on the complete held-out trajectories: the original one-dimensional baseline, residual TD3 without zero-residual regularization, and the proposed regularized residual controller.

The figure summarizes stable-hold time proportion, unsafe-step time proportion, baseline-deviation RMSE, and the valid-contact tangential-to-normal force ratio. Baseline-deviation RMSE measures how strongly the residual controller modifies the original sEMG-derived PowerGrasp command; it is not an intention-decoding error. Stable-hold time is a temporal state proportion rather than an object-level grasp-success rate.

<p align="center">
  <img src="Assets/Fig_Overall.png" width="100%" alt="Held-out one-dimensional residual grasp-control comparison across LORO folds">
</p>

<p align="center"><sub>One-dimensional held-out comparison of the PowerGrasp baseline, unregularized residual TD3, and zero-residual-regularized control.</sub></p>

### Contact-Force and Spectral Characteristics

Contact analysis complements the task-level measures by examining simulated normal force, tangential force, and their ratio during valid finger–object contact. A representative held-out round shows the time evolution of friction demand and stable-contact force, while cross-round summaries describe changes relative to the baseline.

Welch spectral analysis provides a frequency-domain view of force variation. These quantities characterize contact behavior and control smoothness in simulation; the tangential-to-normal ratio represents friction demand derived from simulated contact forces rather than a directly measured material coefficient.

<p align="center">
  <img src="Assets/Fig.Friction_Spectrum.png" width="100%" alt="One-dimensional SimUnity contact-force friction-ratio and Welch spectral analysis">
</p>

<p align="center"><sub>Contact-level evidence from the one-dimensional held-out evaluation: representative force behavior, cross-round summaries, and Welch spectral content.</sub></p>

## Research Scope

| Area | Role in the thesis |
| --- | --- |
| Human intention decoding | Continuous mapping from multichannel sEMG to executable low-dimensional grasp commands |
| Dexterous manipulation | Contact-rich PowerGrasp execution with the Hannes robotic hand |
| Residual reinforcement learning | Bounded task-level compensation around a workable human-derived baseline |
| Intention preservation | Zero residual recovers the baseline; regularization discourages unnecessary intervention |
| Progressive validation | Six-dimensional algorithmic evaluation followed by one-dimensional laboratory-interface adaptation |
| Generalization assessment | Complete recorded rounds held out from SimUnity policy training and checkpoint selection |

## Public Release Scope

**Included**

- High-level research motivation, framework, and experimental progression
- Selected system, task, protocol, and evaluation figures
- Representative descriptive evidence from the thesis

**Not publicly released**

- Human-recorded datasets and participant-related information
- Training, preprocessing, and evaluation source code
- Model checkpoints and internal experimental configurations
- Detailed reward coefficients, complete numerical tables, and unpublished ablations

## Research Status

This repository documents ongoing Master's thesis research. The current evidence combines sEMG-derived human inputs with physics-based simulation: the six-dimensional experiment establishes algorithmic feasibility, and the one-dimensional experiment demonstrates adaptation to a recorded human-in-the-loop laboratory workflow with held-out task-round evaluation. Physical Hannes-hand interaction, broader cross-user and cross-day generalization, and more varied grasp types remain directions for future work.

## Citation

This repository presents ongoing Master's thesis research by **Shiwei Chen**.

Formal thesis and publication citations will be added after the associated work becomes publicly available. Please contact the author before redistributing unpublished figures or research material.
