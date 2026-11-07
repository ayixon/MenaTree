# Changelog

## 0.1.1 — 2026-11-07

- The classical pathway call now requires MenD and MenA (pathway-specific markers) in addition to
  ≥ 4 of the 6 classical markers. In futalosine-pathway genomes (e.g. *Streptomyces*), MenF, MenC,
  MenE and MenB hit paralogs from large families, which produced false `both` calls and paralog-based
  MKv03 trees in 0.1.0.
- Citation request printed at the end of each run; GPL-3.0 with section 7(b) attribution terms.

## 0.1.0 — 2026-11-07

First release.

- `menatree` wrapper: Prodigal protein prediction, pathway profiling of the classical (*men*) and
  futalosine (*mqn*) routes, `auto` mode with one tree per pathway, EasyCGTree supermatrix trees,
  tip labels from NCBI FASTA headers, citation file.
- Marker set **MKv03** (7 markers): MenC and MenF replaced by HAMAP MF_00470 and MF_01935
  (the MKv02 MenC model, NF047905, is *Bdellovibrio*-specific; the MKv02 MenF model, NF011588,
  failed the score filter in most Actinobacteria).
- Marker set **futalosinHMM** (5 markers, HAMAP).
- Apptainer definition (Debian 12, EasyCGTree v4.2 Linux build pinned to commit 71da499,
  IQ-TREE 2.3.6 replacing the bundled 2.2.0).
- Work-arounds for EasyCGTree 4.2: HMMER 3.0 format tag, underscores in model accessions,
  exit error after IQ-TREE, thread count above available cores.
