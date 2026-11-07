# MenaTree user manual

MenaTree answers two questions about a set of prokaryotic genomes:

1. **Can each genome make menaquinone (MK), and by which route?** — the classical *men* pathway,
   the futalosine (*mqn*) pathway, both, or neither (`pathway_profile.tsv`).
2. **How are the genomes related according to their MK biosynthesis genes?** — a supermatrix tree
   built from the pathway markers (`<set>/menatree.<set>.labeled.tree`).

Everything runs from a single Apptainer image (`menatree.sif`); you only need the image and a
folder with your genomes.

---

## 1. Get the image

**Option A — build it** (needs Apptainer ≥ 1.1 and internet access, about 5 minutes):

```bash
git clone https://github.com/ayixon/MenaTree.git
cd MenaTree
sudo apptainer build menatree.sif menatree.def      # or: apptainer build --fakeroot menatree.sif menatree.def
apptainer test menatree.sif
```

**Option B — download a pre-built image** from the repository's *Releases* page, if one is provided.

Check that it works:

```bash
apptainer run menatree.sif --version
apptainer run menatree.sif --help
apptainer run menatree.sif --list-markers
```

> In the examples below, `menatree` stands for `apptainer run menatree.sif`. You can create the
> shortcut once per session with `alias menatree="apptainer run /path/to/menatree.sif"`.

---

## 2. Prepare your input

Put one FASTA file per genome in a folder:

```
genomes/
├── GCA_000797101.1_ASM78619v1.fna.gz
├── GCA_001522255.1_genomic.fna
├── my_isolate.fasta
└── ...
```

- Nucleotide assemblies (contigs or complete genomes) or proteomes; MenaTree detects the type.
- Accepted extensions: `.fa .fna .fasta .fas .faa .ffn .fsa .fn .seq`, optionally gzipped.
- At least **5 genomes** are needed to build a tree (the profile works with any number).
- Genome IDs are taken from the file names (`GCA_000797101.1_...` → `GCA-001522255.1`).

---

## 3. Basic runs

### Classical pathway tree (default)

```bash
menatree -i genomes/ -o results/
```

Profiles both pathways, then builds the MKv03 tree (MenF, MenD, MenC, MenE, MenB, MenA, MenG)
with the genomes in which the classical pathway was detected.

### Let MenaTree decide (recommended for mixed or unknown taxa)

```bash
menatree -i genomes/ -o results/ -m auto
```

Builds one tree per pathway: classical genomes → `results/MKv03/`, futalosine genomes →
`results/futalosinHMM/`. Genomes with neither pathway are listed in the log and in the profile.

### Futalosine pathway only

```bash
menatree -i genomes/ -o results/ -m futalosine
```

### Pathway profile only (fast, no trees)

```bash
menatree -i genomes/ -o results/ --profile-only
```

Useful as a first screen, or with fewer than 5 genomes.

---

## 4. More examples

**More threads and IQ-TREE instead of FastTree** (model selection plus SH-aLRT and UFBoot supports):

```bash
menatree -i genomes/ -o results_iqtree/ -m auto -t 16 --tree-app iqtree
```

**Your own tip labels.** Create a tab-separated file with the genome ID (as in `genome_labels.tsv`)
and the label you want:

```
NE2TTS12-Conting	NE2TTS12 Kocuria sp. nov.
GCA-014636775	Kocuria dechangensis CGMCC 1.12187T
```

```bash
menatree -i genomes/ -o results/ --labels my_labels.tsv
```

**A custom marker set.** Any HMMER3 file works; newer format tags are converted automatically:

```bash
menatree -i genomes/ -o results_custom/ -m my_markers.hmm
```

**Relax the filters** (for draft genomes with missing markers):

```bash
menatree -i genomes/ -o results/ --gene-cutoff 0.6 --genome-cutoff 0.6
```

**Keep intermediate files** (alignments, hmmsearch tables) for inspection:

```bash
menatree -i genomes/ -o results/ --keep-tmp
```

---

## 5. Reading the results

```
results/
├── pathway_profile.tsv        ← start here
├── pathway_bits.tsv
├── profile_hits.tsv
├── genome_labels.tsv
├── MKv03/
│   ├── menatree.MKv03.labeled.tree
│   ├── menatree.MKv03.labeled_underscore.tree
│   ├── menatree.MKv03.tree
│   ├── markers_used.tsv
│   ├── supermatrix.faa
│   └── easycgtree.log
├── proteomes/
├── menatree.log
└── CITATIONS.txt
```

### `pathway_profile.tsv`

One row per genome. Marker columns give the number of copies (hits scoring at least 50% of the
best hit). The last three columns summarise the call:

| genome | … | MenF | MenD | MenC | MenE | MenB | MenA | MenG | MqnA | MqnE | MqnB | MqnC | MqnD | classical_markers | futalosine_markers | pathway |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| NE2TTS12-Conting | … | 2 | 1 | 1 | 1 | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 6/6 | 0/4 | classical |

Pathway calls:

| Call | Rule |
|---|---|
| `classical` | ≥ 4 of MenF, MenD, MenC, MenE, MenB, MenA |
| `futalosine` | ≥ 3 of MqnA, MqnE, MqnC, MqnD |
| `both` | both rules met |
| `incomplete` | some markers, neither rule met (check assembly completeness) |
| `none` | no markers (e.g. organisms without menaquinone) |

MenG and MqnB are reported but not used for the call (MenG is shared with ubiquinone synthesis;
MqnB also matches MtnN nucleosidases).

### Trees

- `*.labeled.tree`: tips like `'GCA-000786195 Kocuria rosea CMS 76or'` (quoted Newick; FigTree,
  iTOL and ete3 read it).
- `*.labeled_underscore.tree`: the same with `_` instead of spaces, for programs that reject quotes.
- Supports: SH-like local supports (FastTree, 0–1) or UFBoot (IQ-TREE, 0–100).
- `markers_used.tsv`: which markers entered the tree and how many columns survived trimming.

### How to interpret

- The **profile** is the chemotaxonomic proxy: it predicts the capacity for MK synthesis and the
  route, not the MK type (MK-7, MK-9…) or its saturation.
- The **tree** is complementary evidence of relatedness based on a handful of genes. Do not use it
  alone to delimit species; combine it with ANI/dDDH and core-genome phylogenomics.

---

## 6. Troubleshooting

| Problem | Solution |
|---|---|
| `input directory not found` with files under `/mnt/c/...` (WSL) or another disk | Apptainer only mounts your home and current folder: add `--bind /mnt/c` (or the disk path), e.g. `apptainer run --bind /mnt/c menatree.sif -i /mnt/c/data/genomes -o results` |
| `tree skipped ... at least 5 are needed` | Fewer than 5 genomes have the pathway or pass the filters; see `pathway_profile.tsv` or lower `--genome-cutoff` |
| A genome is missing from the tree | It lacked the pathway or too many markers; check its row in `pathway_profile.tsv` |
| Tip labels show only the ID | The FASTA header is not NCBI-style or the input is a proteome; use `--labels` |
| `threads requested but only N CPU cores available` | Informational: MenaTree uses the available cores |

---

## 7. Citation

Please cite MenaTree (see `CITATION.cff`) and the tools listed in `CITATIONS.txt`, which every run
writes to the output folder.
