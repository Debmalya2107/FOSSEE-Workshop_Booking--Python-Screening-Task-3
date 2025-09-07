# FOSSEE-Workshop_Booking--Python-Screening-Task-3
Python Screening Task 3: Evaluating Open Source Models for Student Competence Analysis

Approach to identifying & evaluating models.
I will start by surveying open-source models specialized for code and for code-understanding tasks (e.g., BigCode/StarCoder, CodeT5, CodeGen) and complementary static-analysis/NLP tools (Python AST, mypy, pylint) to form candidate combinations. For each candidate I’ll evaluate: (a) whether its tokenizer and pretraining corpus include Python code, (b) ability to produce natural-language explanations and Socratic prompts, (c) inference cost and latency on a modest GPU/CPU, and (d) license/permissibility for educational use. Next I’ll design an evaluation harness: a curated dataset of student-written Python solutions (common beginner problems, with annotated conceptual errors and rubrics), plus unit tests and human-graded labels describing misconception type and depth-of-understanding. I’ll run the model(s) in two modes — (1) analyze-and-classify (identify bug class / misconception) and (2) prompt-generation (produce targeted Socratic questions or hints) — and collect both automated metrics and human ratings.

Test/validation & decision process.
Testing will combine automated checks and human rubrics. Automated checks include: (i) correctness of model-identified bug class (precision/recall against annotations), (ii) whether generated prompts avoid giving full solutions (measured with heuristics: does the prompt include code lines that directly fix the bug?), and (iii) diversity/novelty of questions. Human validation will score prompts on clarity, scaffolding value (how well it nudges thinking without solving), and safety (no harmful code suggestions). Decision-making will weigh empirical performance, pedagogical quality (human scores), cost-to-fine-tune, and interpretability. If one model proves promising but overconfident or hallucinatory, I’ll prefer a hybrid approach: use static analysis + lightweight model outputs and always present model suggestions with confidence signals and suggested follow-up checks.

Reasoning (required)
What makes a model suitable for high-level competence analysis?

A suitable model must (1) understand code semantics, not just syntax — it should map code patterns to underlying concepts (loops, invariants, off-by-one, off-by-condition), (2) produce pedagogically useful natural-language output (Socratic prompts, diagnosis, targeted hints) rather than single-line fixes, (3) be controllable to avoid revealing full solutions, and (4) be efficient enough to run near-interactively with acceptable latency. Finally, for real-world adoption, licensing and ability to fine-tune or prompt-engineer for classroom needs are crucial.

How would you test whether a model generates meaningful prompts?

Dataset & annotation: Create a representative set of student solutions labeled with (a) conceptual error class, (b) expected learning objective, and (c) one or two teacher-written draft prompts that would advance the student’s thinking.

Automated checks: Compare model output to teacher prompts using embedding-similarity and lexical overlap, and check the absence of solution-revealing content using heuristics (e.g., code block length, presence of direct corrected code).

Human evaluation: Have instructors rate prompts on (i) scaffolding effectiveness, (ii) specificity (targets the real gap), and (iii) non-solutionity (doesn’t give answer). Compute inter-rater agreement and average pedagogical score.

Classroom trial (optional): A small A/B test where one group receives model-generated prompts and another receives teacher prompts; measure subsequent student revisions and learning gains.

What trade-offs might exist between accuracy, interpretability, and cost?

Accuracy vs. cost: Larger models (more accurate on tricky subtle bugs) are costlier to run or fine-tune. For scale, smaller models + targeted fine-tuning or few-shot prompting may be preferable.

Accuracy vs. interpretability: High-capacity models may make correct suggestions but with opaque internal reasoning; combining them with deterministic static-analysis results increases transparency.

Interpretability vs. pedagogical richness: Strictly interpretable (rule-based) systems are predictable but can’t generate rich, discourse-level prompts; neural models can generate helpful Socratic dialogue but need guardrails. The pragmatic trade-off is a hybrid: deterministic checks for factual claims and an LLM for pedagogical phrasing.

Why this model(s) was chosen — strengths & limitations

Primary evaluated model: BigCode / StarCoder (or StarCoder-based family).
Why chosen: StarCoder is trained on large public code corpora and is explicitly designed for code generation and code understanding; it has strong tokenizer support for Python and produces coherent natural-language explanations in many reported benchmarks. Because it’s open-source (BigCode community releases) it’s amenable to local deployment and fine-tuning or instruction-tuning for educational prompts.
Strengths: strong code comprehension, can generate explanatory text, permissive to on-premise evaluation, good few-shot performance when prompted with examples of diagnoses + teacher prompts.
Limitations: large size => inference cost and memory needs; can hallucinate (claim incorrect reasoning), may produce solution-like hints unless constrained; licensing/weights may differ between sizes (always confirm current license). Also, out-of-the-box it won’t perfectly identify pedagogical misconception classes — requires supervised fine-tuning with labeled student examples and a human-in-the-loop for safety.

Secondary candidates (comparison):

CodeT5 / CodeGen: Smaller than StarCoder in some variants, fine for code summarization and can be easier to fine-tune on limited hardware; may be less fluent in pedagogical phrasing.

Static analyzers (pylint, mypy, AST pattern matchers): Not ML, but essential complements that provide deterministic signals (type errors, undefined names, unreachable branches) which can be used to seed the model’s prompt-generation and to corroborate model diagnosis.

How I would adapt the chosen model

Pipeline design: (a) run static analysis + unit tests on student code to collect deterministic failure signals; (b) feed code, failing test summary, and static-analysis output as context to the model along with 2–4 few-shot examples mapping bug→teacher prompt; (c) instruct the model to produce 1–3 Socratic prompts and a short explanation of the suspected misconception with a confidence score.

Safety & anti-spoiler controls: Use prompt templates that explicitly forbid giving corrected code and add a post-filter to detect code-like responses. Optionally mask variable names or specific lines to reduce direct solution leaks.

Fine-tuning: If resources permit, fine-tune on an annotated dataset of student code + teacher prompts to improve specificity.

Validation metrics (concrete)

Diagnostic accuracy (per-concept precision & recall) against annotated labels.

Prompt pedagogical score (human-rated, scale 1–5).

Spoiler rate: % of prompts containing direct solution tokens or code blocks.

Latency and cost per inference (GPU memory, time).

Inter-rater agreement (Cohen’s kappa) for human evaluations.
