# MenaTree

**Menaquinone (MK) pathway profiling and phylogeny for prokaryotic genomes.**

MenaTree detects the two known menaquinone biosynthesis routes in prokaryotic genomes -the
classical *men* pathway and the alternative futalosine (*mqn*) pathway- and builds marker-based
supermatrix trees with [EasyCGTree](https://github.com/zdf1987/EasyCGTree4). It is intended as a
genomic proxy for the chemotaxonomic potential of a strain when respiratory quinones have not been
measured experimentally.

> **Scope.** MenaTree predicts *whether* a genome can make menaquinone and *by which route*. It does
> **not** predict the MK type (e.g. MK-7 vs MK-9) or its saturation (e.g. MK-9(H4)): chain length is set
> by polyprenyl diphosphate synthases and saturation by reductases such as MenJ, which are not modelled.
> The marker phylogeny is complementary evidence; species delineation should rely on overall genome
> relatedness indices (ANI/dDDH) and core-genome phylogenomics.

## Quick start

```bash
apptainer run menatree.sif -i genomes/ -o results/            # classical pathway tree (default)
apptainer run menatree.sif -i genomes/ -o results/ -m auto    # profile, then one tree per pathway
apptainer run menatree.sif -i genomes/ -o results/ --profile-only
apptainer run menatree.sif --list-markers
```

See **[MANUAL.md](MANUAL.md)** for step-by-step examples and how to read the results.

Input: one FASTA file per genome in a directory -nucleotide assemblies (proteins are predicted with
Prodigal) or proteomes; `.gz` files are accepted. Extensions: `.fa .fna .fasta .fas .faa .ffn .fsa .fn .seq`.
At least 5 genomes are needed to build a tree.

## Options

| Option | Description |
|---|---|
| `-i, --input DIR` | Directory of genome (nucleotide) or proteome FASTA files |
| `-o, --outdir DIR` | Output directory (default: `menatree_out`) |
| `-m, --markers SET` | `mk` (default), `futalosine`, `auto`, or a custom HMMER3 file |
| `-t, --threads INT` | CPU threads (default: 4; capped at the available cores) |
| `--tree-app` | `fasttree` (default; JTT+CAT, SH-like supports) or `iqtree` (ModelFinder, SH-aLRT and UFBoot, 1000 replicates) |
| `--trim` | trimAl mode: `strict` (default), `gappyout`, `strictplus` |
| `--evalue` | hmmsearch E-value threshold (default: 1e-10) |
| `--gene-cutoff` | Minimum fraction of genomes carrying a marker (default: 0.8) |
| `--genome-cutoff` | Minimum fraction of markers per genome (default: 0.8) |
| `--profile-only` | Report pathway profiles without building trees |
| `--labels FILE` | Tab-separated `genome_id<TAB>label` file to name tree tips |
| `--no-header-labels` | Do not derive tip labels from FASTA headers |
| `--keep-tmp` | Keep intermediate EasyCGTree files |
| `--list-markers` | Show the built-in marker sets and exit |

## Marker sets

**`mk` -MKv03, classical pathway (7 markers)**

| Marker | Model | Enzyme | EC |
|---|---|---|---|
| MenF | HAMAP MF_01935 | isochorismate synthase | 5.4.4.2 |
| MenD | TIGR00173 | SEPHCHC synthase | 2.2.1.9 |
| MenC | HAMAP MF_00470 | o-succinylbenzoate synthase | 4.2.1.113 |
| MenE | NCBIFAM NF005877 | o-succinylbenzoate–CoA ligase | 6.2.1.26 |
| MenB | NCBIFAM NF005637 | DHNA-CoA synthase | 4.1.3.36 |
| MenA | TIGR00751 | DHNA octaprenyltransferase | 2.5.1.74 |
| MenG | Pfam PF01209 | demethylmenaquinone methyltransferase (UbiE/MenG) | 2.1.1.163 |

**`futalosine` -futalosinHMM, futalosine pathway (5 markers)**

| Marker | Model | Enzyme | EC |
|---|---|---|---|
| MqnA | HAMAP MF_00995 | chorismate dehydratase | 4.2.1.151 |
| MqnE | HAMAP MF_00993 | aminodeoxyfutalosine synthase | 2.5.1.120 |
| MqnB | HAMAP MF_00991 | futalosine hydrolase | 3.2.2.26 |
| MqnC | HAMAP MF_00992 | cyclic dehypoxanthine futalosine synthase | 1.21.98.1 |
| MqnD | HAMAP MF_00996 | 1,4-dihydroxy-6-naphthoate synthase | 4.1.99.29 |

## How it works

1. **Proteomes.** Prodigal (single mode; meta mode for inputs < 20 kb) predicts proteins from nucleotide input.
2. **Pathway profile.** All genomes are searched with both marker sets (hmmsearch). A marker is
   *present* when its best hit has E ≤ `--evalue` and a bit score ≥ ¼ of the model length (the
   EasyCGTree filter). Copies are hits scoring ≥ 50% of the best hit, so distant paralogs of large
   families (acyl-CoA ligases, crotonases, enolases) are not counted as copies.
   Each genome is then called:
   - `classical`: MenD and MenA present, and ≥ 4 of MenF, MenD, MenC, MenE, MenB, MenA
   - `futalosine`: ≥ 3 of MqnA, MqnE, MqnC, MqnD
   - `both`, `incomplete` (some markers but below both thresholds) or `none`

   MenG is not used for the call because UbiE/MenG methylates both ubiquinone and menaquinone
   precursors. MqnB is not used because its model also matches MTA/SAH nucleosidases (MtnN).
3. **Trees.** EasyCGTree retrieves the best hit per marker, filters genes and genomes
   (`--gene-cutoff`, `--genome-cutoff`), aligns each marker with MUSCLE 5, trims with trimAl,
   concatenates them and infers a supermatrix tree. Only genomes in which the pathway was called enter
   its tree: `classical`/`both` for MKv03 (`-m mk`) and `futalosine`/`both` for futalosinHMM
   (`-m futalosine`); `auto` builds both. Custom HMM files use all genomes. Each tree needs ≥ 5 genomes.
4. **Labels.** Tree tips are named `accession Genus species strain` from NCBI-style FASTA headers
   (use `--labels` to override).

## Outputs

| File | Content |
|---|---|
| `pathway_profile.tsv` | Copies of each marker per genome, marker counts and pathway call |
| `pathway_bits.tsv` | Best bit score per marker and genome |
| `profile_hits.tsv` | Every hmmsearch hit, flagged `best_hit`, `copy`, `paralog` or `below_threshold` |
| `genome_labels.tsv` | Genome ID, input file and tip label |
| `<set>/menatree.<set>.tree` | Newick tree with genome IDs |
| `<set>/menatree.<set>.labeled.tree` | Tree with quoted `accession species strain` labels |
| `<set>/menatree.<set>.labeled_underscore.tree` | Same, with underscores instead of spaces |
| `<set>/supermatrix.faa` | Concatenated, trimmed alignment |
| `<set>/markers_used.tsv` | Markers in the tree with aligned and trimmed columns |
| `<set>/easycgtree.log` | EasyCGTree log |
| `proteomes/` | Predicted proteomes |
| `menatree.log`, `CITATIONS.txt` | Run log and references |

## Building the image

```bash
sudo apptainer build menatree.sif menatree.def     # or: apptainer build --fakeroot ...
apptainer test menatree.sif
```

Run the build from the repository root (the definition file copies `menatree`, `models/`, the
documentation, `LICENSE`, `NOTICE` and `CITATION.cff` into the image). It needs internet
access to pull `debian:12-slim` and to download, from GitHub, the EasyCGTree v4.2 Linux zip (pinned to
commit `71da499`) and IQ-TREE 2.3.6; both downloads are checksum-verified. IQ-TREE 2.3.6 replaces the IQ-TREE 2.2.0 bundled with EasyCGTree,
which fails on some small supermatrices.

## Notes and limitations

- EasyCGTree ships HMMER 3.0, which only reads the `HMMER3/b` format tag. MenaTree relabels newer
  HMM files automatically, so custom HMMER3 files can be used directly.
- Marker GA/TC/NC lines in the HAMAP-derived models are set to ¼ of the model length; they are not
  curated cut-offs.
- Genomes lacking the selected pathway are dropped from that tree; check `pathway_profile.tsv`.

## Repository layout

```
MenaTree/
├── menatree            # the program (Python 3, standard library only)
├── menatree.def        # Apptainer definition file
├── models/             # MKv03.hmm, futalosinHMM.hmm and their provenance (models/README.md)
├── README.md           # this file
├── MANUAL.md           # user manual with examples
├── CHANGELOG.md
├── CITATION.cff        # how to cite MenaTree
├── LICENSE             # GNU GPL v3
└── NOTICE              # additional attribution terms and third-party components
```

## License and citation

MenaTree is released under the **GNU General Public License v3.0** with additional attribution terms
under section 7(b) (see `LICENSE` and `NOTICE`): copies and derivative works must keep the MenaTree
name, author attribution, `CITATION.cff` and the citation request. If you publish results obtained
with MenaTree, please cite it as given in `CITATION.cff`, together with the tools and marker
databases listed in the `CITATIONS.txt` file written to every output folder, in particular
EasyCGTree (Zhang D-F et al., 2023, *BMC Bioinformatics*, doi:10.1186/s12859-023-05527-2).

Third-party software (EasyCGTree and its bundled binaries, IQ-TREE) is not part of this repository;
it is downloaded when the image is built and keeps its own terms (see `NOTICE`).
