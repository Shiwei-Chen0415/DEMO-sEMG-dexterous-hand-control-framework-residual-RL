# Human-Intent-Preserving Residual Control for Robotic Grasping

**Shiwei Chen**  
Master's Thesis Research

**Research areas:** sEMG · Dexterous Manipulation · Residual Reinforcement Learning · Shared Autonomy · Human–Robot Interaction

This thesis develops human-centered robotic grasp control through two sequential experimental studies. The first study decodes multichannel surface electromyography (sEMG) into a multidimensional hand-synergy command and investigates residual reinforcement learning around that decoded baseline in `SimMuJoCo_Sidegrasp`; the later `SimUnity` study starts from recorded motion and a one-dimensional grasp reference, preserves the spatial trajectory, and applies residual reinforcement learning only to the grasp component.

Across both studies, the human-derived command remains explicit in the control loop. Reinforcement learning augments that command with bounded, task-level corrections informed by simulated hand–object interaction rather than replacing the human reference.

> This repository is a public research showcase for an ongoing Master's thesis. Source code, human-recorded datasets, model checkpoints, detailed experiment configurations, and complete numerical artifacts remain private.

## Research Progression

The two studies address related questions at different levels of representation and must be read as distinct experimental pipelines.

| Study | Human-derived reference | Learned component | Evidence shown here |
| --- | --- | --- | --- |
| **Study I — `SimMuJoCo_Sidegrasp`** | Multichannel sEMG decoded into a multidimensional hand-synergy command | Causal intention decoder followed by bounded residual control in synergy space | TCN/MLP held-out decoding evaluation, system architecture, and simulated manipulation sequence |
| **Study II — `SimUnity`** | Recorded spatial motion plus a one-dimensional grasp reference | Bounded scalar residual correction applied only to grasp | LORO protocol, held-out residual-control comparison, and contact-force/spectral analysis |

The figures and metrics from these studies are therefore grouped separately below. The TCN/MLP decoding results belong to Study I; the held-out control and force results belong to Study II.

## Key Contributions

- A first-stage sEMG pipeline that uses causal temporal modeling to estimate a compact hand-synergy command, with round-level comparison against an MLP reference model.
- A multidimensional residual-control formulation in `SimMuJoCo_Sidegrasp`, where the decoded TCN command remains the baseline and the learned policy provides bounded corrections in hand-synergy space.
- A subsequent one-dimensional residual-control formulation in `SimUnity`, where recorded spatial hand motion is replayed unchanged and adaptation is restricted to the grasp-related command.
- Distinct round-disjoint evaluation designs: held-out recording rounds for intention decoding and complete-trajectory leave-one-round-out evaluation for the SimUnity residual controllers.

## Study I — sEMG Decoding and Multidimensional Residual Control

The first study establishes the sEMG-driven control pathway used in `SimMuJoCo_Sidegrasp`. It first learns the relationship between human forearm muscle activity and a compact hand-synergy representation. The decoded synergy then serves as the human-derived baseline for a bounded residual reinforcement-learning controller in MuJoCo.

### Human sEMG Interface

Forearm activity is measured with a wearable multichannel surface-electromyography interface. The channels capture complementary muscle-activation patterns associated with hand motion and provide the input sequence for supervised intention decoding.

<p align="center">
  <img src="Assets/SemgSensor.jpg" width="42%" alt="Wearable multichannel sEMG interface positioned on the forearm">
</p>

<p align="center"><sub>Wearable forearm sEMG interface used to acquire muscle activity for the first study.</sub></p>

### Causal Intention Decoding: TCN and MLP

A causal Temporal Convolutional Network (TCN) maps recent multichannel sEMG history to the current hand-synergy representation. Its causal construction uses present and preceding signal samples without relying on future observations. An MLP provides a non-temporal reference model for the supervised decoding comparison.

The evaluation is organized by recording round. In each fold, a complete round is held out for testing, a separate round is used for validation, and the remaining rounds are used for training. This preserves round boundaries and prevents neighboring samples from the held-out recording from appearing in the training set.

<p align="center">
  <img src="Assets/Cross-Validation+指标.png" width="100%" alt="Round-level cross-validation and held-out TCN and MLP intention-decoding results">
</p>

<p align="center"><sub>Study I decoding evidence: round-level data allocation and held-out TCN/MLP measures across recording folds.</sub></p>

This figure evaluates the intention-decoding stage only. It does not report the later SimUnity residual-control experiment.

### Multidimensional Residual-Control Architecture

The decoded TCN output is retained as the baseline hand-synergy command. A residual policy observes that baseline together with robotic-hand state, object motion, contact information, interaction forces, and recent control history. It then proposes a bounded correction in the same synergy space. The refined synergy command is clipped to its valid range and mapped to the actuators of the Hannes robotic hand.

The residual formulation makes policy authority explicit: a zero residual reproduces the human-derived TCN command, while a nonzero residual represents the task-level intervention introduced by the controller.

<p align="center">
  <img src="Assets/主框架图新.png" width="100%" alt="sEMG decoding and multidimensional residual reinforcement-learning architecture">
</p>

<p align="center"><sub>Study I architecture: causal sEMG-to-synergy decoding followed by bounded residual adaptation in the MuJoCo grasping task.</sub></p>

```text
Multichannel sEMG
→ causal TCN decoding
→ human-derived synergy baseline
→ bounded multidimensional residual correction
→ Hannes hand actuation
→ hand, object, contact, and force feedback
```

### Demonstrated Manipulation Task

The `SimMuJoCo_Sidegrasp` task covers a complete contact-rich manipulation sequence: approach, grasp establishment, lift, transfer, placement, and release. It therefore evaluates control across changing contact conditions and sustained object transport rather than only an isolated finger-closing action.

<p align="center">
  <img src="Assets/抓握任务全程流程图.png" width="100%" alt="Complete SimMuJoCo manipulation sequence from approach to release">
</p>

<p align="center"><sub>Study I simulated manipulation sequence with the Hannes hand: approach, grasp, lift, transfer, placement, and release.</sub></p>

## Study II — Recorded One-Dimensional Grasp Control

The later `SimUnity` study examines a different control setting. A Unity/ROS workflow provides recorded hand motion and a one-dimensional grasp reference. The recorded spatial trajectory determines where the hand moves and is replayed without policy modification; residual reinforcement learning acts only on the scalar grasp command.

This design isolates grasp adaptation from trajectory generation. The learned controller can respond to kinematics, contact state, force changes, object behavior, and recent control history, but it cannot rewrite the human-provided spatial path.

### Recorded-Motion Control Architecture

The one-dimensional reference is converted into a baseline grasp command and mapped to coordinated Hannes finger actuation. The residual policy adds a bounded scalar correction around this baseline before the final grasp command is applied in MuJoCo. Spatial pose replay and grasp adaptation remain separate channels throughout the experiment.

<p align="center">
  <img src="Assets/Fig_Framework.png" width="100%" alt="SimUnity recorded-motion and one-dimensional residual grasp-control framework">
</p>

<p align="center"><sub>Study II architecture: unchanged Unity/ROS spatial replay combined with adaptive one-dimensional residual grasp control.</sub></p>

```text
Recorded Unity/ROS motion ─────────────→ unchanged spatial pose replay
Recorded one-dimensional grasp reference
→ baseline grasp command
→ bounded scalar residual correction
→ Hannes grasp actuation
→ contact and dynamics feedback
```

### Leave-One-Round-Out Evaluation

The SimUnity residual controllers are assessed with an eight-fold leave-one-round-out (LORO) protocol. One complete recorded round is held out for evaluation, the remaining rounds provide the training data for that fold, and the procedure is repeated until each round has served as the test trajectory. No random train/test mixing is performed between neighboring temporal samples from the same recording.

<p align="center">
  <img src="Assets/Fig_loro_gesture.png" width="100%" alt="SimUnity manipulation phases and leave-one-round-out evaluation protocol">
</p>

<p align="center"><sub>Study II evaluation: representative task phases and complete-round LORO fold construction.</sub></p>

### Held-Out Residual-Control Evidence

The held-out comparison is specific to the one-dimensional SimUnity study. It contrasts the recorded baseline, residual TD3 without the behavior regularization used by the proposed controller, and the behavior-regularized residual policy.

The panels visualize stable-hold time proportion, unsafe-step time proportion, residual-control magnitude, and the tangential-to-normal interaction-force ratio. Baseline-deviation RMSE is a control quantity: it measures the magnitude of the applied residual relative to the recorded human-derived grasp baseline, not intention-decoding error. Likewise, stable-hold time describes the fraction of evaluated time spent in the configured hold state rather than an object-level grasp-success probability.

<p align="center">
  <img src="Assets/Fig_Overall.png" width="100%" alt="SimUnity held-out comparison of baseline residual TD3 and behavior-regularized residual control">
</p>

<p align="center"><sub>Study II held-out results across complete recorded rounds: baseline, residual TD3, and behavior-regularized residual control.</sub></p>

### Contact-Force and Spectral Analysis

The final SimUnity analysis examines normal force, tangential force, and the tangential-to-normal force ratio during valid simulated contact. A representative held-out round provides time-domain views of friction demand and stable-contact force, while cross-round summaries characterize changes relative to the baseline. Welch spectral analysis provides a complementary frequency-domain view of force variation and control smoothness.

<p align="center">
  <img src="Assets/Fig.Friction_Spectrum.png" width="100%" alt="SimUnity contact-force friction-ratio and Welch spectral analysis">
</p>

<p align="center"><sub>Study II contact analysis: representative held-out force behavior, cross-round summaries, and Welch spectral content.</sub></p>

## Relationship Between the Two Studies

The studies share the same human-intent-preserving principle but answer different experimental questions.

| Aspect | Study I: `SimMuJoCo_Sidegrasp` | Study II: `SimUnity` |
| --- | --- | --- |
| Starting signal | Multichannel forearm sEMG | Recorded Unity/ROS motion and scalar grasp reference |
| Baseline representation | Multidimensional decoded hand synergy | One-dimensional recorded grasp command |
| Learned correction | Residual in synergy space | Residual in scalar grasp space |
| Spatial motion | Generated within the first simulated manipulation setup | Replayed from the recording and kept unchanged |
| Evaluation evidence | Round-held-out TCN/MLP decoding and simulated task demonstration | Complete-round LORO control, force, and spectral results |

The second study extends the residual-control principle to recorded motion; it is not a continuation of the TCN/MLP metric series and does not reuse those decoding results as SimUnity outcome measures.

## Research Scope

| Area | Role in the project |
| --- | --- |
| Human intention decoding | Causal mapping from multichannel sEMG to a compact hand-synergy representation |
| Dexterous manipulation | Contact-rich grasping and object transport with the Hannes robotic hand |
| Residual reinforcement learning | Bounded correction around an explicit human-derived baseline |
| Human-centered / shared control | Preservation of human command authority while adding feedback-driven assistance |
| Generalization evaluation | Round-disjoint decoder evaluation and complete-trajectory LORO control assessment |

## Public Release Scope

**Included**

- High-level architecture and experimental design for both research studies
- Selected task, protocol, and evaluation figures
- Representative descriptive evidence

**Not publicly released**

- Human-recorded datasets and participant-related information
- Training, preprocessing, and evaluation source code
- Model checkpoints and internal experiment configurations
- Detailed reward coefficients, complete numerical results, and unpublished ablations

## Research Status

This is ongoing Master's thesis research. The current public showcase is simulation-focused and documents two successive experimental studies with different control representations and evaluation protocols. Full methodology, implementation details, statistical analysis, and supporting research artifacts remain part of the thesis and associated research outputs.

## Citation

This repository presents ongoing Master's thesis research by **Shiwei Chen**.

Formal thesis and publication citations will be added after the associated work becomes publicly available. Please contact the author before redistributing unpublished figures or research material.

