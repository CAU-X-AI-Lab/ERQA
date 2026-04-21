# ERQA Dataset

The **ERQA (Exact Retrieval Question Answering)** dataset is a large-scale benchmark designed to evaluate retrieval-augmented generation systems under the **Exact Retrieval Problem (ERP)** setting — queries that require **all** specified constraints to be satisfied simultaneously.

---

## 📌 Overview

ERQA consists of three domain-specific subsets:

| Subset      | Language | Source Graph                              | #QA Pairs | Avg. Constraints / Query |
|-------------|----------|--------------------------------------------|-----------|--------------------------|
| **FB-ERQA** | English  | FB15K-237 (encyclopedic KG)                | 80,000    | 6.1                      |
| **UD-ERQA** | English  | Academic multi-domain KG (18 disciplines)  | 10,000    | 4.7                      |
| **CM-ERQA** | Chinese  | CPubMed-KG (biomedical KG)                 | 30,000    | 5.4                      |

Additionally, we provide an **approximate-match subset** for robustness testing:

- 1,000 queries where **no exact subgraph match** exists, but approximate matches with edit distance \(x = 1, 2, 3\) are available.

---

## 📂 Data Construction Logic

ERQA is constructed by identifying **Bridge-Star Subgraphs** in large-scale knowledge graphs and generating **multi-constraint factual questions**.

### **1. Bridge-Star Subgraph Pattern**

A Bridge-Star Subgraph is defined as:

- Two **high-degree core nodes** (star nodes) each connected to multiple 1-hop neighbors.
- The two star nodes are **not directly connected**; instead, they share at least **two bridge nodes** which connect them via indirect paths.
- These bridge nodes serve as **shared semantic connectors** and form the basis for **multi-constraint reasoning**.

<p align="center">
  <img src="images/bridge-star.png" width="500">
</p>

### **2. Query Formation**

For each Bridge-Star Subgraph:

1. **Randomly hide one core node** → designated as the **answer**.
2. Collect:
   - All **1-hop neighbors** of the visible core node.
   - All **bridge nodes** linking the two core nodes.
3. Pass this structured context to an **LLM** to generate a **natural language question** that encodes **all constraints**.

**Example:**

> Subgraph: Magnesium ↔ (bridge: Thyroid Health, Bone Formation) ↔ Zinc  
> Visible core node: Magnesium  
> Generated Question:  
> _"Which element is associated with bone formation, thyroid health, hormone metabolism, and immune system?"_  
> Correct Answer: **Magnesium** (must satisfy **all** constraints).

### **3. Constraints and Distractors**

- Each query includes **multiple factual constraints** derived from the subgraph.
- Negative candidates (distractors) are entities that satisfy **some** but not **all** constraints.
