# Radar Phrase Extraction Dataset: Three Multi-Function Radar Emitters

This repository contains a **simulated dataset for radar phrase extraction** from intercepted Multi-Function Radar (MFR) pulse streams. Three distinct MFR emitters — ground-based, shipborne, and airborne — are modeled using the **syntactic modeling framework** [1, 2], where radar commands are expressed as *phrases* composed of sequences of atomic *words* (waveforms). The task is to recover phrase boundaries from continuous word-level observation sequences.

---

## 1. Background: MFR Hierarchical Architecture

Modern MFRs obey a " divide and conquer\ strategy to execute multiple tasks (search, track, acquisition, etc.) simultaneously. They consist of four core components [3, 4]:

| Component             | Role                                           |
| --------------------- | ---------------------------------------------- |
| **Radar Manager**     | Radar Clause: subtask generation               |
| **Command Scheduler** | Radar Phrase: scheduling                       |
| **Radar Controller**  | Radar Word: function generation                |
| **Radar Executor**    | Radar Pulse: pulse-stream physical realization |

The pulse generation process propagates through five hierarchical phases, where high-level tasks are progressively decomposed into concrete radar pulses.

---

## 2. Syntactic Model of MFRs

To formalize this hierarchy, Visnevski et al. [1] proposed modeling MFRs as **syntactic models** adapted from structural pattern recognition, introducing a linguistic analogy:

| Syntactic Unit               | MFR Equivalent  | Description                           |
| ---------------------------- | --------------- | ------------------------------------- |
| **Clause** ($\mathcal{C}$)   | Radar Task      | High-level mission objective          |
| **Phrase** ($\mathcal{P}$)   | Radar Command   | Ordered sequence of functions         |
| **Word** ($\mathcal{W}$)     | Radar Function  | Atomic waveform with fixed parameters |
| **Pulse** ($\mathcal{W}(t)$) | Physical Signal | Modulated RF emission                 |

The MFR is expressed as a four-tuple:


\mathcal{G} = \big( \{ \mathcal{D}_{\mathcal{W}},\ \mathcal{D}_{\mathcal{P}},\ \mathcal{D}_{\mathcal{C}} \},\ \mathcal{D}_{\mathcal{W}(t)},\ \{ \mathcal{S}_{\text{exe}},\ \mathcal{S}_{\text{ctrl}},\ \mathcal{S}_{\text{sche}},\ \mathcal{S}_{\text{man}} \},\ \mathcal{D}_{\mathcal{C}} \big)


Symbols propagate between adjacent layers via production rules:

| Rule                        | Mapping                          | Description                                     |
| --------------------------- | -------------------------------- | ----------------------------------------------- |
| $\mathcal{S}_{\text{man}}$  | $\mathcal{C} \to \mathcal{P}$    | Radar manager: task→command ordering            |
| $\mathcal{S}_{\text{sche}}$ | $\mathcal{P} \to \mathcal{P}$    | Scheduler: queue adaptation & preemption        |
| $\mathcal{S}_{\text{ctrl}}$ | $\mathcal{P} \to \mathcal{W}$    | Controller: template-based command→word mapping |
| $\mathcal{S}_{\text{exe}}$  | $\mathcal{W} \to \mathcal{W}(t)$ | Executor: word→physical pulse realization       |

---

## 3. Radar Phrase Generation: $\mathcal{S}_{\text{ctrl}}$

The **phrase extraction task** targets $\mathcal{S}_{\text{ctrl}}$ — the radar controller is production rules. Formally, $\mathcal{S}_{\text{ctrl}}$ is the finite set of composition rules:


\mathcal{S}_{\text{ctrl}} = \left\{ \mathcal{P} \to [\mathcal{W}_1,\ \mathcal{W}_2,\ \dots,\ \mathcal{W}_k] \mid \mathcal{P} \in \mathcal{D}_{\mathcal{P}},\ \mathcal{W}_i \in \mathcal{D}_{\mathcal{W}} \right\}


**In plain terms:** a radar phrase $\mathcal{P}$ is a sequence of $ radar words $[\mathcal{W}_1, \dots, \mathcal{W}_k]$. The composition rules are deterministic and table-driven — once you know the radar type and operating mode, the phrase-to-word mapping is fixed.

The goal of the phrase extraction task is to **segment a continuous word stream $\dots \mathcal{W}_i \mathcal{W}_{i+1} \dots$ into its constituent phrases $\mathcal{P}_1, \mathcal{P}_2, \dots$**, effectively recovering the underlying command structure from intercepted pulse data.

---

## 4. Dataset Overview

Three simulated MFR emitters are constructed across typical operational platforms, with a clear progression in grammatical complexity:

| Radar                       | Platform     | Word Vocab. | Phrase Dict. | Phrase Length | Complexity |
| --------------------------- | ------------ | ----------- | ------------ | ------------- | ---------- |
| **Radar A** (Mercury) [1]   | Ground-based | 9           | 42           | 4 (fixed)     | Low        |
| **Radar B** (Shipborne) [5] | Shipborne    | 7           | 3            | 3–4           | Medium     |
| **Radar C** (Airborne) [2]  | Airborne     | 18          | 1,015        | 2–8           | High       |

**Data generation:**

- **5,000** sequence samples per radar
- Train / Validation / Test split: **0.8 : 0.1 : 0.1**
- Each sequence contains a **random number of phrases** from 1 to {\max}=10$
- All sequences are annotated with ground-truth phrase boundaries

> **Why three radars?** Radar A tests basic boundary detection (fixed length). Radar B introduces interleaved heterogeneous phrases. Radar C stresses long-range context modeling with a massive dictionary and highly variable lengths.

---

## 5. Radar A — Ground-Based MFR (Mercury)

### 5.1 System Description

The **Mercury** radar [1] is a classic Canadian ground-based MFR. It operates with a vocabulary of **9 distinct radar words** ($\mathcal{W}_1 \sim \mathcal{W}_9$), all sharing a uniform pulse duration of **7.14 ms** and a **5-region word structure**.

### 5.2 Radar Words

All 9 words share identical basic parameters (the distinction lies in intra-pulse regions, not shown here for brevity).

### 5.3 Radar Phrases

The 9 words are composed into **42 phrases**, each containing exactly **4 words**. Phrases are organized into 5 operating modes:

| Mode                         | Sub-Mode           | Phrase ID                             | Pattern                                    |
| ---------------------------- | ------------------ | ------------------------------------- | ------------------------------------------ |
| **Search (S)**               | 4-Word Search (FS) | $\mathcal{P}_1$–$\mathcal{P}_4$       | Cyclic shifts of $[1,2,4,5]$               |
|                              | 3-Word Search (TS) | $\mathcal{P}_5$–$\mathcal{P}_7$       | $[1,3,5,1]$, $[3,5,1,3]$, $[5,1,3,5]$      |
| **Acquisition (Acq)**        | —                  | $\mathcal{P}_8$–$\mathcal{P}_{12}$    | $[x,x,x,x]$, \in \{1..5\}$                 |
| **Non-Adaptive Track (NAT)** | —                  | $\mathcal{P}_{13}$–$\mathcal{P}_{17}$ | $[x,6,6,6]$, \in \{1..5\}$                 |
| **Range Resolution (RR)**    | RR1–RR3            | $\mathcal{P}_{18}$–$\mathcal{P}_{20}$ | $[y,6,6,6]$, \in \{7,8,9\}$                |
| **Shared**                   | Acq / NAT / TTM    | $\mathcal{P}_{21}$                    | $[6,6,6,6]$                                |
| **Track Maintenance (TM)**   | 3-Word Track (TTM) | $\mathcal{P}_{22}$–$\mathcal{P}_{39}$ | $[x,y,y,y]$, \in \{1..6\}$, \in \{7,8,9\}$ |
|                              | 4-Word Track (FTM) | $\mathcal{P}_{40}$–$\mathcal{P}_{42}$ | $[y,y,y,y]$, \in \{7,8,9\}$                |

> **Key property:** All phrases are uniformly 4 words long. Different modes may share the same phrase (e.g., $\mathcal{P}_{21} = [6,6,6,6]$ serves Acq, NAT, and TTM). This **rigid, fixed-length grammar** makes Radar A the simplest test case.

---

## 6. Radar B — Shipborne MFR

### 6.1 System Description

This radar models a representative naval MFR based on parameters from [5]. It operates at a **fixed carrier frequency of 3 GHz** with a compact vocabulary of **7 distinct words** ($\mathcal{W}_1 \sim \mathcal{W}_7$), consolidated by merging words with identical waveform parameters across all operational modes.

### 6.2 Radar Words

| Word            | Category | RF (GHz) | PRI (μs) | BW (MHz) | PW (μs) |
| --------------- | -------- | -------- | -------- | -------- | ------- |
| $\mathcal{W}_1$ | LPRF     | 3        | 2,000    | 5        | 51      |
| $\mathcal{W}_2$ | LPRF     | 3        | 2,000    | 5        | 25      |
| $\mathcal{W}_3$ | LPRF     | 3        | 2,000    | 5        | 12.5    |
| $\mathcal{W}_4$ | MPRF     | 3        | 500      | 10       | 20      |
| $\mathcal{W}_5$ | MPRF     | 3        | 333      | 10       | 20      |
| $\mathcal{W}_6$ | MPRF     | 3        | 200      | 10       | 20      |
| $\mathcal{W}_7$ | MPRF     | 3        | 142      | 10       | 20      |

Words fall into two categories:

- **LPRF** ($\mathcal{W}_1$–$\mathcal{W}_3$): low PRF (500 Hz), narrow BW (5 MHz), group-switched PW (51 / 25 / 12.5 μs)
- **MPRF** ($\mathcal{W}_4$–$\mathcal{W}_7$): high staggered PRI (500–7,042 Hz), wide BW (10 MHz), fixed PW (20 μs)

**Word boundary cue:** adjacent words are distinguished by jumps in either PW or PRI.

### 6.3 Radar Phrases

| Phrase          | Mode   | Word Sequence                                                | Length |
| --------------- | ------ | ------------------------------------------------------------ | ------ |
| $\mathcal{P}_1$ | Search | $[\mathcal{W}_1, \mathcal{W}_2, \mathcal{W}_3]$              | 3      |
| $\mathcal{P}_2$ | Track  | $[\mathcal{W}_4, \mathcal{W}_5, \mathcal{W}_6, \mathcal{W}_7]$ | 4      |
| $\mathcal{P}_3$ | Track  | $[\mathcal{W}_4, \mathcal{W}_5, \mathcal{W}_7]$              | 3      |

> **Key property:** Only 3 phrases but with **variable lengths** (3 or 4 words). The search phrase completes one PW group-switching cycle; the track phrases complete PRI staggering cycles. **Heterogeneous phrase interleaving** within the same mode increases decoupling difficulty.

---

## 7. Radar C — Airborne MFR (Apfeld)

### 7.1 System Description

This agile airborne MFR is based on the hierarchical linguistic framework of Apfeld et al. [2]. Unlike legacy radars, this system **adaptively selects waveform parameters** based on tactical demands. All words share a **fixed RF of 10 GHz**; each word is uniquely determined by its **PRF and PW**.

### 7.2 Radar Words

The vocabulary contains **18 words**, strictly partitioned by PRF regime:

| Word               | Category | PRF (Hz) | PW (μs) |
| ------------------ | -------- | -------- | ------- |
| $\mathcal{W}_1$    | MPRF     | 8,880    | 12.387  |
| $\mathcal{W}_2$    | MPRF     | 10,850   | 10.138  |
| $\mathcal{W}_3$    | MPRF     | 12,040   | 9.136   |
| $\mathcal{W}_4$    | MPRF     | 12,820   | 8.580   |
| $\mathcal{W}_5$    | MPRF     | 14,110   | 7.795   |
| $\mathcal{W}_6$    | MPRF     | 14,800   | 7.432   |
| $\mathcal{W}_7$    | MPRF     | 15,980   | 6.883   |
| $\mathcal{W}_8$    | MPRF     | 16,770   | 6.559   |
| $\mathcal{W}_9$    | HPRF     | 75,420   | 1.201   |
| $\mathcal{W}_{10}$ | HPRF     | 77,930   | 1.201   |
| $\mathcal{W}_{11}$ | HPRF     | 79,310   | 1.201   |
| $\mathcal{W}_{12}$ | HPRF     | 82,210   | 1.201   |
| $\mathcal{W}_{13}$ | HPRF     | 86,150   | 1.201   |
| $\mathcal{W}_{14}$ | HPRF     | 90,200   | 1.201   |
| $\mathcal{W}_{15}$ | HPRF     | 96,200   | 1.201   |
| $\mathcal{W}_{16}$ | HPRF     | 98,512   | 1.201   |
| $\mathcal{W}_{17}$ | HPRF     | 100,120  | 1.201   |
| $\mathcal{W}_{18}$ | HPRF     | 105,380  | 1.201   |

- **MPRF subset** ($\mathcal{W}_1$–$\mathcal{W}_8$): 8 words, PRF 8,880–16,770 Hz, PW 12.4–6.6 μs
- **HPRF subset** ($\mathcal{W}_9$–$\mathcal{W}_{18}$): 10 words, PRF 75,420–105,380 Hz, fixed PW 1.201 μs

### 7.3 Radar Phrases

A total of **1,015 phrases** are organized into 10 structural modes across MPRF and HPRF regimes:

#### MPRF Modes (247 phrases)

| Mode    | $    | Combinations        | Phrases | Construction                  |
| ------- | ---- | ------------------- | ------- | ----------------------------- |
| Track   | 2    | $\binom{8}{2} = 28$ | 28      | Distinct words, ascending PRF |
| Track   | 3    | $\binom{8}{3} = 56$ | 56      | Distinct words, ascending PRF |
| Confirm | 4    | $\binom{8}{4} = 70$ | 70      | Distinct words, ascending PRF |
| Track   | 5    | $\binom{8}{5} = 56$ | 56      | Distinct words, ascending PRF |
| Track   | 6    | $\binom{8}{6} = 28$ | 28      | Distinct words, ascending PRF |
| Track   | 7    | $\binom{8}{7} = 8$  | 8       | Distinct words, ascending PRF |
| Search  | 8    | $\binom{8}{8} = 1$  | 1       | All 8 words                   |

MPRF phrases use **distinct words only**, arranged in ascending PRF order (no repetitions, no permutations).

#### HPRF Modes (768 phrases)

| Mode    | $    | Combinations                                          | Phrases | Construction                             |
| ------- | ---- | ----------------------------------------------------- | ------- | ---------------------------------------- |
| Track   | 2    | \times 10 = 100$                                      | 100     | Any word pair (repetition allowed)       |
| Track   | 3    | $\binom{10}{1} + A_9^2 = 10 + 72 = 82$                | 82      | 1 anchor + 2 distinct others OR all-same |
| Confirm | 4    | $\binom{10}{1} + A_9^2 + A_9^3 = 10 + 72 + 504 = 586$ | 586     | 1 anchor + up to 3 distinct others       |

HPRF phrases use a **baseline anchor word** (typically $\mathcal{W}_9$) with permutations of remaining words — dramatically expanding the combinatorial space.

---

## 8. Citation

| Ref  | Source                                                       |
| ---- | ------------------------------------------------------------ |
| [1]  | N. A. Visnevski, Syntactic modeling of multi-function radars, Ph.D. dissertation, McMaster University, 2005. |
| [2]  | S. Apfeld and D. Heberling,  Machine learning for electronic intelligence, Ph.D. dissertation, RWTH Aachen University, 2021. |
| [3]  | A. Wang and V. Krishnamurthy,  Signal interpretation of multifunction radars: Modeling and statistical signal processing with stochastic context free grammar, *IEEE Trans. Signal Process.*, vol. 56, no. 3, pp. 1106–1119, 2008. |
| [4]  | Y. Zhao, X. Wang, and Z. Huang,  Multi-function radar modeling: A review, *IEEE Sensors J.*, vol. 24, no. 20, pp. 31658–31680, 2024. |
| [5]  | T. Tian et al.,  Shipborne multi-function radar working mode recognition based on DP-ATCN, *Remote Sensing*, vol. 15, no. 13, p. 3415, 2023. |

