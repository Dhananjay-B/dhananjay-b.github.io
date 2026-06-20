# ☸️ KubeCon + CloudNativeCon India 2026: Notes and Learnings

![Secure AI Architecture](../../assets/images/KubeCon.png)
_Image source - https://events.linuxfoundation.org/_

I attended KubeCon + CloudNativeCon India 2026 for the first time this year. Rather than documenting every session, I wanted to capture the ideas, technologies, and trends that stood out.

The talks I attended largely fell into three buckets:

* AI and Agentic Systems
* Security and Supply Chain Security
* Platform Engineering and Reliability

What became apparent throughout the conference, however, was that these domains are increasingly converging.

Below are my notes and takeaways.

# 🤖 AI Meets Infrastructure

## 🚦 LLMd: Intelligent Routing for AI Inference

Traditional load balancers assume all replicas are equal.

For LLM workloads, that's often not true.

LLMd proposes routing inference requests based on:

* Current pod load
* Cache hit probability
* Context locality

Instead of blindly distributing traffic, requests can be routed to pods that already contain relevant context, reducing recomputation and improving GPU utilization.

### What stood out

* AI workloads introduce cache locality as a scheduling concern.
* Infrastructure decisions now directly influence inference performance.
* GPU efficiency is becoming as important as raw compute availability.

**Takeaway:** The best destination for an AI request is not always the least-loaded pod.

## ⚙️ Running AI Agents on Kubernetes

This session focused on understanding three concepts that increasingly appear together:

* AI Agents
* MCP Servers
* Dynamic Resource Allocation (DRA)

Rather than viewing them independently, the talk showed how they combine to form adaptive AI platforms.

One particularly interesting idea was using agents to influence infrastructure decisions dynamically, allowing workloads to optimize GPU allocation and scheduling in real time.

### What stood out

Kubernetes is no longer just hosting AI workloads.

AI workloads are beginning to participate in infrastructure decisions themselves.

**Takeaway:** Kubernetes is becoming a platform increasingly managed by AI systems.

## 🧠 Building Production-Ready Agentic AI Systems

One recurring theme throughout KubeCon was that an AI application is much more than an LLM endpoint.

Production systems require:

* Memory
* Tool calling
* RAG
* Guardrails
* Observability
* Resilience

The session also emphasized LangChain4J as a Java-native framework for building agent workflows.

Interesting orchestration patterns included:

* Prompt chaining
* Reflection loops
* Parallel execution
* Intelligent routing

### What stood out

As systems evolve from:

```text
1 Service
1 Model
```

to:

```text
N Services
M Models
K Agents
```

the challenge shifts away from prompting and toward orchestration, state management, and reliability.

**Takeaway:** Agentic systems should be treated as distributed systems.

# 🔒 Security Boundaries and Isolation

## 👤 Root Without Risk: User Namespaces

A long-standing container security problem is that root inside a container often maps to root on the host.

Kubernetes User Namespaces address this by mapping container root to an unprivileged user on the host.

Benefits include:

* Reduced impact of container escapes
* Better isolation boundaries
* Lower blast radius from runtime vulnerabilities

The session also demonstrated how this mitigates several container escape scenarios.

**Takeaway:** Root inside a container should not imply root on the host.

## 🔒 LLMs Behind Bars: Sandboxes at Scale

The moment an AI system executes generated code, it effectively becomes a remote code execution platform.

The challenge is obvious:

* Host compromise
* Data exfiltration
* Secret exposure
* Resource abuse
* Network abuse

The session focused on building production-grade sandboxes using:

* Containers
* Filesystem isolation
* Restricted syscalls
* Resource limits
* Network controls

The emphasis was not only on security but also on operational challenges such as startup latency, observability, and scaling thousands of short-lived execution environments.

**Takeaway:** Treat AI-generated code exactly like code submitted by an unknown internet user.

## ⛏️ Stopping a Crypto-Mining Attack Triggered by a Next.js RCE

This session demonstrated how a Next.js RCE vulnerability can evolve into a Kubernetes security incident.

Attack progression:

```text
Application RCE
        ↓
Container Access
        ↓
Payload Download
        ↓
Crypto Mining
        ↓
Resource Abuse
```

Defensive controls included:

* Falco
* Kyverno
* Resource limits
* Workload isolation
* Least privilege

### What stood out

Patching is necessary, but it is rarely sufficient.

The real outcome depends on the controls available after compromise.

**Takeaway:** Defense-in-depth determines the blast radius of a successful attack.

# 🌍 Governance, Compliance and Trust

## 🌍 Your Prompt Is a Cross-Border Data Transfer

Every AI request creates a chain of data movement.

A seemingly simple prompt may involve:

* Vector databases
* RAG pipelines
* MCP servers
* External tools
* Logging systems
* Third-party models

Organizations often know where their databases reside but cannot explain where a prompt traveled.

The proposed concept was an **AI Receipt** using OpenTelemetry traces to reconstruct the full lifecycle of an AI request.

### Why it matters

As AI adoption increases, data sovereignty and governance become observability problems.

Questions such as:

* Which systems processed the prompt?
* Which country handled the data?
* Which tools received context?

need verifiable answers.

**Takeaway:** AI observability must include data lineage, not just application telemetry.

## 🌳 Cryptographic SBOM Auditing Without IP Leakage

As SBOM requirements become more common, organizations face a difficult tradeoff:

* Auditors want transparency.
* Vendors want confidentiality.

The proposed solution used Merkle-tree commitments to separate verification from disclosure.

Instead of sharing an entire SBOM:

* Publish a cryptographic commitment.
* Reveal only the components required for an audit.
* Provide proof that the disclosed components belong to the committed SBOM.

### Why it matters

This introduces the concept of selective disclosure for compliance.

Organizations can prove integrity without exposing their entire software composition.

**Takeaway:** Compliance may increasingly rely on cryptographic proofs rather than trust-based reporting.

## 📜 Automating Compliance with Policy-as-Code

One of the more practical sessions focused on translating compliance requirements directly into machine-enforceable policies.

The model consisted of three layers:

### Proactive Controls

Catch violations during CI/CD.

### Preventive Controls

Block non-compliant workloads from reaching production.

### Detective Controls

Continuously gather evidence and monitor environments.

### Why it matters

Compliance often becomes a periodic audit exercise.

Policy-as-Code shifts compliance into the software delivery lifecycle itself.

**Takeaway:** Compliance should be an engineering capability, not an audit event.

# 🏗️ Operating Platforms at Scale

## 💥 Chaos Engineering at Scale

Flipkart shared how they built a centralized chaos engineering platform on top of LitmusChaos.

Interesting implementation details included:

* Hybrid multi-tenancy
* DaemonSet-based chaos injection
* Script Runner for dynamic targeting
* Support for VM workloads

The platform transformed chaos engineering from isolated experiments into a reusable organizational capability.

**Takeaway:** Reliability practices become impactful when they are self-service and repeatable.

## 📜 Why Your Cluster-Wide Policies Are a Risk

Cluster-wide policies provide strong governance but also create large blast radii.

A single policy mistake can impact every workload in a cluster.

The session explored namespace-scoped policy models that provide:

* Better ownership
* Team autonomy
* Reduced operational bottlenecks
* Smaller failure domains

### What stood out

Not every policy needs to be cluster-wide.

Many application-specific controls are better managed closer to the workloads they govern.

**Takeaway:** Policy scope is a security decision.

## 🔗 Five Tactics for a Hardened Software Chain

This lightning talk framed software supply chain security around five trust boundaries:

1. Source
2. Build
3. Registry
4. Admission
5. Runtime

Recommended controls included:

* Golden paths
* SBOM generation
* Artifact signing
* Admission policies
* eBPF runtime protection

### What stood out

The message was simple:

Security failures often occur between stages rather than within stages.

**Takeaway:** Secure the entire software delivery lifecycle, not just the Kubernetes cluster.

## 🎯 Pruning Kernel CVEs with Reachability Analysis

Most vulnerability management programs still rely heavily on version matching.

This session argued that many kernel vulnerabilities may not be exploitable because the vulnerable code:

* Is not compiled
* Is not enabled
* Is not loaded
* Is not reachable

The proposed approach maps CVEs to vulnerable functions and performs reachability analysis using:

* Kconfig
* Loaded modules
* Call graphs
* Runtime context

### Why it matters

This can significantly reduce vulnerability noise while helping teams focus on vulnerabilities that represent practical risk.

**Takeaway:** Risk should be based on reachability, not simply the presence of vulnerable code.

## 👥 The Human Cost of Security in the AI Era

This keynote was less about technology and more about the people responsible for maintaining it.

Security discussions often focus on:

* CVEs
* Advisories
* Vulnerability scores
* Supply chain incidents

What is often ignored is the human effort required to:

* Investigate
* Patch
* Communicate
* Respond

The session explored how AI may simultaneously increase and reduce this burden.

**Takeaway:** Every vulnerability has a CVSS score, but every vulnerability also has a human cost.

# 🔍 Themes That Kept Appearing

Across AI, security, and platform engineering sessions, three ideas surfaced repeatedly.

## 🧠 1. AI Is Becoming Infrastructure

The conversation is no longer about calling LLM APIs.

It is about:

* Scheduling GPUs
* Routing inference traffic
* Running agents
* Sandboxing generated code
* Operating AI platforms

Kubernetes is rapidly becoming the default platform for AI systems.

## ✅ 2. Verification Is Replacing Trust

Several talks focused on proving security properties instead of assuming them.

Examples included:

* Cryptographic SBOMs
* Policy-as-Code
* Artifact signing
* Runtime enforcement
* Reachability analysis

The common theme was verifiable security.

## ⚙️ 3. Platforms Are Absorbing Complexity

Whether the topic was:

* Chaos engineering
* Compliance
* AI infrastructure
* Security policies

the goal was consistent:

Move operational complexity into the platform so application teams can focus on delivering business value Overall, KubeCon 2026 reinforced that Kubernetes is no longer just a container orchestration platform. It is increasingly becoming the operating system for AI, security, compliance, and modern platform engineering.

If any of these topics interest you, feel free to reach out. I would be happy to share session links, references, and more detailed notes.

### 📢 Share this post

[![Twitter](https://img.shields.io/badge/Share-Twitter-1DA1F2?style=for-the-badge\&logo=twitter)](https://twitter.com/intent/tweet?text=Checkout%20this%20post%20by%20Dhananjay%20Bhujbal%20...%20KubeCon%20%2B%20CloudNativeCon%20India%202026%3A%20Notes%20and%20Learnings&url=https://onemorelens.co.in/posts/Container-Security/20-06-2026-kubecon-cloudnativecon-mumbai.html)

[![LinkedIn](https://img.shields.io/badge/Share-LinkedIn-0A66C2?style=for-the-badge\&logo=linkedin)](https://www.linkedin.com/sharing/share-offsite/?url=https://onemorelens.co.in/posts/Container-Security/20-06-2026-kubecon-cloudnativecon-mumbai.html)

[![WhatsApp](https://img.shields.io/badge/Share-WhatsApp-25D366?style=for-the-badge\&logo=whatsapp)](https://api.whatsapp.com/send?text=Checkout%20this%20post%20by%20Dhananjay%20Bhujbal%20...%20KubeCon%20%2B%20CloudNativeCon%20India%202026%3A%20Notes%20and%20Learnings.%20https://onemorelens.co.in/posts/Container-Security/20-06-2026-kubecon-cloudnativecon-mumbai.html)
