# SEMANTIC_COLLAPSE_TEST.md

## Inference-gap review scenario

Design status: `DESIGN_ONLY`; no embedding model, detector, threshold validation, or test run is included.

**Question:** Does a conclusion omit a premise or scope change that a reviewer needs in order to assess the inference?

List the source claim, intermediate premises, conclusion, definitions, context, and evidence references. Ask whether the conclusion follows under the stated reasoning and whether translation, summarization, or domain transfer changed a material meaning. Semantic-vector distance may prioritize review only when the model and threshold have been validated for the task; it does not establish a logical gap or falsehood.

Record `GAP_FOUND`, `NO_GAP_FOUND_WITHIN_REVIEW`, or `UNRESOLVED`, with the reasoning rule and scope. Do not describe an unexecuted scenario as passed.
## Case variations

Check summaries that omit a negation, unit, uncertainty interval, comparison group, temporal qualifier, conditional premise, legal jurisdiction, population restriction, or source attribution. Include valid paraphrases as negative controls and meaning-changing edits as positive controls. Keep the original and transformed text side by side.

For every flagged difference, state whether it changes logical entailment, evidential scope, user interpretation, or only surface wording. If an automated similarity measure is used, evaluate false alarms and missed material changes on the same task-specific set. A detector score must not substitute for an explicit explanation of the changed proposition.
