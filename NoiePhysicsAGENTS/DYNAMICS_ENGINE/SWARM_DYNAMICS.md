# SWARM_DYNAMICS.md

## L3 — Multi-agent physical dynamics v2.3

**Scope:** Represent interacting moving agents and physical group patterns. This module is not a fleet controller, distributed consensus service, mission policy, or truth-verification method.

## 1. State and interaction model

For each agent i, define position, velocity, orientation if relevant, physical limits, and measurement or estimation uncertainty. Define the environment, obstacle geometry, update interval, coordinate frame, and a time-varying interaction graph. The graph edge i→j means a specified information or physical interaction is available; it does not imply that information is accurate or independent.

Declare whether motion is continuous, discrete-time, event-driven, or hybrid. Include acceleration and speed limits, sensing and communication range, latency, message loss, actuator response, and failure behavior. Local-neighbor models and all-to-all models have materially different connectivity and scaling assumptions.

## 2. Boids-style local rules

A Boids-style model often combines:

- **Separation:** steer away from neighbors inside a short-range exclusion region.
- **Alignment:** adjust heading or velocity toward a local-neighbor statistic.
- **Cohesion:** steer toward a local-neighbor position centroid.

These are modeling rules, not physical laws. Define neighborhoods, distance metric, weights, speed and turn limits, boundary conditions, obstacle handling, and tie-breaking when no neighbors exist. Scaling one term can dominate the others; report parameter sensitivity and behavior changes. Do not imply that the three rules guarantee collision avoidance, stable flocking, or goal completion.

## 3. Order parameters

A common polarization statistic for nonzero velocities is:

$$P=\frac{1}{N}\left\|\sum_{i=1}^N\frac{v_i}{\|v_i\|}\right\|,\qquad 0\le P\le1.$$

Define behavior for zero or near-zero speed, missing agents, weighted observations, and finite samples. Polarization measures directional alignment only; it does not establish safety, coordination quality, correct information, or consensus on a claim. Other observables such as density, neighbor count, connectivity, collision rate, and task coverage answer different questions.

## 4. Consensus-like physical dynamics

For a declared undirected connected communication graph with Laplacian L, the continuous-time averaging model ẋ=−Lx converges toward a common value under its standard assumptions. Directed, switching, delayed, quantized, or lossy networks need different conditions and may fail to converge. This is agreement about a state variable under a model; it is not epistemic truth or authorization.

Keep physical agreement, task allocation, belief exchange, and Byzantine consensus separate. A swarm simulation that converges does not show that a value is correct, that agents are independent, or that a deployed network meets the assumed graph.

## 5. Safety, resources, and failure

Check collision margins, communication partitions, stale neighbor state, localization drift, actuator saturation, energy limits, obstacle changes, agent loss, and emergency stopping. A single aggregate group metric can hide a failing or isolated member. Record individual and group state when relevant, plus the actual observations supporting each.

Any deployment requires a host-attested controller, platform, communication system, geofence or other constraints, emergency stop path, and field validation. A simulated group trajectory is conditional evidence and cannot authorize a real deployment.

## 6. Validation report

State the question and metric before simulation; model and parameters; graph and delay assumptions; initial-condition ensemble; obstacle and failure cases; timestep and solver; sensitivity; run identifier; and observed failures. Distinguish DESIGN_ONLY, NOT_RUN, RUN_FAILED, and RUN_COMPLETED_WITH_LIMITS. Report the tested scope and unmodeled effects.
