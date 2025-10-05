---
title: Model Definition
output: html_document
bibliography: references.bib
---

# Model Definition

The MACM (Multi-purpose Application Composition Model) is a graph-based formalism that can be adopted to represent a System under Analysis (SuA).

In general, a MACM is a **Property Graph**, namely a graph in which each node/arc is characterized by some *labels* and *properties*. From the semantic perspective, each node of the graph represents an asset of the SuA, while each edge represents a relationship between two assets.

## Modeling Assumptions

The MACM model provides a formal and structured representation of a SuA, with the goal of supporting reasoning, validation, and security assessment.

To ensure clarity and precision, the MACM relies on a number of fundamental assumptions:

- **Snapshot Semantics.**  
  The MACM captures the architecture of a system as a snapshot—a static and coherent view of its structure at a specific point in time.  
  It does not model runtime behavior, dynamic reconfiguration, failover, or autoscaling mechanisms.
- **Abstract and Realized Assets.**  
  The MACM model can be used to represent both concrete systems (e.g., for security analysis) and planned or hypothetical architectures (e.g., during system design).  
  Therefore, not all modeled assets must necessarily exist in the deployed system: they may also represent intended, planned, or provisioned components.
- **Interpretation of Constraints.**  
  Some constraints in the MACM specification (e.g., unique hosting of services) are valid under the snapshot assumption, but may not hold globally in systems with dynamic infrastructures (e.g., Kubernetes).  
  In such contexts, the model should be interpreted as one possible concrete deployment instance.
- **Finite and Closed World.**  
  The MACM assumes a closed and finite set of nodes and relationships.  
  External elements not modeled are considered unknown and have no implicit effect on validation.

These assumptions guide the formal structure of the model and the semantics of the constraints defined in later sections.

## Formal Definition

Let us consider an infinite set of labels $\mathbb{L}$ for edges and nodes, an infinite set of property names $\mathbb{P}$, an infinite set of atomic values $\mathbb{V}$, and a finite set of datatypes $Q$ (e.g., String, Integer).

### Property Graph

A Property Graph [@anglesPropertyGraphDatabase; @bellomariniModelIndependentDesignKnowledge2023] is defined as a tuple $ G = (N,E,\rho,\lambda,\sigma) $, where:

- $N$ is a finite set of nodes;
- $E$ is a finite set of edges (disjoint from $N$);
- $\rho: E \to (N \times N)$ is a function, known as the *incidence function*, which associates each edge with a pair of "source" and "target" nodes;
- $\lambda: (N \cup E)\to \mathrm{SET^+}(\mathbb{L})$[^1] is a function that assigns to each node/edge the set of labels describing them, taken from the infinite label set $\mathbb{L}$;
- $\sigma: (N \cup E) \times \mathbb{P} \to \mathrm{SET^+}(\mathbb{V})$ is a function that assigns to each node/edge a set of properties, taken from the infinite set $\mathbb{P}$, and to each $(node/edge, property)$ pair a set of values, taken from the infinite set of atomic values $\mathbb{V}$.

[^1]: Note that, given a set $\mathbb{K}$ we denote $\mathrm{SET}(\mathbb{K})$  the set of all finite subsets of $\mathbb{K}$, and $\mathrm{SET^+}(\mathbb{K})$ excludes the empty set.

### Property Graph Schema

Since the sets $\mathbb{L}, \mathbb{P}, \mathbb{V}$ are infinite, it is necessary to define which subsets are allowed for a given model. This leads to the concept of a **Property Graph Schema**, which defines the admissible types of nodes and edges, and the properties allowed for each of them.

A Property Graph Schema is defined as a tuple $S=(L_N, L_E, \beta, \delta)$, where:

- $L_N \subset \mathbb{L}$ is a finite set of labels, representing node types;
- $L_E \subset \mathbb{L}$ is a finite set of labels, representing edge types;
- $\beta: (L_N \cup L_E) \times \mathbb{P} \to Q $ is a function that defines the allowed properties for each node or edge type, and the corresponding datatype, taken from a finite set $Q$;
- $\delta: (L_N, L_N) \to \mathrm{SET^+}(L_E)$ is a function that defines the admissible edge types between two nodes of given types.

### MACM Property Graph Schema

Note that in the previous definition, no hierarchy is defined over the elements of the set $L_N$. This is unfortunately a limitation for the intended use of MACM, which led us to extend this definition by making it more rigid in certain aspects.

Since MACM is designed to model entire IT infrastructures, characterized by heterogeneous components and services, it is appropriate to classify the assets, and therefore the graph nodes, with multiple levels of granularity. We already introduced, at the beginning of this section, the *primary labels*, which define a high-level classification of nodes in the model, denoting the main roles or categories of assets in the system, and the *secondary labels*, which provide a finer semantic classification of nodes within the general role defined by the primary labels. Therefore, the label set $L_N$ is divided into two sets, $L_P$ and $L_S$. Thus, as it will be defined formally later, we developed an extended definition of a property graph as $M = (N,E,\rho,\lambda_N,\lambda_E,\sigma)$, where $\lambda_N$ concerns the node types, while $\lambda_E$ is related to edge types.

Furthermore, two new sets are introduced in the schema, containing the constraints to be satisfied for a MACM to be considered correct. In particular, we distinguish between two categories of constraints: syntactic constraints and semantic constraints. **Syntactic constraints** are mandatory and must be satisfied for the MACM model to be considered structurally valid. A violation of a syntactic constraint results in an invalid or unprocessable model. **Semantic constraints** represent optional validation rules that capture domain-specific aspects, best practices, or semantic consistency conditions. Although not strictly mandatory, violations of semantic constraints generate warnings and typically indicate potential design issues or misconfigurations.

Consequently, the definition of a MACM Property Graph Schema becomes the following.

A MACM Property Graph Schema is defined as a tuple $S_M=(L_P, L_S, R, \beta, \delta, \Sigma, \Gamma)$, where:

- $L_P \subset \mathbb{L}$ is a finite set of primary labels, which provide a high-level classification of the nodes in the model;
- $L_S \subset \mathbb{L}$ is a finite set of secondary labels, with a function $LS : L_P \to \mathrm{SET}(L_S)$ mapping each primary label to its corresponding subset of admissible secondary labels; the secondary labels provide a finer-grained semantic classification of the nodes within the general role;
- $R \subset \mathbb{L}$ is a finite set of labels representing edge types;
- $\beta: (L_P \times L_S \cup R) \times \mathbb{P} \to Q$ is a partial function that assigns to each node label pair $(PrimaryLabel, SecondaryLabel)$ or edge type the admissible properties and the corresponding datatype from a finite set $Q$;
- $\delta: (L_P, L_P) \to \mathrm{SET^+}(R)$ is a function defining the allowed edge types between two nodes with given primary labels;
- $\Sigma$ is the set of  *syntactic constraints*;
- $\Gamma$ is the set of *semantic constraints*.

### Syntactic Constraints

The *semantic constraints* may vary according to the application domain and the schema configuratiion, so they will be described in the following sections. The *syntactic constraints* are fixed and here listed:

- $P_m \subseteq \mathrm{dom}(\beta)$, the set of mandatory properties (must always be present for nodes/edges of the specified type);
- $P_o \subseteq \mathrm{dom}(\beta)$, the set of optional properties (may be omitted), with $P_o \cap P_m = \emptyset$ and $P_o \cup P_m = \mathrm{dom}(\beta)$;
- $P_u \subseteq P_m$, a subset of mandatory properties that must have unique values (e.g., *id*);
- $\mathit{Cardinality}: \mathrm{dom}(\beta) \to \mathbb{N} \times (\mathbb{N}\cup{+\infty})$, a function that assigns to each pair $(t,p)\in \mathrm{dom}(\beta)$ an interval $(m_p, M_p)$, where $m_p$ is the minimum and $M_p$ the maximum allowed cardinality for the set of values of property $p$ in objects of type $t$;
- Graph Connectivity, the MACM property graph $M = (N,E,\rho,\lambda_N,\lambda_E,\sigma)$ must be *weakly connected*;
let $M' = (N,E',\rho,\lambda_N,\lambda_E,\sigma)$ be the undirected version of $M$, then $\forall n_i,n_j\in N$, there exists a path in $M'$ connecting them;
- Acyclicity over Specified Relations
Let $R_{\mathrm{acyc}} \subseteq R$ be the subset of relation types that must not form cycles; let $M_{\mathrm{acyc}} = (N,E,\rho,\lambda_N,\lambda_{E,\mathrm{acyc}},\sigma)$, where $\forall e \in E, \lambda_{E,\mathrm{acyc}}(e) \in R_{\mathrm{acyc}}$;
then $M_{\mathrm{acyc}}$ must be a directed acyclic graph (DAG).

### Schema-Instance Consistency for MACM

A  (property) graph is a MACM only if it is consistent with the MACM Property Graph Schema, resulting both syntactically and semantically correct, according to the following definition.

Let $S_M = (L_P, L_S, R, \beta, \delta, \Sigma, \Gamma)$ be a MACM schema, and let $M = (N,E,\rho,\lambda_N,\lambda_E,\sigma)$ be a property graph, where:

- $\lambda_N : N \to L_P \times (L_S \cup {\emptyset})$ assigns to each node the pair $(PrimaryLabel, SecondaryLabel)$;
- $\lambda_E : E \to R$ assigns to each edge a single relation type.

We say that $M$ is *valid with respect to* $S_M$, and thus that \emph{$M$ is a MACM}, if all the following conditions hold:

1. **Label Validity**

     - $\forall n \in N \implies  \lambda_N(n) \in L_P \times (L_S \cup {\emptyset})$;
     - $ \forall e \in E \implies  \lambda_E(e) \in R$.

2. **Valid Relationship Patterns**

    $\forall e \in E: \rho(e) = (n_s,n_t) \implies \lambda_E(e) \in \delta(\lambda_N(n_s),\lambda_N(n_t))$.

3. **Admissible and Typed Properties**

    Assuming that for every $v \in \mathbb{V}$, the function $type(v)$ returns the datatype of $v$, then:

    - $\forall (n,p)=V_p \in \mathrm{dom}(\sigma),  \bigl(\lambda_N(n),p\bigr) \in \mathrm{dom}(\beta) \wedge \forall v \in V_p, type(v) = \beta\bigl(\lambda_N(n),p\bigr)$;

    - $\forall (e,p)=V_p \in \mathrm{dom}(\sigma),  \bigl(\lambda_E(e),p\bigr) \in \mathrm{dom}(\beta) \wedge \forall v \in V_p, type(v) = \beta\bigl(\lambda_E(e),p\bigr)$.

4. **Mandatory Properties**

    $ \forall(x,p)\in P_m \subseteq \mathrm{dom}(\beta) \land  \forall o \in N \cup E: \lambda(o)=x \implies (o,p)\in \mathrm{dom}(\sigma)$.

5. **Unique Properties**

    Let $P_u \subseteq P_m$ be the set of mandatory properties that must have unique values, then:

    - $\forall n_1,n_2 \in N: \bigl(n_1 \neq n_2 \wedge \lambda_N(n_1) = \lambda_N(n_2)\bigr) \implies \forall p \in P_u, \sigma(n_1,p) \neq \sigma(n_2,p)$
  
    - $\forall e_1,e_2 \in E:\; \bigl(e_1 \neq e_2 \wedge \lambda_E(e_1) = \lambda_E(e_2)\bigr) \implies \forall p \in P_u, \sigma(e_1,p) \neq \sigma(e_2,p)$

6. **Cardinality of Property Values**

     - $\forall n \in N, \forall \bigl(\lambda_N(n),p\bigr)\in \mathrm{dom}(\beta): (m_p,M_p) = \mathit{Cardinality}\bigl(\lambda_N(n),p\bigr) \implies m_p \leq |\sigma(n,p)| \leq M_p$;
     - $\forall e \in E, \forall \bigl(\lambda_E(e),p\bigr)\in \mathrm{dom}(\beta): (m_p,M_p) = \mathit{Cardinality}\bigl(\lambda_E(e),p\bigr) \implies m_p \leq |\sigma(e,p)| \leq M_p$.  

7. **Additional Constraints**

    All constraints in $\Sigma$ (e.g., connectivity, acyclicity) and in $\Gamma$ must be satisfied.
