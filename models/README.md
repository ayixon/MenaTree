# Marker models

Both files use the HMMER3 text format with the `HMMER3/b [3.0 | March 2010]` tag, because EasyCGTree
ships HMMER 3.0. Every model has exactly 23 header lines between the format tag and
`STATS LOCAL FORWARD`. Emission and transition parameters are identical to the source models.

## MKv03.hmm — classical menaquinone pathway (men)

| NAME | ACC | Source | LENG |
|---|---|---|---|
| MenF | MF_01935 | HAMAP | 447 |
| MenD | TIGR00173.1 | TIGRFAMs / NCBIFAM | 435 |
| MenC | MF_00470 | HAMAP | 334 |
| MenE | NF005877.1 | NCBIFAM (PRK07824) | 362 |
| MenB | NF005637.0 | NCBIFAM (PRK07396) | 273 |
| MenA | TIGR00751.1 | TIGRFAMs / NCBIFAM | 284 |
| MenG | PF01209.26 | Pfam (Ubie_methyltran) | 233 |

## futalosinHMM.hmm — futalosine pathway (mqn)

| NAME | ACC | Source | LENG |
|---|---|---|---|
| MqnA | MF_00995 | HAMAP | 266 |
| MqnE | MF_00993 | HAMAP | 364 |
| MqnB | MF_00991 | HAMAP | 229 |
| MqnC | MF_00992 | HAMAP | 362 |
| MqnD | MF_00996 | HAMAP | 275 |

## Header edits

- `NAME` set to the gene name; original name and accession kept in `DESC`.
- HAMAP models had no `ACC`, `GA`, `TC`, `NC`, `BM` or `SM` lines. `ACC` is the HAMAP rule ID;
  `GA`/`TC`/`NC` are set to ¼ of the model length (the EasyCGTree score filter), not curated cut-offs.
- HAMAP models were taken from the HMMER3 build distributed with Prokka (`db/hmm/HAMAP.hmm`).

## Known caveats

- MenG/UbiE (PF01209) also detects UbiE of the ubiquinone pathway.
- MqnB (MF_00991) also detects MTA/SAH nucleosidases (MtnN); MenaTree does not use it to call the
  futalosine pathway.
- MenF often has a second hit above 50% of the best score (possibly a second isochorismate synthase).
