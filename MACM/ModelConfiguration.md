---
title: Model Configuration
---

# Model Configuration

In this section, we define the concrete instantiation of the sets and functions introduced in the formal definition of the MACM model. While previous sections described the abstract structure of MACM as a generic and parametric formalism, here we specify the exact values adopted in a particular domain-specific configuration.

This configuration establishes the permitted primary and secondary labels, the derived asset types, the allowed relationship types, and the structure of parameter keys and values. These elements form the semantic foundation of any MACM-compliant instance and are required for the correct validation and interpretation of models.

Unless otherwise stated, all constraints, rules, and type-checking functions described in the previous sections are assumed to operate on the configuration defined here.

## Nodes Configuration

### Primary Labels

We define the set of admissible primary labels:

$$
L_P = \\{ \texttt{CSC}, \texttt{CSP}, \texttt{Service}, \texttt{HW}, \texttt{Network}, \texttt{Data} \\}
$$

### Secondary Labels

For each $\ell_p \in L_P$, we define the associated secondary label set $L_S(\ell_p) \subseteq \mathbb{L}_S$. For instance:

$$
\begin{aligned}
    & L_S(\texttt{Network}) = \\{ \texttt{5G}, \texttt{WAN}, \texttt{LAN}, \texttt{PAN} \\} \cr
    & L_S(\texttt{Service}) = \\{ \texttt{5G}, \texttt{COTS}, \texttt{IaaS}, \texttt{PaaS}, \texttt{SaaS} \\} \cr
    & L_S(\texttt{HW}) = \\{ \texttt{Server}, \texttt{IoT}, \texttt{Device}, \texttt{Microcontroller}, \texttt{SOC} \\} \cr
    & L_S(\texttt{CSC}) = \emptyset \cr
    & L_S(\texttt{CSP}) = \emptyset \cr
    & L_S(\texttt{Data}) = \emptyset
\end{aligned}
$$

### Asset Types

This section presents the set of asset types supported by the default MACM configuration.

Each asset type $t \in \mathbb{T}$ represents a concrete classification of a node within the system and is associated with a primary label $\ell_p \in L_P$ and a secondary label $\ell_s \in \mathbb{L}_S \cup \\{ \emptyset \\}$. While the asset type is not necessarily a direct composition of its labels, it is uniquely related to a pair $(\ell_p, \ell_s)$ through the function $\texttt{LabelPair}$, as defined in the formal specification.

The first component of every asset type corresponds to its primary label and determines the high-level role of the asset in the model. The secondary label, if present, provides a more specific categorization within that role.

The following table enumerates all asset types defined in the default MACM configuration, along with their corresponding label pair and a brief description.

<!-- Table omitted for brevity, see original for full list -->

### Node Parameter Domain

In the default configuration of the MACM model, a single parameter is defined for all nodes: `compromise_state`. This parameter encodes the compromise status of a node along six independent security-relevant dimensions.

#### Parameter Key Set

The set of admissible parameter keys for nodes is defined as:

$$
\mathcal{K}_n = \\{\texttt{compromise\_state}\\}
$$

#### Admissible Values

Let $\mathcal{CS} = \\{n, p, f, ?\\}$ denote the compromise state alphabet, where:

- **n** = not compromised
- **p** = partially compromised
- **f** = fully compromised
- **?** = unknown

Each node must specify a 6-tuple $\mathbf{c} = (c_1, c_2, c_3, c_4, c_5, c_6) \in \mathcal{CS}^6$, where each $c_i \in \mathcal{CS}$. These six elements correspond to the six dimensions of the extended CIA+ taxonomy adopted by MACM:

- $c_1$: **Confidentiality** — unauthorized disclosure of information.
- $c_2$: **Integrity** — unauthorized modification or corruption of data or functions.
- $c_3$: **Availability** — denial or degradation of service or resources.
- $c_4$: **Authentication** — compromise of identity verification mechanisms.
- $c_5$: **Authorization** — abuse or bypass of access control policies.
- $c_6$: **Accountability** — loss of traceability or auditability guarantees.

The corresponding value domain function is:

$$
\texttt{AllowedValues}_n(\texttt{compromise\_state}) = \mathcal{CS}^6
$$

#### Cardinality Constraint

Nodes must declare exactly one such tuple:

$$
\texttt{ValueCardinality}(\texttt{compromise\_state}) = (1, 1)
$$

#### Parameter Applicability

The parameter `compromise_state` is allowed for all asset types:

$$
\forall t \in T, \quad \texttt{AllowedParams}_n(t) = \\{\texttt{compromise\_state}\\}
$$

This parameter provides a uniform mechanism to annotate nodes with structured compromise information across multiple security properties, enabling advanced reasoning tasks such as impact analysis, recursive threat propagation, and multi-dimensional risk modeling.

#### Example

Consider a node $n \in N$ representing a server, whose compromise state is specified as:

$$
\texttt{compromise\_state}(n) = (n, p, n, f, n, ?)
$$

While the notation $\texttt{compromise\_state}(n)$ is used here for clarity, it is formally an abbreviation. In the MACM model, parameter values are stored as key-value pairs in the node's parameter set $P_n$, i.e.,

$$
(\texttt{compromise\_state}, \\{(n, p, n, f, n, ?)\\}) \in P_n.
$$

This corresponds to the following security status:

- **Confidentiality**: not compromised
- **Integrity**: partially compromised
- **Availability**: not compromised
- **Authentication**: fully compromised
- **Authorization**: not compromised
- **Accountability**: unknown

Such a configuration reflects a scenario where the node’s authentication mechanisms have been fully bypassed (e.g., via credential theft), and its integrity is at risk due to unauthorized modifications. However, availability remains intact, and no clear evidence exists yet about accountability failures.

## Relationship Configuration

### Relationship Types

We define the set of admissible relationship types:

$$
R = \\{ \texttt{uses}, \texttt{hosts}, \texttt{provides}, \texttt{connects} \\}
$$

### Relationship Parameter Domain

We define all the parameters eligible for relationships:

$$
\mathcal{K}_e = \\{ \texttt{data\_link\_protocol}, \texttt{network\_protocol}, \texttt{transport\_protocol},
                   \texttt{session\_protocol}, \texttt{presentation\_protocol}, \texttt{application\_protocol} \\}
$$

### Supported Protocols per Relationship Type

In this section, we define the set of protocols that are admissible for each relationship type in the MACM default configuration.

#### OSI Layer Set

Let $\mathcal{L}$ be the set of ISO/OSI layers. Each layer $\lambda \in \mathcal{L}$ represents a conceptual level in the standardized communication stack, from physical signal transmission to high-level application semantics.

$$
\mathcal{L} = \\{\texttt{Physical},\;\texttt{Data Link},\;\texttt{Network},\;\texttt{Transport},\;\texttt{Session},\;\texttt{Presentation},\;\texttt{Application}\\}
$$

These layers provide the foundation for classifying supported protocols within the model, and are referenced in the definition of the `ProtocolLayerMapping` relation.

Each layer $\lambda \in \mathcal{L}$ is semantically associated with a unique parameter key $k \in \mathcal{K}_e$. This mapping enables each protocol $p \in \mathcal{P}$ to be positioned within the OSI model and attached to relationships via its corresponding parameter key, as used in the MACM model semantics.

#### Protocol Domain

Let $\mathcal{P}$ denote the set of all protocols supported by the MACM model. Each protocol $p \in \mathcal{P}$ corresponds to a standardized communication mechanism.

#### Protocol-to-OSI Layer Mapping

We define the relation:

$$
\texttt{ProtocolLayerMapping} \subseteq \mathcal{P} \times \mathcal{L}
$$

where $\mathcal{P}$ is the set of supported protocols (as defined in the Protocol Domain), and $\mathcal{L}$ is the set of ISO/OSI layers.

Each pair $(p, \lambda) \in \texttt{ProtocolLayerMapping}$ indicates that protocol $p \in \mathcal{P}$ operates at ISO/OSI layer $\lambda \in \mathcal{L}$.

#### Protocol-to-Relation Mapping

We define the relation:

$$
\texttt{ProtocolRelationMapping} \subseteq \mathcal{P} \times R
$$

as the set of all valid protocol-to-relationship associations permitted in the MACM model.

Each pair $(p, r) \in \texttt{ProtocolRelationMapping}$ denotes that protocol $p \in \mathcal{P}$ is semantically allowed to appear as the value of a protocol-related parameter on a relationship of type $r \in R$.

#### Protocol Semantics Triples

We define the relation:

$$
\texttt{ProtocolSemantics} \subseteq \mathcal{P} \times \mathcal{L} \times R
$$

Each triple $(p, \lambda, r) \in \texttt{ProtocolSemantics}$ expresses that:

- protocol $p \in \mathcal{P}$ operates at OSI layer $\lambda \in \mathcal{L}$
- protocol $p$ is valid as a value for the corresponding parameter key associated with $\lambda$
- protocol $p$ is allowed in the context of relationship type $r \in R$

This relation provides a unified semantics for protocol parameters in the MACM model and is defined extensionally by the supported protocols table.

#### Protocol Dependencies

Let

$$
\texttt{ProtocolDependencies} \subseteq \mathcal{P} \times \mathcal{K}_e \times \mathcal{P}
$$

be the set of semantic dependencies between protocols defined in the MACM model.

Each element $(p_h, k_l, p_l) \in \texttt{ProtocolDependencies}$ expresses the following constraint:

If a relationship includes a parameter key $k_h \in \mathcal{K}_e$ whose associated value set contains the protocol $p_h \in \mathcal{P}$, then it must also include a parameter key $k_l \in \mathcal{K}_e$ whose associated value set contains the protocol $p_l \in \mathcal{P}$, where $k_l$ typically corresponds to a lower OSI layer.

This dependency ensures semantic coherence across protocol stacks within a relationship by enforcing mandatory protocol compositions.

<!-- Table omitted for brevity, see original for full list -->

## Constraints

The following constraints refine and specialize the general validation rules defined in the formal MACM specification.  
They apply specifically to the default configuration presented in this section.

As in the core model, we distinguish between:

- **Syntactic Constraints** — Configuration-specific structural requirements that must be enforced for model validity.
- **Semantic Constraints** — Best-practice conditions and consistency checks derived from domain knowledge and expected modeling behavior.

Syntactic constraints in this section restrict the accepted patterns, label combinations, or graph structures to those valid under the default configuration.  
Semantic constraints, while optional, highlight potential misconfigurations, such as unsupported protocol stacks or possible protocol dependencies.

### Syntactic Constraint: Relationship pattern validity

Let

$$
\begin{aligned}
    \texttt{AllowedPattern} = \\{ 
        & (\texttt{CSC}, \texttt{uses}, \texttt{Service}), \cr
        & (\texttt{Service}, \texttt{uses}, \texttt{Service}), \cr
        & (\texttt{CSP}, \texttt{provides}, \texttt{Service}), \cr
        & (\texttt{CSP}, \texttt{provides}, \texttt{Network}), \cr
        & (\texttt{CSP}, \texttt{provides}, \texttt{HW}), \cr
        & (\texttt{Service}, \texttt{hosts}, \texttt{Service}), \cr
        & (\texttt{HW}, \texttt{hosts}, \texttt{HW}), \cr
        & (\texttt{HW}, \texttt{hosts}, \texttt{Service}), \cr
        & (\texttt{Network}, \texttt{connects}, \texttt{IaaS}), \cr
        & (\texttt{Network}, \texttt{connects}, \texttt{HW})
    \\}
\end{aligned}
$$

Then:

$$
\forall (n_s, r, n_t, P_e) \in E, \quad (\texttt{PrimaryLabel}(n_s), r, \texttt{PrimaryLabel}(n_t)) \in \texttt{AllowedPattern}.
$$

### Syntactic Constraint: Single host/provide per service

Each service node must be hosted or provided by exactly one other node.  
This guarantees that every service has a unique deployment context in the system.

$$
\forall n \in N \text{ with } \texttt{PrimaryLabel}(n)=\texttt{Service}, \quad 
\bigl|\\{(n',r,n,P_e) \in E \mid r \in \\{\texttt{hosts},\texttt{provides}\\} \\}\bigr| = 1
$$

> **Remark:** This relies on the snapshot assumption of the MACM model. In design-time scenarios or highly dynamic infrastructures, this condition applies to the specific instance of the architecture captured by the model, and not to the system's full lifecycle.

### Syntactic Constraint: Alternate path for uses

A `uses` relationship between two nodes must be supported by an alternative communication path that does not itself depend on a `uses` edge.

This ensures that functional dependencies (captured by `uses`) are semantically meaningful only when there exists a viable infrastructure or network path that enables actual communication or interaction.

Formally, let $G' = (N, E')$ be the subgraph of the MACM obtained by removing all `uses` relationships:

$$
E' = \\{ (n_s, n_t) \mid (n_s, r, n_t, P_e) \in E,\; r \neq \texttt{uses} \\}
$$

Then:

$$
\forall (n_1, \texttt{uses}, n_2, P_e) \in E,\quad \exists \text{ a path in } G' \text{ from } n_1 \text{ to } n_2
$$

### Syntactic Constraint: CSC only uses

$$
\forall(n_s,r,n_t,P_e)\in E,\;\texttt{PrimaryLabel}(n_s)=\texttt{CSC}\;\Rightarrow\;r=\texttt{uses}.
$$

### Syntactic Constraint: CSP nodes may only appear as source in provides relationships

Nodes labeled `CSP` are only allowed to initiate `provides` relationships.

$$
\forall (n_s, r, n_t, P_e) \in E, \quad \texttt{PrimaryLabel}(n_s) = \texttt{CSP} \Rightarrow r = \texttt{provides}
$$

### Syntactic Constraint: Services must be hosted by Firmware or OS

Every service node that is not itself of type `Firmware` or `OS` must be hosted by another service node whose type is either `Firmware` or `OS`.

Formally:

$$
\forall (n_h, \texttt{hosts}, n_s, P_e) \in E,
\texttt{PrimaryLabel}(n_s) = \texttt{Service} \wedge
\texttt{SecondaryLabel}(n_s) \notin \\{ \texttt{Firmware}, \texttt{OS} \\}
\Rightarrow
\texttt{AssetType}(n_h) \in \\{ (\texttt{Service}, \texttt{Firmware}),\; (\texttt{Service}, \texttt{OS}) \\}
$$

### Syntactic Constraint: HW nodes can only host Firmware or OS services

If a node with asset type in category `HW` hosts a `Service`, that service must be of type `Firmware` or `OS`.

Formally:

$$
\forall (n_s, \texttt{hosts}, n_t, P_e) \in E, \texttt{PrimaryLabel}(n_s) = \texttt{HW} \wedge \texttt{PrimaryLabel}(n_t) = \texttt{Service} \Rightarrow \texttt{AssetType}(n_t) \in \\{ \texttt{Service.Firmware}, \texttt{Service.OS} \\}
$$

### Semantic Constraint: Default Value Cardinality for Protocol Parameters

In the default MACM configuration, each protocol parameter key is assigned an admissible range of values through the function:

$$
\texttt{ValueCardinality}(k) =
\begin{cases}
(0,\;1) & \text{if } k \in \\{\texttt{data\_link\_protocol},\;\texttt{network\_protocol},\;\texttt{transport\_protocol}, \cr
    \texttt{session\_protocol},\;\texttt{presentation\_protocol}\\} \cr
(0,\;+\infty) & \text{if } k = \texttt{application\_protocol}
\end{cases}
$$

This constraint specifies that each protocol key may be omitted, but if present, its associated value set must respect the defined bounds. All lower-layer protocol keys admit at most one value; the application layer admits one or more values if used.

### Semantic Constraint: Protocol dependency enforcement

For each declared dependency $(p_h, k_l, p_l) \in \texttt{ProtocolDependencies}$, if a relationship declares $p_h$ within the value set associated with some key $k_h$, then it must also include $p_l$ within the value set associated with $k_l$.

Formally:

$$
\forall (n_s, r, n_t, P_e) \in E,\quad
\forall (p_h, k_l, p_l) \in \texttt{ProtocolDependencies},
$$
$$
(\exists V_h\; :\; (k_h, V_h) \in P_e\ \wedge\ p_h \in V_h)\quad 
\Rightarrow\quad (\exists V_l\; :\; (k_l, V_l) \in P_e\ \wedge\ p_l \in V_l)
$$