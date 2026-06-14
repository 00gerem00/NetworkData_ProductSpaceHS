# **Dataset Description - Product Space (HS)**

The following *markdown* file is a technical reference for the dataset used in this project. It documents the **provenance**, the **meaning of every variable**, and the **caveats** that
affect interpretation. The conceptual explanation of how the network is built (RCA, proximity, worked example) lives in the analysis report (`.Rmd`); this file is the concise reference card.

---

## **1. Overview**

The **Product Space (HS)** is a network of traded products. Nodes are products
(Harmonized System classification); an edge between two products means the same
countries tend to be competitive exporters of both, interpreted as the two
products requiring **similar productive capabilities**.

The network is a one-mode **projection** of an underlying bipartite
country-product network. It is built *from* country export data, but the country
layer is collapsed away and is **not** present in these files.

## **2. Provenance**


| field | value |
|---|---|
| Dataset name | `product_space (HS)` |
| Source package | `PS_HS.zip` - Michele Coscia, https://www.michelecoscia.com/?page_id=223 |
| Underlying data | UN Comtrade worldwide trade patterns, via the Atlas of Economic Complexity |
| Classification | Harmonized System (HS), 4-digit product codes |
| Construction rule | Maximum Spanning Tree + proximity links with phi > 0.55 |
| Primary reference | Hidalgo, Klinger, Barabasi, Hausmann - *Science* 317:482-487 (2007) |
| Catalogued by | ICON - Colorado Index of Complex Networks (https://icon.colorado.edu) |

## **3. Network structure (verified)**

| property | value |
|---|---|
| Nodes (products) | 866 |
| Edges | 2,532 |
| Directed | No |
| Connected components | 1 (fully connected) |
| Isolated nodes | 0 |
| Self-loops / multi-edges | 0 / 0 |
| Author communities | 12 (encoded in node colour - see section 5) |

## **4. Files and variables**

### `nodes.csv`
| raw column | true meaning | notes |
|---|---|---|
| `index` | 0-based node id (used by the edge list) | |
| `pid` | product id (HS code, 4 digits) | |
| `community` | author community id | **all zeros** - not populated; real community is in the colour (section 5) |
| `size` | product market share | **non-linear, layout-only** (section 6) |
| `pos` | layout coords `array([x, y])` | parsed into `x`, `y` |
| `leamer` | **product name** | column mislabelled (section 6) |
| `name` | **node colour (hex)** = community | column mislabelled (section 6) |
| `_pos` | secondary coordinates | unused |

### `edges.csv`
| raw column | true meaning | notes |
|---|---|---|
| `source`, `target` | the two products connected (by `index`) | |
| `width` | link strength as a **drawing thickness** | **non-linear, layout-only** (section 6) |
| `color` | link colour | redundant with width |

### `gprops.csv`
Dataset-level metadata: name, description, source, citation, tags.

## **5. The colour encodes the community**

Per the official documentation: *"the node color is a map corresponding to node
community."* The explicit `community` column is empty, but the authors' partition
survives in the node colour: **12 distinct colours = 12 communities** (sizes from
252 down to 19 products). This is used as a **reference partition** to validate
our own community detection.

## **6. Caveats (must read)**

1. **`width` and `size` are layout-only and non-linear.** The official docs state
   both are "not linearly connected to its strength / market share" and "should
   be used exclusively for layout purposes." Consequently the edge `width` is
   **not** the proximity φ and cannot be converted back to it. It is retained as
   an **ordinal** edge weight and used only for weighted-vs-unweighted robustness
   comparisons; `size` is likewise kept as an ordinal/visual cue. Primary analyses
   treat the network as **unweighted**, with weighted variants reported as a
   robustness check.

2. **Mislabelled columns (HS vs SITC headers).** Headers follow the SITC layout,
   but this is the HS file. So the column named `leamer` actually holds the **product name**, and the column named `name` actually holds the **hex colour / community**. The cleaning pipeline corrects this.

3. **The country layer is gone.** As a projection, country identities, the
   country-product matrix, RCA values, and the raw proximity matrix are **not recoverable** from these files. No country-level or geographic analysis is possible; the project is product-side only.

## **7. Processed outputs**

The cleaning pipeline (`.Rmd`) produces, in `data/processed/`:

| file | content |
|---|---|
| `product_space.rds` | cleaned undirected igraph (ordinal `weight` from `width_layout`; primary analyses unweighted) with corrected attributes + provenance metadata |
| `layout_xy.rds` | node coordinate matrix for the canonical layout |
| `nodes_clean.csv` | human-readable cleaned node table |
| `edges_clean.csv` | human-readable cleaned edge table |
