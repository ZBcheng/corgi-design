# Corgi Design

## 📚 Document Navigation

### Quick Start Guide

**Recommended Approach**:

📖 **Read this README.md from top to bottom**. It will guide you through the complete design journey and reference other documents at the appropriate points in the narrative.

- The README naturally flows through: Problem Analysis → High-Level Design → Detailed Design → Solutions
- Follow the document references as they appear in context
- You'll encounter links to [00-high-level-design.md](design/00-high-level-design.md) and [solutions.md](solutions.md) when they're relevant

**Optional Deep Dive**: If you're interested in technical implementation details, the README also references **[Detailed Design Documents (01-06)](design/)** - explore these only if you need module-specific specifications.

### Document Structure

| Document | Purpose | How to Use |
|----------|---------|------------|
| **[README.md](README.md)** (This file) | ⭐ **Your guide** - Problem-solving approach, design process, with contextual references to all other documents | Read from top to bottom, follow references as they appear |
| **[questions.txt](questions.txt)** | Original problem statements and requirements | Referenced in README's "Requirements Analysis" section |
| **[design/00-high-level-design.md](design/00-high-level-design.md)** | System architecture, design principles, component responsibilities | Referenced in README's "High-Level Design" section |
| **[design/01-06-*.md](design/)** | Layer-by-layer technical specifications and implementation details | Referenced throughout README and solutions.md when technical depth is needed |
| **[solutions.md](solutions.md)** | Complete answers to all questions with diagrams and references | Referenced in README's "Summary" section |

---


## 🎯 Problem-Solving Approach

This document follows a systematic, AI-assisted problem-solving methodology:

### Step-by-Step Process

1. **Read and Understand the Problems** ([questions.txt](questions.txt))
   - Analyze each problem statement carefully
   - Identify key requirements and constraints

2. **Divide and Conquer Strategy**
   - If problems are independent → Solve each one separately
   - If problems are related → Merge related problems, then solve systematically

3. **Extract Requirements and Define the System** ([System Definition](#system-defination))
   - Identify core challenges from the problem statements
   - Generate a cohesive system definition document
   - Define key architectural pillars

4. **Learn Unknown Concepts** ([Design Deep Dive & Technical Q&A](#design-deep-dive--technical-qa-gemini-3-pro))
   - Research and understand unfamiliar technical concepts
   - Document key learnings (e.g., GBDT, XGBoost, Holdout methods, Shadow mode)

5. **Generate High-Level Design** ([design/00-high-level-design.md](design/00-high-level-design.md))
   - Break down the system into sub-problems (modules/layers)
   - Define module responsibilities and interactions
   - Create architecture diagrams

6. **Detailed Design for Each Module** ([design/01-06-*.md](design/))
   - Design each module/layer with technical specifications
   - Define technology stack, data models, and interfaces
   - Document performance requirements and trade-offs

7. **Generate Solution Summary** ([solutions.md](solutions.md))
   - Map each original question to the design solution
   - Reference specific sections in design documents
   - Provide visual diagrams and implementation details

### AI Tools Used

**All steps above were completed with AI assistance**, using multiple tools:

GitHub Copilot, Google Gimini, Google AI, Kiro, Warp AI

This approach demonstrates how AI can accelerate the entire system design process, from problem analysis to detailed implementation specifications.

---

## 📋 Requirements Analysis

### Prompt for System Definition (Claude Sonnet 4.5)

```text
Role: You are a Senior Principal Solution Architect. Your task is to synthesize a set of fragmented technical questions into a cohesive System Design Specification.

Context: I have been brainstorming various components of a technical challenge. I will provide a list of specific questions I have asked.

Task: 
1. Identify the Core Objective: Based on these questions, define what kind of system I am trying to build (e.g., "A Real-time Fraud Detection Pipeline"). 
2. Synthesize System Requirements: Group the questions into functional themes (e.g., Data Reliability, Model Serving, Storage Efficiency). 
3. Highlight Primary Constraints: Identify the non-functional requirements (like latency, cost-efficiency, or data consistency) that these questions imply.

The Questions: 
1. How would you design a system to ingest data from a payment processor API, run ETL,
and store in a database for ML model development on a continuous basis (ideally
optimized for data storage and runtime) and then deliver the model for near real time
inference? You can select any payment processor. Please be as detailed and specific
as possible.
2. One of the classical problems in payments data science is how to measure the
effectiveness of a new fraud model, without exposing your business to fraud by
substituting a model that performs well in training but fails in production. A potential
solution is to use a small holdout sample, which is not subject to any fraud prevention
algorithms or rules, and use that sample to A/B test new models vs existing models.
How would you design this system? What are the business considerations that you
need to take into account? Do you have ideas for other less risky solutions?
3. We’re working with a new payment provider, and they don’t have an API we can use
to ingest data. What kind of system/process could you come up with to receive their
data, process it and then deliver the model for inference on their architecture?
Assume they already have a decision engine on their side that works well.
4. What’s the best way to take a 3,000 variables and deep learning model (GBDT,
XgBoost, Random Forest) for payment fraud to run inference on an incoming
payments object with near-real-time inference? (hint: think about high frequency trading)

Output Format: Please provide a structured summary including "System Definition," "Key Architectural Pillars,".
```


### System Defination

A Real-time Payment Fraud Detection and ML Operations Platform - An end-to-end MLOps system that ingests payment transaction data from heterogeneous sources, maintains optimized storage for analytics and operations, continuously trains and validates fraud detection models, and serves near real-time predictions at high frequency with built-in safety mechanisms for model experimentation.

### Key Architectural Pillars

- Multi-Source Data Ingestion & Integration
  - API-based ingestion from payment processors with continuous ETL
  - Non-API integration layer for legacy payment providers (file-based, batch, or custom protocols)
  - Unified data schema and normalization layer

- Optimized Dual-Storage Architecture
  - Operational data store for low-latency model serving
  - Analytical data warehouse optimized for ML feature engineering and training
  - Storage optimization for cost and runtime efficiency

- Continuous ML Pipeline
  - Automated feature engineering with 3000+ variable support
  - Model training orchestration for ensemble methods (GBDT, XGBoost, Random Forest)
  - Model versioning and registry

- Safe Model Deployment & Experimentation
  - Holdout sample mechanism (unaffected by fraud rules) for unbiased evaluation
  - A/B testing framework with gradual rollout
  - Shadow mode deployment for risk mitigation
  - Champion/challenger model strategy

- High-Performance Inference Engine
  - Near real-time prediction serving (sub-100ms latency)
  - High-frequency transaction processing capability
  - Feature store for real-time feature retrieval
  - Model optimization (quantization, pruning) for large ensemble models

- Cross-Platform Integration
  - Inference API for external decision engines
  - Model export/deployment to third-party architectures
  - Bidirectional data synchronization

## 💡 Design Deep Dive & Technical Q&A (Gemini 3 Pro)

```markdown
### 1. Data Acquisition Strategy (No-API Providers)

**Q: For a provider like Stripe, if they don't offer an API to query transaction information, how can I typically get their data?**

To retrieve transaction data without directly querying the API, you typically rely on the following methods:

*   **Dashboard Exports**: Manually downloading or scheduling CSV/Excel reports directly from the provider's interface.
*   **Webhooks**: Configuring the provider to push real-time data to your system whenever an event (like a payment) occurs.
*   **Native Data Tools**: Using built-in features like Stripe Sigma (for SQL querying) or Data Pipelines to sync data to a warehouse.
*   **Third-Party Connectors**: Leveraging no-code tools like Zapier or Fivetran to automatically sync data to spreadsheets or databases.

**Q: If the provider offers files, what formats are typically used?**

If a provider (like Stripe, PayPal, or a bank) offers downloadable files for transaction data, the most typical formats are:

1.  **CSV** (Comma-Separated Values)
2.  **XLSX** (Microsoft Excel)
3.  **JSON** (JavaScript Object Notation)

### 2. Machine Learning Pipeline Architecture

**Q: What features do ML pipelines typically provide?**

Machine learning (ML) pipelines focus on these core functions:

*   **Data Automation**: Automating data ingestion, cleaning, and feature engineering to ensure a consistent, high-quality data flow.
*   **Model Training & Tuning**: Systematic training of algorithms and automated hyperparameter optimization to find the best-performing model.
*   **Evaluation & Validation**: Rigorous testing against performance metrics (e.g., accuracy, loss) to ensure the model is production-ready.
*   **Orchestration & Versioning**: Managing the execution sequence and tracking versions of code, data, and models for reproducibility.
*   **Deployment & Monitoring**: Rolling models into production and tracking data drift or performance decay to trigger alerts or retraining.

**Q: What do 3,000+ variables signify in feature engineering, and how do they impact system performance bottlenecks?**

*   **Compute & Memory (RAM Saturation)**: Massive feature matrices can trigger Out-Of-Memory (OOM) errors.
*   **I/O Blocking**: Transferring 3,000+ columns creates a massive throughput bottleneck, leaving GPUs/CPUs idle while waiting for data.
*   **Inference Latency**: Calculating 3,000+ features in real-time during production significantly increases request-response latency, making high-speed applications impossible.
*   **Storage & Orchestration**: Versioning and syncing 3,000+ features in a Feature Store creates immense operational complexity and increases storage costs.

### 3. Model Logic & Algorithms

**Q: What are GBDT, XGBoost, and Random Forest, and what are their commonalities?**

#### Core Logic (The Difference)
*   **GBDT**: A sequential approach. It builds trees one after another, where each new tree focuses on correcting the errors (residuals) of the previous ones.
*   **XGBoost**: The optimized evolution of GBDT. It adds regularization to prevent overfitting and enables high-speed parallel computing, making it the industry standard in 2026.
*   **Random Forest**: A parallel approach. It builds many independent trees simultaneously using random subsets of data/features and combines them via "voting" or "averaging."

#### Core Commonalities
*   **Ensemble Learning**: They all combine multiple Decision Trees to create a single strong model.
*   **Tabular Specialists**: They remain the top choice for structured (SQL-like) data, often outperforming deep learning in 2026.
*   **Non-linear Power**: They naturally handle complex relationships without needing data scaling or normalization.
*   **Feature Importance**: All three provide built-in transparency by ranking which variables have the most predictive power.
*   **Dual-Tasking**: Each supports both Classification and Regression.

### 4. Validation & Deployment Strategies

**Q: What are Holdout, Shadow mode, and the Champion/Challenger model strategy?**

#### 1. Holdout Method (Validation)
The Holdout method is a foundational data-splitting technique used to evaluate model performance on unseen data.
*   **How it works**: A portion of the original labeled dataset (typically 20%) is "held out" and never shown to the model during training.
*   **Purpose**: It acts as a final, unbiased test to estimate how the model will perform in the real world, preventing overfitting (where a model memorizes the training data instead of learning patterns).
*   **Context**: While K-Fold Cross-Validation is more robust for small datasets, the Holdout method is the standard for the massive datasets common in 2026 due to its speed and computational efficiency.

#### 2. Shadow Mode (Deployment)
Shadow Mode (or "Dark Launching") is a risk-free deployment strategy where a new model runs in production alongside the live model.
*   **How it works**: Real production traffic is sent to both models simultaneously. However, only the live model's results are delivered to users. The shadow model's predictions are merely logged for analysis.
*   **Purpose**: It allows engineers to verify that the new model can handle real-world load and data edge cases without any risk of affecting business operations or user experience.

#### 3. Champion/Challenger Model (Strategy)
The Champion/Challenger strategy is an operational framework for continuous model optimization in production.
*   **The Champion**: The current best-performing model (the "baseline") that is actively making decisions.
*   **The Challenger(s)**: One or more new or re-trained models competing to prove they are better than the Champion.
*   **The Process**: Both models are monitored against the same KPIs (e.g., accuracy, profit, or latency). If a Challenger consistently outperforms the Champion over a set period, it is "promoted" to become the new Champion.
*   **Comparison**: Unlike Shadow Mode, which is purely for safety testing, Champion/Challenger is a long-term A/B testing framework used to drive constant incremental improvements.
```

## 🏗️ High-Level Design

### Prompt for High-Level Design (Claude Sonnet 4.5)

```text
# Role
You are a Senior Systems Architect specializing in transforming conceptual requirements into robust, scalable, and modular High-Level Designs (HLD).

# Context
I will provide you with a **System Definition** and a set of **Key Architectural Pillars**. Your goal is to design a system architecture that adheres to these foundations while addressing specific problem statements.

# Task
Please generate a High-Level Design (HLD) document consisting of the following sections:

### 1. Architectural Strategy
- **Design Principles**: Outline the core philosophies guiding this design (e.g., separation of concerns, statelessness, event-driven).
- **Non-Functional Requirements (NFRs)**: Define how the system handles scalability, availability, maintainability, and security.

### 2. System Decomposition & Module Responsibilities
- Break down the system into logical modules/sub-systems.
- For each module, clearly define its **Primary Responsibility** and its **Role** within the ecosystem. 
- *Note: Focus on "what" each module does rather than the specific internal code implementation.*

### 3. Visualizations (Mermaid)
- **Architecture Diagram**: Provide a high-level block diagram showing the components and their relationships.
- **Workflow/Sequence Diagram**: Provide a diagram illustrating the end-to-end flow for a primary use case.

### 4. Problem-to-Module Mapping
- I have provided a list of specific problems/challenges. For each problem, identify which module(s) are responsible for resolving it and briefly explain the logic.

# Constraints
- Maintain a high-level perspective. Avoid deep-diving into specific database schemas or low-level API signatures unless necessary for clarity.
- Use Mermaid syntax for all diagrams.
- Ensure the design strictly aligns with the "Key Architectural Pillars" provided.
```

### Prompt for HLD Evaluation (Claude Opus 4.5)

```text
# Role
You are an Expert Technical Reviewer and Systems Architect.

# Context
I have developed a High-Level Design (HLD) based on a specific **System Definition** and **Key Architectural Pillars**. I need you to audit this design to ensure it is robust, compliant with the requirements, and professionally complete.

# Task
Please perform a dual-phase review of the provided HLD:

### Phase 1: Validation & Compliance Check
- Cross-reference the HLD against the **System Definition** and **Key Architectural Pillars**.
- Identify if any core requirements are missing or if the design deviates from the established pillars.
- Verify if the proposed modules and workflows effectively address the initial **Problem Statements**.

### Phase 2: Gap Analysis & Improvement Suggestions
- As a professional HLD document, identify any missing sections that should be included (e.g., Data Sovereignty, Error Handling Strategies, Scalability Bottlenecks, or Integration Patterns).
- Suggest ways to improve the clarity of the **Module Responsibilities** or the logic of the **Sequence Diagrams**.
- Point out any "Non-Functional Requirements" that might have been overlooked but are critical for this specific type of system.

# Output Format
1. **Executive Summary**: A brief pass/fail assessment of the design's alignment.
2. **Detailed Critique**: Specific points of alignment or misalignment.
3. **Actionable Recommendations**: A prioritized list of additions or refinements to elevate the HLD to industry standards.
```

### HDL Documents

**[High-Level Design (HLD)](design/00-high-level-design.md)** - Comprehensive system architecture, design principles, and component responsibilities

## 🔧 Detailed Design

### Prompt for Detailed Design (Claude Sonnet 4.5)

```text
# Role
You are a Senior Software Engineer and Technical Lead.

# Context
We have finalized the High-Level Design (HLD). It is now time to transition into the **Detailed Design** phase by decomposing the architecture into specific, actionable technical specifications.

# Task
Please generate a structured technical documentation set for the system. You are required to:

### 1. Create a Project Directory
- Initialize a directory named `detailed-design/`.

### 2. Generate Module-Specific Documents
- For each module identified in the HLD, create a dedicated markdown file (e.g., `module-name-design.md`).
- Each module document should include:
    - **Technology Stack Selection**: Justify the choice of specific technologies, frameworks, and tools for this module (e.g., PostgreSQL vs MongoDB, gRPC vs REST, Kafka vs RabbitMQ).
    - **Interface Definitions**: Define API endpoints, message schemas, or internal service interfaces using abstract representations (e.g., Golang interfaces, Python abstract classes, Rust traits).
    - **Data Models**: Describe the core data structures or local database schemas relevant to this module.
    - **Logic & Algorithms**: Explain the internal logic, state machines, or specific algorithms used.
    - **Error Handling**: Detail how the module manages failures and edge cases.
    - **Dependencies**: List external services, libraries, or other modules this component relies on.

# Constraints
- Maintain technical consistency across all documents.
- Use Mermaid for any detailed logic or internal state diagrams within the module files.
- Focus on implementation-ready details while adhering to the previously established Design Principles.
- **DO NOT provide detailed implementation code**. Instead, use language-specific abstractions where appropriate:
  - Rust: Trait definitions
  - Python: Abstract Base Classes (ABC), Protocol classes
- Provide architectural patterns and contracts, not line-by-line implementations.

# Technical Decision Guidelines

### Programming Language Selection
- **Rust**: Use for performance-critical or security-sensitive services where:
  - Sub-millisecond latency is required (e.g., high-frequency inference)
  - Memory safety is paramount (e.g., payment processing)
  - Zero-cost abstractions and minimal runtime overhead are needed
  - High-throughput network services (e.g., API gateways, data ingestion)
  - System-level concurrency without garbage collection pauses
- **Python**: Use for modules requiring rich ecosystems:
  - ML/Data Science (scikit-learn, PyTorch, TensorFlow)
  - Rapid prototyping and experimentation
  - Integration with existing Python-based data tools
  - Data processing pipelines (ETL, feature engineering)
  - Orchestration and automation scripts

### Database & Storage Selection
When selecting databases, consider the consistency, performance, and operational requirements of your use case.

#### Recommended: PostgreSQL (for strong consistency requirements)
- **Use when**: ACID guarantees, complex queries, and relational integrity are critical
- **MUST specify isolation level** for each use case:
  - `READ COMMITTED`: Default for OLTP operations
  - `REPEATABLE READ`: For financial transactions requiring snapshot isolation
  - `SERIALIZABLE`: For critical operations requiring strict serializability (note: performance impact)
- **Trade-offs**: Document consistency vs performance implications
- **Optimizations**: Connection pooling, prepared statements, read replicas

#### Alternative Options (justify your choice)
- **MySQL/MariaDB**: If existing ecosystem or specific features are required (e.g., Galera Cluster)
- **CockroachDB**: For distributed SQL with horizontal scalability and built-in geo-distribution
- **FoundationDB**: For strong consistency with high write throughput (>100k writes/sec)
- **MongoDB**: Only if document model is genuinely better fit (must justify over JSONB in PostgreSQL)
- **Cassandra/ScyllaDB**: For extreme write-heavy workloads with eventual consistency (must document consistency trade-offs)

**Selection Criteria**: For any database choice, document:
1. Consistency guarantees provided vs required
2. Performance characteristics (read/write patterns, latency)
3. Operational complexity (backup, replication, monitoring)
4. Cost implications (licensing, infrastructure)
5. Team expertise and learning curve

- **Data Synchronization**: When syncing between OLAP, OLTP, and Cache:
  - **MUST address consistency model**: Eventual consistency vs Strong consistency
  - Consider using Change Data Capture (CDC) patterns (e.g., Debezium, PostgreSQL logical replication)
  - Document acceptable staleness/lag for each consumer
  - Implement reconciliation mechanisms for drift detection
  - Consider using distributed transactions (2PC/Saga) only when absolutely necessary

### Performance & Capacity Planning
For each module, **MUST provide**:
1. **Traffic Estimation**:
   - Expected QPS (Queries Per Second) / TPS (Transactions Per Second)
   - Peak vs average load characteristics
   - Growth projections (6-month, 1-year, 3-year)

2. **Resource Requirements**:
   - **Bandwidth**: Network I/O calculations (ingress/egress)
   - **Storage**: Data volume growth rate, retention policies
   - **Compute**: CPU/Memory requirements per instance
   - **IOPS**: Disk I/O for database-heavy modules

3. **Performance Bottleneck Analysis**:
   - Identify primary bottleneck (CPU, Memory, I/O, Network)
   - Document scaling strategy:
     - Vertical scaling limits
     - Horizontal scaling approach (stateless vs stateful)
     - Partitioning/Sharding strategy
   - Provide latency breakdown (P50, P95, P99)


### Additional Considerations
- **Observability**: Every module must define key metrics, logging strategy, and distributed tracing approach
- **Security**: Document authentication, authorization, encryption (in-transit and at-rest), and compliance requirements (PCI-DSS, GDPR)
- **Testing Strategy**: Unit test coverage targets, integration test approach, load testing plan
```

### Detailed Design Documents

> **📖 Reading Guide**: You don't need to read every detailed design document in full. The **[solutions.md](solutions.md)** file provides comprehensive answers to all core questions with direct references to relevant sections in these detailed design documents. Use the detailed designs as reference material when you need deeper technical context.

- **[01 - Ingestion Layer](design/01-ingestion-layer.md)** - Real-time and batch data ingestion from payment providers
- **[02 - Processing Layer](design/02-processing-layer.md)** - ETL pipeline, data validation, and normalization
- **[03 - Storage Layer](design/03-storage-layer.md)** - OLTP, OLAP, Feature Store, and Model Registry
- **[04 - Inference Layer](design/04-inference-layer.md)** - Real-time fraud scoring service and model serving
- **[05 - Decision Layer](design/05-decision-layer.md)** - Business rules engine and fraud decision-making
- **[06 - ML Pipeline](design/06-ml-pipeline.md)** - Model training, validation, and deployment workflow

---


## 📝 Summary: Design Solutions to Core Challenges

### Prompt for Summary

```text
# Role: Expert System Architect & Technical Writer


## Context
We have finalized the system design. The core architecture is outlined in `design/00-high-level-design.md`. I need to document how the issues and requirements listed in `questions.md` are addressed within this design.

## Task
Analyze the provided documents and **write the following information into `solutions.md`**. This file should serve as a comprehensive record of how each problem in `questions.md` is resolved.

### Requirements:
1. **Design Logic Summary**: For each question in `questions.md`, provide a concise summary of the **design thinking** and strategy used to solve the problem.
2. **Layered Collaboration**: Explain how different layers collaborate to implement the solution.
3. **Visual Representation**: Where necessary, use **Mermaid.js** (e.g., sequence or flow diagrams) to visually represent the interaction or logic for clarity.
4. **Source Traceability**: You **must** explicitly cite the source files and specific sections for every solution provided.
   - *Example*: `[Ref: [design/00-high-level-design.md](design/00-design/00-high-level-design.md#1-architectural-strategy) - Architectural Strategy]`

## Output Format
- Professional Markdown structure within `solutions.md`.
- Clear headings for each question from `questions.md`.
- Concise "Design Logic" paragraphs followed by "Implementation Details."
- Integrated Mermaid diagrams for technical clarity where appropriate.
- Precise citations for auditability.
```

### Solution Document

**[Solutions](solutions.md)** - Detailed design solutions and architectural decisions addressing the core technical questions
