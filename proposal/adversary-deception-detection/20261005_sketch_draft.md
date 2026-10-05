---
type: proposal
source: "[[research-proposal-log]]"
created: 2026-10-05
---

# Sketch — Real-Time Harm-Intent Detection Against Adaptive Attackers

Direction sketch (Edge 1: Contact → Persuasion). Reframes the earlier draft in this folder (`20261003_sketch_draft.md`) around one core idea. Written before supervisor outreach; supervisor-specific framing belongs in the cover email, not here.

Citations are author-year. Full references are at the end. Source summaries are in `Sources/`.

## Problem

Scam and impersonation attacks now run as live conversations over chat, phone, and video call, with LLM-written text, cloned voices, and deepfake video. Detectors that look at wording or at whether media is synthetic can be evaded by an attacker who rewrites the message or uses clean media. AI generation and fraudulent intent are also different things: benign content can be synthetic, and a scam need not be (Zhao et al., 2026).

One part of the attack cannot be removed. To succeed, the attacker must steer the victim to a harmful action: pay, disclose credentials, install software, or move to an unmonitored channel. This sketch calls that the **harmful ask**.

The problem: **detect, turn by turn, that a live interaction is moving toward a harmful ask, early enough to intervene, and stay accurate when the attacker adapts to the detector.**

Four open problems for this setting:

1. **Adaptive attackers.** Existing real-time detectors are tested against fixed or scripted attackers. Adversarial evaluation is listed as future work (Zhao et al., 2026; Hossain et al., 2026). An attacker that rewrites its message from detector feedback exists for single emails only (Qi et al., 2025).
2. **Earliness.** Detection is only useful before the victim acts. Most detectors are scored on whole conversations or single items, not on how many turns they need. Turn-by-turn detection on phone calls keeps recall but loses precision (Shen et al., 2025).
3. **Many inputs.** Chat carries text only, a phone call adds speech, and a video call adds video. Multimodal scam detectors score independent items, not one live interaction (Zhao et al., 2026), and real-time detectors are text-only (Hossain et al., 2026; Tewari et al., 2026).
4. **False-positive constraint.** A detector that warns too often will be ignored or switched off, so gains in earliness and robustness only count at a fixed, low false-alarm rate. Both tend to raise false alarms: deciding earlier lowers precision (Shen et al., 2025), and catching more manipulation raises false positives (Ma et al., 2025). The hardest benign cases are legitimate asks, since real banks and employers also request payments and identity details.

## Research Questions

1. **Detection.** Can a live interaction be flagged as moving toward a harmful ask early enough to intervene, while keeping false alarms low?
   - 1a. How many turns does it need, compared with conversation-level detectors, at the same false-alarm rate?
   - 1b. Does the false-alarm rate hold on hard benign cases such as legitimate requests for payment or identity?
2. **Robustness.** How well does the detector hold up against an attacker that adapts to it, and can training against that attacker help?
   - 2a. How much can an adaptive attacker evade the detector while the attack still works?
   - 2b. Does training against one attacker reduce evasion by an unseen attacker, and at what false-positive cost?
   - 2c. Does robustness hold on tactics and scenarios not seen in training?
3. **Inputs.** What do speech, video, and affective cues add over text, and at what cost?
   - 3a. Do they improve earliness or robustness?
   - 3b. What do they cost in latency, and does the detector still work when an input is missing?

**Proposed scope:** RQ2 is the core contribution. RQ1 is the precondition. RQ3 is studied through method variants and may be a separate paper. This needs a supervisor decision.

## Literature Review

### Closest prior work

| Work | What it does | Overlap with this sketch |
|---|---|---|
| Tewari et al. (2026), PhishSim and PhishGate | LLM phishing simulator that counts compromise only when the simulated victim completes an off-chat action; real-time multi-agent risk scoring | **Direct overlap** on real-time scoring and outcome-verified attacks. Text-only, one recruiter scenario, scripted attacker. |
| Hossain et al. (2026), AI-in-the-loop | Per-message and running scam-risk score, LLM scambaiting, federated training | Overlaps real-time scoring. Text-only, largely synthetic data; adaptive adversary simulation is future work. |
| Shen et al. (2025) | Prompted LLM judges a phone call after every turn from the speech transcript; optional UNCERTAIN label | **Direct overlap** on earliness and on the false-positive constraint. Transcript only, Chinese, no trained detector, attacker does not adapt. |
| Qi et al. (2025), SpearBot | LLM writes a spear-phishing email, LLM critics say why it looks like phishing, the email is rewritten until no critic flags it | **Direct overlap** on the adaptive attacker. Single emails, no interaction, no outcome check, no detector trained against it. |
| Ai et al. (2024), SEConvo and ConvoSentinel | LLM-generated social-engineering dialogues; message-level detection of sensitive-information requests, retrieval, then conversation-level decision | Overlaps the ask-based signal and early detection. Text-only; attacker does not adapt. |
| Zhao et al. (2026), Check-Agent | LLM agents coordinate per-modality expert detectors over text, audio, image, and video | Overlaps multimodal input. Item-level, in-distribution, no adversarial test. |
| Han et al. (2025), ScamGen | Template generation of tele-scam dialogues from WHO, WHY, and WHAT (the action demanded) | Supplies a taxonomy of asks and tactics. Chinese, text-only, offline. |
| Ma et al. (2025), intent-aware prompting | Summarises each speaker's intent, then detects manipulation | Overlaps intent as an intermediate step. Fictional dialogues, one LLM. |
| Dinges et al. (2024); Nam et al. (2023); Joshi et al. (2025) | Deception detection from facial, gaze, emotion, and physiological cues | Overlaps the affective-cue variant. Staged lies, small samples, offline. |
| Arora et al. (2024) | Cheap encoder first, LLM only on uncertain inputs | Design pattern for latency. Customer-support intents, not fraud. |

### Key findings

**PhishSim and PhishGate (Tewari et al., 2026).**

- *Method.* An LLM attacker poses as a recruiter. An LLM victim, conditioned on one of 192 personas, counts as compromised only if it submits credentials off-chat. PhishGate scores risk per turn with a tactic agent and a severity agent, and the running risk never decreases (R_t = max(R_{t−1}, Δ_t)).
- *Results.* 1,833 conversations with a 57.7% verified compromise rate among phishing dialogues. PhishGate takes about 1.58 s to 2.68 s per turn depending on the LLM backend, and flags risk before the link appears. Masking the link lowers F1 from 0.861 to 0.851 on the best backend, and more on weaker ones.
- *Limitations.* One scenario and a staged attacker prompt. The paper says simulated victims may not match real users and that it does not model how warnings are shown.

**ConvoSentinel (Ai et al., 2024).** 1,400 GPT-4-Turbo conversations, 400 annotated (Fleiss' kappa 0.63). 80% of annotated conversations contain a sensitive-information request, usually early. ConvoSentinel reaches F1 0.74 after five messages and 0.80 overall, against 0.78 for two-shot GPT-4-Turbo. Training data is 40 conversations, and test data comes from the same generator.

**Check-Agent (Zhao et al., 2026).** At least 84.05% accuracy on each modality and 99.15% on text. Latency is 5.36 s for text, 15.55 s for audio, and 88.60 s for video per sample. The authors state the results are in-distribution only and that the dataset has no adversarial, cross-platform, or cross-lingual content. Data is available on request.

**Real-time phone-scam detection (Shen et al., 2025).**

- *Method.* After each turn a prompted LLM labels the call FRAUD or SAFE from the transcript so far. Five LLMs, Chinese authentic and synthetic call transcripts; dataset sizes are not reported.
- *Results.* Real-time recall is 0.98–1.00. Precision on the synthetic set falls to 0.70–0.77 for most models, against 0.95–1.00 when the whole call is judged afterwards. False positives are benign calls with identity checks, payments, or urgent decisions. An UNCERTAIN label raises precision but lowers recall to 0.90–0.98 and delays the alert; in one example from utterance 6 to utterance 10, when the caller explicitly asks for ID card information.
- *Limitations.* No acoustic or video input, no adaptive attacker, no trained component, no latency figures.

**SpearBot (Qi et al., 2025).**

- *Method.* GPT-4 generates an email; GPT-4, Claude-3-Sonnet, and gpt-3.5-turbo act as critics; the email is rewritten from their reasons for up to 10 rounds.
- *Results.* On 1,000 generated emails, XGBoost detects 21.70% (97.80% on older datasets) and pre-trained language models 1.00–3.00%. GPT-4 defenders detect 21.30–45.00%. Detection by the best defender is 70.3% without critics and 45.00% with all three. Cost is about $0.13–0.15 per email.
- *Limitations.* Critics and the strongest defenders are the same model family. Defenders are never retrained on SpearBot emails. Deception is rated by people, not verified by a completed action. Emails asking to confirm personal information were the hardest for one GPT-4 defender (17% detected) and the easiest for another (56%).

**AI-in-the-loop (Hossain et al., 2026).** A formal threat model with a risk score at each timestep and a detection threshold. Baselines reach F1 near 0.99 on the synthetic sets, which suggests those sets are easy.

**Deception-cue papers.** Dinges et al. (2024) report about 63–68% accuracy on low-stakes lies and cross-dataset transfer that is often worse than chance; emotion cues were the most useful. Nam et al. (2023) reach 70.79% on an interrogation-protocol set. Joshi et al. (2025) reach 75–79% with audio, video, gaze, and skin-response fusion. Constâncio et al. (2023) find that about 70% of the data in this field is mock or laboratory data.

**Other sources.**

- *Ma et al. (2025).* Reducing false negatives by 30.5% raised false positives by 14.6%.
- *Wang et al. (2024), MentalManip.* GPT-4 Turbo labelled 312 of 899 non-manipulative dialogues as manipulative.
- *Han et al. (2025).* Template generation matched the target category in 100 of 100 samples; prompted LLMs scored 82 to 96.
- *Zhi et al. (2025), D-STAR.* Scam-call transcript classifier; 800 transcripts, partly ChatGPT-augmented, no live test.
- *Jabir et al. (2025).* Review of human factors in GenAI-enabled phishing; the source through which SpearBot was found.
- *Zhou et al. (2026).* Scam typology: lure of gain and fear of loss, with urgency, authority, and fear as triggers.
- *Zhang et al. (2026).* Identity-fraud review; recommends adversarial training and names scam detection as its least-covered use case.

### Gap status

Based on source summaries, with full text read for Ai et al. (2024), Shen et al. (2025), and Qi et al. (2025), and selected sections of Zhao et al. (2026), Hossain et al. (2026), and Tewari et al. (2026). A full literature search is still needed before any novelty claim.

| Gap | Status | Closest evidence |
|---|---|---|
| 1. Adaptive attackers | **Partly covered** for single emails. Open for live interaction, for outcome-verified evasion, and for a detector trained against the attacker. | Qi et al. (2025); Tewari et al. (2026) |
| 2. Earliness | **Partly covered** for chat text and call transcripts. | Shen et al. (2025); Ai et al. (2024); Tewari et al. (2026) |
| 3. Many inputs | **Partly covered** at item level, not for a live interaction. | Zhao et al. (2026) |
| 4. False-positive constraint | **Problem shown, not solved.** False alarms rise with earlier decisions and with more sensitive detection. The only remedy tested is an UNCERTAIN label, which costs recall and time. No work reports robustness to an attacker at a fixed false-alarm rate. | Shen et al. (2025); Ma et al. (2025); Wang et al. (2024) |

**Main finding.** Real-time text detection of social engineering already exists, including outcome-verified simulation. A feedback-driven attacker also exists, but only for single emails and without a check that the attack still works. The sketch's novelty must come from combining the two: an adaptive attacker in a live interaction, evasion measured only when the attack still works, and a detector trained against that attacker.

## Candidate Method

**Core: ask-trajectory detector.** At each turn the detector estimates (a) what is being asked of the target, (b) how harmful that action would be, and (c) how far the conversation has moved toward it. These feed a running risk score with an alert threshold.

**Latency.** A small encoder scores every turn; only uncertain turns go to an LLM (Arora et al., 2024).

**Adaptive attacker.** An LLM attacker rewrites or re-plans its dialogue using limited feedback from the detector, such as flagged or not flagged. This extends the critic loop of Qi et al. (2025) from one email to a multi-turn interaction. Attacker and detector are trained in alternating rounds.

**Input variants.** The same detector is built in four variants, so each input's contribution can be measured:

| Variant | Inputs | What it adds |
|---|---|---|
| V1 | Text (chat, or speech transcript) | Baseline; all closest prior work is here |
| V2 | V1 + speech | Prosody and pacing, and synthetic-voice signals |
| V3 | V2 + video | Synthetic-face signals and cross-modal consistency, following Zhao et al. (2026) |
| V4 | Any of V1–V3 + affective cues | Attacker-side pressure (urgency, fear appeals) and target-side stress or hesitation |

- **Fusion.** Late fusion that tolerates missing inputs, since a chat has no audio and a phone call has no video.
- **Affective cues (V4).** Treated as an auxiliary signal, not a lie detector. The evidence for facial deception cues is weak outside staged settings, and a cloned voice or deepfake face carries no genuine cues from the attacker.
- **Synthetic-media signals.** Kept as a separate output from harm intent, as in Zhao et al. (2026).

**Other method choices:**

- **Threat model.** Black-box attacker with a limited number of queries. Text adaptation is the core; speech and video attacks use existing synthetic media, not a new generator.
- **Training against the attacker.** For LLM components this means fine-tuning or updating retrieved examples with attacker outputs, not gradient-based adversarial training.

## Data

No dataset has live, multimodal scam interactions with turn-level labels. Candidates:

- Ai et al. (2024), SEConvo: LLM-generated, English, text.
- Tewari et al. (2026), PhishSim: outcome-labelled, text, one scenario.
- Han et al. (2025), ScamGen: 89,594 Chinese tele-scam dialogues; its taxonomy can define held-out tactics.
- Scam-bait corpora used by Hossain et al. (2026): closest to real scammer text; quality not yet checked.
- Shen et al. (2025): Chinese phone-call transcripts, authentic and synthetic, described as open-source; sizes and licence not yet checked.
- Zhao et al. (2026): multimodal but item-level and on request only.

Simulated dialogues fill the gaps, with speech and video rendered from text scripts for V2 and V3. Test data must come from a different generator than training data. Real conversation data needs ethics approval and a data-access agreement.

## Evaluation

- **Baselines.** ConvoSentinel (Ai et al., 2024), PhishGate (Tewari et al., 2026), the running risk score of Hossain et al. (2026), a per-turn prompted LLM (Shen et al., 2025), and a whole-conversation LLM classifier. Check-Agent (Zhao et al., 2026) only if it can be reproduced.
- **Metrics.** Time-to-detection in turns; recall at a fixed low false-alarm rate; latency per turn. Accuracy is not a headline metric.
- **Detection (RQ1).** Fix the false-alarm rate first, then compare time-to-detection at that rate. Measure the rate on a hard benign set built around legitimate requests for payment or identity, following the failure cases in Shen et al. (2025), not only on ordinary benign conversations.
- **Robustness (RQ2).**
  - *Evasion.* An evasion counts only if the rewritten attack still leads to the harmful action. Follow the outcome check of Tewari et al. (2026): a simulated victim must complete the action. Validate a sample with human raters.
  - *Robust training.* Train against one attacker model, test against another. Report false positives on benign conversations before and after.
  - *Held-out tactics.* Leave one tactic or scenario out, using the ScamGen or Zhou et al. (2026) categories.
- **Inputs (RQ3).** Run V1 to V4 on the same interactions, including with an input removed. Report the gain per input and the latency cost.
- **Sim-to-real.** Compare results on simulated data with results on scam-bait transcripts.

## Risks

- **PhishGate overlap.** Tewari et al. (2026) already do real-time, outcome-verified detection in text. The contribution must be defined against it; the adaptive attacker is the main difference, and Qi et al. (2025) already show the idea for email.
- **Attacker and detector too alike.** In Qi et al. (2025) the critics and the best defenders are the same model family. Evasion measured against a detector similar to the attacker's own critic may not transfer, in either direction.
- **Judging a working attack.** A simulated victim may not behave like a real one (Tewari et al., 2026). The evasion metric depends on this proxy.
- **Patient attackers.** An attacker can delay the ask. The detector may only catch these late.
- **Rendered speech and video.** Media rendered from scripts may not match real scam calls, so V2 and V3 results may not transfer.
- **Affective cues.** Evidence comes from staged lies and transfers poorly (Dinges et al., 2024). Target-side sensing raises consent and fairness concerns. V4 may show no gain.
- **Video latency.** Check-Agent needs 88.60 s per video sample, which is far from real time.
- **False-positive constraint too tight.** At a low fixed false-alarm rate the detector may show little gain in earliness or robustness, since both have come with more false positives in prior work (Shen et al., 2025; Ma et al., 2025).
- **Easy synthetic data.** Near-perfect scores on synthetic sets (Hossain et al., 2026) can hide weak generalisation.
- **Overfitting to the attacker.** Robustness to one's own attacker may not hold for others.
- **Scope.** Three RQs, with an adaptive-attacker loop and four input variants, may be more than one project. Needs a supervisor decision.
- **Dual use.** An adaptive scam attacker is itself harmful; release needs a restricted protocol.

## Supervisor Fit

Mapped from the cluster descriptions in [[research-proposal-log]]. Check each supervisor's recent papers before pitching.

| Priority | Supervisor | Fit | Caution |
|---|---|---|---|
| 1 | Nalin Arachchilage (RMIT) | Co-author of Tewari et al. (2026), the closest prior work; phishing and scam defence | Less focus on speech and video. |
| 2 | Thanh Thi Nguyen (USC) | Deepfake and LLM-generated content detection; fits V2 and V3 | Supervision capacity unconfirmed after recent move. |
| 3 | Sabrina Caldwell (UNSW Canberra) | Media authenticity and deception detection | Recent institutional move to confirm. |
| 4 | Roland Goecke (UNSW Canberra) | Affective computing and multimodal fusion; fits V4 | Co-supervision option with Caldwell. |
| 5 | Campbell Wilson (Monash/AiLECS) | Threat detection from text under class imbalance and evasion; his group's papers on hate-speech and life-threatening-text detection are close analogues for detecting harmful intent | Current projects are online-exploitation and forensics, not financial. No speech or video-call work found in his five most recent papers. |
| Option | Bo Liu (UTS) | AI security; fits the adaptive-attacker component. Co-author of Zhang et al. (2026) | Industry link unconfirmed. |
| Option | Ling Chen (UTS) | Co-author of Ma et al. (2025), intent-aware manipulation detection; LLM-guardrail and safe-agent work fits the intent-detection core | Main fraud work is graph-based and centralised, not conversational. Already first choice for the federated graph sketch. |
| Option | Monica Whitty (Monash) | Scam victimology and persuasion and deception theory; her hyperpersonal-AI work on how deepfakes raise trust fits the sketch's deepfake-call setting and the target-side affective cues in V4 | Psychology-led with a strict social-science qualification bar; the detector and adaptive-attacker work would need co-supervision from a technical supervisor. |

## References

Full texts are in `Sources/pdfs/` unless marked. **[Background only]** marks a reference cited only in the literature review; it is not used in the problem, method, data, evaluation, or risks.

1. Ai, L., Kumarage, T., Bhattacharjee, A., Liu, Z., Hui, Z., Davinroy, M., Cook, J., Cassani, L., Trapeznikov, K., Kirchner, M., Basharat, A., Hoogs, A., Garland, J., Liu, H., & Hirschberg, J. (2024). Defending against social engineering attacks in the age of LLMs. arXiv:2406.12263. https://arxiv.org/abs/2406.12263
2. Arora, G., Jain, S., & Merugu, S. (2024). Intent detection in the age of LLMs. *EMNLP 2024 Industry Track*, 1559–1570. https://aclanthology.org/2024.emnlp-industry.114 (URL derived from the paper ID).
3. Constâncio, A. S., Tsunoda, D. F., Silva, H. de F. N., Silveira, J. M. da, & Carvalho, D. R. (2023). Deception detection with machine learning: A systematic review and statistical analysis. *PLOS ONE*, 18(2), e0281323. https://doi.org/10.1371/journal.pone.0281323 **[Background only]**
4. Dinges, L., Fiedler, M.-A., Al-Hamadi, A., Hempel, T., Abdelrahman, A., Weimann, J., Bershadskyy, D., & Steiner, J. (2024). Exploring facial cues: Automated deception detection using artificial intelligence. *Neural Computing and Applications*, 36, 14857–14883. https://doi.org/10.1007/s00521-024-09811-x
5. Han, X., Li, Q., Qi, Y., Cao, H., Pedrycz, W., & Wang, W. (2025). ScamGen: Unveiling psychological patterns in tele-scam through advanced template-augmented corpus generation. *Computers in Human Behavior*, 162, 108451. https://doi.org/10.1016/j.chb.2024.108451
6. Hossain, I., Puppala, S., Alam, M. J., & Talukder, S. (2026). AI-in-the-loop: Privacy preserving real-time scam detection and conversational scambaiting by leveraging LLMs and federated learning. arXiv:2509.05362. https://arxiv.org/abs/2509.05362
7. Jabir, R., Le, J., & Nguyen, C. (2025). Phishing attacks in the age of generative artificial intelligence: A systematic review of human factors. *AI*, 6, 174. https://doi.org/10.3390/ai6080174 **[Background only]**
8. Joshi, G., et al. (2025). Multimodal machine learning for deception detection using behavioral and physiological data. *Scientific Reports*, 15, 8943. https://doi.org/10.1038/s41598-025-92399-6 **[Background only]**
9. Ma, J., Na, H., Wang, Z., Hua, Y., Liu, Y., Wang, W., & Chen, L. (2025). Detecting conversational mental manipulation with intent-aware prompting. *COLING 2025*, 9176–9183. https://aclanthology.org/2025.coling-main.616/ (URL derived from the paper ID).
10. Nam, B., Kim, J. Y., Bark, B., Kim, Y., Kim, J., So, S. W., Choi, H. Y., & Kim, I. Y. (2023). FacialCueNet: Unmasking deception, an interpretable model for criminal interrogation using facial expressions. *Applied Intelligence*, 53, 27413–27427. https://doi.org/10.1007/s10489-023-04968-9 **[Background only]**
11. Qi, Q., Luo, Y., Xu, Y., Guo, W., & Fang, Y. (2025). SpearBot: Leveraging large language models in a generative-critique framework for spear-phishing email generation. arXiv:2412.11109. https://arxiv.org/abs/2412.11109 (arXiv version read; Jabir et al. (2025) list a journal version in *Information Fusion*, 122, 103176, not opened).
12. Shen, Z., Yan, S., Zhang, Y., Luo, X., Ngai, G., & Fu, E. Y. (2025). "It warned me just at the right moment": Exploring LLM-based real-time detection of phone scams. arXiv:2502.03964. https://arxiv.org/abs/2502.03964
13. Tewari, T., Arachchilage, N., Challa, J. S., & Kumar, D. (2026). From trust to compromise: Outcome-verified LLM phishing simulation and real-time defense. *ACL 2026*, 11831–11845. https://aclanthology.org/2026.acl-long.543.pdf
14. Wang, Y., Yang, I., Hassanpour, S., & Vosoughi, S. (2024). MentalManip: A dataset for fine-grained analysis of mental manipulation in conversations. *ACL 2024*, 3747–3764. https://aclanthology.org/2024.acl-long.206 (URL derived from the paper ID). **[Background only]**
15. Zhang, C., Gill, A. Q., Liu, B., & Anwar, M. (2026). AI-based identity fraud detection: A systematic review. *Applied Artificial Intelligence*, 40(1), e2731995. https://doi.org/10.1080/08839514.2026.2731995 **[Background only]**
16. Zhao, Y., Xu, Z., & Wu, X. (2026). Check-Agent: Multimodal fraud detection via a multi-agent framework. *Knowledge-Based Systems*, 350, 116511. https://doi.org/10.1016/j.knosys.2026.116511
17. Zhi, B. H. J., Connie, T., Ong, T. S., & Teoh, A. B. J. (2025). Classifying scam calls through content analysis with dynamic sparsity top-k attention regularization. *IEEE Access*, 13, 111733–111750. https://doi.org/10.1109/ACCESS.2025.3582906 **[Background only]**
18. Zhou, S., Liu, X. F., Wang, X., Zhang, X., Lin, F., Nah, F. F.-H., Jiang, L. C., Zhang, R., Zhi, P., Ai, Y., & Gong, J. (2026). Beyond deception: Fighting scams in an age of digital inauthenticity. *Journal of Information Technology Case and Application Research*. https://doi.org/10.1080/15228053.2026.2724564

Ingested in `Sources/`: [[tewari-2026-phishsim-outcome-verified-phishing]], [[hossain-2026-ai-in-the-loop-scam-scambaiting-fl]], [[ai-2024-seconvo-convosentinel]], [[han-2025-scamgen-tele-scam-corpus-generation]], [[ma-2025-intent-aware-prompting-mental-manipulation]], [[wang-2024-mentalmanip-mental-manipulation-dataset]], [[arora-2024-intent-detection-age-of-llms]], [[zhi-2025-d-star-scam-call-sparse-attention]], [[dinges-2024-facial-cues-automated-deception-detection]], [[nam-2023-facialcuenet-interpretable-deception-facial]], [[joshi-2025-cognimodal-d-multimodal-deception-detection]], [[constancio-2023-deception-detection-ml-systematic-review]], [[jabir-2025-phishing-genai-human-factors-review]], [[zhou-2026-beyond-deception-fighting-scams-digital-inauthenticity]], [[zhang-2026-ai-identity-fraud-detection-slr]], [[shen-2025-llm-real-time-phone-scam-detection]], [[qi-2025-spearbot-generative-critique-spear-phishing]]. Topic note: [[academic-zhao-2026-check-agent-multimodal-fraud-multiagent]].

Read for the earlier draft and not cited here: [[xia-2018-zero-shot-intent-capsule-networks]] (benign text intents), [[deepfake-apac-banks-fintechnews]] (industry context).
