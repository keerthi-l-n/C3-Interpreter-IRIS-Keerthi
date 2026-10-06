# C3 second-tier interpreter

**A calculator to separate propionic acidemia, methylmalonic acidemia and B12-related causes after a raised C3 newborn screen**

Keerthi Lakshmi Narayanan : Submission for IRIS National Fair 2026–27 - Computational Biology & Bioinformatics

**▶ [Open the calculator](https://<USERNAME>.github.io/c3-ratio-pa-mma/)**

>**This is a research prototype. Not a medical device.** This page must not be used to diagnose or treat any patient. A diagnosis needs confirmatory testing by a metabolic specialist.
>**NOTE REGARDING USAGE OF AI**: While the research question, data extraction, decision path,the actual coding in the Python notebooks that analyzed patient data, and all other research and findings is done by me, I used Claude for the HTML code of this webpage (web calculator), but the content, design and decision flow is by me. 

---

## Contents

1. [Background](#background)
2. [What the calculator does](#what-the-calculator-does)
3. [How to use it](#how-to-use-it)
4. [The decision path](#the-decision-path)
5. [Where the ratio comes from](#where-the-ratio-comes-from)
6. [How the cut-offs were checked](#how-the-cut-offs-were-checked)
7. [Limitations](#limitations)
8. [Running the page yourself](#running-the-page-yourself)
9. [Sources](#sources)
10. [License](#license)

---

## Background

Most countries screen newborns for rare inherited metabolic disorders from a few drops of blood on a card (a dried blood spot). One marker on this screen is **propionylcarnitine (C3)**. A raised C3 can mean several different conditions:

| Condition | What goes wrong |
|---|---|
| **Propionic acidemia (PA)** | The enzyme propionyl-CoA carboxylase does not work, so propionyl-CoA builds up. |
| **Methylmalonic acidemia (MMA)** | The enzyme methylmalonyl-CoA mutase (or a cofactor pathway such as CblA, CblB or CblC) does not work, so methylmalonyl-CoA builds up. |
| **B12-related causes** | Low vitamin B12 in the baby, often passed on from a mother with low B12, or a cobalamin disorder such as CblC. B12 is needed by the mutase enzyme and by methionine synthase. |

These conditions need different treatment, so it matters which one a baby has. After a raised C3, many labs run **second-tier tests** on the same blood spot, measuring:

- **Methylmalonic acid (MMA)**: high in MMA and B12-related causes
- **2-Methylcitric acid (MCA)**: high in PA, and often in MMA too
- **Total homocysteine (tHcy)**: high when methionine synthase is affected (B12 deficiency, CblC)

Normally each value is compared with its own lab cut-off. This project asks whether **combining** the values gives a clearer answer, especially in milder disease states, where differentiating between the different diseases can get harder..

---

## What the calculator does

You enter the three second-tier values and your lab's cut-offs. The page then:

1. flags each value as high (**H**) or within range (**N**)
2. follows a three-step decision path and shows a suggested category: PA, isolated MMA, B12-related cause, grey zone, or not consistent with PA/MMA
3. shows each step with the numbers used, and greys out steps that were not needed
4. plots the sample's MMA ÷ MCA ratio on a log scale next to **30 published patient groups and samples**, so you can see where it falls compared with real data


---

## How to use it

1. **Enter the results** in µmol/L:
   - Methylmalonic acid (MMA)
   - 2-Methylcitric acid (MCA)
   - Total homocysteine (optional, needed for step 3)
2. **Enter your lab's cut-offs.** Cut-offs differ between labs (published MCA cut-offs range from 0.08 to 0.70 µmol/L). The defaults are those of Hu et al. 2021. The ratio in step 2 does not depend on cut-offs.
3. **Or load a published case** from the drop-down menu:

   | Example | Source | Expected result |
   |---|---|---|
   | Maternal B12 deficiency, newborn | Hu 2021 | B12-related cause |
   | Propionic acidemia, newborn median | Hu 2021 | PA |
   | Mutase MMA, newborn median | Hu 2021 | Isolated MMA |
   | CblC, newborn median | Hu 2021 | B12-related cause |
   | Mutase MMA on treatment | Dubland 2021 | Isolated MMA |
   | Unaffected newborn, control median | Dubland 2021 | Not consistent with PA or MMA |

4. **Read the interpretation.** The coloured box gives the suggested category and a next step. The step list underneath shows how the answer was reached.
5. **Check the scale.** The red marker shows where your sample's ratio sits among published PA (blue) and MMA-type (orange) patients.

---

## The decision path

| Step | Question | Rule | Outcome |
|---|---|---|---|
| **01** | Is MMA or MCA above the lab cut-off? | Both within cut-off | **Not consistent with PA or MMA** (likely a false-positive C3; consider the mother's B12) |
| **02** | Ratio MMA ÷ MCA | Below 1 | **Suggests propionic acidemia** |
| | | 1 to 10 | **Grey zone**: the ratio cannot decide; rely on confirmatory tests |
| | | Above 10 | MMA-type disorder → go to step 3 |
| **03** | Total homocysteine | Above cut-off (default 10 µmol/L) | **Suggests a B12-related cause** (maternal B12 deficiency or a cobalamin disorder such as CblC) |
| | | Within cut-off | **Suggests isolated methylmalonic acidemia** (mutase, CblA or CblB) |

**Why a ratio?** In PA, MCA rises a lot while MMA stays fairly low, so the ratio is small. In MMA-type disorders, MMA rises far more than MCA, so the ratio is large. Dividing one by the other cancels out some of the differences between labs, sample handling and how severe the disease is.

If MCA is 0, the ratio cannot be calculated; enter the assay's detection limit instead.

---

## Where the ratio comes from

The ratio was not picked by trial and error. It came out of a computer simulation of human metabolism that I wrote in Python.

- **Model:** Human-GEM (Robinson et al. 2020), a genome-scale model of human metabolism with 12,877 reactions, 8,460 metabolites and 2,848 genes.
- **Method:** flux balance and flux variability analysis using COBRApy. The model calculates the **maximum amount of each metabolite the body could export**, not the blood concentration.
- **Virtual patients:** 24 versions of the model with amino acid intake varied by up to ±30%, to represent differences in diet.
- **Diseases simulated:** PA (propionyl-CoA carboxylase limited), genetic MMA (methylmalonyl-CoA mutase limited) and B12 deficiency (mutase and methionine synthase limited together).
- **Severity:** each disease at 0%, 10%, 25%, 50%, 75% and 90% of normal enzyme activity.
- **Two-stage screen:**
  - *Stage 1:* about 1,550 candidate metabolites tested at the most severe level (0% enzyme) in 12 virtual patients, to find the ones worth testing further.
  - *Stage 2:* the 35 shortlisted metabolites tested in all 24 patients × 6 severities × 3 conditions (432 simulations). Patients 12–23 were not used in Stage 1.
- **Scoring:** how well each marker separates the diseases (AUC) and how consistently it does so in matched patients, with pass criteria set before running.

**What the simulation predicted:**

- Methylmalonate on its own becomes less reliable at separating PA from MMA as the disease gets milder.
- The **MMA ÷ MCA ratio** stays reliable at every severity level.
- In the methyl-group pathway, **formate** was the most robust predicted marker for separating B12 deficiency from genetic MMA. Formate is **not** used in the calculator, because it has not yet been measured for this purpose in patients (it does rise in B12-deficient rats; MacMillan et al. 2018).

To my knowledge, the MMA ÷ MCA ratio has not previously been proposed and tested as a single measure to tell PA from MMA. Published second-tier studies read MMA and MCA as separate markers.

The simulation code will be added to this repository.

---

## How the cut-offs were checked

I tested the ratio against published second-tier results from three studies: **30 rows covering 180 patient-samples**, including newborn screening samples, samples taken on treatment and external quality-control samples.

| Check | Ratio MMA ÷ MCA | MMA alone |
|---|---|---|
| Gap between the PA and MMA-type groups | **11.8×** | 1.3× |
| Subsets of the data where the gap is wider | **8 of 8** | – |
| Accuracy in each study when the cut-off was learned from the other two | **100%** in every study | 75–100% |

Applying the full decision path to the same data:

- **30 / 30** rows sorted correctly into PA vs MMA-type by the ratio
- **10 / 10** rows that reported homocysteine sorted correctly into isolated MMA vs B12-related MMA
- No published patient falls between a ratio of 0.87 and 10.3, which is why 1 and 10 were chosen as the edges of the grey zone

**Important:** this is a consistency check, not independent validation. The cut-offs of 1 and 10 were read from the same data they were checked on.

---

## Limitations

- **Not tested on individual patients.** Most published values are group medians, not single babies. The next step is testing on individual patient records.
- **Small data set.** Three studies, with only a few PA and B12-deficiency cases.
- **Capacity, not concentration.** The model predicts how much of a metabolite could be made, not how much is in the blood. For some B12 markers, the model and real blood move in opposite directions.
- **Virtual patients differ only in diet,** not in genetics, age or other illnesses.
- **Mild B12 deficiency** can give near-normal values and may pass step 1.
- **Lab cut-offs vary,** so step 1 and step 3 depend on using the right cut-offs for the lab that ran the test.

---

## Running the page yourself

The calculator is a single file, `index.html`, with no build step and no server.

- **Online:** use the GitHub Pages link at the top.
- **Offline:** download `index.html` and open it in any web browser.

To host your own copy: fork this repository, then go to **Settings → Pages → Deploy from a branch → main → / (root)**.

---

## Sources

1. Hu Z, Yang J, Lin Y, et al. (2021). [Determination of methylmalonic acid, 2-methylcitric acid, and total homocysteine in dried blood spots by liquid chromatography–tandem mass spectrometry: a reliable follow-up method for propionylcarnitine-related disorders in newborn screening](https://journals.sagepub.com/doi/10.1177/0969141320937725). *Journal of Medical Screening*.
2. Monostori P, et al. (2017). [Simultaneous determination of 3-hydroxypropionic acid, methylmalonic acid and methylcitric acid in dried blood spots: second-tier LC-MS/MS assay for newborn screening of propionic acidemia, methylmalonic acidemias and combined remethylation disorders](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0184897). *PLOS ONE* 12: e0184897.
3. Dubland JA, et al. (2021). [Analysis of 2-methylcitric acid, methylmalonic acid, and total homocysteine in dried blood spots by LC-MS/MS for application in the newborn screening laboratory: a dual derivatization approach](https://www.sciencedirect.com/science/article/pii/S2667145X21000067). *Journal of Mass Spectrometry and Advances in the Clinical Lab* 20.
4. MacMillan L, et al. (2018). [Cobalamin deficiency results in increased production of formate secondary to decreased mitochondrial oxidation of one-carbon units in rats](https://www.sciencedirect.com/science/article/pii/S0022316622109983). *Journal of Nutrition*.
5. Robinson JL, et al. (2020). [An atlas of human metabolism (Human-GEM)](https://www.science.org/doi/10.1126/scisignal.aaz1482). *Science Signaling* 13: eaaz1482.
6. Ebrahim A, et al. (2013). [COBRApy: COnstraints-Based Reconstruction and Analysis for Python](https://bmcsystbiol.biomedcentral.com/articles/10.1186/1752-0509-7-74). *BMC Systems Biology* 7: 74.

---


## License

Code: [MIT](LICENSE). The published patient values shown on the scale belong to the studies cited above and are reproduced for research comparison only.
