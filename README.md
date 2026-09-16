# Target-Conditioned Peptide Binder Design with PepMLM and Boltz-1

An end-to-end computational pipeline for generating target-conditioned
peptide binders, predicting their protein–peptide complex structures,
and evaluating whether predicted binding poses reproduce experimentally
observed peptide-binding sites.

The project combines protein language modeling, structure prediction,
structural superposition, negative controls, and statistical analysis.

---

## Overview

The workflow consists of six main stages:

1. **Target selection**
   - MDM2 (`1YCR`)
   - BCL-XL (`1BXL`)
   - A third protein–peptide complex identified automatically through the
     RCSB Search API (`10KU`)

2. **Target-conditioned peptide generation**
   - Candidate peptides are generated using the pretrained
     `ChatterjeeLab/PepMLM-650M` model.
   - Four candidates are generated for each target.
   - Generation confidence is measured using the mean log-probability of
     sampled amino acids.

3. **Composition-matched negative controls**
   - Each generated peptide is randomly scrambled while preserving its
     amino-acid composition.
   - These controls test whether downstream structural confidence is
     sensitive to peptide sequence order rather than composition alone.

4. **Protein–peptide structure prediction**
   - Every target–peptide pair is co-folded using Boltz-1.
   - Both PepMLM candidates and scrambled controls are evaluated.
   - In total, 24 protein–peptide complexes are predicted.

5. **Structural pose validation**
   - Predicted target structures are superimposed onto their experimental
     RCSB crystal structures.
   - The predicted peptide is transformed into the reference coordinate frame.
   - Pose agreement is quantified using an `overlap_fraction`, defined as
     the fraction of predicted peptide Cα atoms lying within 8 Å of the
     experimentally observed peptide footprint.

6. **Statistical analysis and ranking**
   - Mann–Whitney U tests compare Boltz-1 ipTM scores between generated and
     scrambled peptides.
   - Spearman correlation evaluates whether PepMLM generation confidence
     predicts Boltz-1 structural confidence.
   - Candidates are ranked using a composite score combining structural
     confidence, pose overlap, interface contacts, and generation confidence.

---

## Pipeline

```text
RCSB target structures
        |
        v
     PepMLM
        |
        +----> generated peptides
        |
        +----> composition-matched scrambled controls
                         |
                         v
                     Boltz-1
                         |
                         v
              predicted complexes
                         |
            +------------+-------------+
            |                          |
            v                          v
      confidence scores       structural superposition
            |                          |
            v                          v
          ipTM                  pose overlap
            \                          /
             \                        /
              +---- statistical -----+
                     analysis
                         |
                         v
                 composite ranking
