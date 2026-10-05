---
type: proposal
source: "[[research-proposal-log]]"
created: 2026-10-04
---

# Sketch — Federated Graph-Based Anomaly Detection for Financial Institutions

Direction sketch (Edge 2). Written before supervisor outreach; supervisor-specific framing belongs in the cover email, not here.

Citations are author-year (for example, "Tan et al., 2023"). Full references are at the end. Source summaries are in `Sources/`.

## Problem

Banks and payment providers each see part of the same fraud network. Mule accounts, laundering chains, and scam rings often cross institution boundaries, but each institution only trains on its own transaction graph, so cross-institution structure is missed. Raw transaction graphs cannot easily be shared because of privacy and competition law.

The problem: **detect anomalous accounts and transactions across several institutions, each holding a partial graph, without sharing raw records.**

Federated graph models have seen limited adoption in fraud detection, which leaves considerable scope for experimental work. The area has many open problems. For this setting, this sketch focuses on four:

1. **Heterogeneity.** Institutions differ in customers, feature schema, label definitions, and class balance. Standard federated averaging (McMahan et al., 2017) assumes homogeneous clients, and federated graph learning degrades on non-IID data (He et al., 2021; Liu et al., 2025). Shared classifiers may not transfer across feature sets (Chen et al., 2025).
2. **Privacy.** In graphs, edges and structure leak information, so anything shared (embeddings, structure encoders, prototypes, gradients) is an attack surface. Current federated graph anomaly detection does not measure this leakage (Fang et al., 2026).
3. **Entity alignment.** The same account can appear at several institutions, so cross-institution edges need a matching step. Matching is itself a privacy problem, and the closest work assumes non-overlapping nodes (Fang et al., 2026).
4. **Rare topologies.** Fraud typologies are rare and structurally different (mule chains, star-shaped collection accounts, laundering cycles), with few labels per institution. Federated averaging dilutes localised patterns (Park et al., 2025), and no graph work has tested typology-level rarity.

## Research Questions

1. **Heterogeneity.** Can a federated graph model detect fraud better than baseline models when institutions differ in customers, relation types, and label definitions?
2. **Privacy.** Can institutions share what the model needs without exposing information about their customers' relationships?
3. **Rare topologies.** Can the model detect fraud whose network structure is rare within and across institutions?

Entity alignment is a precondition for RQ1 and RQ2. It is treated as an assumption and an open design choice for now; if the supervisor agrees, it becomes a fourth question.

**Proposed scope:** RQ1 is the core contribution. RQ2 is a constraint measured in the evaluation. RQ3 is an ablation. The three questions could otherwise be three papers. This needs a supervisor decision.

## Literature Review

### Closest prior work

| Work | What it does | Overlap with this sketch |
|---|---|---|
| Fang et al. (2026), FedGAD | Unsupervised federated graph anomaly detection over subgraphs, with a neighbour learner, cross-client neighbour reconstruction, and multiscale contrastive learning | **Direct overlap.** Subgraph-per-client setting, unsupervised, cross-client structure. Banks appear only in its Fig. 1 illustration; no bank data is used. |
| Al Tfaily et al. (2025), FedGATSage | Client-side GAT with Louvain communities; server-side GraphSAGE on a community overlay | Overlaps heterogeneity and partly privacy. Privacy is informal; data is IoT network flow. |
| Tan et al. (2023), FedStar | Shares a structure encoder across clients; feature encoder stays local | Overlaps heterogeneity and rare-structure sharing. Graph classification only. |
| Zheng et al. (2026), FedProD | Groups clients by uncertainty; exchanges reliability-weighted class prototypes | Overlaps rare-class handling. Intrusion detection, not graphs. |
| Li et al. (2025a), relation-aware heterogeneous GNN | Relation-aware aggregation over typed edges | Overlaps heterogeneity, but centralised. Ling Chen is a co-author. |
| Park et al. (2025), federated gradient boosting | Four federated GBDT models on banking fraud; FedXGBBagging performed best | Non-graph federated baseline. Reports instability and dilution of localised fraud. |
| Ghourabi and Khaldi (2026), federated ensemble | Federated XGBoost, CatBoost, and MLP ensemble | Non-graph baseline. No differential privacy or secure aggregation. |
| Zhu et al. (2025), POWER | Federated continual GNN with replay | Tangential; relevant to drift and new typologies. |
| Liu et al. (2025), federated GNN survey | Survey and taxonomy of federated GNNs | Background and benchmark partitioning. |

### Key findings

**FedGAD (Fang et al., 2026).**

- *Method.* A neighbour learner predicts each node's degree and neighbour features from masked neighbourhoods. A multiscale contrastive objective (node vs. node, node vs. ego-net) scores anomalies. Training is FedAvg across clients.
- *Cross-client step.* Each node's neighbours are partly reconstructed from other clients' embeddings (Eqs. 9–12, Algorithm 1). This is how the paper handles heterogeneity.
- *Results.* Reddit (three clients): 96.42% accuracy, 3.2 points above GADNR. Cora: 91.42%, 0.04 points below GADNR. Versus local-only training, +2.84 on Reddit (five clients) and +0.65 on Weibo (three clients, imbalanced).
- *Data.* Benchmark graphs only: Reddit, Weibo, Disney, Books, Tolokers, Questions, and Cora (synthetic). No financial transaction graph is used. The paper gives three different dataset counts.
- *Partitions.* Louvain (class-balanced) and Dirichlet β = 0.2 (imbalanced), with 3, 5, and 10 clients.
- *Limitations.*
  1. The no-data-sharing claim conflicts with the method: cross-client reconstruction passes node embeddings through the server. No leakage analysis is reported.
  2. The cross-client update (Eq. 24) is underspecified. It is unclear whether raw data or only parameters move.
  3. No entity alignment; the no-overlap assumption fails for banks.
  4. The anomaly threshold ("m%") is unspecified. If tuned on labels, the unsupervised framing is weakened.
  5. Node-level only. RQ2 concerns edges, which FedGAD does not address.
  6. Three repetitions, no significance tests, accuracy as the headline metric despite rare anomalies. Result tables did not extract from the PDF, so the numbers come from body text.
  7. Code availability is not stated in the text read.

**FedStar (Tan et al., 2023).** A degree one-hot and random-walk structure embedding feed a GCN structure encoder, averaged on the server. The GIN feature encoder stays local. Sharing only the structure encoder beat sharing all parameters (Table 2). Gains over local training: +2.58 to +4.41. Not studied: privacy, node-level tasks, transaction data. It does not model relation types.

**FedProD (Zheng et al., 2026).** Clients are graded by Monte Carlo dropout uncertainty on a server-side public set. Good-client prototypes are weighted by reliability. The local loss adds a KL term toward good and poor prototypes (λ = 0.25); the authors note the negative KL term has no finite lower bound. Removing good prototypes hurts more than removing poor ones. Public-data sweep: 96.29% with no public data, 97.79% with 75% of it.

**Other sources.**

- *Park et al. (2025).* Federated GBDT on real banking data. Shows dilution of localised fraud (ATM skimming) and instability under data-quantity skew and bank churn.
- *Ghourabi and Khaldi (2026).* Federated tabular ensemble with no privacy mechanism. Non-graph baseline.
- *Li et al. (2025a).* Relation-aware heterogeneous GNN, centralised. Shows typed relations can be modelled at scale.
- *Johannessen and Jullum (2023).* Heterogeneous GNN for money laundering on DNB's real bank graph. Centralised; data is confidential.
- *Xiang et al. (2023).* Semi-supervised attribute-driven graph for credit-card fraud under label scarcity. Centralised.
- *Chen et al. (2025), FED-SPFD.* Federated transfer for credit-card fraud under heterogeneous features.
- *Lamptey et al. (2025), FedMuL.* Federated multi-label learning on streams, not graphs.
- *Li et al. (2025b), CGNN.* Centralised graph fraud detection with very few labels.
- *Wang et al. (2020), CMLP.* Multi-label propagation for fraud, centralised.
- *Yuan et al. (2020), XGNN.* Model-level GNN explanations, centralised.
- *Motie and Raahemi (2023).* Review of GNNs for financial fraud. Centralised; predates FedGAD.
- *He et al. (2021), FedGraphNN.* Federated GNN benchmark with a documented LDA partitioning method. No financial graph.
- *Corcuera Bárcena et al. (2022), Fed-XAI.* Peripheral; gives the post-hoc vs. interpretable-by-design vocabulary.

### Gap status

Based on full texts where marked, and on abstracts and introductions otherwise. A full-text review is still needed before any novelty claim.

| Gap | Status | Closest evidence |
|---|---|---|
| 1. Heterogeneity | **Partly covered.** Structure sharing and cross-client reconstruction handle it without relation types. Typed relations exist only in centralised models. | FedStar, FedGAD, Li et al. (2025a) |
| 2. Privacy | **Open as a formal result.** Neither closest paper measures leakage from shared structure. | FedGAD, FedStar, Ghourabi and Khaldi (2026) |
| 3. Entity alignment | **Open.** The closest work assumes no overlapping nodes. | FedGAD |
| 4. Rare topologies | **Partly covered.** Prototype reliability works for rare classes, not rare topologies. Label scarcity is covered only centrally. | FedProD, Park et al. (2025), Li et al. (2025b) |

**Main finding.** FedGAD already covers the subgraph-per-client, unsupervised, cross-client setting, and it assumes away entity alignment. The sketch's novelty must come from what FedGAD does not do: a federated typed-relation model (gap 1), measured and bounded leakage (gap 2), privacy-preserving matching (gap 3), or a rare-topology mechanism that does not need a public clean set (gap 4).

**Secondary gaps.** Knowledge distillation into local models; a shared fraud-pattern knowledge base; explainability of federated graph decisions (open for federated graphs); and multi-label propagation for accounts with several typologies.

## Candidate Method

**Setting: semi-supervised.** Each institution has a few confirmed fraud labels and a large unlabelled graph, which matches how banks receive labels: late and in small numbers. Training combines a supervised loss on labelled accounts with a propagation or contrastive term on unlabelled ones, so the method is not just FedGAD's unsupervised objective.

The method is organised around the three research questions.

**RQ1 (heterogeneity): shared structure, local features.** Based on Tan et al. (2023). Each institution's feature encoder stays local, since feature sets differ. The structure encoder is averaged on the server. The bank version adds relation types to the structure channel, which FedStar does not model. Li et al. (2025a) show one way to model typed relations centrally.

**RQ2 (privacy): controlled sharing.** Only the structure encoder, prototypes, and gradients cross institution boundaries, and each is measured for leakage. Privacy is treated as a constraint on the method, not a separate contribution. Entity alignment is part of this boundary: the matching step must state what crosses it (node embeddings, edge aggregates, or gradients). One candidate is matching on hashed, salted identifiers within a consortium, which needs a re-identification check before use.

**RQ3 (rare topologies): prototype exchange.** Based on Zheng et al. (2026). Institutions share class prototypes weighted by reliability. Each pulls its own fraud and benign prototypes toward reliable ones. Fraud prototypes rest on few labels, so their variance needs checking.

**Other method choices:**

- **Learning setting (decided: semi-supervised).** A purely unsupervised version would mostly overlap FedGAD. A fully supervised version would overlap labelled federated fraud work (Lamptey et al., 2025; Chen et al., 2025; Park et al., 2025). Semi-supervised fits label-scarce settings (Li et al., 2025b; Xiang et al., 2023), which have a centralised precedent but no federated version. Supervised and unsupervised variants stay as ablations.
- **Multi-label propagation head.** Follows Wang et al. (2020), federated as in Lamptey et al. (2025), for accounts that carry several typologies.
- **Threshold.** Scores use a threshold fixed on validation data, not test labels. FedGAD's m% rule is unspecified.

## Data

Public or synthetic transaction graphs partitioned into institutions, to be selected in `Datasets/`. Real institutional data is not assumed. Any use of employer data must be checked for confidentiality first. Partitioning one graph into institutions is artificial and must be documented.

No existing dataset fits the setting:

- Fang et al. (2026): social, review, and crowdsourcing graphs, plus one synthetic graph.
- Al Tfaily et al. (2025): IoT network flow.
- He et al. (2021): no financial graph, but a documented partitioning method.
- Johannessen and Jullum (2023): DNB's real heterogeneous graph. Closest realistic structure, but not public; reference only.
- Park et al. (2025): a private banking dataset (Financial Security Institute) and a public banking dataset. The public one is a candidate if its features and labels suit a graph.

The dataset must be chosen before experiments begin.

## Evaluation

- **Baselines.**
  - Centralised GNN (upper bound), federated averaging GNN, and local-only GNN.
  - Federated graph baselines: FedGAD (Fang et al., 2026) and FedGATSage (Al Tfaily et al., 2025), run on the same partitions with the threshold rule fixed.
  - Structure sharing: FedStar (Tan et al., 2023).
  - Centralised references: CGNN (Li et al., 2025b) for label scarcity; Li et al. (2025a) for typed relations.
  - Non-graph federated baselines: Park et al. (2025) and Ghourabi and Khaldi (2026). A graph model must beat them to justify its cost.
- **Metrics.** Recall at fixed precision, PR-AUC, F1, and performance on rare typologies. Leakage under a membership or edge-inference attack. Explanation fidelity against Yuan et al. (2020), stating whether explanations are post-hoc or interpretable by design. Accuracy is not a headline metric.
- **Heterogeneity (gap 1).** Non-IID partitions by institution size, customers, and fraud-type mix, using FedGAD's Louvain and Dirichlet (β = 0.2) splits and He et al.'s LDA method. Report variance across partition seeds.
- **Structure-sharing ablation.** Share only the structure encoder, all parameters, or nothing, on the same partitions.
- **Leakage (gap 2).** Edge-inference attack on each shared quantity (structure encoder, prototypes, gradients). Measure privacy for each configuration; do not assume it.
- **Rare topologies (gap 4).**
  - Label-fraction sweep: 1%, 5%, and 10% labelled accounts per institution, against supervised-only and FedGAD.
  - Prototype ablation: reliable and unreliable prototypes, reliable only, unreliable only, and none.
- **Clean reference set (gap 4).** Test with and without a small consented reference set. Zheng et al.'s public-data sweep (96.29% vs. 97.79%) sets the reference size of the gap.
- **Robustness (Park et al., 2025).** Data-quantity skew, and institutions joining or leaving mid-training.
- **Entity alignment (gap 3).** Measure how missed and false matches change detection, with and without the matching step.
- **Splits.** Time-based train/test split, since fraud is temporal. Threshold fixed on validation data.

## Risks

- **FedGAD overlap.** Fang et al. (2026) already cover the subgraph-per-client setting. Banks are illustrative only, and the paper assumes away entity alignment. The contribution must be defined against it; the semi-supervised setting is the main difference.
- **Relation-aware claim.** Heterogeneity is partly covered by FedStar and FedGAD. The relation-aware claim needs a specific federated mechanism, since Li et al. (2025a) is centralised.
- **Informal privacy claims.** FedGAD's cross-client step shares embeddings with no leakage analysis. Gap 2 needs a formal edge-privacy result.
- **Entity alignment.** No precedent in this set. Matching can leak identity, so it may need a separate study.
- **Supervised overlap.** Labelled federated fraud work (Lamptey et al., 2025; Chen et al., 2025; Park et al., 2025) means the semi-supervised framing must stay distinct.
- **Reference set.** FedProD needs a clean public set that banks may not have. Without it, the no-reference variant may underperform. The negative KL term may also make training unstable.
- **Bank conditions.** Park et al. (2025) report instability under data-quantity skew and bank churn. These can lower results as well as test robustness.
- **Scope.** The three RQs may be three papers. Needs a supervisor decision.
- **Threshold and metrics.** FedGAD's threshold is unspecified, so its results cannot be compared directly until the rule is recovered. Accuracy would hide weak rare-fraud detection.
- **Data.** No public dataset matches cross-institution structure. Partitioning is artificial. Employer data needs a confidentiality check.
- **Utility cost.** Privacy guarantees can reduce utility; measure the trade-off.
- **Rare-topology variance.** Few labels mean high variance in prototypes and label-fraction results.
- **Review coverage.** Motie and Raahemi (2023) predates FedGAD and does not settle federated coverage.
- **Unverified references.** Zheng et al.'s conclusion, FedGAD's result tables, and Johannessen and Jullum's journal details are unchecked. The FedAvg reference (McMahan et al., 2017) is from memory.

## Supervisor Fit

Mapped from the Cluster D and E descriptions in [[research-proposal-log]]. Check each supervisor's recent papers before pitching.

| Priority | Supervisor | Fit | Caution |
|---|---|---|---|
| 1 | Ling Chen (UTS) | Graph anomaly detection for transaction fraud; closest match to the relation-aware GNN core (gap 1) | Check current federated work. Co-author of Li et al. (2025a) and Xiang et al. (2023); both centralised. |
| 2 | Zahir Tari (RMIT) | FL work on poisoning defence and FL intrusion detection; active project on topology-adaptive, distillation-driven FL (maps to gap 1) | Graph-specific FL record and project details not yet verified. |
| 3 | Bo Liu (UTS) | AI security and privacy; fits gap 2 and gap 3 | Industry link (RBA) unconfirmed. No vault source by this author. |
| 4 | Shui Yu (UTS) | Privacy-preserving analytics; second option for gaps 2 and 3 | No fraud-specific content. |
| Reserve | Chengqi Zhang (UTS, emeritus) | Federated and non-IID graph work; strongest topic match for gap 1. Co-author of Tan et al. (2023) | Emeritus status and capacity unconfirmed. Co-supervision only. |

### Reference papers by supervisor

- **Ling Chen (UTS):** Li et al. (2025a), relation-aware heterogeneous GNN; Xiang et al. (2023), semi-supervised credit card fraud detection.
- **Chengqi Zhang (UTS, emeritus):** Tan et al. (2023), FedStar.
- **Bo Liu (UTS):** candidates, not yet read; titles and authorship unverified. Two Google Scholar entries and one IEEE article (2026, vol. 2, no. 11235980) were noted.
- **Zahir Tari (RMIT):** not yet in the reference list. Candidates from search, not yet read: Xiong, Dong, Sohrabi & Tari (2025), FedLAD, federated poisoning defence (arXiv 2508.02136); FedGDD, federated Sybil poisoning defence, *IEEE Transactions on Reliability* (2026); Zhao, Tari, Sohrabi, Wang & Xia (2026), split learning with local epoch regulation, *IEEE TDSC* 23, 170–191 (venue details from search summary). Also IEEE Xplore 11415392 (DOI 10.1109/TSC.2026.3668916; title and authors not yet read; journal inferred from the DOI prefix, so *IEEE Transactions on Services Computing* is unconfirmed).
- **Shui Yu (UTS):** four candidates from his Google Scholar profile, not yet read (titles, venues, and authorship unverified).

## References

Full texts are in `Sources/pdfs/`.

1. Al Tfaily, F., Ghalmane, Z., Brahmia, M. el A., Hazimeh, H., Jaber, A., & Zghal, M. (2025). Graph-based federated learning approach for intrusion detection in IoT networks. *Scientific Reports*, 15, 41264. https://doi.org/10.1038/s41598-025-25175-1
2. Chen, Y., Zhang, K., Zhu, H., & Qiu, Z. (2025). A novel federated transfer learning framework for credit card fraud detection (FED-SPFD). *Risks*, 13(11), 208. https://doi.org/10.3390/risks13110208
3. Corcuera Bárcena, J. L., Daole, M., Ducange, P., Marcelloni, F., Renda, A., Ruffini, F., & Schiavo, A. (2022). Fed-XAI: Federated learning of explainable artificial intelligence models. *Proceedings of XAI.it 2022, CEUR Workshop Proceedings*, 3277. https://ceur-ws.org/Vol-3277/paper8.pdf
4. Fang, H., Gao, Y., Zhang, P., Zhou, S., Chen, H., Bu, J., & Wang, H. (2026). Contrastive federated learning for graph anomaly detection (FedGAD). *IEEE Transactions on Neural Networks and Learning Systems*, 37(1). https://doi.org/10.1109/TNNLS.2025.3601449
5. Ghourabi, A., & Khaldi, K. (2026). A federated ensemble learning framework for distributed fraud detection. *Applied Sciences*, 16(9), 4508. https://doi.org/10.3390/app16094508
6. He, C., Balasubramanian, K., Ceyani, E., Yang, C., Xie, H., Sun, L., He, L., Yang, L., Yu, P. S., Rong, Y., Zhao, P., Huang, J., Annavaram, M., & Avestimehr, S. (2021). FedGraphNN: A federated learning benchmark system for graph neural networks. arXiv:2104.07145. https://arxiv.org/abs/2104.07145
7. Johannessen, F., & Jullum, M. (2023). Finding money launderers using heterogeneous graph neural networks. arXiv:2307.13499. https://arxiv.org/abs/2307.13499. Published in *Machine Learning with Applications* (2025); journal details not yet verified.
8. Lamptey, K. O., Ayekai, B. J., & Din, S. U. (2025). Federated learning on multilabel evolving data streams (FedMuL). *IEEE Internet of Things Journal*, 12(20). https://doi.org/10.1109/JIOT.2025.3592954
9. Li, E., Ouyang, J., Xiang, S., Qin, L., & Chen, L. (2025a). Efficient relation-aware heterogeneous graph neural network for fraud detection. *World Wide Web*, 28, 55. https://doi.org/10.1007/s11280-025-01369-5
10. Li, P., Yu, H., & Luo, X. (2025b). Context-aware graph neural network for graph-based fraud detection with extremely limited labels (CGNN). *AAAI-25*, 12112.
11. Liu, R., Xing, P., Deng, Z., Li, A., Guan, C., & Yu, H. (2025). Federated graph neural networks: Overview, techniques, and challenges. *IEEE TNNLS*, 36(3). https://doi.org/10.1109/TNNLS.2024.3360429
12. McMahan, B., Moore, E., Ramage, D., Hampson, S., & y Arcas, B. A. (2017). Communication-efficient learning of deep networks from decentralized data (FedAvg). *AISTATS*. Details from memory; not checked against the paper.
13. Motie, S., & Raahemi, B. (2023). Financial fraud detection using graph neural networks: A systematic review. *Expert Systems with Applications*, 240, 122156. Author attribution lower-confidence; verify before formal citation.
14. Park, D.-Y., Ko, I.-Y., Lee, T.-H., & Lee, J. (2025). Federated gradient boosting for financial fraud detection: An empirical study in the banking sector. *ACM CIKM*. https://doi.org/10.1145/3746252.3760891
15. Tan, Y., Liu, Y., Long, G., Jiang, J., Lu, Q., & Zhang, C. (2023). Federated learning on non-IID graphs via structural knowledge sharing (FedStar). *AAAI-23*, 9953.
16. Wang, H., Li, Z., Huang, J., Hui, P., Liu, W., Hu, T., & Chen, G. (2020). Collaboration based multi-label propagation for fraud detection (CMLP). *IJCAI-20*, 2477.
17. Xiang, S., Zhu, M., Cheng, D., Li, E., Zhao, R., Ouyang, Y., Chen, L., & Zheng, Y. (2023). Semi-supervised credit card fraud detection via attribute-driven graph representation. *AAAI-23*. https://doi.org/10.1609/aaai.v37i12.26702
18. Yuan, H., Tang, J., Hu, X., & Ji, S. (2020). XGNN: Towards model-level explanations of graph neural networks. *KDD '20*. https://doi.org/10.1145/3394486.3403085
19. Zheng, S., Liu, W., Ji, J., Wong, K.-C., Chen, J., & Lin, Q. (2026). Robust federated intrusion detection via prototype distillation (FedProD). *Knowledge-Based Systems*, 352, 117028. https://doi.org/10.1016/j.knosys.2026.117028
20. Zhu, Y., Hu, M., & Wu, D. (2025). Federated continual graph learning (POWER). *KDD '25*. https://doi.org/10.1145/3711896.3736956

Ingested in `Sources/`: [[fang-2026-fedgad-contrastive-federated-graph-anomaly]], [[tan-2023-fedstar-federated-graph-structural-knowledge]], [[altfaily-2025-fedgatsage-graph-federated-ids]], [[liu-2025-federated-gnn-survey]], [[zhu-2025-federated-continual-graph-learning]], [[he-2021-fedgraphnn-benchmark-gnn]], [[corcuera-barcena-2022-fed-xai-federated-explainable]], [[li-2025-cgnn-graph-fraud-extremely-limited-labels]], [[wang-2020-collaboration-multilabel-propagation-fraud]], [[lamptey-2025-federated-multilabel-evolving-streams]], [[chen-2025-fedspfd-federated-transfer-credit-card-fraud]], [[zheng-2026-fedprod-prototype-distillation-intrusion]], [[yuan-2020-xgnn-model-level-gnn-explanations]], [[mothukuri-2022-fl-anomaly-detection-iot]], [[preuveneers-2018-chained-anomaly-blockchain-fl]], [[sana-2025-advancing-fl-slr]], [[park-2025-federated-gradient-boosting-fraud]], [[ghourabi-khaldi-2026-federated-ensemble-fraud]], [[motie-raahemi-2023-gnn-systematic-review]], [[johannessen-jullum-2023-gnn-money-laundering]], [[xiang-2023-semi-supervised-graph-fraud]], [[li-2025-relation-aware-heterogeneous-gnn-fraud]].
