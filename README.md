# (H-mas)
(h-mas) Hierarchical Multi-Agent Systems

TECHNICAL CONCEPT REPORT

see website for better report:
https://h-masresearchpaper.tiiny.site

Hierarchical
Multi-Agent Systems
(H-MAS)

A proposed architecture for coordinating AI agents with different capabilities

Prepared and credited to

Not Rigit

Contact

spark.x.pvt.ltd@gmail.com
notrigit@gmail.com

Document type

Independent technical concept and research overview

Version

1.0 | 11 November 2026

STATUS: PROPOSED DESIGN. This report describes an architecture and a set of engineering hypotheses. It
does not claim that the complete system has been built, experimentally validated, or adopted as an industry
standard.

H-MAS Technical Concept Report | Not Rigit

Page 1

Contents
1. Executive Summary
2. Scope and Terminology
3. Design Objectives
4. System Architecture
5. Hierarchical Agent Roles
6. Orchestration and Scheduling
7. Communication and Shared State
8. Verification and Probability
9. Software and Hardware Design
10. Applications and Example Workflow
11. Evaluation Framework
12. Risks and Safeguards
13. Development Roadmap
14. Conclusion
References

H-MAS Technical Concept Report | Not Rigit

Page 2

1. Executive Summary
Hierarchical Multi-Agent Systems (H-MAS) is the descriptive name used in this report for a proposed architecture in
which AI agents are arranged into levels according to their capabilities and responsibilities. Rather than sending every
request to one large model, the system decomposes a goal into smaller tasks, routes work to appropriate agents,
combines their outputs, and verifies important results.
The proposed design has four functional levels: a central orchestrator, high-capability reasoning agents, intermediate
specialist agents, and lightweight task agents. The orchestrator plans and coordinates; it is not expected to do all the
work. Small agents execute bounded subtasks in parallel. Medium agents integrate and review groups of results. Large
agents handle difficult reasoning, high-impact decisions, and unresolved conflicts.
An illustrative configuration is one orchestrator, one large reasoning agent, four medium agents, and sixty-four small
agents. These numbers are examples onlyâ€”not an industry standard, recommended ratio, or proven optimum. The best
arrangement depends on workload, reliability, cost, latency, and available hardware.
The central proposition is that a well-designed hierarchy may improve task allocation and throughput while preserving
access to stronger reasoning when needed. Its value must be demonstrated through controlled testing; adding more
agents does not automatically improve accuracy or efficiency.

2. Scope and Terminology
2.1 Agent
An agent is a software component that receives a task, uses an AI model and/or tools, maintains limited task state, and
returns an output with a status. An agent may be a separate model instance, a specialized service, or a model operating
under a defined role. Different agents do not necessarily require different model architectures.

2.2 Hierarchy
A hierarchy establishes levels of responsibility and escalation. Lower-level agents perform bounded work; higher-level
agents coordinate, review, or resolve tasks that exceed lower-level authority. This differs from a flat multi-agent system
where agents interact more directly as peers.

2.3 Relationship to existing technologies
Multi-Agent Systems (MAS), distributed artificial intelligence, model routing, orchestration, and LLM-based agent
collaboration are established areas. Mixture of Experts (MoE) is related but distinct: an MoE typically routes tokens or
internal representations among expert subnetworks within one model. H-MAS, as used here, coordinates separate
agents that may have independent prompts, tools, state, and workflows.
This report does not claim that H-MAS is a newly standardized scientific term. It presents a proposed combination of
known design patterns. The exact roles, task hierarchy, and illustrative 1 + 4 + 64 arrangement are part of this proposal
and require empirical evaluation.

3. Design Objectives
â€¢ Capability-aware routing: assign each task to an agent likely to complete it reliably at an appropriate cost.
â€¢ Parallel execution: run independent subtasks concurrently where dependencies allow.
â€¢ Controlled integration: normalize and review outputs before combining them into a final result.
â€¢ Traceability: record task ownership, inputs, outputs, versions, decisions, and verification results.
â€¢ Fault containment: prevent one agent failure from silently corrupting the whole project.
â€¢ Resource efficiency: balance capability against compute, memory, latency, energy, and monetary cost.
â€¢ Domain flexibility: support different tools and specialist roles without redesigning the entire hierarchy.

H-MAS Technical Concept Report | Not Rigit

Page 3

4. System Architecture
The architecture can be divided into a control plane and a work plane. The control plane manages planning,
permissions, scheduling, task state, and monitoring. The work plane contains agents that execute tasks and produce
results. Separating these responsibilities makes progress easier to track and recover.
Layer

Responsibility

Typical output

Orchestrator

Interpret goal, build task graph, assign work, track
dependencies, escalate failures, coordinate delivery.

Task plan, assignments, project
state.

Large reasoning
agent

Complex reasoning, cross-domain trade-offs,
ambiguity, high-impact review, conflict resolution.

Decision records, high-level
analysis.

Medium specialist
agents

Integrate groups of subtasks, check interfaces,
normalize outputs, review quality.

Integrated modules, review
findings.

Small task agents

Execute narrow, repeatable tasks such as extraction,
classification, calculations, or first-pass checks.

Structured task results and local
checks.

Shared data
services

Persist project state, artifacts, logs, schemas, and
version history.

Authoritative records and
artifacts.

Conceptual flow: goal â†’ orchestrator â†’ task decomposition â†’ small-agent execution â†’ medium-agent
integration/review â†’ large-agent escalation when required â†’ final validation and delivery. Not every task needs
every layer.

5. Hierarchical Agent Roles
5.1 Orchestrator
The orchestrator converts a goal into a dependency-aware task graph, assigns unique task IDs, selects agents, enforces
permissions, monitors progress, and decides when to retry or escalate. It should maintain durable project state outside its
conversational context. If it fails, the system should reconstruct progress from the task database and event log.

5.2 Large reasoning agents
Large agents are reserved for tasks that justify their higher computational cost: ambiguous synthesis, cross-domain
reasoning, high-impact decisions, and conflicts that cannot be resolved through tests or evidence. A large model is not
automatically correct. Decisions should include assumptions, evidence, and review requirements.

5.3 Medium specialist agents
Medium agents act as integration specialists or team leads. They combine outputs from small agents, enforce shared
formats, check interface compatibility, and identify missing information. They can also independently review conclusions
from large agents.

5.4 Small task agents
Small agents are suited to bounded subtasks: extracting facts from documents, classifying data, performing simple
calculations, detecting anomalies, or checking whether requirements are present. Each task needs explicit inputs,
constraints, output format, and acceptance criteria.

5.5 Dynamic roles
Roles should be logical, not permanently tied to parameter count. The same model family may serve different roles with
different tools and instructions. Deterministic software may be preferable to an LLM for arithmetic, validation, or other
exact operations. Routing should be based on measured performance, not model size alone.

H-MAS Technical Concept Report | Not Rigit

Page 4

6. Orchestration and Scheduling
Work should be represented as a dependency graph. A task becomes eligible when its prerequisites are complete.
Independent tasks can run in parallel; dependent tasks wait for required inputs. Iterative workflows should have explicit
iteration limits and stopping criteria.
â€¢ Task contract: define task ID, objective, input references, output schema, constraints, deadline, owner, and
acceptance tests.
â€¢ Routing policy: consider task type, expected accuracy, context length, tool access, latency, cost, and risk.
â€¢ Escalation: retry or escalate when output is missing, malformed, inconsistent, uncertain, or fails tests.
â€¢ Concurrency control: assign file/resource ownership and use version checks or locks to prevent conflicting writes.
â€¢ Completion policy: accept work only when its output meets defined criteria; an agent claiming completion is not
enough.

7. Communication and Shared State
7.1 Structured communication
Agents should exchange structured records rather than relying only on free-form conversation. A message can include
task_id, sender, recipient, status, input_refs, output_ref, errors, confidence, evidence_refs, and review_required. The
schema should be versioned so agents using different model families can interoperate.

7.2 Shared project memory
Durable project records include approved requirements, decisions, schemas, task states, test results, and artifact
versions. Temporary reasoning should be isolated and disposable. The orchestrator should consult an authoritative store
rather than assume every agent shares the same conversational memory.

7.3 Namespaces and ownership
Prefixes such as sm08_data_cleaning_result or med02_api_review can reduce naming collisions. Prefixes alone are not
sufficient. Stronger isolation comes from module boundaries, scoped variables, separate workspaces, file ownership,
database transactions, and explicit API contracts.

7.4 Communication topology
A hub-and-spoke pattern is a practical default: agents report to their coordinator, and coordinators communicate upward.
Limited peer-to-peer communication can be enabled for specific tasks. Unrestricted all-to-all messaging can create
unnecessary traffic, duplicated work, and conflicting state.

8. Verification, Consensus and Probability
Verification must be separate from generation. An agent confidence score is not automatically a calibrated probability.
Confidence becomes useful only when compared with observed outcomes on representative tasks.

8.1 Independent checks
Important conclusions should be checked using different methods: independent model review, deterministic calculations,
source verification, unit tests, simulation, formal constraints, or human review. Reviewers should, where possible, assess
the task and evidence without being anchored by the first answer.

8.2 Voting is not proof
If 48 of 64 agents agree, that is a voteâ€”not proof of correctness. Agents may share training data, prompts, tools, or
assumptions and therefore make correlated errors. A majority can repeat the same mistake. Voting should be combined
with evidence quality, diversity, calibrated reliability, and task-specific tests.

H-MAS Technical Concept Report | Not Rigit

Page 5

8.3 Statistical aggregation
Depending on the problem, aggregation may use majority voting, weighted voting, median, mean, Bayesian updating, or
Monte Carlo simulation. The method must match the data and assumptions. Numerical estimates should include
uncertainty ranges and sensitivity analysis rather than presenting one percentage as certain.

8.4 Verification sequence
â€¢ Validate output schema and required fields.
â€¢ Check factual claims against trusted sources or supplied evidence.
â€¢ Run deterministic tests and calculations where possible.
â€¢ Compare independent results and identify disagreements.
â€¢ Escalate unresolved or high-impact conflicts.
â€¢ Record the final decision, rationale, evidence, and remaining uncertainty.

9. Software and Hardware Design
9.1 Software implementation
A prototype can use model APIs or locally hosted models, an orchestration service, a task queue, a relational database,
object storage, and monitoring. Start with a small number of agents and a measurable workflow. Increasing agent count
before validating coordination often increases cost and failure modes without improving results.
Possible components include a scheduler, agent runtime, tool gateway, policy engine, shared-state database, artifact
store, event log, evaluation harness, and monitoring dashboard. These can initially run in one application and be
separated as scale requires.

9.2 Hardware implementation
The software hierarchy does not require a special processor. It can run across CPU and GPU servers, cloud instances,
or edge devices. A specialized hardware deployment could distribute workloads across multiple compute devices, but
this is optional and separate from the agent architecture itself.
Relevant hardware concepts include multi-chip modules, chiplets, high-bandwidth memory (HBM), local SRAM/cache,
high-speed interconnects, silicon interposers, and wafer-scale integration. These technologies can improve compute
density or data movement; they do not automatically provide intelligent coordination.

9.3 Intermediate storage
Tier

Use

Consideration

Local SRAM/cache

Immediate working state and frequently
reused data.

Very fast, limited capacity, often device-local.

Shared
memory/fabric

Exchange small results and
synchronization data.

Requires consistency and contention
management.

NVMe SSD

Large intermediate artifacts, checkpoints,
datasets.

Higher latency than memory; manage
concurrent writes.

Database/object
store

Durable state, logs, versions, final
outputs.

Needs backups, access control, integrity
checks.

10. Applications and Example Workflow
Potential applications are domain-independent in principle, but suitability depends on available tools, data quality, and
validation methods.

H-MAS Technical Concept Report | Not Rigit

Page 6

Scientific research
Search and classify literature, extract measurements, reproduce calculations, compare hypotheses, and prepare a
synthesis for expert review. Provenance and reproducibility are essential.

Engineering and design
Explore design alternatives, check component constraints, analyze simulation outputs, and review subsystem interfaces.
Safety-critical designs require qualified engineering approval.

Healthcare support
Organize literature, summarize records, or support administrative workflows. Diagnosis and treatment decisions require
qualified professionals and validated systems.

Education
Generate differentiated explanations, practice material, answer checks, and feedback. Educators should retain oversight
of grading and consequential decisions.

Business and economics
Consolidate reports, compare scenarios, detect anomalies, and prepare forecasts. Forecasts must expose assumptions
and uncertainty.

Robotics and operations
Use high-level planning to decompose goals while local controllers handle real-time execution. Safety interlocks and
deterministic controls must remain independent.

Environmental analysis
Process sensor data, compare models, identify unusual readings, and prepare monitoring summaries. Measurement
quality and provenance remain critical.

10.1 Example: environmental monitoring
1. The orchestrator receives a goal to assess unusual changes in a monitored river segment.
2. Small agents process sensor batches, flag missing readings, and extract weather or flow observations.
3. Medium agents combine regional summaries, check time alignment, and identify inconsistent measurements.
4. A large reasoning agent reviews competing explanations and uncertainty, without replacing field verification.
5. The orchestrator assembles a traceable report with sources, tests, unresolved anomalies, and a request for expert
review where needed.

11. Evaluation Framework
A credible implementation should be compared with a single-agent baseline and simpler multi-agent alternatives. The
purpose is to determine whether hierarchy improves outcomes enough to justify coordination costs.
Metric

Meaning

Why it matters

Task success rate

Share of tasks meeting predefined acceptance
criteria.

Measures completion quality.

End-to-end latency

Time from request to accepted result.

Captures parallelism and coordination
overhead.

Cost per accepted
task

Compute, model, storage, and tool costs per
successful result.

Tests economic viability.

Retry/escalation rate

Frequency of rework or upward escalation.

Reveals routing weaknesses.

H-MAS Technical Concept Report | Not Rigit

Page 7

Metric

Meaning

Why it matters

Integration error rate

Incompatible outputs, duplicated work, or
broken interfaces.

Measures coordination quality.

Calibration

Agreement between confidence and observed
accuracy.

Tests whether confidence can guide
routing.

Resource/energy use

CPU/GPU time, memory, network traffic, and
energy.

Measures operational efficiency.

Recommended experiment: select representative tasks; run them with a single strong model, a flat multi-agent setup, and
the proposed hierarchy; repeat trials; keep tools and information consistent; score outputs against explicit criteria; and
report averages, variance, failures, latency, and cost. Do not claim improvement unless results support it.

12. Risks and Safeguards
â€¢ Coordination overhead: more agents create messages, scheduling work, and integration points. Parallelize only
when tasks are separable.
â€¢ Correlated errors: agents may share biases or sources. Use independent evidence and deterministic checks rather
than relying on headcount.
â€¢ Orchestrator bottleneck: use queues, checkpointing, failover, and recoverable state.
â€¢ State conflicts: use ownership, version control, transactions, and contract tests to prevent overwrites and schema
drift.
â€¢ Security and privacy: apply least-privilege permissions, secret isolation, audit logs, and approval gates.
â€¢ Untrusted content: treat retrieved content as data, constrain tool permissions, and validate instructions at the policy
layer.
â€¢ Accountability: preserve provenance and assign a responsible owner for high-impact actions.
â€¢ Cost and energy: measure total system cost, not just the cost of individual agents.
â€¢ False confidence: require evidence, calibration, and domain-appropriate review.
For high-impact domains, the system should default to advisory operation. Human approval should be required
before consequential actions such as medical decisions, financial transactions, safety-critical control, or
irreversible production changes.

13. Development Roadmap
Phase

Objective

Exit criterion

1. Prototype

One orchestrator and two or three task agents
on a narrow workflow.

Tasks are assigned, tracked, and
repeatably completed.

2. Structured state

Add task database, artifact storage, message
schemas, and versioning.

Progress survives restart and can be
reconstructed.

3. Hierarchical
integration

Add medium-level integration and large-model
escalation.

Integration defects and escalation
outcomes are measurable.

4. Evaluation

Add test harness, independent checks, baseline
comparisons, calibration.

Claims are supported by reproducible
experiments.

5. Scale and
harden

Increase concurrency; add permissions, failover,
monitoring, security review.

System passes load and failure tests.

H-MAS Technical Concept Report | Not Rigit

Page 8

Prove the coordination model on a small system before expanding to dozens of agents or specialized hardware. Consider
a 1 + 4 + 64 deployment only if workload measurements demonstrate that this scale is useful.

14. Conclusion
Hierarchical Multi-Agent Systems offer a design direction for distributing complex work across agents with different
capabilities. The proposed model separates coordination from execution, assigns bounded subtasks to lightweight
agents, uses intermediate agents for integration, and reserves high-capability reasoning for difficult or high-impact
decisions.
The central engineering challenge is not simply creating more agents. It is decomposing work correctly, communicating
through reliable contracts, maintaining consistent shared state, and validating outputs with independent evidence.
Without these controls, additional agents can increase complexity and cost without improving results.
H-MAS should therefore be treated as a testable engineering proposal. Its value must be demonstrated through
controlled comparisons with simpler systems and transparent reporting of accuracy, latency, cost, failure rates, and
resource use.

References
[1] X. Li, S. Wang, S. Zeng, Y. Wu, and Y. Yang, â€œA survey on LLM-based multi-agent systems: workflow, infrastructure, and challenges,â€ 2024.
DOI: 10.1007/s44336-024-00009-2. https://link.springer.com/article/10.1007/s44336-024-00009-2
[2] T. Guo et al., â€œLarge Language Model Based Multi-agents: A Survey of Progress and Challenges,â€ IJCAI 2024, pp. 8048â€“8057. DOI:
10.24963/ijcai.2024/890. https://www.ijcai.org/proceedings/2024/0890
[3] K.-T. Tran et al., â€œMulti-Agent Collaboration Mechanisms: A Survey of LLMs,â€ arXiv:2501.06322, 2025. https://arxiv.org/abs/2501.06322
[4] N. Shazeer et al., â€œOutrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer,â€ ICLR, 2017.
https://research.google/pubs/outrageously-large-neural-networks-the-sparsely-gated-mixture-of-experts-layer/
[5] R. M. Aratchige and W. M. K. S. Ilmini, â€œLLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based
Multi Agent Systems,â€ arXiv:2504.01963, 2025. https://arxiv.org/abs/2504.01963
[6] â€œThe Five Ws of Multi-Agent Communication: Who Talks to Whom, When, What, and Why,â€ Transactions on Machine Learning Research,
2026. https://openreview.net/pdf?id=LGsed0QQVq

Reference note: These sources provide background on multi-agent systems, LLM-agent collaboration, communication, and Mixture of
Experts. They do not validate this reportâ€™s exact proposed configuration. The architecture and recommendations here are a synthesis and
proposal credited to Not Rigit.

Prepared and credited to: Not Rigit
Contact: spark.x.pvt.ltd@gmail.com | notrigit@gmail.com

H-MAS Technical Concept Report | Not Rigit

Page 9

