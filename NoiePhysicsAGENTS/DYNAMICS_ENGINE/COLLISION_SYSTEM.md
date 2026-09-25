# COLLISION_SYSTEM.md

## L3 — Collision geometry and contact dynamics v2.3

**Scope:** Describe collision queries and physical contact-response models. This document is not a collision library, rigid-body engine, or safety controller. Detection, contact generation, and response are separate stages.

## 1. Collision pipeline

1. **Broad phase:** cheaply return candidate body pairs using conservative bounds and spatial partitioning. Bounds may include spheres, AABBs, OBBs, or a hierarchy. They must enclose the represented geometry; false positives are acceptable, false negatives are not.
2. **Narrow phase:** test candidate geometry pairs using a method appropriate to their shape and motion. Return separation, contact, penetration, or indeterminate with numerical tolerances.
3. **Contact generation:** construct contact points or a manifold, oriented normals, separation or penetration depth, and feature identifiers.
4. **Response:** apply a declared force, impulse, constraint, or compliant-contact model using mass, inertia, velocity at contact, and material parameters.
5. **Integration and validation:** update the state and check penetration, energy, momentum, angular momentum, and solver residuals where conservation or balance is expected.

A broad-phase overlap is not a confirmed collision. A geometric contact is not itself a physical response law.

## 2. Separating axis theorem

For two convex polyhedra in three dimensions, the separating axis theorem says they are disjoint if a separating axis exists among the face normals of each polyhedron and cross products of edge directions. Project both convex shapes onto each non-degenerate candidate axis; a gap larger than the declared tolerance proves separation for the represented geometry. If no candidate axis separates them, the test reports intersection or touching according to the chosen boundary convention.

Normalize axes only after checking their magnitude. Parallel edges yield near-zero cross products; skip those axes rather than divide by a near-zero norm. Nearly touching geometry, poorly conditioned coordinates, degenerate faces, and floating-point tolerance can change classification. SAT in this form applies to convex polyhedra, not arbitrary non-convex meshes without decomposition or another appropriate method.

## 3. Bounds and candidate generation

A bounding volume hierarchy organizes conservative bounds so disjoint subtrees can be rejected. The bound type and split/update method affect performance, not the correctness condition that enclosed geometry remain enclosed. Refit or rebuild dynamic hierarchies when motion invalidates bounds. A sphere hierarchy is one possible bounding hierarchy; “BVH” does not imply spheres specifically.

Candidate generation must avoid self-pairs and duplicate pairs while preserving all potentially colliding pairs. High object speed relative to the time step can cause tunneling even with correct discrete overlap queries. Use continuous collision detection, swept bounds, or a sufficiently justified time-step strategy when fast motion matters.

## 4. Contact response

For rigid bodies, contact response depends on the normal relative velocity at the contact point, translational inverse masses, rotational inertia, restitution, and friction. Restitution is a model parameter describing rebound in a declared impact regime; it is not a universal material constant across speed, temperature, surface state, or repeated impacts.

Coulomb friction constrains tangential impulse magnitude by the normal impulse and a friction coefficient under its idealized model. More detailed material behavior may require compliance, damping, adhesion, deformation, or rate dependence. Multiple simultaneous contacts require a coupled or iterative solution; resolving contacts independently can create order dependence, jitter, excessive penetration, or energy artifacts.

Contact normals and points must come from actual geometry or a justified approximation. The line between body centers is not generally a valid contact normal for arbitrary shapes. Penetration correction should not add unphysical energy; report any stabilization method and its effect.

## 5. Edge cases and diagnostics

Check exact touching versus overlap, initial interpenetration, degenerate geometry, very small or large scales, nearly parallel features, high-speed motion, resting contact, stacks, frictional sliding, rotational impacts, and multiple simultaneous contacts. Track tolerance, time step, maximum penetration, solver residual, and convergence sensitivity.

For an isolated frictionless collision with a suitable rigid-body model, momentum and angular momentum about the system boundary should satisfy the declared impact law; kinetic energy is conserved only for an elastic impact and otherwise changes according to restitution. For open or driven systems, include external work and dissipation instead of treating energy drift as an automatic solver defect.

## 6. Validation and capability

Validate simple analytic cases, symmetry and frame transformations, shape-bound containment, timestep refinement, and independent reference results where available. Test broad-phase conservatism separately from narrow-phase correctness and response-law behavior. A passing unit test does not establish safety for an untested geometry or deployment.

The host must attest any collision engine, geometry representation, supported shapes, tolerances, continuous-collision capability, solver, version, and validation. This specification provides none of these capabilities.
