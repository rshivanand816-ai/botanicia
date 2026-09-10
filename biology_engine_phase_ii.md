# BioGenesis Phase II: Biology Engine Architecture Expansion Specification
**Version:** 2.0.0-draft  
**Status:** Architectural Blueprint  
**Authors:** Joint Taskforce of Computational Biologists, Systems Architects, and Simulation Engineers

---

## Executive Summary
This document specifies the Phase II expansion of the **BioGenesis Biology Engine**. Phase I established a static semantic knowledge representation map. Phase II transforms this map into a dynamic, continuous-time simulation platform. By combining ordinary differential equations (ODEs), stochastic reaction solvers (Gillespie algorithm), constraint-based metabolic flux models, and an Entity-Component-System (ECS) graph architecture, the engine models biological life from single-nucleotide transcription rates to systemic human organ homeostasis.

---

## System Overview & Layered Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          ENVIRONMENT ENGINE                             │
│     (Radiation, Pathogens, Toxins, Nutrients, Vitals, Age, Stress)      │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ (Alters local rates / coefficients)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       BIOLOGICAL PHYSICS LAYER                          │
│     (Diffusion, Osmosis, Nernst Potentials, Fluid Dynamics, pH, Temp)    │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ (Establishes gradient states)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ENTITY-COMPONENT-SYSTEM (ECS)                        │
│          (Systems: Transcription, Translation, Folding, etc.)           │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ (Computes state updates)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         RULE & STATE ENGINES                            │
│           (Kinetics, State Transitions, Dynamic Feedbacks)              │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ (Serializes state vector)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       VISUAL SIMULATION ENGINE                          │
│       (Pulsing Voids, Blood flow heatmaps, Microscopic visualizers)     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## TASK 1: Critical Analysis of Phase I Architecture

A critical review of the Phase I architecture reveals several fundamental limitations:

### 1. Static Graph Representation vs. Dynamic States
- **What Exists**: Vertices representing genes, proteins, and cells connected by semantic edges (e.g. `Gene` $\rightarrow$ `Protein`).
- **What is Missing**: Dynamic state variables, chemical concentrations, and time-dependent rates.
- **Why it is Missing**: Phase I prioritized database mapping over execution.
- **How it should Evolve**: Move from a static relationship graph to an **Entity-Component-System (ECS)** where nodes contain numerical state components, and systems run physics/kinetics scripts on these nodes.

### 2. Lack of Feedback Loop Execution
- **What Exists**: Hardcoded pathways mapping specific inputs to predefined outputs.
- **What is Missing**: True cellular homeostatic feedback (e.g., hypoxia triggering EPO expression, which in turn increases erythrocyte count to resolve the hypoxia).
- **Why it is Missing**: Hardcoded rules do not scale when multiple mutations interact.
- **How it should Evolve**: Implement a unified event-driven **Rule Engine** executing on topological sweep order, with a numerical state integration loop.

### 3. Missing Physical Constraint Layers
- **What Exists**: Abstract descriptions of tissue functions (e.g., "viscous mucus").
- **What is Missing**: Conservation of mass, thermodynamic constraints (Gibbs free energy), fluid mechanics, and concentration diffusion profiles.
- **Why it is Missing**: Simulated systems were represented as localized text descriptors rather than numerical state vectors.
- **How it should Evolve**: Implement a **Biological Physics Layer** that defines global thermodynamic, osmotic, and fluid dynamics equations that constrain rate constants.

---

## TASK 2: Simulation Layer (State Variables)

Every biological entity maintains a dynamic state component defined by the following variables.

### 1. DNA Integrity
- **Purpose**: Tracks cumulative double-strand and single-strand DNA breaks.
- **Range**: $[0.0, 1.0]$ (where $1.0$ is pristine).
- **Units**: Dimensionless ratio.
- **Default Value**: $1.0$
- **Relationships**: Decreases with radiation, UV, and oxidative stress; increases with DNA repair rate.
- **Update Rule**: 
  $$\frac{d(\text{DNA\_Integrity})}{dt} = -(\alpha_{UV} \cdot I_{UV} + \alpha_{ROS} \cdot [\text{ROS}]) + \beta_{repair} \cdot \text{Repair\_Efficiency}$$
- **Visualization**: Chromatid fragment animations, glow intensity maps.

### 2. Chromatin Accessibility
- **Purpose**: Represents local chromatin opening at promoter loci.
- **Range**: $[0.0, 1.0]$ (where $1.0$ is completely open euchromatin).
- **Units**: Dimensionless probability.
- **Default Value**: $0.8$ for active housekeep genes; $0.1$ for tissue-specific genes.
- **Relationships**: Modulated by Histone Acetylation (+) and DNA Methylation (-).
- **Update Rule**:
  $$\text{Accessibility} = \frac{1}{1 + e^{-\gamma (\text{Acetylation} - \text{Methylation})}}$$
- **Visualization**: Double-helix unwinding animation, glow radius changes.

### 3. Transcript Abundance
- **Purpose**: Measures mature mRNA copy count in the cytoplasm.
- **Range**: $[0, 10^5]$
- **Units**: Copies per cell.
- **Default Value**: $100$
- **Relationships**: Increased by Transcription rate; decreased by mRNA decay.
- **Update Rule**:
  $$\frac{d[\text{mRNA}]}{dt} = \text{Transcription\_Rate} - k_{decay} \cdot [\text{mRNA}]$$
- **Visualization**: Floating particle count in microscopic render.

### 4. Protein Folding Score
- **Purpose**: Measures the ratio of correctly folded tertiary proteins to total proteins.
- **Range**: $[0.0, 1.0]$
- **Units**: Ratio.
- **Default Value**: $0.98$
- **Relationships**: Decreases with heat stress and mutations; increases with chaperone concentration.
- **Update Rule**:
  $$\text{Folding\_Score} = \frac{1}{1 + e^{-(\Delta G_{folding} + \beta_{chaperone} \cdot [\text{Chaperone}]) / RT}}$$
- **Visualization**: Structural ribbon diagrams morphing from folded to uncoiled.

### 5. Binding Affinity
- **Purpose**: Defines molecular interactions (ligand-receptor, enzyme-substrate).
- **Range**: $[10^{-12}, 10^{-3}]$
- **Units**: Molar ($M$, Dissociation constant $K_d$).
- **Default Value**: $10^{-6}$
- **Relationships**: Directly affects signal transduction rates.
- **Update Rule**:
  $$K_d = e^{\Delta G_{binding} / RT}$$
- **Visualization**: Receptors docking animations with neon connection pulses.

### 6. Catalytic Efficiency
- **Purpose**: Measures enzyme substrate conversion capability.
- **Range**: $[10^3, 10^9]$
- **Units**: $M^{-1}s^{-1}$ ($k_{cat} / K_m$).
- **Default Value**: $10^5$
- **Relationships**: Dictates metabolic flux rates.
- **Update Rule**:
  $$k_{cat} = k_{cat\_wild} \cdot \text{Folding\_Score} \cdot (1.0 - \text{Active\_Site\_Disruption})$$
- **Visualization**: Substrate cleavage rate indicator bars.

### 7. Aggregation Probability
- **Purpose**: Tracks misfolded protein self-assembly into toxic amyloid structures.
- **Range**: $[0.0, 1.0]$
- **Units**: Probability.
- **Default Value**: $0.001$
- **Relationships**: High when Folding Score is low.
- **Update Rule**:
  $$\text{Aggregation\_Rate} = k_{agg} \cdot [\text{Protein}_{misfolded}]^2$$
- **Visualization**: Sticky fibers growing on cell membranes.

### 8. Membrane Localization
- **Purpose**: Measures the fraction of target proteins successfully localized to active membranes.
- **Range**: $[0.0, 1.0]$
- **Units**: Ratio.
- **Default Value**: $0.95$
- **Relationships**: Drops to zero if target targeting sequences are deleted.
- **Update Rule**:
  $$\text{Localization} = \text{Folding\_Score} \cdot \mathbb{I}(\text{Signal\_Peptide\_Intact})$$
- **Visualization**: Channel protein glowing arrays on cell boundaries.

### 9. ATP Consumption Rate
- **Purpose**: Measures metabolic energy demand of active systems.
- **Range**: $[0.0, 1000.0]$
- **Units**: $\mu\text{mol/sec/cell}$
- **Default Value**: $10.0$
- **Relationships**: Increases with active transport, repair, and transcription.
- **Update Rule**:
  $$\text{ATP\_Cons} = \sum J_{active\_transport} + J_{transcription} + J_{translation} + J_{repair}$$
- **Visualization**: Pulsing orange battery status indicators.

### 10. Reactive Oxygen Species (ROS)
- **Purpose**: Monitors intracellular oxidative stress levels.
- **Range**: $[0.0, 100.0]$
- **Units**: $\mu M$
- **Default Value**: $1.0$
- **Relationships**: Generated by mitochondria; cleared by catalase/SOD.
- **Update Rule**:
  $$\frac{d[\text{ROS}]}{dt} = \text{Mito\_Leak\_Rate} - k_{scavenger} \cdot [\text{Antioxidants}] \cdot [\text{ROS}]$$
- **Visualization**: Spiky neon-purple particle clouds surrounding mitochondria.

### 11. Cell Viability
- **Purpose**: Monitors health of local cell populations.
- **Range**: $[0.0, 100.0]$
- **Units**: Percent.
- **Default Value**: $100.0$
- **Relationships**: Falls as damage, ATP deficits, or inflammatory triggers accumulate.
- **Update Rule**:
  $$\frac{d(\text{Viability})}{dt} = -(\lambda_{necrosis} \cdot \mathbb{I}(\text{ATP} < \text{Threshold}) + \lambda_{apoptosis} \cdot \text{Caspase\_Active})$$
- **Visualization**: Color shifting from cyan to ash-gray.

### 12. Inflammatory State
- **Purpose**: Tracks local tissue immune mobilization.
- **Range**: $[0.0, 10.0]$
- **Units**: Dimensionless index.
- **Default Value**: $0.0$
- **Relationships**: Driven by cytokine secretion.
- **Update Rule**:
  $$\frac{d(\text{Inflammation})}{dt} = \kappa \cdot [\text{TNF\_alpha}] - d_{clear} \cdot \text{Inflammation}$$
- **Visualization**: Pulsing red overlays across tissue boundaries.

### 13. Organ Performance
- **Purpose**: Tracks macro-organ health capacity.
- **Range**: $[0.0, 100.0]$
- **Units**: Percent.
- **Default Value**: $100.0$
- **Relationships**: Aggregates tissue health parameters.
- **Update Rule**:
  $$\text{Performance} = \text{Mean}(\text{Tissue\_Viability}) \times (1.0 - \text{Damage\_Index})$$
- **Visualization**: Dynamic dial gauge consoles with warning sirens.

### 14. Oxygen Delivery Rate
- **Purpose**: Tracks system-wide oxygen delivery to capillary beds.
- **Range**: $[0.0, 100.0]$
- **Units**: $mL/min/kg$
- **Default Value**: $5.0$ at rest.
- **Relationships**: Dictated by cardiac output, hemoglobin affinity, and alveolar ventilation.
- **Update Rule**:
  $$\text{O2\_Delivery} = \text{Cardiac\_Output} \times [\text{Hemoglobin}] \times \text{Saturation}_{\text{arterial}}$$
- **Visualization**: Flow particle streams shifting from bright red to blue-purple.

### 15. Homeostasis Score
- **Purpose**: Systemic metric for whole organism stability.
- **Range**: $[0.0, 1.0]$
- **Units**: Score.
- **Default Value**: $1.0$
- **Relationships**: Falls when pH, temperature, or oxygen deviate from nominal limits.
- **Update Rule**:
  $$\text{Homeostasis} = \prod (1.0 - \text{Deviation}_{\text{vitals}})$$
- **Visualization**: Master EKG monitoring feed stability.

---

## TASK 3: Biological State Engine

The engine models state transitions using a continuous finite-state machine (FSM) governed by physiological thresholds.

```
       ┌───────────┐    Recovery    ┌──────────────┐
       │  Healthy  │◄───────────────┤  Recovering  │
       └─────┬─────┘                └──────▲───────┘
             │                             │
             │ Stress / Mutation           │ Resolution
             ▼                             │
       ┌───────────┐    Therapy     ┌──────┴───────┐
       │Stressed / ├───────────────►│  Repairing   │
       │ Inflamed  │                └──────▲───────┘
       └─────┬─────┘                       │
             │                             │ Healing
             │ Critical Strain             │
             ▼                             │
       ┌───────────┐    Failure     ┌──────┴───────┐
       │ Damaged / ├───────────────►│  Compensating │
       │ Critical  │                └──────────────┘
       └─────┬─────┘
             │ Severe Damage
             ▼
       ┌───────────┐
       │   Dead    │
       └───────────┘
```

### Transition Logic:
- **Healthy $\rightarrow$ Stressed**: Triggered when $ROS > 5.0\,\mu M$ or ATP generation drops by $20\%$.
- **Stressed $\rightarrow$ Damaged**: Triggered when cell viability drops below $80\%$, or membrane potential falls below $-50\,mV$.
- **Damaged $\rightarrow$ Repairing**: Triggered when the environment provides sufficient amino acid reserves and ATP levels exceed $60\%$.
- **Compensating**: A state where one parameter is degraded (e.g. anemia) but systemic vitals are sustained by boosting secondary systems (e.g., increasing heart rate to 120 BPM to sustain oxygen delivery).

---

## TASK 4: Rule Engine (Executable Edges)

Instead of static semantic links, relationship edges in the graph contain executable rule scripts with clear execution rules.

### Rule Execution Model:
- **Rule Definition**:
  ```typescript
  interface BioRule {
    id: string;
    priority: number; // [0 = lowest, 100 = highest]
    conditions: () => boolean;
    consequences: (delta: Map<string, number>) => void;
  }
  ```
- **Rule Conflict Resolution**:
  1. **Topological Order**: Rules are evaluated starting at the genotype layer and propagating downstream.
  2. **Priority Sweeps**: Within a layer, rules with higher priority are executed first.
  3. **Delta Blending**: Changes are stored in a temporary delta buffer. When conflicts occur (e.g. system A tries to increase cell size, system B tries to shrink it), the values are blended using a weighted average.

---

## TASK 5: Processes as First-Class Objects

Biological processes are modeled as active objects that run continuous update loops.

### Process Registry Matrix

| Process | Inputs | Outputs | Base Rate | Failure Mode | Associated Diseases |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Replication** | dNTPs, DNA, Helicase | Duplicated DNA | $1000\,bp/s$ | Replication fork collapse | Bloom Syndrome |
| **Transcription** | NTPs, DNA, TFs | pre-mRNA | $50\,nt/s$ | Polymerase stalling | Thalassemia |
| **Splicing** | pre-mRNA, Spliceosomes | Mature mRNA | $1.5\,min/exon$ | Exon skipping | Spinal Muscular Atrophy |
| **Translation** | AminoAcids, mRNA, tRNAs | Polypeptide | $15\,AA/s$ | Ribosome stalling | Diamond-Blackfan Anemia |
| **Protein Folding** | Nascent Peptide, ATP | Folded Protein | Milliseconds | Hydrophobic aggregation | Cystic Fibrosis, Cataracts |
| **Apoptosis** | Caspase-3, cytochrome-c | Apoptotic bodies | $2-6$ Hours | Fail to activate | Oncogenesis, Leukemia |
| **DNA Repair** | DNA damage, Ligase | Repaired DNA | $10-50\,bp/hr$ | Error insertion | Xeroderma Pigmentosum |
| **Metabolism** | Glucose, Oxygen, ADP | ATP, CO2, H2O | Variable | Lactic acidosis | Mitochondrial Myopathy |

---

## TASK 6: Temporal Dynamics (Time Engine)

The simulation engine decouples visual rendering from physical calculations using a variable-step integrator.

```
                  [ Realtime Frame tick ]
                             │
                             ▼
                ┌──────────────────────────┐
                │ Calculate elapsed time   │
                │        (dt_real)         │
                └────────────┬─────────────┘
                             │
                             ▼
                ┌──────────────────────────┐
                │ Scale by user multiplier │
                │   (dt_sim = dt * scale)  │
                └────────────┬─────────────┘
                             │
                             ├──────────────────────────┐
                             ▼                          ▼
                ┌──────────────────────────┐  ┌──────────────────────────┐
                │ Loop: step by micro-ticks│  │   Stochastic System      │
                │  (Run ODE integration)   │  │  (Run Gillespie Solver)  │
                └────────────┬─────────────┘  └─────────┬────────────────┘
                             │                          │
                             └────────────┬─────────────┘
                                          │
                                          ▼
                             ┌──────────────────────────┐
                             │ Push state changes to UI │
                             └──────────────────────────┘
```

- **Simulation Ticks**: The physics engine ticks at $20\,Hz$ (50ms cycles).
- **Temporal Latency**: Epigenetic changes (e.g. methylation) have a time delay constant $\tau = 10^5$ seconds, creating delayed effects that appear hours after exposure.
- **Historical State Log**: Maintains a circular ring buffer of state vectors over the last 10,000 steps, enabling rewinding and playbacks.

---

## TASK 7: Biological Physics Layer

Biological processes are constrained by physical laws:

- **Diffusion**: Governed by Fick's Second Law:
  $$\frac{\partial \phi}{\partial t} = D \nabla^2 \phi$$
  Models hormone, cytokine, and oxygen diffusion gradients through cellular boundaries.
- **Ion Gradients & Electrochemical Potentials**: Governed by the Nernst-Planck equation:
  $$E_m = \frac{RT}{F} \ln \left( \frac{P_{K}[\text{K}^+]_{out} + P_{Na}[\text{Na}^+]_{out} + P_{Cl}[\text{Cl}^-]_{in}}{P_{K}[\text{K}^+]_{in} + P_{Na}[\text{Na}^+]_{in} + P_{Cl}[\text{Cl}^-]_{out}} \right)$$
  Calculates membrane potential deviations inside cell membranes.
- **Cardiovascular Fluid Dynamics**: Governed by Poiseuille's law:
  $$\Delta P = \frac{8 \mu L Q}{\pi R^4}$$
  Determines systemic blood pressure changes when capillary structures collapse.

---

## TASK 8: Environmental Engine

The environment applies external pressures by modifying rate equations:

- **Radiation**: Directly introduces DNA double-strand breaks:
  $$\Delta(\text{DNA\_Damage}) = \text{Radiation\_Rad} \times \text{Exposure\_Time} \times (1.0 - \text{Melanin\_Index})$$
- **Sleep Quality**: Affects the clearance rate of metabolic waste products in brain tissue:
  $$\text{Clearance}_{\text{amyloid}} = \text{Clearance}_{\text{basal}} \times \text{Sleep\_Quality\_Index}$$
- **Medications**: Binds competitive active sites:
  $$V = \frac{V_{max} [S]}{K_m (1 + [I]/K_i) + [S]}$$
  Where $[I]$ is the medication concentration.

---

## TASK 9: Visual Simulation Engine

Dynamic telemetry variables link directly to visual parameters:

- **Cell Viability $\rightarrow$ Color Shift**: Cell color shifts from neon cyan to dark ash gray:
  $$\text{Color} = \text{Lerp}(\text{Cyan}, \text{Gray}, 1.0 - \text{Viability}/100.0)$$
- **Vaso-occlusion $\rightarrow$ Microscopic Particle Streams**: Flow velocity determines the speed of floating red blood cell particles.
- **Acidosis $\rightarrow$ Tissue Glow**: When local pH drops below 7.3, the tissue emits a pulsing orange border glow.

---

## TASK 10: Speculative Biology & Evidence Mapping

To balance game discovery with real science, every relationship node includes an **Evidence Certainty Parameter (ECP)**.

| Relationship Edge | ECP Value | Source | Confidence | Classification |
| :--- | :--- | :--- | :--- | :--- |
| **HBB (GTG) $\rightarrow$ HbS Polymerization** | $0.99$ | ClinVar / OMIM | Confirmed | Established Science |
| **CFTR (F508) $\rightarrow$ folding Degradation** | $0.98$ | PubMed ID 182901 | Confirmed | Established Science |
| **Synthetic Enzyme X $\rightarrow$ Amyloid Cleavage** | $0.35$ | Hypothetical | Low | Speculative Simulation |
| **CRISPR-Cas9 Vector Y $\rightarrow$ HBB Locus Correction** | $0.85$ | Clinical Trial Phase II | High | Predicted Biology |

---

## TASK 11: Extensibility & ECS Architecture

To support massive scaling without redesign:
- **Entity-Component-System (ECS)**:
  - **Entities**: Simple numeric IDs representing genes, proteins, cells, or organs.
  - **Components**: Plain data structures (e.g. `PositionComponent`, `ConcentrationComponent`, `DNASeqComponent`).
  - **Systems**: Isolated execution loops processing specific components (e.g. `TranscriptionSystem` queries all entities with both `DNASeqComponent` and `TranscriptionFactorComponent`).
- **Dynamic Registration**: Systems are loaded via dynamic libraries. Adding a drug simulation system does not require modifying transcription or translation loops.

---

## TASK 12: Architectural Critique & Bottleneck Analysis

A critical analysis identifies several potential bottlenecks:

- **Computational Complexity of ODE Solvers**: Solving thousands of ordinary differential equations (ODEs) per tick for cell metabolism would crash the browser main thread.
  - *Redesign*: We utilize a **hybrid multi-scale solver**. Genotype and translation layers use event-based triggers; metabolism uses constraint-based Flux Balance Analysis (FBA) calculated only when state boundaries shift.
- **High-Frequency Graph Updates**: Graph traversal algorithms (BFS/DFS) on 20,000+ nodes will lag.
  - *Redesign*: Use a **Dependency DAG**. When a value changes, the engine schedules updates only for downstream dependents, skipping unaffected nodes entirely.
- **Serialization Size**: Logging history arrays for thousands of entities will quickly saturate memory limits.
  - *Redesign*: Use a **Sparse State Delta Log** where only changed fields are recorded in history buffers, rather than duplicate copies of the entire state vector.
