# CAUSAL_GRAPHS

This directory is reserved for causal-model examples and documentation. The current repository includes the inference contract at `../LOGIC_ENGINE/CAUSAL_INFERENCE.md` and counterfactual assumptions at `../LOGIC_ENGINE/COUNTERFACTUAL.md`; no graph database or learned-graph collection is included.

A model record should identify its question, target population, variables, time order, graph or structural equations, assumptions, provenance, validation, and known limitations. Label a model `DAG` only when its represented structure is acyclic under the stated semantics. Feedback or temporal systems may require another representation.

A drawn graph is a hypothesis, not proof of causal structure. Causal identification, numeric estimation, and decision relevance are separate checks. Do not label a graph “validated” unless the validating evidence, method, scope, and responsible tool or reviewer are recorded.

The root `AGENTS.md` and `NoieLogicAGENTS.md` define the active contracts. Older v2.2 safety banners in archived or illustrative material do not override them. No runtime storage, learner, validator, or sandbox is supplied here.