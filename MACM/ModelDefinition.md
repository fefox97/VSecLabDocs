---
title: Model Definition
---

# Model Definition

Il MACM (Multi-purpose Application Composition Model) è un formalismo basato su grafi che può essere adottato per rappresentare un Sistema sotto Analisi (SuA). Il MACM è un grafo diretto definito come una coppia:

$$
\mathcal{M} = (N, E)
$$

dove:

- $N$ è un insieme finito di nodi, che rappresentano gli asset del sistema target.
- $E \subseteq \mathbb{E}$ è un insieme finito di archi diretti etichettati, che rappresentano le relazioni tra gli asset.

## Modeling Assumptions

Il modello MACM fornisce una rappresentazione formale e strutturata di un sistema sotto analisi (SuA), con l'obiettivo di supportare il ragionamento, la validazione e la valutazione della sicurezza.

Per garantire chiarezza e precisione, il MACM si basa su alcune assunzioni fondamentali:

- **Snapshot Semantics.**  
  Il MACM cattura l'architettura di un sistema come uno snapshot—una vista statica e coerente della sua struttura in un momento specifico.  
  Non modella il comportamento a runtime, la riconfigurazione dinamica, il failover o i meccanismi di autoscaling.

- **Asset astratti e realizzati.**  
  Il modello MACM può essere usato per rappresentare sia sistemi concreti (es. per analisi di sicurezza) sia architetture pianificate o ipotetiche (es. durante la progettazione).  
  Non tutti gli asset modellati devono necessariamente esistere nel sistema distribuito: possono rappresentare anche componenti previsti o pianificati.

- **Interpretazione dei vincoli.**  
  Alcuni vincoli nella specifica MACM (es. hosting unico dei servizi) sono validi sotto l'assunzione di snapshot, ma potrebbero non valere globalmente in sistemi con infrastrutture dinamiche (es. Kubernetes).  
  In tali contesti, il modello va interpretato come una possibile istanza concreta di deployment.

- **Mondo finito e chiuso.**  
  Il MACM assume un insieme chiuso e finito di nodi e relazioni.  
  Elementi esterni non modellati sono considerati sconosciuti e non hanno effetto implicito sulla validazione.

Queste assunzioni guidano la struttura formale del modello e la semantica dei vincoli definiti nelle sezioni successive.

## Insiemi utili

Prima di caratterizzare formalmente i nodi e gli archi che compongono il MACM, è opportuno introdurre gli insiemi che verranno utilizzati nelle definizioni seguenti.

**Definizione (Primary Label Set)**  
Sia $L_P$ l'insieme finito delle etichette primarie ammesse nel modello:

$$
L_P \subseteq \mathbb{L}_P, \quad \emptyset \notin L_P
$$

dove $\mathbb{L}_P$ è l'insieme universale delle possibili etichette primarie.

Ogni elemento di $L_P$ definisce una classificazione di alto livello di un nodo nel modello MACM, indicando i principali ruoli o categorie di asset nel sistema.

Ogni nodo $n \in N$ deve essere associato esattamente a una $\ell_p \in L_P$; l'etichetta vuota non è ammessa.

**Definizione (Secondary Label Set)**  
Sia $\mathbb{L}_S$ l'insieme universale di tutte le etichette secondarie ammissibili.  
Ogni elemento di $L_S(\ell_p) \subseteq \mathbb{L}_S$ rappresenta una sottocategoria o specializzazione specifica di dominio dell'etichetta primaria $\ell_p$, fornendo una classificazione semantica più fine del nodo.

La funzione

$$
L_S: L_P \rightarrow 2^{\mathbb{L}_S}
$$

mappa ogni etichetta primaria al suo insieme di etichette secondarie valide. L'insieme $L_S(\ell_p)$ può essere vuoto, nel qual caso i nodi con $\ell_p$ possono omettere l'etichetta secondaria o impostare $\ell_s = \emptyset$.

**Definizione (Asset Type Set)**  
Sia $\mathbb{T}$ l'insieme universale dei tipi di asset nel modello MACM.

Ogni tipo di asset $t \in \mathbb{T}$ è associato univocamente a una etichetta primaria $\ell_p \in L_P$ e a una etichetta secondaria $\ell_s \in \mathbb{L}_S \cup \{ \emptyset \}$.

Definiamo una mappatura totale:

$$
\texttt{LabelPair} : \mathbb{T} \rightarrow L_P \times (\mathbb{L}_S \cup \{ \emptyset \})
$$

tale che per ogni tipo di asset $t \in \mathbb{T}$:

$$
\texttt{LabelPair}(t) = (\ell_p, \ell_s)
$$

L'etichetta primaria $\ell_p$ determina la categoria di alto livello dell'asset e corrisponde sempre al prefisso del tipo di asset. L'etichetta secondaria $\ell_s$ può essere vuota e raffina la classificazione all'interno della categoria primaria.

**Definizione (Relationship Type Set)**  
Sia $\mathbb{R}$ l'insieme universale di tutti i tipi di relazione ammissibili. Ogni elemento $r \in R \subseteq \mathbb{R}$ rappresenta il tipo di collegamento diretto tra due nodi $n_s, n_t \in N$, dove $n_s$ è il nodo sorgente e $n_t$ il nodo destinazione della relazione.

**Definizione (Parameter Domain)**  
Siano $\mathbb{K}$ e $\mathbb{V}$ gli insiemi universali, rispettivamente, delle chiavi e dei valori di parametro.

Definiamo:

- $\mathcal{K}_n \subseteq \mathbb{K}$ come l'insieme delle chiavi di parametro ammesse per i nodi.
- $\mathcal{V}_n \subseteq \mathbb{V}$ come l'insieme dei valori ammissibili per i parametri dei nodi.
- $\mathcal{K}_e \subseteq \mathbb{K}$ come l'insieme delle chiavi di parametro ammesse per le relazioni.
- $\mathcal{V}_e \subseteq \mathbb{V}$ come l'insieme dei valori ammissibili per i parametri delle relazioni.

L'insieme dei parametri associato a un nodo è definito come:

$$
P_n \subseteq \mathcal{K}_n \times 2^{\mathcal{V}_n}
$$

dove ogni chiave è associata a un insieme finito di valori.

L'insieme dei parametri associato a una relazione è definito come:

$$
P_e \subseteq \mathcal{K}_e \times 2^{\mathcal{V}_e}
$$

dove ogni chiave è associata a un insieme finito di valori, abilitando attributi multi-valore.

**Definizione (Universal Relationship Set)**  
Sia $\mathbb{E}$ l'insieme universale di tutte le relazioni sintatticamente ben formate sul modello MACM:

$$
\mathbb{E} = N \times R \times N \times P_e
$$

Questo insieme cattura tutto lo spazio delle relazioni sintatticamente costruibili dai nodi in $N$, tipi in $R$ e parametri in $P_e$, senza imporre alcuna validità semantica.

L'insieme effettivo delle relazioni usate in una istanza MACM è denotato da:

$$
E \subseteq \mathbb{E}
$$

ed è soggetto ai vincoli semantici specificati dal modello.

## Definizione di Nodo

Ogni nodo $n \in N$ è una 5-upla:

$$
n = (\texttt{id}, \ell_p, \ell_s, t, P_n)
$$

dove:

- $\texttt{id} \in \mathbb{ID}$ è un identificatore univoco.
- $\ell_p \in L_P$ è l'etichetta primaria (obbligatoria).
- $\ell_s \in L_S(\ell_p) \cup \{ \emptyset \}$ è l'etichetta secondaria (opzionale).
- $t \in \mathbb{T}$ è il tipo di asset (obbligatorio).
- $P_n \subseteq \mathcal{K}_n \times 2^{\mathcal{V}_n}$ è un insieme opzionale di coppie chiave-insieme di valori associati al nodo.

**Vincolo sintattico (Identificatori univoci dei nodi)**

$$
\forall n_i,n_j\in N,\;n_i\neq n_j\;\Rightarrow\;\texttt{id}(n_i)\neq\texttt{id}(n_j)
$$

Questo vincolo assicura che tutti i nodi nell'insieme $N$ abbiano identificatori univoci.

## Definizione di Relazione

Ogni relazione $e \in E$ è una 4-upla:

$$
e = (n_s, r, n_t, P_e)
$$

dove:

- $r \in R$ è il tipo di relazione.
- $n_s, n_t \in N$ sono nodi tali che $n_s \neq n_t$.
- $n_s$ è il **nodo sorgente** della relazione.
- $n_t$ è il **nodo destinazione** della relazione.
- $P_e \subseteq \mathcal{K}_e \times 2^{\mathcal{V}_e}$ è un insieme opzionale di proprietà chiave-valore.

> **Nota:** Formalmente $E \subseteq \mathbb{E} = N \times R \times N \times P_e$, ma non ogni quadrupla possibile in questo prodotto cartesiano rappresenta una relazione semanticamente valida nel modello MACM. La validità effettiva di una quadrupla $(n_s, r, n_t, P_e) \in E$ è determinata dai vincoli semantici definiti nella sezione seguente, come vincoli sui tipi di nodo, direzioni delle relazioni e regole di compatibilità specifiche del contesto.

## Funzioni utili

In questa sezione definiamo alcune funzioni utili, che verranno usate in seguito.

**Definizione (Funzioni di proiezione delle etichette dei nodi)**

Sia $n = (\texttt{id}, \ell_p, \ell_s, t, P_n) \in N$ un nodo nel modello MACM.

Definiamo le seguenti funzioni di proiezione:

$$
\texttt{PrimaryLabel} : N \rightarrow L_P,\qquad \texttt{PrimaryLabel}(n) = \ell_p
$$

$$
\texttt{SecondaryLabel} : N \rightarrow \mathbb{L}_S \cup \{ \emptyset \},\qquad \texttt{SecondaryLabel}(n) = \ell_s
$$

Queste funzioni permettono l'accesso esplicito alle componenti di etichetta di un nodo e sono usate nei vincoli semantici e nella costruzione dei tipi di asset.

**Definizione (Tipo di asset di un nodo)**

Sia $n = (\texttt{id}, \ell_p, \ell_s, t, P_n) \in N$ un nodo nel modello MACM.

Definiamo la funzione:

$$
\texttt{AssetType} : N \rightarrow \mathbb{T}
\quad \text{tale che} \quad \texttt{AssetType}(n) = t
$$

Il tipo di asset $t \in \mathbb{T}$ assegnato al nodo è semanticamente associato alle etichette $\ell_p$ e $\ell_s$ tramite la funzione $\texttt{LabelPair}$, come definito nell'insieme dei tipi di asset.

Nota che più tipi di asset possono condividere la stessa coppia di etichette $(\ell_p, \ell_s)$.

**Definizione (Parametri ammessi per tipo di asset)**

Definiamo una funzione totale:

$$
\texttt{AllowedParams}_n : \mathbb{T} \rightarrow 2^{\mathcal{K}_n}
$$

che mappa ogni tipo di asset $t \in \mathbb{T}$ al sottoinsieme di chiavi ammesse per quel tipo.

Un nodo è semanticamente valido rispetto ai suoi parametri se:

$$
\forall (\texttt{id}, \ell_p, \ell_s, t, P_n) \in N,\quad \forall (k, V_k) \in P_n,\quad k \in \texttt{AllowedParams}_n(t)
$$

**Definizione (Valori ammessi per chiave di parametro nodo)**

Siano $\mathcal{K}_n$ e $\mathcal{V}_n$ gli insiemi delle chiavi e dei valori di parametro ammissibili per i nodi.

Definiamo la funzione totale:

$$
\texttt{AllowedValues}_n : \mathcal{K}_n \rightarrow 2^{\mathcal{V}_n}
$$

tale che, per ogni chiave $k \in \mathcal{K}_n$, il valore $\texttt{AllowedValues}_n(k)$ restituisce l'insieme di tutti i valori ammissibili per quella chiave.

**Definizione (Parametri ammessi per relazione)**

Definiamo una funzione totale:

$$
\texttt{AllowedParams}_e : \mathbb{R} \rightarrow 2^{\mathcal{K}_e}
$$

che mappa ogni tipo di relazione $r \in \mathbb{R}$ al sottoinsieme di chiavi di parametro ammesse per quel tipo.

Una relazione è semanticamente valida rispetto ai suoi parametri se:

$$
\forall (n_s, r, n_t, P_e) \in E,\quad \forall (k, V_k) \in P_e,\quad k \in \texttt{AllowedParams}_e(r)
$$

**Definizione (Valori ammessi per chiave di parametro relazione)**

Siano $\mathcal{K}_e$ e $\mathcal{V}_e$ gli insiemi delle chiavi e dei valori di parametro ammissibili per le relazioni.

Definiamo la funzione totale:

$$
\texttt{AllowedValues}_e : \mathcal{K}_e \rightarrow 2^{\mathcal{V}_e}
$$

tale che, per ogni chiave $k \in \mathcal{K}_e$, il valore $\texttt{AllowedValues}_e(k)$ restituisce l'insieme di tutti i valori ammissibili per quella chiave.

**Definizione (Cardinalità dei valori per chiave di parametro)**

Definiamo una funzione totale:

$$
\texttt{ValueCardinality} : (\mathcal{K}_n \cup \mathcal{K}_e) \rightarrow \mathbb{N}^+ \times \mathbb{N}^+
$$

che mappa ogni chiave di parametro ammissibile $k$ a una coppia $(m_k, M_k)$, dove:

- $m_k \in \mathbb{N}^+$ è la cardinalità minima ammessa per l'insieme di valori associato.
- $M_k \in \mathbb{N}^+$ è la cardinalità massima ammessa per l'insieme di valori associato.

## Vincoli

Il modello MACM è soggetto a un insieme di vincoli formali che ne governano la ben formatura e la coerenza semantica.  
Distinguiamo tra due categorie di vincoli:

- **Vincoli sintattici** — Sono obbligatori e devono essere soddisfatti affinché il modello MACM sia considerato strutturalmente valido. La violazione di un vincolo sintattico rende il modello invalido o non processabile.
- **Vincoli semantici** — Rappresentano regole di validazione opzionali che catturano aspettative specifiche di dominio, best practice o condizioni di coerenza semantica. Pur non essendo strettamente obbligatori, la violazione di vincoli semantici genera warning e tipicamente indica possibili problemi di progettazione o configurazione.

Salvo diversa specifica, i vincoli semantici sono trattati come avvisi e dovrebbero essere mostrati agli utenti durante la validazione del modello per supportare il troubleshooting.

**Vincolo sintattico (Connettività del grafo)**

Il modello MACM deve essere debolmente connesso.

Sia $G = (N, E')$ la versione non orientata del grafo delle relazioni, dove:

$$
E' = \{ \{n_s, n_t\} \mid (n_s, r, n_t) \in E \}
$$

Allora, per ogni coppia di nodi $n_i, n_j \in N$, esiste un cammino non orientato in $G$ che li connette:

$$
\forall n_i, n_j \in N, \quad \exists \text{ un cammino in } G \text{ da } n_i \text{ a } n_j
$$

> **Nota:** Attenzione. Non sono sicuro della validità considerando un sottinsieme degli archi, inoltre sappiamo che il grafo è ciclico con la relazione use.

**Vincolo sintattico (Acriticità su relazioni specifiche)**

Sia $R_{\mathrm{acyclic}} \subseteq R$ il sottoinsieme di tipi di relazione che non devono formare cicli.

Sia $G_{\mathrm{acyc}} = (N, E_{\mathrm{acyc}})$ il sottografo diretto, dove:

$$
E_{\mathrm{acyc}} = \{ (n_s, n_t, P_e) \mid (n_s, r, n_t, P_e) \in E,\; r \in R_{\mathrm{acyclic}} \}
$$

Allora, il sottografo $G_{\mathrm{acyc}}$ non deve contenere cicli, cioè:

$$
G_{\mathrm{acyc}} \text{ è un DAG (Directed Acyclic Graph)}
$$

**Vincolo semantico (Valori validi per parametri nodo)**

Un'assegnazione di parametri è semanticamente valida se:

$$
\forall (k, V_k) \in P_n, \quad V_k \subseteq \texttt{AllowedValues}_n(k)
$$

**Vincolo semantico (Valori validi per parametri relazione)**

Un'assegnazione di parametri è semanticamente valida se:

$$
\forall (k, V_k) \in P_e, \quad V_k \subseteq \texttt{AllowedValues}_e(k)
$$

**Vincolo semantico (Cardinalità dell'insieme di valori per parametro)**

Un'assegnazione di parametri è semanticamente valida se:

$$
\forall (k, V_k) \in P_n \cup P_e,\quad m_k \leq |V_k| \leq M_k
\quad \text{dove} \quad (m_k, M_k) = \texttt{ValueCardinality}(k)
$$