---
title: Model Definition
---

# Model Definition

The MACM (Multi-purpose Application Composition Model) is a graph-based formalism that can be adopted to represent a System under Analysis (SuA). The MACM is a directed graph defined as a pair:

$$
\mathcal{M} = (N, E)
$$

where:

- $N$ is a finite set of nodes, representing the assets of the target system.
- $E \subseteq \mathbb{E}$ is a finite set of labeled directed edges, representing the relationships between the assets.

## Modeling Assumptions

The MACM model provides a formal and structured representation of a system under analysis (SuA), with the goal of supporting reasoning, validation, and security assessment.

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

## Useful sets

Before formally characterizing the nodes and arcs that make up the MACM, it is appropriate to introduce the sets that will be used in the following definitions.

### Primary Label Set
Let $L_P$ be the finite set of primary labels allowed in the model:

$$
L_P \subseteq \mathbb{L}_P, \quad \emptyset \notin L_P
$$

where $\mathbb{L}_P$ is the universal set of possible primary labels.

Each element of $L_P$ defines a high-level classification of a node in the MACM model, denoting the main roles or categories of assets in the system.

Every node $n \in N$ must be associated with exactly one $\ell_p \in L_P$; the empty label is not admissible.

### Secondary Label Set
Let $\mathbb{L}_S$ be the universal set of all admissible secondary labels.  
Each element of $L_S(\ell_p) \subseteq \mathbb{L}_S$ represents a domain-specific subcategory or specialization of the primary label $\ell_p$, providing a finer-grained semantic classification of the node within the broader role defined by $L_P$.

The function

$$
L_S: L_P \rightarrow 2^{\mathbb{L}_S}
$$

maps each primary label to its corresponding set of valid secondary labels. The set $L_S(\ell_p)$ may be empty, in which case nodes with $\ell_p$ may omit the secondary label or set $\ell_s = \emptyset$.

### Asset Type Set
Let $\mathbb{T}$ be the universal set of asset types in the MACM model.

Each asset type $t \in \mathbb{T}$ is uniquely associated with a primary label $\ell_p \in L_P$ and a secondary label $\ell_s \in \mathbb{L}_S \cup \{ \emptyset \}$.

We define a total mapping:

$$
\texttt{LabelPair} : \mathbb{T} \rightarrow L_P \times (\mathbb{L}_S \cup \{ \emptyset \})
$$

such that for each asset type $t \in \mathbb{T}$, we have:

$$
\texttt{LabelPair}(t) = (\ell_p, \ell_s)
$$

The primary label $\ell_p$ determines the high-level category of the asset, and always corresponds to the prefix of the asset type. The secondary label $\ell_s$ may be empty and refines the classification within the primary category.

### Relationship Type Set
Let $\mathbb{R}$ be the universal set of all admissible relationship types. Each element $r \in R \subseteq \mathbb{R}$ represents the type of a directed link between two nodes $n_s, n_t \in N$, where $n_s$ is the *source node* of the relationship, while $n_t$ is the *target node* of the relationship.

### Parameter Domain
Let $\mathbb{K}$ and $\mathbb{V}$ be the universal sets of, respectively, parameter keys and parameter values.

We define:

- $\mathcal{K}_n \subseteq \mathbb{K}$ as the set of allowed parameter keys for nodes.
- $\mathcal{V}_n \subseteq \mathbb{V}$ as the corresponding set of admissible values for node parameters.
- $\mathcal{K}_e \subseteq \mathbb{K}$ as the set of allowed parameter keys for relationships.
- $\mathcal{V}_e \subseteq \mathbb{V}$ as the corresponding set of admissible values for relationship parameters.

The parameter set associated with a node is defined as:

$$
P_n \subseteq \mathcal{K}_n \times 2^{\mathcal{V}_n}
$$

where each key is associated with a finite set of values.

The parameter set associated with a relationship is defined as:

$$
P_e \subseteq \mathcal{K}_e \times 2^{\mathcal{V}_e}
$$

where each key is associated with a finite set of values, enabling multi-valued attributes.

### Universal Relationship Set
Let $\mathbb{E}$ be the universal set of all syntactically well-formed relationships over the MACM model:

$$
\mathbb{E} = N \times R \times N \times P_e
$$

This set captures the entire space of relationships that are syntactically constructible from nodes in $N$, types in $R$ and parameters in $P_e$, without enforcing any semantic validity.

The actual set of relationships used in a MACM instance is denoted by:

$$
E \subseteq \mathbb{E}
$$

and is subject to the semantic constraints specified by the model.

## Node Definition

Each node $n \in N$ is a 5-tuple:

$$
n = (\texttt{id}, \ell_p, \ell_s, t, P_n)
$$

where:

- $\texttt{id} \in \mathbb{ID}$ is a unique identifier.
- $\ell_p \in L_P$ is the primary label (mandatory).
- $\ell_s \in L_S(\ell_p) \cup \{ \emptyset \}$ is the secondary label (optional).
- $t \in \mathbb{T}$ is the asset type (mandatory).
- $P_n \subseteq \mathcal{K}_n \times 2^{\mathcal{V}_n}$ is an optional set of parameter key-set pairs associated with the node.

### Syntactic Constraint: Unique node identifiers

$$
\forall n_i,n_j\in N,\;n_i\neq n_j\;\Rightarrow\;\texttt{id}(n_i)\neq\texttt{id}(n_j)
$$

This constraint ensures that all nodes within the set $N$ have unique identifiers. Specifically, for any two distinct nodes $n_i$ and $n_j$, their respective identifiers must differ, maintaining the uniqueness property.

## Relationship Definition

Each relationship $e \in E$ is a 4-tuple:

$$
e = (n_s, r, n_t, P_e)
$$

where:

- $r \in R$ is the relationship type.
- $n_s, n_t \in N$ are nodes such that $n_s \neq n_t$.
- $n_s$ is the **source node** of the relationship.
- $n_t$ is the **target node** of the relationship.
- $P_e \subseteq \mathcal{K}_e \times 2^{\mathcal{V}_e}$ is an optional set of key-value properties.

> **Remark:** Although formally $E \subseteq \mathbb{E} = N \times R \times N \times P_e$, not every possible quadruple in this Cartesian product represents a semantically valid relationship in the MACM model. The actual validity of a quadruple $(n_s, r, n_t, P_e) \in E$ is determined by the semantic constraints defined in the following section, such as constraints on node types, relationship directions, and context-specific compatibility rules.

## Useful functions

In this section we define some useful functions, which will be used later.

### Node Label Functions
Let $n = (\texttt{id}, \ell_p, \ell_s, t, P_n) \in N$ be a node in the MACM model.

We define the following projection functions:

$$
\texttt{PrimaryLabel} : N \rightarrow L_P,\qquad \texttt{PrimaryLabel}(n) = \ell_p
$$

$$
\texttt{SecondaryLabel} : N \rightarrow \mathbb{L}_S \cup \{ \emptyset \},\qquad \texttt{SecondaryLabel}(n) = \ell_s
$$

These functions allow explicit access to the label components of a node and are used in the semantic constraints and asset type construction.

### Asset Type of a Node
Let $n = (\texttt{id}, \ell_p, \ell_s, t, P_n) \in N$ be a node in the MACM model.

We define the function:

$$
\texttt{AssetType} : N \rightarrow \mathbb{T}
\quad \text{such that} \quad \texttt{AssetType}(n) = t
$$

The asset type $t \in \mathbb{T}$ assigned to the node is semantically associated with the labels $\ell_p$ and $\ell_s$ via the function $\texttt{LabelPair}$, as defined in the Asset Type Set.

Note that multiple asset types may share the same label pair $(\ell_p, \ell_s)$.

### Allowed Parameters per Asset Type
We define a total function:

$$
\texttt{AllowedParams}_n : \mathbb{T} \rightarrow 2^{\mathcal{K}_n}
$$

that maps each asset type $t \in \mathbb{T}$ to the subset of keys permitted for that type.

A node is semantically valid with respect to its parameters if:

$$
\forall (\texttt{id}, \ell_p, \ell_s, t, P_n) \in N,\quad \forall (k, V_k) \in P_n,\quad k \in \texttt{AllowedParams}_n(t)
$$

### Allowed Values per Node Parameter Key
Let $\mathcal{K}_n$ and $\mathcal{V}_n$ be the sets of admissible parameter keys and values for nodes, as defined in the Parameter Domain.

We define the total function:

$$
\texttt{AllowedValues}_n : \mathcal{K}_n \rightarrow 2^{\mathcal{V}_n}
$$

such that, for each key $k \in \mathcal{K}_n$, the value $\texttt{AllowedValues}_n(k)$ returns the set of all admissible values for that parameter key.

### Allowed Parameters per Relationship
We define a total function:

$$
\texttt{AllowedParams}_e : \mathbb{R} \rightarrow 2^{\mathcal{K}_e}
$$

that maps each relationship type $r \in \mathbb{R}$ to the subset of parameter keys permitted for that type.

A relationship is semantically valid with respect to its parameters if:

$$
\forall (n_s, r, n_t, P_e) \in E,\quad \forall (k, V_k) \in P_e,\quad k \in \texttt{AllowedParams}_e(r)
$$

### Allowed Values per Relationship Parameter Key
Let $\mathcal{K}_e$ and $\mathcal{V}_e$ be the sets of admissible parameter keys and values for relationships, as defined in the Parameter Domain.

We define the total function:

$$
\texttt{AllowedValues}_e : \mathcal{K}_e \rightarrow 2^{\mathcal{V}_e}
$$

such that, for each key $k \in \mathcal{K}_e$, the value $\texttt{AllowedValues}_e(k)$ returns the set of all admissible values for that parameter key.

### Value Cardinality per Parameter Key
We define a total function:

$$
\texttt{ValueCardinality} : (\mathcal{K}_n \cup \mathcal{K}_e) \rightarrow \mathbb{N}^+ \times \mathbb{N}^+
$$

that maps each admissible parameter key $k$ to a pair $(m_k, M_k)$, where:

- $m_k \in \mathbb{N}^+$ is the minimum allowed cardinality for the associated value set.
- $M_k \in \mathbb{N}^+$ is the maximum allowed cardinality for the associated value set.

## Constraints

The MACM model is subject to a set of formal constraints that govern its well-formedness and semantic consistency.  
We distinguish between two categories of constraints:

- **Syntactic Constraints** — These are mandatory and must be satisfied for the MACM model to be considered structurally valid. Violating a syntactic constraint results in an invalid or unprocessable model.
- **Semantic Constraints** — These represent optional validation rules that capture domain-specific expectations, best practices, or semantic consistency conditions. While not strictly mandatory, violations of semantic constraints generate warnings and typically indicate possible design issues or misconfigurations.

Unless otherwise specified, semantic constraints are treated as advisory and should be surfaced to users during model validation to support decision-making and troubleshooting.

### Syntactic Constraint: Graph Connectivity

The MACM model must be weakly connected.

Let $G = (N, E')$ be the undirected version of the relationship graph, where:

$$
E' = \{ \{n_s, n_t\} \mid (n_s, r, n_t) \in E \}
$$

Then, for every pair of nodes $n_i, n_j \in N$, there exists an undirected path in $G$ connecting them:

$$
\forall n_i, n_j \in N, \quad \exists \text{ a path in } G \text{ from } n_i \text{ to } n_j
$$

> **Note:** Attention. Not sure about the validity considering a subset of the edges, also we know the graph is cyclic with the use relation.

### Syntactic Constraint: Acyclicity over Specified Relations

Let $R_{\mathrm{acyclic}} \subseteq R$ be the subset of relationship types that must not form cycles.

Let $G_{\mathrm{acyc}} = (N, E_{\mathrm{acyc}})$ be the directed subgraph, where:

$$
E_{\mathrm{acyc}} = \{ (n_s, n_t, P_e) \mid (n_s, r, n_t, P_e) \in E,\; r \in R_{\mathrm{acyclic}} \}
$$

Then, the subgraph $G_{\mathrm{acyc}}$ must not contain cycles. That is:

$$
G_{\mathrm{acyc}} \text{ is a Directed Acyclic Graph (DAG)}
$$

### Semantic Constraint: Valid Node Parameter Values

A parameter assignment is semantically valid if:

$$
\forall (k, V_k) \in P_n, \quad V_k \subseteq \texttt{AllowedValues}_n(k)
$$

### Semantic Constraint: Valid Relationship Parameter Values

A parameter assignment is semantically valid if:

$$
\forall (k, V_k) \in P_e, \quad V_k \subseteq \texttt{AllowedValues}_e(k)
$$

### Semantic Constraint: Parameter Value Set Cardinality Constraint

A parameter assignment is semantically valid if:

$$
\forall (k, V_k) \in P_n \cup P_e,\quad m_k \leq |V_k| \leq M_k
\quad \text{where} \quad (m_k, M_k) = \texttt{ValueCardinality}(k)
$$