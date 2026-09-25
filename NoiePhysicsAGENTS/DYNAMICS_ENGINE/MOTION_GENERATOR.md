# MOTION_GENERATOR.md

## L3 — Motion equations and planning models v2.3

**Scope:** Select and describe mathematical models for motion, constraints, trajectory prediction, and action-conditioned state forecasts. This is a reference specification, not an equation generator, controller, sensor, actuator, or simulator.

## 1. Model declaration

A motion model must state:

- system boundary, bodies or fields, degrees of freedom, coordinate frame, units, and time base;
- generalized coordinates or state variables, initial conditions, and uncertainty;
- governing equations, force/interaction laws, material parameters, and their provenance;
- constraints, environment, boundary conditions, and omitted effects;
- validity range and whether the model is conservative, open, dissipative, stochastic, relativistic, quantum, or hybrid;
- numerical method, tolerances, event handling, convergence checks, and runtime identity if a solver was actually used.

Do not infer a unique model from a label such as “rigid body,” “robot,” or “agent.” Different representations may be appropriate for different scales and observables.

## 2. Lagrangian formulation

For generalized coordinates q and velocities q̇, define a Lagrangian L(q,q̇,t), often T−V for a conservative mechanical system. The Euler–Lagrange equations with generalized non-conservative forces Qᵢ are:

$$\frac{d}{dt}\frac{\partial L}{\partial \dot q_i}-\frac{\partial L}{\partial q_i}=Q_i.$$

This representation is useful when coordinates and constraints simplify the problem. It assumes the chosen L and force terms adequately represent the system; dissipation, impacts, control inputs, and open-system fluxes require explicit treatment.

For holonomic constraints fₐ(q,t)=0, Lagrange multipliers may represent constraint forces. Nonholonomic velocity constraints require a declared variational or d'Alembert formulation; they cannot be converted to holonomic constraints by notation alone. Record constraint rank, admissibility, and treatment of redundant or singular constraints.

## 3. Hamiltonian formulation

Where the Legendre transform is regular, define canonical momenta pᵢ=∂L/∂q̇ᵢ and H(q,p,t)=Σᵢpᵢq̇ᵢ−L. Hamilton's equations are:

$$\dot q_i=\frac{\partial H}{\partial p_i},\qquad \dot p_i=-\frac{\partial H}{\partial q_i}.$$

A Hamiltonian formulation is not automatically available in the same form for every constrained, dissipative, or singular system. The Hamiltonian equals total mechanical energy only under appropriate time-independence and model conditions. State the symplectic structure and constraint handling when these matter.

## 4. Numerical propagation and trajectory planning

A prediction requires an initial state, model, input sequence, and solver. Select integrators based on stiffness, smoothness, conservation structure, constraints, and discontinuities. Symplectic methods can preserve geometric structure for suitable Hamiltonian systems; they still have discretization error. Adaptive integration controls local error only under its solver assumptions and does not guarantee stability or detect every event.

For a trajectory plan, declare state transition model, objective, admissible controls, physical constraints, horizon, and uncertainty. A generic constrained problem may minimize an accumulated cost subject to the model and state/control bounds, but the objective and constraints are domain-specific. Report infeasible constraints, local-optimum limits, sensitivity, and whether the result is a plan, simulated trajectory, or observed motion.

Collision events, impacts, contact, actuator saturation, and mode changes may require event-driven or hybrid integration. Check step-size convergence, conservation or balance laws justified by the model, and limiting cases. A numerically converged trajectory remains conditional on the inputs and model validity.

## 5. Expected free energy and action-conditioned inference

Active inference is one model-based planning framework. In a discrete generative model, a common risk/ambiguity decomposition is:

$$G(\pi)=D_{KL}\!\left[Q(o\mid\pi)\,\|\,P(o)\right]+E_{Q(s\mid\pi)}\!\left[H(P(o\mid s))\right].$$

Here π is a policy; Q(o|π) and Q(s|π) are predicted outcome and state distributions; P(o) encodes preferred outcomes; and P(o|s) is the observation likelihood. Risk measures divergence from the declared outcome preferences; ambiguity reflects uncertainty in the state-to-observation mapping. Definitions and sign conventions vary across formulations, so use only with a specified generative model and horizon.

This objective is not a generic utility score, a measured physical potential, or a guarantee of safe behavior. Preferences must not be mistaken for evidence about the world. Action selection still requires feasible controls, permission, external safety constraints, and a host-attested implementation. The Free Energy Principle and active inference are theoretical frameworks, not a general theorem reducing motion planning to least action or Newtonian mechanics.

## 6. Hierarchical and multi-agent models

A hierarchical planner may separate task-level intent, motion primitives, and low-level control. Each boundary needs explicit state and time scales, interface units, handoff conditions, and failure behavior. A high-level plan does not verify low-level stability; a controller's local stability does not establish task completion.

For a group model, declare agents, interaction graph, communication or sensing range, update timing, delays, loss, obstacles, and collision constraints. Distinguish a simulated group trajectory from a deployed fleet. Agreement of trajectories or beliefs is a property of the stated model or protocol, not evidence that a shared belief is true.

## 7. Result and failure states

Use explicit outcomes such as MODEL_UNDECLARED, INPUT_INCOMPLETE, OUT_OF_DOMAIN, CONSTRAINT_INFEASIBLE, SOLVER_UNAVAILABLE, NOT_CONVERGED, RESOURCE_LIMIT, PREDICTION_WITHIN_SCOPE, or OBSERVED_AND_VERIFIED. Include uncertainty and prediction horizon. If state, geometry, force law, or capability is unknown, narrow the claim or return INDETERMINATE.

No motion generator, active-inference planner, whole-body controller, drone fleet, or simulator is included in this repository. A deployment must attest the actual model, solver/controller, version, supported state space, safety boundary, and validation.
