# SCALE_MODULES/README.md

## Scale Modules Index

**Version:** v1.0  
**Status:** Active  
**Created:** 2026-03-17

---

## Overview

This document is the index directory for NoiePhysicsAGENTS scale modules. According to the physical scale permission hierarchy defined in NoiePhysicsAGENTS.md §1, these modules provide physical frameworks from sub-quantum to cosmic scales.

---

## Scale Level Reference Table

| PS-L Level | Name | Characteristic Scale | Dominant Physical Theory | Corresponding Module |
|------------|------|---------------------|------------------------|---------------------|
| **PS-L(-1)** | Sub-quantum/Topological | < 10⁻³⁵ m | Quantum Gravity | `QUANTUM_GRAVITY.md` |
| **PS-L0** | Quantum | 10⁻³⁵ ~ 10⁻⁹ m | Quantum Mechanics, QFT | `QUANTUM_MECHANICS.md`, `QUANTUM_FIELD_THEORY.md` |
| **PS-L1** | Micro/Statistical | 10⁻⁹ ~ 10⁻³ m | Statistical Mechanics | `STATISTICAL_MECHANICS.md` |
| **PS-L2** | Human/Classical | 10⁻³ ~ 10³ m | Classical Mechanics | `CLASSICAL_MECHANICS.md`, `CONTINUUM_MECHANICS.md`, `FLUID_DYNAMICS.md` |
| **PS-L3** | Earth/Geological | 10³ ~ 10⁷ m | Continuum Mechanics | `CONTINUUM_MECHANICS.md` |
| **PS-L4** | Celestial/Relativistic | > 10⁷ m | General Relativity | `GENERAL_RELATIVITY.md` |
| **PS-LR** | Relativistic Effects | v > 0.1c | Special Relativity | `SPECIAL_RELATIVITY.md` |
| **-** | Plasma | Various scales | Plasma Physics | `PLASMA_PHYSICS.md` |

---

## Module Descriptions

### QUANTUM_GRAVITY.md

**Title:** Quantum Gravity  
**Scale:** PS-L(-1) (< 10⁻³⁵ m)  
**Description:** Handles spacetime microstructure and Planck scale physics

**Core Contents:**
- String theory foundations
- Loop quantum gravity
- Spacetime emergence
- Planck scale phenomena

---

### QUANTUM_MECHANICS.md

**Title:** Quantum Mechanics  
**Scale:** PS-L0 (10⁻³⁵ ~ 10⁻⁹ m)  
**Description:** Handles quantum phenomena at atomic and molecular scales

**Core Contents:**
- Wave functions and Schrödinger equation
- Measurement theory
- Quantum state evolution
- Uncertainty principle

---

### QUANTUM_FIELD_THEORY.md

**Title:** Quantum Field Theory  
**Scale:** PS-L0 (10⁻³⁵ ~ 10⁻⁹ m)  
**Description:** Handles quantum fields and particle physics

**Core Contents:**
- Field quantization
- Standard Model
- Feynman diagrams
- Renormalization

---

### STATISTICAL_MECHANICS.md

**Title:** Statistical Mechanics  
**Scale:** PS-L1 (10⁻⁹ ~ 10⁻³ m)  
**Description:** Statistical description connecting micro to macro scales

**Core Contents:**
- Ensemble theory
- Thermodynamics
- Phase transitions
- Critical phenomena

---

### CLASSICAL_MECHANICS.md

**Title:** Classical Mechanics  
**Scale:** PS-L2 (10⁻³ ~ 10³ m)  
**Description:** Handles classical physical phenomena at human scale

**Core Contents:**
- Newton's laws of motion
- Lagrangian mechanics
- Hamiltonian mechanics
- Rigid body dynamics
- Fundamentals of elasticity
- Vibrations and waves

---

### CONTINUUM_MECHANICS.md

**Title:** Continuum Mechanics  
**Scale:** PS-L2, PS-L3 (10⁻³ ~ 10⁷ m)  
**Description:** Continuum description of macroscopic matter

**Core Contents:**
- Elasticity
- Plasticity
- Stress-strain relations
- Finite deformation

---

### FLUID_DYNAMICS.md

**Title:** Fluid Dynamics  
**Scale:** PS-L2, PS-L3 (10⁻³ ~ 10⁷ m)  
**Description:** Handles fluid motion

**Core Contents:**
- Navier-Stokes equations
- Laminar and turbulent flow
- Boundary layers
- Multiphase flow

---

### PLASMA_PHYSICS.md

**Title:** Plasma Physics  
**Scale:** Various scales  
**Description:** Handles ionized gases

**Core Contents:**
- MHD equations
- Plasma diagnostics
- Nuclear fusion
- Space weather

---

### SPECIAL_RELATIVITY.md

**Title:** Special Relativity  
**Scale:** PS-LR (v > 0.1c)  
**Description:** Handles high-speed motion

**Core Contents:**
- Minkowski spacetime
- Time dilation/length contraction
- Mass-energy equivalence
- Lorentz transformations

---

### GENERAL_RELATIVITY.md

**Title:** General Relativity  
**Scale:** PS-L4 (> 10⁷ m)  
**Description:** Handles gravity and spacetime geometry

**Core Contents:**
- Einstein field equations
- Black hole physics
- Cosmology
- Gravitational waves

---

## Usage Guide

### When to Use These Modules

According to the scale selection logic in NoiePhysicsAGENTS.md §1.2:

1. **Automatic Selection:** When the characteristic scale of the physical environment is clear, the system automatically selects the corresponding scale module
2. **Manual Selection:** When performing cross-scale operations, manual switching of scale modules is required
3. **Mixed Usage:** When involving multi-scale coupling, multiple scale modules can be loaded simultaneously

### Cross-Scale Coupling

When operations involve multiple scales, refer to the cross-scale coupling mechanism in NoiePhysicsAGENTS.md §1.1:

```
┌─────────────────────────────────────────────────────────────┐
│                 Cross-Scale Coupling Scenarios              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Macro→Quantum:                                            │
│    Trigger: Macro entity manipulates quantum-level objects  │
│    Protocol: Compute energy scale, evaluate decoherence     │
│                                                             │
│  Quantum→Macro:                                            │
│    Trigger: Quantum effects influence macro behavior       │
│    Protocol: Track quantum state evolution, compute        │
│              decoherence time                               │
│                                                             │
│  Thermal↔Mechanical:                                       │
│    Trigger: Thermal perturbations coupled with mechanical  │
│             motion                                         │
│    Protocol: Thermo-mechanical coupling analysis            │
│                                                             │
│  Entanglement↔Geometry:                                    │
│    Trigger: Entanglement changes lead to effective         │
│             geometric changes                              │
│    Protocol: Ryu-Takayanagi evaluation                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Dynamic Loading Strategy

According to the context loading strategy in NoiePhysicsAGENTS.md §3:

- **L1 (Root file):** Always load NoiePhysicsAGENTS.md
- **L2 (Core layer):** Load AXIOMS, FIELD_PERCEPTION, DYNAMICS_ENGINE, PHYSICS_KNOWLEDGE, SAFETY_PROTOCOLS based on task type
- **L3 (Scale layer):** Load corresponding modules only when specific scale physics is explicitly needed

**It is strictly prohibited to load all scale dynamics manifolds simultaneously** to prevent computational resource waste and context window pollution.

---

## Module Compatibility

| Module | Dependencies | Conflicts |
|--------|-------------|-----------|
| QUANTUM_GRAVITY | AXIOMS | - |
| QUANTUM_MECHANICS | AXIOMS | - |
| QUANTUM_FIELD_THEORY | AXIOMS, QUANTUM_MECHANICS | - |
| STATISTICAL_MECHANICS | AXIOMS | - |
| CLASSICAL_MECHANICS | AXIOMS | - |
| CONTINUUM_MECHANICS | AXIOMS, STATISTICAL_MECHANICS | - |
| FLUID_DYNAMICS | AXIOMS, CONTINUUM_MECHANICS | - |
| PLASMA_PHYSICS | AXIOMS, FLUID_DYNAMICS | - |
| SPECIAL_RELATIVITY | AXIOMS | GENERAL_RELATIVITY |
| GENERAL_RELATIVITY | AXIOMS | SPECIAL_RELATIVITY |

---

## Evolution Log

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2026-03-17 | Initial version |

---

*This document is the index directory for NoiePhysicsAGENTS scale modules.*
*Refer to individual module documents for detailed content.*
