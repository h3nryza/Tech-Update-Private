# Academic Research: AI-Assisted Coding, Vibe Coding, and LLM-Driven Development

**Compiled:** 2026-03-22
**Papers Catalogued:** 38
**Sources:** arXiv, ACM Digital Library, IEEE Xplore, Science, Google Scholar, MIT CSAIL, Stanford HAI, Georgetown CSET, CMU, METR

---

## 1. Executive Summary

The academic literature on AI-assisted coding from 2022-2026 reveals a field characterized by rapid capability gains, surprising productivity paradoxes, and unresolved quality-security tensions. Key themes across 38 papers:

- **Productivity gains are real but context-dependent.** Controlled experiments show 21-56% speedups on well-scoped tasks, but experienced developers working on mature codebases may see *negative* productivity effects (19% slowdown in the METR RCT). The benefit skews toward less-experienced developers and simpler tasks.
- **Code quality is a double-edged sword.** GitHub's own RCT found Copilot code passes more unit tests, but independent studies (GitClear, CMU/MSR) document persistent increases in code complexity, churn, and static analysis warnings when AI tools are used at scale.
- **Security vulnerabilities are pervasive.** Between 12-65% of AI-generated code snippets trigger CWE-classified vulnerabilities depending on model, language, and prompt. Package hallucination rates reach 44.7% in some models.
- **Autonomous agents are advancing rapidly.** SWE-bench solve rates jumped from ~4% (2023) to 72%+ (2025), and multi-agent frameworks (ChatDev, MetaGPT, ALMAS) demonstrate end-to-end SDLC automation.
- **"Vibe coding" has entered the academic lexicon.** Stanford HAI's 2025 AI Index formally recognized the term, and multiple arxiv surveys now study it as a distinct paradigm with specific risk profiles.
- **AIOps is maturing** with LLM integration enabling natural-language incident triage, but adversarial telemetry manipulation poses new attack surfaces.

The consensus: AI coding tools are transformational but require deliberate governance, review processes, and expertise to realize net benefits without accumulating technical debt and security risk.

---

## 2. Papers by Category

### 2.1 AI-Assisted Code Generation: Quality, Correctness, and Productivity

#### Paper 1: The Impact of AI on Developer Productivity: Evidence from GitHub Copilot
- **Authors:** Sida Peng, Eirini Kalliamvakou, Peter Cihon, Mert Demirer
- **Institution:** GitHub / Microsoft Research
- **Year:** 2023
- **Venue:** arXiv (2302.06590)
- **Key Findings:** In a controlled experiment, developers with Copilot access completed an HTTP server implementation task 55.8% faster (71 min vs. 161 min, p=0.0017). Less experienced developers, older programmers, and those coding more hours/day benefited most.
- **Link:** [https://arxiv.org/abs/2302.06590](https://arxiv.org/abs/2302.06590)
- **Relevance to Vibe Coding:** Establishes the foundational productivity case for AI-assisted coding that vibe coding builds upon.

#### Paper 2: Measuring GitHub Copilot's Impact on Productivity
- **Authors:** Albert Ziegler, Eirini Kalliamvakou, et al.
- **Institution:** GitHub
- **Year:** 2024
- **Venue:** Communications of the ACM
- **Key Findings:** Survey of 2,000+ developers found 46% of code written with Copilot assistance. Developers reported higher satisfaction and perceived productivity. Blind code reviews showed Copilot-assisted code had 3.62% better readability, 2.94% better reliability, and 4.16% better conciseness.
- **Link:** [https://dl.acm.org/doi/10.1145/3633453](https://dl.acm.org/doi/10.1145/3633453)
- **Relevance to Vibe Coding:** Quantifies that AI-generated code can meet or exceed human-written quality on multiple dimensions.

#### Paper 3: Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity
- **Authors:** METR (Model Evaluation & Threat Research)
- **Institution:** METR
- **Year:** 2025
- **Venue:** arXiv (2507.09089)
- **Key Findings:** RCT with 16 experienced open-source developers completing 246 tasks. AI tools (Claude 3.5 Sonnet, Cursor Pro) *increased* task completion time by 19%, despite developers believing they were 20% faster. The effect held across task types and experience levels.
- **Link:** [https://arxiv.org/abs/2507.09089](https://arxiv.org/abs/2507.09089)
- **Relevance to Vibe Coding:** A critical counterpoint -- vibe coding may produce an illusion of speed while actually slowing experienced developers on complex, real-world tasks.

#### Paper 4: How Much Does AI Impact Development Speed? An Enterprise-Based Randomized Controlled Trial
- **Authors:** Sundaram, Patel, et al.
- **Institution:** Google
- **Year:** 2024
- **Venue:** arXiv (2410.12944)
- **Key Findings:** RCT with 96 Google software engineers on an enterprise-grade C++ task (~474 LOC across 10 files). AI group completed 21% faster (~96 min vs. ~114 min). Developers who code more hours/day and senior developers saw larger gains.
- **Link:** [https://arxiv.org/abs/2410.12944](https://arxiv.org/abs/2410.12944)
- **Relevance to Vibe Coding:** Shows that even in a highly controlled enterprise setting with experienced developers, AI provides meaningful but moderate speedups.

#### Paper 5: Examining the Use and Impact of an AI Code Assistant on Developer Productivity and Experience in the Enterprise
- **Authors:** IBM Research team
- **Institution:** IBM
- **Year:** 2024
- **Venue:** arXiv (2412.06603)
- **Key Findings:** Enterprise-scale study found AI assistants increased net productivity despite variable output quality. Gains were unevenly distributed across users. Highlights the importance of organizational context in AI tool adoption.
- **Link:** [https://arxiv.org/abs/2412.06603](https://arxiv.org/abs/2412.06603)
- **Relevance to Vibe Coding:** Demonstrates that vibe coding benefits depend heavily on individual skill and organizational support.

#### Paper 6: AI-Assisted Programming Decreases the Productivity of Experienced Developers by Increasing Technical Debt and Maintenance Burden
- **Authors:** Feiyang Xu, Poonacha K. Medappa, Murat M. Tunc, Martijn Vroegindeweij, Jan C. Fransoo
- **Institution:** Multiple (SSRN/arXiv)
- **Year:** 2025
- **Venue:** arXiv (2510.10165)
- **Key Findings:** After GitHub Copilot introduction in OSS projects, peripheral (less experienced) developers drove productivity increases, but core (experienced) developers reviewed 6.5% more code and saw a 19% drop in original code productivity. AI-written code required more rework, indicating increased technical debt.
- **Link:** [https://arxiv.org/abs/2510.10165](https://arxiv.org/abs/2510.10165)
- **Relevance to Vibe Coding:** Warns that vibe coding at scale may shift burden to senior developers who must review and fix AI-generated code.

#### Paper 7: Speed at the Cost of Quality: How Cursor AI Increases Short-Term Velocity and Long-Term Complexity in Open-Source Projects
- **Authors:** He et al.
- **Institution:** Carnegie Mellon University
- **Year:** 2026
- **Venue:** MSR '26 (Mining Software Repositories)
- **Key Findings:** Difference-in-differences analysis of Cursor-adopting GitHub projects found a statistically significant but *transient* increase in development velocity, alongside a *substantial and persistent* increase in static analysis warnings and code complexity. "Complexity debt" accumulates when AI is used heavily.
- **Link:** [https://arxiv.org/abs/2511.04427](https://arxiv.org/abs/2511.04427)
- **Relevance to Vibe Coding:** Directly measures the long-term quality cost of the vibe coding paradigm -- speed now, complexity later.

#### Paper 8: The Multi-Company GitHub Copilot Productivity Study
- **Authors:** Microsoft, Accenture, Fortune 100 Enterprise teams
- **Institution:** Microsoft Research / Accenture
- **Year:** 2024
- **Venue:** Industry RCT
- **Key Findings:** Nearly 5,000 developers across three organizations showed an average 26% productivity increase with Copilot access. Effect sizes varied by organization, task type, and developer experience.
- **Link:** Referenced in [https://addyo.substack.com/p/the-reality-of-ai-assisted-software](https://addyo.substack.com/p/the-reality-of-ai-assisted-software)
- **Relevance to Vibe Coding:** Largest multi-org RCT to date, establishing a baseline productivity multiplier for AI-assisted coding.

#### Paper 9: Assessing and Analyzing the Correctness of GitHub Copilot's Code Suggestions
- **Authors:** Multiple authors
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** ACM Transactions on Software Engineering and Methodology
- **Key Findings:** Empirical study across all 2,033 LeetCode problems: 70% received at least one correct Copilot suggestion. Language-specific rates: C (29.7%), Java (57.7%), JavaScript (54.1%), Python (41.0%).
- **Link:** [https://dl.acm.org/doi/10.1145/3715108](https://dl.acm.org/doi/10.1145/3715108)
- **Relevance to Vibe Coding:** Quantifies the correctness ceiling -- vibe coders must verify outputs since 30-70% of suggestions contain errors depending on language.

---

### 2.2 LLM Agents for Software Engineering

#### Paper 10: SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering
- **Authors:** Yang et al.
- **Institution:** Princeton University
- **Year:** 2024
- **Venue:** NeurIPS 2024 (arXiv 2405.15793)
- **Key Findings:** SWE-agent introduces a custom agent-computer interface (ACI) for LLM agents to navigate repos, edit code, and run tests. Achieved 12.5% pass@1 on SWE-bench and 87.7% on HumanEvalFix. The ACI design is critical -- simple changes yielded 40% relative performance gains.
- **Link:** [https://arxiv.org/abs/2405.15793](https://arxiv.org/abs/2405.15793)
- **Relevance to Vibe Coding:** Establishes the technical foundation for autonomous coding agents that power the vibe coding experience.

#### Paper 11: OpenHands: An Open Platform for AI Software Developers as Generalist Agents
- **Authors:** Wang et al.
- **Institution:** Multiple (open-source community)
- **Year:** 2024/2025
- **Venue:** ICLR 2025 (arXiv 2407.16741)
- **Key Findings:** Open-source platform evaluated across 15 benchmarks. Achieved 72% resolution rate on SWE-bench Verified (Claude Sonnet 4.5 with extended thinking). Supports multiple agent architectures and coordination mechanisms.
- **Link:** [https://arxiv.org/abs/2407.16741](https://arxiv.org/abs/2407.16741)
- **Relevance to Vibe Coding:** Provides the open-source infrastructure that democratizes autonomous coding agent development.

#### Paper 12: SWE-Bench Pro: Can AI Agents Solve Long-Horizon Software Engineering Tasks?
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2509.16941)
- **Key Findings:** 1,865 problems from 41 repositories designed to capture enterprise-level complexity. Tasks require hours to days for human engineers and span multiple files. Live-SWE-agent achieves 45.8% on SWE-Bench Pro, demonstrating progress on realistic long-horizon tasks.
- **Link:** [https://arxiv.org/abs/2509.16941](https://arxiv.org/abs/2509.16941)
- **Relevance to Vibe Coding:** Tests whether vibe coding can scale beyond simple tasks to real enterprise software engineering.

#### Paper 13: The Rise of AI Teammates in Software Engineering (SE) 3.0
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2507.15003)
- **Key Findings:** Analyzes the AIDev dataset spanning 456,000+ pull requests by five leading agents (OpenAI Codex, Devin, GitHub Copilot, Cursor, Claude Code) across 61,000 repos and 47,000 developers. Documents the transition from AI-as-tool to AI-as-teammate.
- **Link:** [https://arxiv.org/abs/2507.15003](https://arxiv.org/abs/2507.15003)
- **Relevance to Vibe Coding:** The largest empirical study of AI agent adoption in real software projects to date.

#### Paper 14: Live-SWE-agent: Can Software Engineering Agents Self-Evolve on the Fly?
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2511.13646)
- **Key Findings:** First live software agent that autonomously and continuously evolves during runtime. Achieves 77.4% on SWE-bench Verified and 45.8% on SWE-Bench Pro. Demonstrates self-improving agent architectures.
- **Link:** [https://arxiv.org/abs/2511.13646](https://arxiv.org/abs/2511.13646)
- **Relevance to Vibe Coding:** Points toward a future where vibe coding agents improve themselves -- reducing the need for human intervention over time.

---

### 2.3 AI Pair Programming Studies

#### Paper 15: The Impact of AI-Assisted Pair Programming on Student Motivation, Programming Anxiety, and Performance
- **Authors:** Multiple
- **Institution:** Multiple (Education Research)
- **Year:** 2025
- **Venue:** International Journal of STEM Education (Springer)
- **Key Findings:** Quasi-experimental study with 234 undergraduates over two academic years. AI-assisted pair programming significantly improved intrinsic motivation, reduced programming anxiety, and improved performance compared to both human-human pair programming and individual programming.
- **Link:** [https://link.springer.com/article/10.1186/s40594-025-00537-3](https://link.springer.com/article/10.1186/s40594-025-00537-3)
- **Relevance to Vibe Coding:** Suggests vibe coding may be especially beneficial for learning and reducing barriers to entry.

#### Paper 16: From Developer Pairs to AI Copilots: A Comparative Study
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2506.04785)
- **Key Findings:** Compared human-human and human-AI pair programming. When developers pair with humans, AI tool interactions decrease -- developers prefer human input. AI pair programming metrics are less comprehensive than human pair programming metrics, highlighting measurement gaps.
- **Link:** [https://arxiv.org/abs/2506.04785](https://arxiv.org/abs/2506.04785)
- **Relevance to Vibe Coding:** Suggests vibe coding may work differently in team vs. solo contexts, and that human collaboration remains preferred when available.

#### Paper 17: Is AI the Better Programming Partner? Human-Human vs. Human-AI Pair Programming
- **Authors:** Sherry et al.
- **Institution:** Carnegie Mellon University
- **Year:** 2023
- **Venue:** CEUR Workshop Proceedings
- **Key Findings:** Empirical comparison of both modalities. Mixed results -- AI partners excel at rapid code generation but human partners provide better high-level design guidance and error detection in complex scenarios.
- **Link:** [https://www.cs.cmu.edu/~sherryw/assets/pubs/2023-pair.pdf](https://www.cs.cmu.edu/~sherryw/assets/pubs/2023-pair.pdf)
- **Relevance to Vibe Coding:** Highlights that vibe coding loses the design-level feedback that human pair programming provides.

#### Paper 18: Exploring the Problems, Their Causes and Solutions of AI Pair Programming
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** Journal of Systems and Software (ScienceDirect)
- **Key Findings:** Analysis of GitHub and Stack Overflow discussions. Identified systematic categories of problems developers face with AI pair programming including incorrect suggestions, context misunderstanding, and over-reliance.
- **Link:** [https://www.sciencedirect.com/science/article/abs/pii/S0164121224002486](https://www.sciencedirect.com/science/article/abs/pii/S0164121224002486)
- **Relevance to Vibe Coding:** Catalogs the failure modes that vibe coders will encounter and must learn to manage.

---

### 2.4 Prompt Engineering for Code

#### Paper 19: Promptware Engineering: Software Engineering for LLM Prompt Development
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2503.02400)
- **Key Findings:** Defines "promptware" as a new paradigm where natural language prompts replace traditional code. Outlines a comprehensive roadmap for promptware engineering, treating prompts as first-class software artifacts requiring versioning, testing, and maintenance.
- **Link:** [https://arxiv.org/abs/2503.02400](https://arxiv.org/abs/2503.02400)
- **Relevance to Vibe Coding:** Directly theorizes the engineering discipline needed to make vibe coding reliable and reproducible.

#### Paper 20: Enhancing Computer Programming Education with LLMs: Effective Prompt Engineering for Python Code Generation
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2407.05437)
- **Key Findings:** Systematic categorization of prompt engineering strategies for code generation. Experiments with GPT-4, GPT-4o, Llama3-8b, and Mixtral-8x7b revealed that tailored prompt strategies significantly enhance LLM performance for code generation tasks.
- **Link:** [https://arxiv.org/abs/2407.05437](https://arxiv.org/abs/2407.05437)
- **Relevance to Vibe Coding:** Provides the empirical basis for prompt crafting techniques that improve vibe coding outcomes.

#### Paper 21: Natural Language Boosts LLM Performance in Coding, Planning, and Robotics
- **Authors:** MIT CSAIL researchers
- **Institution:** MIT CSAIL
- **Year:** 2024
- **Venue:** MIT News / arXiv
- **Key Findings:** Natural Language Embedded Programs (NLEPs) combine natural language and programming to solve tasks. Achieved >90% accuracy on symbolic reasoning tasks with GPT-4, and 30% greater accuracy than task-specific prompting methods.
- **Link:** [https://news.mit.edu/2024/natural-language-boosts-llm-performance-coding-planning-robotics-0501](https://news.mit.edu/2024/natural-language-boosts-llm-performance-coding-planning-robotics-0501)
- **Relevance to Vibe Coding:** The "vibe" in vibe coding -- natural language prompts can actually outperform rigid programming-only approaches.

---

### 2.5 Vibe Coding as a Paradigm

#### Paper 22: A Survey of Vibe Coding with Large Language Models
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2510.12399)
- **Key Findings:** Comprehensive survey of the vibe coding paradigm. Despite its transformative potential, empirical evidence reveals unexpected productivity losses and fundamental challenges in human-AI collaboration. Establishes taxonomy of vibe coding practices, tools, and outcomes.
- **Link:** [https://arxiv.org/abs/2510.12399](https://arxiv.org/abs/2510.12399)
- **Relevance to Vibe Coding:** The first dedicated academic survey of vibe coding as a distinct software development paradigm.

#### Paper 23: Professional Software Developers Don't Vibe, They Control: AI Agent Use for Coding in 2025
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2512.14012)
- **Key Findings:** Professional developers value AI agents for productivity but retain agency in design and implementation. Developers employ strategies to control agent behavior, leveraging their expertise to maintain software quality. "Controlled coding" rather than pure vibe coding is the professional norm.
- **Link:** [https://arxiv.org/abs/2512.14012](https://arxiv.org/abs/2512.14012)
- **Relevance to Vibe Coding:** Key finding that professionals adapt vibe coding into a controlled, expertise-driven workflow rather than fully trusting AI output.

#### Paper 24: Vibe Coding vs. Agentic Coding: Fundamentals and Practical Implications
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2505.19443)
- **Key Findings:** Distinguishes vibe coding (intuitive, human-in-the-loop, prompt-based) from agentic coding (autonomous, goal-driven, minimal human intervention). Vibe coding supports ideation and experimentation; agentic coding enables autonomous task execution. Different risk profiles for each.
- **Link:** [https://arxiv.org/abs/2505.19443](https://arxiv.org/abs/2505.19443)
- **Relevance to Vibe Coding:** Provides the formal taxonomic distinction between vibe coding and fully autonomous agentic coding.

#### Paper 25: FeatBench: Evaluating Coding Agents on Feature Implementation for Vibe Coding
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2509.22237)
- **Key Findings:** Purpose-built benchmark for evaluating coding agents specifically in the vibe coding context -- feature implementation from natural language descriptions. Tests agents on realistic feature-level tasks rather than bug fixes.
- **Link:** [https://arxiv.org/abs/2509.22237](https://arxiv.org/abs/2509.22237)
- **Relevance to Vibe Coding:** First benchmark designed specifically to evaluate vibe coding agent performance.

#### Paper 26: Stanford HAI AI Index Report 2025 -- Vibe Coding Section
- **Authors:** Stanford HAI team
- **Institution:** Stanford University, Human-Centered AI Institute
- **Year:** 2025
- **Venue:** Stanford HAI AI Index Report
- **Key Findings:** Formally recognizes "vibe coding" (coined by Andrej Karpathy) as an emerging paradigm. Two groups benefit most: experienced coders (who can fix errors) and complete beginners (learning). Risks include errors and security vulnerabilities in deployed code.
- **Link:** [https://hai.stanford.edu/ai-index/2025-ai-index-report](https://hai.stanford.edu/ai-index/2025-ai-index-report)
- **Relevance to Vibe Coding:** Institutional recognition of vibe coding by a top AI research institute.

---

### 2.6 Multi-Agent Systems for Development

#### Paper 27: ChatDev: Communicative Agents for Software Development
- **Authors:** Qian et al.
- **Institution:** Tsinghua University / OpenBMB
- **Year:** 2023/2024
- **Venue:** arXiv (2307.07924)
- **Key Findings:** 7 specialized LLM agents collaborate through chat-based communication across design, coding, and testing phases. Demonstrates that multi-agent systems can simulate full SDLC workflows, though with significant token overhead (183.7K for HumanEval).
- **Link:** [https://arxiv.org/abs/2307.07924](https://arxiv.org/abs/2307.07924)
- **Relevance to Vibe Coding:** Pioneering work showing that vibe coding can extend beyond single-agent interaction to team-simulated development.

#### Paper 28: MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework
- **Authors:** Hong et al.
- **Institution:** Multiple
- **Year:** 2023/2024
- **Venue:** arXiv (2308.00352)
- **Key Findings:** Agents communicate through structured documents and diagrams rather than dialogue. Achieves 85.9% and 87.7% Pass@1 on code generation benchmarks. More efficient than dialogue-based approaches by reducing communication overhead.
- **Link:** [https://arxiv.org/abs/2308.00352](https://arxiv.org/abs/2308.00352)
- **Relevance to Vibe Coding:** Shows that structured artifacts (not just chat) improve multi-agent coding quality -- implications for vibe coding prompt design.

#### Paper 29: AgentCoder: Multi-Agent Code Generation with Iterative Testing and Optimisation
- **Authors:** Huang et al.
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2312.13010)
- **Key Findings:** Three-agent framework (programmer, test designer, test executor) achieves 91.5% pass@1 with GPT-4 on HumanEval. Significantly lower token overhead (56.9K) than ChatDev (183.7K) and MetaGPT (138.2K).
- **Link:** [https://arxiv.org/abs/2312.13010](https://arxiv.org/abs/2312.13010)
- **Relevance to Vibe Coding:** Demonstrates that test-driven multi-agent approaches produce higher quality vibe-coded output with less waste.

#### Paper 30: ALMAS: An Autonomous LLM-based Multi-Agent Software Engineering Framework
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2510.03463)
- **Key Findings:** Follows SDLC philosophy with agents aligned to agile roles. Can work within agile teams, performing tasks end-to-end while integrating with human developers. Modular design allows selective agent deployment.
- **Link:** [https://arxiv.org/abs/2510.03463](https://arxiv.org/abs/2510.03463)
- **Relevance to Vibe Coding:** Shows how vibe coding can be embedded within existing agile workflows rather than replacing them.

#### Paper 31: LLM-Based Multi-Agent Systems for Software Engineering: Literature Review, Vision and the Road Ahead
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2404.04834)
- **Key Findings:** Systematic review of 71 papers on LLM-based multi-agent systems for SE. Proposes research agenda for strengthening individual agent competence and optimizing inter-agent collaboration. Identifies key challenges in role specialization and communication protocols.
- **Link:** [https://arxiv.org/abs/2404.04834](https://arxiv.org/abs/2404.04834)
- **Relevance to Vibe Coding:** The definitive literature review of multi-agent approaches that underpin advanced vibe coding platforms.

---

### 2.7 Security Implications of AI-Generated Code

#### Paper 32: State of the Art of the Security of Code Generated by LLMs: A Systematic Literature Review
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** IEEE (10795572)
- **Key Findings:** 12-65% of generated code snippets are non-compliant with secure coding standards or trigger CWE-classified vulnerabilities. Prevalent weaknesses: buffer overflows, unchecked return values, hard-coded credentials, SQL injection, code injection, path traversal.
- **Link:** [https://ieeexplore.ieee.org/document/10795572/](https://ieeexplore.ieee.org/document/10795572/)
- **Relevance to Vibe Coding:** Quantifies the security risk inherent in accepting AI-generated code without review -- the core risk of vibe coding.

#### Paper 33: Cybersecurity Risks of AI-Generated Code
- **Authors:** Jessica Ji, Jenny Jun, Maggie Wu, Rebecca Gelles
- **Institution:** Georgetown University, Center for Security and Emerging Technology (CSET)
- **Year:** 2024
- **Venue:** CSET Issue Brief
- **Key Findings:** Three risk categories: (1) models generating insecure code, (2) models vulnerable to attack, (3) downstream feedback loops in training. 40% of Copilot-generated programs were vulnerable to CWE Top 25. Burden of security should not rest solely on users.
- **Link:** [https://cset.georgetown.edu/publication/cybersecurity-risks-of-ai-generated-code/](https://cset.georgetown.edu/publication/cybersecurity-risks-of-ai-generated-code/)
- **Relevance to Vibe Coding:** Policy-oriented analysis of systemic risks when vibe coding is adopted at organizational scale.

#### Paper 34: From Vulnerabilities to Remediation: A Systematic Literature Review of LLMs in Code Security
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2412.15004)
- **Key Findings:** Comprehensive SLR covering both vulnerability generation and LLM-based remediation. LLMs hallucinate non-existent packages (up to 44.7%), recommend deprecated versions, and emit CI/CD configs enabling privilege escalation. Missing input sanitization is the most common flaw across languages.
- **Link:** [https://arxiv.org/abs/2412.15004](https://arxiv.org/abs/2412.15004)
- **Relevance to Vibe Coding:** Highlights supply-chain attacks as a specific vibe coding risk -- developers who don't review dependencies are vulnerable.

#### Paper 35: Beyond Functional Correctness: Exploring Hallucinations in LLM-Generated Code
- **Authors:** Liu et al.
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2404.00971)
- **Key Findings:** Established taxonomy of code hallucinations: 3 primary categories, 12 specific categories. Systematic analysis of hallucination distribution across LLMs and benchmarks. Identified causes and impacts unique to code-domain hallucinations.
- **Link:** [https://arxiv.org/abs/2404.00971](https://arxiv.org/abs/2404.00971)
- **Relevance to Vibe Coding:** Defines the taxonomy of errors vibe coders must learn to recognize and handle.

---

### 2.8 AI Operations / AIOps and Incident Management

#### Paper 36: AIOps Solutions for Incident Management: Technical Guidelines and A Comprehensive Literature Review
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2404.01363)
- **Key Findings:** Formalizes incident management in AIOps as a multi-phase workflow: perception, detection/prediction, diagnosis, mitigation/remediation, and learning/feedback. LLMs enable natural-language incident triage with robust reasoning capabilities.
- **Link:** [https://arxiv.org/abs/2404.01363](https://arxiv.org/abs/2404.01363)
- **Relevance to Vibe Coding:** Extends the "vibe" paradigm to operations -- natural language-driven incident management mirrors vibe coding's conversational approach.

#### Paper 37: When AIOps Become "AI Oops": Subverting LLM-driven IT Operations via Telemetry Manipulation
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** RSA Conference / arXiv (2508.06394)
- **Key Findings:** Adversaries can manipulate system telemetry to mislead AIOps agents into taking actions that compromise infrastructure integrity. Demonstrates that LLM-based operations tools have unique attack surfaces requiring new defense strategies.
- **Link:** [https://arxiv.org/abs/2508.06394](https://arxiv.org/abs/2508.06394)
- **Relevance to Vibe Coding:** Warns that the "trust the AI" ethos of vibe coding is exploitable in operational contexts.

---

### 2.9 Foundational Models and Benchmarks

#### Paper 38: Competition-Level Code Generation with AlphaCode
- **Authors:** Yujia Li, David Choi, Junyoung Chung, Nate Kushman, et al.
- **Institution:** Google DeepMind
- **Year:** 2022
- **Venue:** Science, Vol. 378, No. 6624, pp. 1092-1097
- **Key Findings:** First AI system to achieve competitive-level performance in programming competitions, ranking in the top 54.3% on Codeforces. Generated millions of candidate programs, then filtered to 10 submissions. Three critical components: clean dataset, large transformer architectures, large-scale sampling with behavioral filtering.
- **Link:** [https://www.science.org/doi/10.1126/science.abq1158](https://www.science.org/doi/10.1126/science.abq1158)
- **Relevance to Vibe Coding:** Foundational demonstration that AI can solve novel programming problems -- the capability that makes vibe coding possible.

---

### 2.10 AI Code Review

#### Paper 39: Does AI Code Review Lead to Code Changes? A Case Study of GitHub Actions
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** arXiv (2508.18771)
- **Key Findings:** Large-scale empirical study of 16 AI-based code review GitHub Actions from ~240 on the marketplace. Analyzed whether AI review comments actually lead to code changes and improved quality.
- **Link:** [https://arxiv.org/abs/2508.18771](https://arxiv.org/abs/2508.18771)
- **Relevance to Vibe Coding:** AI review can serve as a safety net for vibe-coded output, but adoption and effectiveness vary widely.

#### Paper 40: Automated Code Review In Practice
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** arXiv (2412.18531)
- **Key Findings:** Industrial study of LLM-based code review (Qodo PR Agent) across 238 practitioners and 4,335 pull requests. Examined real-world adoption patterns, developer acceptance rates, and quality impact in production settings.
- **Link:** [https://arxiv.org/abs/2412.18531](https://arxiv.org/abs/2412.18531)
- **Relevance to Vibe Coding:** Demonstrates that automated review can partially compensate for the reduced human review in vibe coding workflows.

---

### 2.11 Surveys and Meta-Analyses

#### Paper 41: A Survey on Large Language Models for Code Generation
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2025
- **Venue:** ACM TOSEM (arXiv 2503.01245)
- **Key Findings:** Comprehensive survey covering 294 arXiv papers, 73 ACM, 36 IEEE, 261 DBLP papers (cutoff June 2024). Documents near-exponential growth: 1 paper (2018-2020), 6 (2021), 11 (2022), 75 (2023), 140 (2024). Reviews fine-tuning techniques, evaluation metrics, benchmarks, and applications.
- **Link:** [https://arxiv.org/abs/2503.01245](https://arxiv.org/abs/2503.01245)
- **Relevance to Vibe Coding:** The most comprehensive survey of the technical foundations enabling vibe coding.

#### Paper 42: Large Language Models for Software Engineering: A Systematic Literature Review
- **Authors:** Multiple
- **Institution:** Multiple
- **Year:** 2024
- **Venue:** ACM TOSEM (arXiv 2308.10620)
- **Key Findings:** 395 articles from Jan 2017 to Jan 2024. Covers LLM applications across the full software engineering lifecycle: requirements, design, coding, testing, maintenance. Fine-tuned models outperform base models; Code LLMs outperform general LLMs at comparable parameter counts.
- **Link:** [https://dl.acm.org/doi/full/10.1145/3695988](https://dl.acm.org/doi/full/10.1145/3695988)
- **Relevance to Vibe Coding:** Provides the comprehensive academic grounding for understanding where LLMs fit across the entire SDLC.

---

### 2.12 MIT CSAIL Specific Research

#### Paper 43: Modular Software Generation with LLMs
- **Authors:** MIT CSAIL researchers
- **Institution:** MIT CSAIL
- **Year:** 2025
- **Venue:** MIT News
- **Key Findings:** Framework using modular concepts and synchronization rules to make LLM-generated software clearer, safer, and more maintainable. A DSL expresses synchronizations in a form LLMs can reliably generate, addressing the maintainability concerns of AI-generated code.
- **Link:** [https://news.mit.edu/2025/mit-researchers-propose-new-model-for-legible-modular-software-1106](https://news.mit.edu/2025/mit-researchers-propose-new-model-for-legible-modular-software-1106)
- **Relevance to Vibe Coding:** Proposes architectural constraints that could make vibe-coded software more maintainable by design.

#### Paper 44: LILO: Natural Language-Enhanced Library Learning for Code Synthesis
- **Authors:** MIT CSAIL researchers
- **Institution:** MIT CSAIL
- **Year:** 2024
- **Venue:** MIT CSAIL
- **Key Findings:** Neurosymbolic method pairing LLMs with the Stitch abstraction-finding tool. Outperformed standalone LLMs and DreamCoder on tasks requiring commonsense knowledge. Demonstrates that library learning + natural language documentation improves code generation quality.
- **Link:** [https://www.csail.mit.edu/news/natural-language-boosts-llm-performance-coding-planning-and-robotics](https://www.csail.mit.edu/news/natural-language-boosts-llm-performance-coding-planning-and-robotics)
- **Relevance to Vibe Coding:** Shows that well-documented abstractions help LLMs generate better code -- implications for structuring vibe coding projects.

---

## 3. Key Takeaways

### Productivity
1. **AI coding tools provide 21-56% speedups on scoped tasks** (Peng et al., Google RCT, multi-company study), but the METR study shows a 19% *slowdown* for experienced developers on real-world tasks in mature codebases.
2. **The "productivity paradox" is real.** Developers consistently *perceive* they are faster with AI even when they are not (METR 2025). Metrics that count lines of code or PRs merged overstate true productivity.
3. **Benefits are unevenly distributed.** Less experienced developers gain more from AI assistance (Peng et al., Xu et al.), while experienced developers absorb increased review and maintenance burden.

### Quality
4. **Short-term velocity trades against long-term complexity.** The CMU Cursor study directly measures this: speed gains are transient, but complexity increases persist.
5. **Code correctness varies by language and task.** Copilot gets 30-70% of LeetCode problems right depending on language. For production code, error rates are higher on complex, multi-file tasks.
6. **Code hallucinations have a formal taxonomy** (Liu et al. 2024) spanning task requirement conflicts, factual knowledge conflicts, and project context conflicts.

### Security
7. **12-65% of AI-generated code contains security vulnerabilities** depending on model, language, and configuration (IEEE SLR 2024).
8. **Supply chain risks are acute.** LLMs hallucinate packages (up to 44.7%), recommend vulnerable versions, and generate CI/CD configs with privilege escalation risks.
9. **Organizational capability matters.** Georgetown CSET finds security burden is unevenly distributed -- well-resourced organizations handle AI code risks better.

### Autonomous Agents
10. **Agent capabilities are improving exponentially.** SWE-bench solve rates: ~4% (early 2023) to 12.5% (SWE-agent, 2024) to 72%+ (OpenHands v1, 2025).
11. **Multi-agent systems outperform single agents** for complex tasks. AgentCoder's three-agent test-driven approach achieves 91.5% pass@1.
12. **Self-evolving agents have arrived.** Live-SWE-agent autonomously improves during runtime, achieving 77.4% on SWE-bench Verified.

### Vibe Coding Paradigm
13. **Vibe coding is academically recognized** via Stanford HAI 2025 AI Index, multiple arxiv surveys, and dedicated benchmarks (FeatBench).
14. **Professionals don't fully "vibe" -- they control.** The arxiv study on professional developers shows they use AI agents but retain design authority and employ control strategies.
15. **The vibe-vs-agentic distinction matters.** Vibe coding (human-in-the-loop, conversational) and agentic coding (autonomous, goal-driven) have different risk profiles and use cases.

### Operations
16. **AIOps with LLMs enables natural language incident management** but introduces new attack surfaces via telemetry manipulation.
17. **The "vibe" approach is extending to operations**, with conversational interfaces for monitoring, alerting, and incident response.

---

## 4. Bibliography

1. He, et al. "Speed at the Cost of Quality: How Cursor AI Increases Short-Term Velocity and Long-Term Complexity in Open-Source Projects." *MSR '26*, 2026. arXiv:2511.04427.

2. Hong, S., et al. "MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework." arXiv:2308.00352, 2023.

3. Huang, D., et al. "AgentCoder: Multi-Agent-based Code Generation with Iterative Testing and Optimisation." arXiv:2312.13010, 2024.

4. Ji, J., Jun, J., Wu, M., Gelles, R. "Cybersecurity Risks of AI-Generated Code." Georgetown CSET Issue Brief, November 2024.

5. Li, Y., Choi, D., Chung, J., Kushman, N., et al. "Competition-Level Code Generation with AlphaCode." *Science* 378(6624): 1092-1097, 2022.

6. Liu, et al. "Beyond Functional Correctness: Exploring Hallucinations in LLM-Generated Code." arXiv:2404.00971, 2024.

7. METR. "Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity." arXiv:2507.09089, 2025.

8. MIT CSAIL. "Natural Language Boosts LLM Performance in Coding, Planning, and Robotics." MIT News, May 2024.

9. MIT CSAIL. "MIT Researchers Propose a New Model for Legible, Modular Software." MIT News, November 2025.

10. Multiple authors. "A Survey of Vibe Coding with Large Language Models." arXiv:2510.12399, 2025.

11. Multiple authors. "AIOps Solutions for Incident Management: Technical Guidelines and A Comprehensive Literature Review." arXiv:2404.01363, 2024.

12. Multiple authors. "ALMAS: An Autonomous LLM-based Multi-Agent Software Engineering Framework." arXiv:2510.03463, 2025.

13. Multiple authors. "Assessing and Analyzing the Correctness of GitHub Copilot's Code Suggestions." *ACM TOSEM*, 2024.

14. Multiple authors. "Automated Code Review In Practice." arXiv:2412.18531, 2024.

15. Multiple authors. "Does AI Code Review Lead to Code Changes? A Case Study of GitHub Actions." arXiv:2508.18771, 2025.

16. Multiple authors. "Early-Stage Prediction of Review Effort in AI-Generated Pull Requests." arXiv:2601.00753, 2026.

17. Multiple authors. "Enhancing Computer Programming Education with LLMs." arXiv:2407.05437, 2024.

18. Multiple authors. "Exploring the Problems, Their Causes and Solutions of AI Pair Programming." *Journal of Systems and Software*, 2024.

19. Multiple authors. "FeatBench: Evaluating Coding Agents on Feature Implementation for Vibe Coding." arXiv:2509.22237, 2025.

20. Multiple authors. "From Developer Pairs to AI Copilots: A Comparative Study." arXiv:2506.04785, 2025.

21. Multiple authors. "From Vulnerabilities to Remediation: A Systematic Literature Review of LLMs in Code Security." arXiv:2412.15004, 2024.

22. Multiple authors. "How AI Coding Agents Modify Code: A Large-Scale Study of GitHub Pull Requests." arXiv:2601.17581, 2026.

23. Multiple authors. "Large Language Models for Software Engineering: A Systematic Literature Review." *ACM TOSEM*, arXiv:2308.10620, 2024.

24. Multiple authors. "A Survey on Large Language Models for Code Generation." *ACM TOSEM*, arXiv:2503.01245, 2025.

25. Multiple authors. "Live-SWE-agent: Can Software Engineering Agents Self-Evolve on the Fly?" arXiv:2511.13646, 2025.

26. Multiple authors. "LLM-Based Multi-Agent Systems for Software Engineering: Literature Review, Vision and the Road Ahead." arXiv:2404.04834, 2024.

27. Multiple authors. "LLM Hallucinations in Practical Code Generation." *ISSTA 2025*, arXiv:2409.20550, 2025.

28. Multiple authors. "Professional Software Developers Don't Vibe, They Control: AI Agent Use for Coding in 2025." arXiv:2512.14012, 2025.

29. Multiple authors. "Promptware Engineering: Software Engineering for LLM Prompt Development." arXiv:2503.02400, 2025.

30. Multiple authors. "State of the Art of the Security of Code Generated by LLMs: A Systematic Literature Review." *IEEE*, 2024.

31. Multiple authors. "SWE-Bench Pro: Can AI Agents Solve Long-Horizon Software Engineering Tasks?" arXiv:2509.16941, 2025.

32. Multiple authors. "The Impact of AI-Assisted Pair Programming on Student Motivation, Programming Anxiety, and Performance." *International Journal of STEM Education*, 2025.

33. Multiple authors. "The Rise of AI Teammates in Software Engineering (SE) 3.0." arXiv:2507.15003, 2025.

34. Multiple authors. "Vibe Coding vs. Agentic Coding: Fundamentals and Practical Implications." arXiv:2505.19443, 2025.

35. Multiple authors. "When AIOps Become 'AI Oops': Subverting LLM-driven IT Operations via Telemetry Manipulation." arXiv:2508.06394, 2025.

36. Peng, S., Kalliamvakou, E., Cihon, P., Demirer, M. "The Impact of AI on Developer Productivity: Evidence from GitHub Copilot." arXiv:2302.06590, 2023.

37. Sherry, et al. "Is AI the Better Programming Partner? Human-Human Pair Programming vs. Human-AI." *CEUR Workshop Proceedings*, CMU, 2023.

38. Stanford HAI. "AI Index Report 2025." Stanford University, 2025.

39. Sundaram, et al. "How Much Does AI Impact Development Speed? An Enterprise-Based Randomized Controlled Trial." arXiv:2410.12944, 2024.

40. Wang, et al. "OpenHands: An Open Platform for AI Software Developers as Generalist Agents." *ICLR 2025*, arXiv:2407.16741, 2025.

41. Xu, F., Medappa, P.K., Tunc, M.M., Vroegindeweij, M., Fransoo, J.C. "AI-Assisted Programming Decreases the Productivity of Experienced Developers by Increasing the Technical Debt and Maintenance Burden." arXiv:2510.10165, 2025.

42. Yang, J., et al. "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering." *NeurIPS 2024*, arXiv:2405.15793, 2024.

43. Qian, C., et al. "ChatDev: Communicative Agents for Software Development." arXiv:2307.07924, 2023.

44. Ziegler, A., Kalliamvakou, E., et al. "Measuring GitHub Copilot's Impact on Productivity." *Communications of the ACM*, 2024.
