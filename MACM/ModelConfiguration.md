---
title: Model Configuration
---

# Schema Configuration

So far, we have provided the definition of a MACM model as a Property Graph and formalized its schema. At this point, the remaining step is to configure the schema by explicitly specifying all the sets defined above.

## Primary Labels

We define the set of admissible primary labels $L_P = \\{ \mathtt{Party}, \mathtt{CSP}, \mathtt{Service}, \mathtt{HW}, \mathtt{Network}, \mathtt{Virtual}, \mathtt{SystemLayer} \\}$.

## Secondary Labels

For each $\ell_p \in L_P$, the associated secondary label set $LS(\ell_p) \subseteq L_S$ are:

- $ LS(\mathtt{Network}) = \\{ \mathtt{WAN}, \mathtt{LAN}, \mathtt{PAN} \\}$
- $LS(\mathtt{CSP}) = \emptyset$
- $LS(\mathtt{Virtual}) = \\{ \mathtt{VM}, \mathtt{Container} \\}$
- $LS(\mathtt{SystemLayer}) = \\{\mathtt{OS},\mathtt{Firmware},\mathtt{ContainerRuntime}, \mathtt{Hypervisor} \\}$
- $LS(\mathtt{Data}) = \emptyset$
- $LS(\mathtt{Service}) = \\{ \mathtt{App}, \mathtt{Server} \\}$
- $LS(\mathtt{Party}) = \\{ \mathtt{Human}, \mathtt{LegalEntity}, \mathtt{Group} \\}$
- $LS(\mathtt{HW}) = \\{ \mathtt{Server}, \mathtt{IoT}, \mathtt{Device}, \mathtt{Microcontroller}, \mathtt{SOC}, \mathtt{MEC}, \mathtt{PC} \\}$

## Relationship Types

The set of admissible relationship types is $R = \\{ \mathtt{uses, hosts, provides, connects, interacts} \\}$.

## Relationship pattern validity

The primary labels of the source and target nodes, together with the relationship type, must correspond to one of the patterns defined in the following table. Each row of the table has to be intended as $\delta(Source Primary Label,Target Primary Label) = Relationship Type$.

| Source Primary Label | Relationship Type | Target Primary Label                                                                                           |
|-----------------------------|----------------------------|-------------------------------------------------------------------------------------------------------------------------|
| $\mathtt{Party}$            | $\mathtt{interacts}$       | $\mathtt{Service}, \mathtt{HW}, \mathtt{Network}, \mathtt{Party}, \mathtt{Virtual}, \mathtt{SystemLayer}, \mathtt{CSP}$ |
| $\mathtt{Service}$          | $\mathtt{uses}$            | $\mathtt{Service}, \mathtt{Virtual}$                                                                                    |
| $\mathtt{Service}$          | $\mathtt{hosts}$           | $\mathtt{Service}$                                                                                                      |
| $\mathtt{Virtual}$          | $\mathtt{hosts}$           | $\mathtt{SystemLayer}$                                                                                                  |
| $\mathtt{SystemLayer}$      | $\mathtt{hosts}$           | $\mathtt{SystemLayer}, \mathtt{Virtual}, \mathtt{Service}$                                                              |
| $\mathtt{SystemLayer}$      | $\mathtt{uses}$            | $\mathtt{HW}$                                                                                                           |
| $\mathtt{HW}$               | $\mathtt{hosts}$           | $\mathtt{HW}, \mathtt{SystemLayer}$                                                                                     |
| $\mathtt{CSP}$              | $\mathtt{provides}$        | $\mathtt{Service}, \mathtt{Network}, \mathtt{HW}, \mathtt{Virtual}, \mathtt{SystemLayer}$                               |
| $\mathtt{Network}$          | $\mathtt{connects}$        | $\mathtt{Network}, \mathtt{Virtual}, \mathtt{SystemLayer}, \mathtt{HW}, \mathtt{CSP}$                                   |

## Properties

The Property configuration within the schema is defined through the function $\beta$, as introduced in the *Macm Property Graph Schema*, that assigns to each pair $(PrimaryLabel,~SecondaryLabel)$ or to each edge type the admissible properties and their corresponding datatype.

According to the definition, we would need to list a potentially long set of entries of the form $\beta\bigl((x,y),p\bigr)=q$, where $x$ is the primary label, $y$ is the secondary label, $p$ is the property, and $q$ is the datatype. Since these properties apply to all nodes regardless of type, we will use the wildcard pair $(\cdot,\cdot)$ to denote all node types and thus simplify the configuration.

As previously stated in the introduction of this section, there are three mandatory properties for nodes: the ID, the name, and the Asset Type. In particular, the Asset Type represents an additional, fine-grained classification of an asset. An Asset Type is tightly coupled with the concepts of Primary and Secondary Label. Indeed, given an Asset Type, the pair $(PrimaryLabel, SecondaryLabel)$ is uniquely determined. The list of all Asset Types defined in this configuration, along with their corresponding label pair and a brief description, is available in the [Asset Type Catalogue](https://pennet.vseclab.it/catalogs/asset-types).
We now introduce the following definition, useful for expressing some constraints.

### Asset Type Set
    
Let $T$ be the set of Asset Types in the MACM model. Each asset type $t \in T$ is uniquely associated with a primary label $\ell_p \in L_P$ and a secondary label $\ell_s \in L_S \cup \\{ \emptyset \\}$.
We define a total mapping $\tau : T \rightarrow L_P \times (L_S \cup \\{ \emptyset \\})$ such that for each asset type $t \in T$, we have $\tau(t) = (\ell_p, \ell_s)$.

Therefore, the following configurations apply to the nodes:

- $\beta\bigl((\cdot,\cdot),id\bigr) = \mathtt{Integer}$

- $\beta\bigl((\cdot,\cdot),name\bigr) = \beta\bigl((\cdot,\cdot),asset\\_type\bigr)=\mathtt{String}$

In terms of the edge properties configuration, properties are defined for the protocols used in some of the relationships between the assets, whenever communications occur over a network. We have chosen to adopt all the layers of the standard ISO/OSI communication stack, excluding the physical layer.

Each layer is associated with a property:

$$
    p \in O = \\{ \mathtt{data\\_link\\_protocol}, \mathtt{network\\_protocol}, \mathtt{transport\\_protocol}, \mathtt{session\\_protocol}, \mathtt{presentation\\_protocol}, \mathtt{application\\_protocol} \\}
$$

The following configuration applies:

- $\beta(\text{connects}, p) = \mathtt{String},\quad \forall p \in \\{ \mathtt{data\\_link\\_protocol}, \mathtt{network\\_protocol}, \mathtt{application\\_protocol} \\}$

- $\beta(\text{uses}, p) = \mathtt{String},\quad \forall p \in \\{ \mathtt{transport\\_protocol}, \mathtt{session\\_protocol}, \mathtt{presentation\\_protocol}, \mathtt{application\\_protocol} \\}$

At this point, it is necessary to determine which of the above configurations $\beta(\cdot,\cdot)$ belong to the special sets $P_m$ (mandatory properties), $P_o$ (optional properties), or $P_u$ (properties with unique values):

- $((\cdot, \cdot), id) \in P_u$
- $\forall x \in \\{ ((\cdot, \cdot), name), ((\cdot, \cdot), asset\\_type) \\}, x \in P_m$
- $\forall x \in \left\\{
    \\begin{array}{l}
    (connects, data\\_link\\_protocol),
    (connects, network\\_protocol), \\\\
    (connects, application\\_protocol),
    (uses, transport\\_protocol), \\\\
    (uses, session\\_protocol),
    (uses, presentation\\_protocol), \\\\
    (uses, application\\_protocol)
    \\end{array}
    \right\\}, x \in P_o$

Since $P_u \subseteq P_m$, the property $id$ is both unique and mandatory. The cardinality of each property is defined by the function $Cardinality(\cdot,p)$ and reported in the following table.

| Property                                     | Cardinality |
|-----------------------------------------------------|----------------------|
| $((\cdot,\cdot),id)$, $((\cdot,\cdot),name)$, $((\cdot,\cdot),asset\\_type)$ | $(1,1)$              |
| $(uses, presentation\\_protocol)$                    | $(0,1)$              |
| $(uses, application\\_protocol)$                     | $(0,+\infty)$        |

According to this configuration, each property related to a protocol may be omitted; however, if present, its associated value set must comply with the defined limits. All protocol properties allow at most one value, except for the application-level property, which allows one or more values if used.

> **Disambiguation**: In this section, we have defined the Property Graph as a graph in which nodes are characterized by labels and properties. We also stated that the labels $l \in L$ define the “type” of a node. In the case of MACM, we have defined a hierarchy consisting of two types of labels: Primary Labels and Secondary Labels. The former provide a higher-level classification, while the latter serve to further specialize this classification. For example, to represent a LAN network, we use a node whose Primary Label is $\mathtt{Network}$ and whose Secondary Label is $\mathtt{LAN}$. However, in MACM we have also defined Asset Types, which represent the most fine-grained classification assigned to a node. In the case of a LAN network, the Asset Type would be $\mathtt{Network.LAN}$. It should first be noted that an Asset Type is not always composed of $\mathtt{PrimaryLabel.SecondaryLabel}$. Furthermore, Asset Types are not labels but properties, and therefore it would be inaccurate to say that a node representing a LAN network is of “type” $\mathtt{Network.LAN}$.
Nevertheless, throughout this document, the “type” of a node will often be referred to by indicating either its labels or its Asset Type. The reader can easily distinguish between the two classifications—label-level or Asset Type—by noting whether the indicated “type” contains a dot or not.

## Constraints

The last element to be configured in the schema is the set of semantic constraints $\Gamma$, which, as previously explained, are used to express semantic characteristics specific to the application domain—in this case, IT systems.

### Asset Type validity

Previously, we defined the set of Asset Types $T$ and stated that each Asset Type $t \in T$ is associated with a pair $(PrimaryLabel,SecondaryLabel)$. We now define a constraint ensuring that this relationship holds for every Asset Type of each node in the MACM:

$$
    \forall n \in N, \sigma(n,\mathtt{asset\\_type}) \in T \land  \tau \bigl(\sigma(n,\mathtt{asset\\_type})\bigr)=\lambda_N(n) 
$$

### Single host/provide per service

The second constraint requires that each service node must be hosted or provided by exactly one other node. This guarantees that every service has a unique deployment context in the system.

$$
    \forall n_t \in N: \lambda_N(n_t)=(\mathtt{Service},\cdot), \exists! n_s \in N, \exists e \in E:\lambda_E(e) \in \\{ \mathtt{hosts},\mathtt{provides} \\} \land \rho(e) = (n_s,n_t)
$$

This constraint relies on the snapshot assumption of the MACM model (see Model Definition). In design-time scenarios or highly dynamic infrastructures, this condition applies to the specific instance of the architecture captured by the model, and not to the system's full lifecycle.

### Alternate path for uses

The following semantic sonstraint requires that each  $\mathtt{uses}$ relationship between two nodes must be supported by an alternative communication path that does not itself depend on a $\mathtt{uses}$ or an $\mathtt{interacts}$ edge. This ensures that functional dependencies are semantically meaningful only when there exists a viable infrastructure or network path that enables actual communication.

Let $M = (N,E,\rho,\lambda_N,\lambda_E,\sigma)$ a MACM property graph and $M' = (N,E',\rho,\lambda_N,\lambda_E,\sigma)$ the subgraph of $M$ with $E' = \bigl\\{ e \in E \mid \lambda_E(e) \notin \\{ \mathtt{uses}, \mathtt{interacts} \\} \bigr\\}$, then $\forall e \in E$, where $\lambda_E(e)=\mathtt{uses}$ and $\rho(e)=(n_s,n_t)$, exists a path in $M'$ from $n_s$ to $n_t$.

### SystemLayer hosting SystemLayer node validity

We introduce constraints for *SystemLayer* nodes, representing middleware. This primary label includes four asset types: $\mathtt{SystemLayer.OS}$ (e.g., Operating System), $\mathtt{SystemLayer.Firmware}$, $\mathtt{SystemLayer.ContainerRuntime}$ (e.g., Docker), and $\mathtt{SystemLayer.HyperVisor}$ (e.g., Xen, VMware).
First, we constrain the $\delta(\mathtt{SystemLayer, SystemLayer})=\mathtt{hosts}$ pattern from Relationship Pattern Validity. A $\mathtt{SystemLayer.OS}$ may host a $\mathtt{SystemLayer.ContainerRuntime}$ or a $\mathtt{SystemLayer.HyperVisor}$, but other direct hosting between *SystemLayer* nodes is invalid. For instance, a *HyperVisor* cannot host an *OS* without an intermediate *Virtual* node.
The following rule defines that a
$\mathtt{SystemLayer.OS}$ can host only $\mathtt{SystemLayer.ContainerRuntime}$ or $\mathtt{SystemLayer.HyperVisor}$ nodes.

$$
    \forall e \in E,  \big( \lambda_E(e) = \mathtt{"hosts"} \land \rho(e) = (n_s, n_t) \land \lambda_N(n_s) = \lambda_N(n_t) = \mathtt{(SystemLayer, \cdot)} \big) \implies \big( \sigma(n_s, \mathtt{asset\\_type}) = \mathtt{"SystemLayer.OS"} \land
    \sigma(n_t, \mathtt{asset\\_type}) \in \\{ \mathtt{"SystemLayer.ContainerRuntime"}, \mathtt{"SystemLayer.HyperVisor"} \\} \big)
$$

### SystemLayer hosting Virtual node validity

The second relationship pattern we wish to constrain is $\delta(\mathtt{SystemLayer,Virtual})=\mathtt{hosts}$. In this case, since the Primary Label $\mathtt{Virtual}$ is used to represent either virtual machines (of type $\mathtt{Virtual.VM}$) or containers (of type $\mathtt{Virtual.Container}$), only the following relationships are valid: $\mathtt{SystemLayer.ContainerRuntime}$ $\mathtt{hosts}$ $\mathtt{Virtual.Container}$ and $\mathtt{SystemLayer.HyperVisor}$ $\mathtt{hosts}$ $\mathtt{Virtual.VM}$.

$$
    \forall e \in E, \big( \lambda_E(e) = \mathtt{"hosts"} \land \rho(e) = (n_s, n_t) \land \lambda_N(n_s) = (SystemLayer, \cdot) \land \lambda_N(n_t) = (Virtual, \cdot) \implies \big( \sigma(n_s, \mathtt{asset\\_type}) \in \\{ \mathtt{"SystemLayer.ContainerRuntime"}, \mathtt{"SystemLayer.HyperVisor"} \\} \big), \\\\
    \sigma(n_t,asset\\_type) =
    \\begin{cases}
        \\begin{alignedat}{2}
            &\mathtt{"Virtual.Container"} && \quad \text{if } \sigma(n_s,asset\\_type) = \mathtt{"SystemLayer.ContainerRuntime"}   \\\\
            &\mathtt{"Virtual.VM"} && \quad \text{if } \sigma(n_s,asset\\_type) = \mathtt{"SystemLayer.HyperVisor"}
        \\end{alignedat}
    \\end{cases}
$$

### SystemLayer hosting Service node validity

Regarding SystemLayer nodes that host Service nodes, the former can only be operating systems ($\mathtt{SystemLayer.OS}$) or firmware ($\mathtt{SystemLayer.Firmware}$). Therefore, the following constraint applies.

If a SystemLayer node hosts a Service node, the source node must be of type $\mathtt{SystemLayer.OS}$ or $\mathtt{SystemLayer.Firmware}$.

$$
    \forall e \in E, \big( \lambda_E(e) = \mathtt{"hosts"} \land \rho(e) = (n_s, n_t) \land \lambda_N(n_s) = (\mathtt{SystemLayer},\cdot) \land \lambda_N(n_t) = (\mathtt{Service}, \cdot) \big) \implies \sigma(n_s, \mathtt{asset\\_type}) \in \\{ \mathtt{"SystemLayer.OS"}, \mathtt{"SystemLayer.Firmware"} \\}
$$

### Virtual hosting SystemLayer node validity

Furthermore, with regard to nodes of type $\mathtt{SystemLayer}$, we also want to constrain the pattern $\delta(\mathtt{Virtual},\mathtt{SystemLayer})$. Specifically, it is valid for a $\mathtt{Virtual.VM}$ node to host a node of type $\mathtt{SystemLayer.OS}$ or $\mathtt{SystemLayer.Firmware}$, but it is not valid for a $\mathtt{Virtual.VM}$ node to host a $\mathtt{SystemLayer.HyperVisor}$ node.

If a Virtual node hosts a SystemLayer node, the target node must be of type $\mathtt{SystemLayer.OS}$ or $\mathtt{SystemLayer.Firmware}$.

$$
    \forall e \in E, \big( \lambda_E(e) = \mathtt{"hosts"} \land \rho(e) = (n_s, n_t) \land \lambda_N(n_s) = (\mathtt{Virtual},\cdot) \land \lambda_N(n_t) = (\mathtt{SystemLayer},\cdot) \big) \implies \big( \sigma(n_t, \mathtt{asset\\_type}) \in \\{ \mathtt{"SystemLayer.OS"}, \mathtt{"SystemLayer.Firmware"} \\} \big)
$$

### Hardware hosting SystemLayer node validity

A final constraint concerning $\mathtt{SystemLayer}$ nodes aims to restrict the cases in which they are hosted by hardware, specifically constraining the pattern $\delta(\mathtt{HW}, \mathtt{SystemLayer})=\mathtt{hosts}$. In particular, it is not valid for a node of type $\mathtt{HW}$ to directly host a node of type $\mathtt{SystemLayer.ContainerRuntime}$.

If a HW (hardware) node hosts a SystemLayer node, the target node can not be of type $\mathtt{SystemLayer.ContainerRuntime}$.

$$
\forall e \in E, \big( \lambda_E(e) = \mathtt{"hosts"} \land \rho(e) = (n_s, n_t) \land \lambda_N(n_s) = (\mathtt{HW},\cdot) \land \lambda_N(n_t) = (\mathtt{SystemLayer},\cdot) \big) \implies \sigma(n_t, \mathtt{asset\\_type}) \neq \mathtt{"SystemLayer.ContainerRuntime"}
$$

### Protocol validity

At this point, we define a constraint for the edge parameters related to protocols. The list of supported protocols is available [online](https://pennet.vseclab.it/catalogs/protocols). For each protocol, the type of relationship to which it applies, its corresponding layer in the ISO/OSI stack, and a brief description are provided. Therefore, the following constraint must hold.

Let $Y \subseteq R \times P \times V$ be the set of triples $(r, o, v)$ representing the protocols allowed in the MACM, where $r$ is a type of relationship (e.g., $\mathtt{uses}$), $o$ is a property key associated with a layer of the ISO/OSI model (e.g., $application\_protocol$), and $v$ is the name of a valid protocol (e.g., HTTP).

$$
    \forall e \in E, \forall p \in O \subseteq P, \big( \lambda_E(e) \in \\{ \mathtt{connects}, \mathtt{uses} \\} \land (e, p) \in \mathrm{dom}(\sigma) \big) \implies  \forall v \in \sigma(e, p), (\lambda_E(e), p, v) \in Y
$$

At this stage, the reader should have gathered that, in the simplest form of a MACM, each node is characterized by a *Primary Label*, an (optional) *Secondary Label*, and an *Asset Type*, where these three elements describe, at different levels, the type of asset that the node represents. In addition, there is an *id* (unique) and a *name* describing the asset. Clearly, further properties can be added as needed. As for the edges, they are simply characterized by a type, the pair of nodes they connect and, optionally, properties related to protocols.
