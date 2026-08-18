# Chapter 1 — cervical kinematics, activation and neck pain

Results are filed by hypothesis, matching the numbering used in the chapter. A
reader arriving from Chapter 1 holding "H6" in mind will find `H6_acute_pain/`.

| Folder | Hypothesis | Question |
|---|---|---|
| `H1_angles_and_pain/` | H1 | Do cyclists with more at-home neck pain move differently — reduced range of motion, more variability, or different mean angles? |
| `H2_activation/` | H2 | Is at-home neck pain associated with muscle activation (%MVC)? |
| `H3_cocontraction/` | H3 | Is it associated with co-contraction (CCI)? |
| `H4_asymmetry/` | H4 | Is it associated with left–right activation asymmetry? |
| `H5_anthropometrics_fitness/` | H5 | Do body measures and fitness relate to pain, and do fitness groups differ in neck angles? |
| `H6_acute_pain/` | H6 | Does pain that develops *during* the session track biomechanical change across it? |
| `H7_angle_emg_coupling/` | H7 | How do cervical angle and muscle activation move together through the protocol? |
| `H8_scm_activation/` | H8 | Does the sternocleidomastoid stand out? — see the note inside; no separate output |
| `H9_clustering/` | H9 | Do participants fall into distinct kinematic phenotypes, and do those relate to pain? |

One folder sits outside the numbering. **`supplementary/`** holds two
cross-hypothesis analyses that span several of the folders above rather than
belonging to any one of them: the Pain versus No-Pain group comparisons for
H1–H4, and the ordinal analyses across the three pain groups (None, Mild,
Moderate-High).


## Where to start

`H9_clustering/READ_ME.md` defines the four kinematic features the clustering
is built on, and is the best single entry point to the chapter.

For how any individual measure is computed — the formula, units, sign
conventions and exclusions — see the METHOD documents in
[`../../docs/`](../../docs/). For one participant followed from raw signal to a
reported number, see [`../../docs/PIPELINE.md`](../../docs/PIPELINE.md).

## Reading these tables

**Pain is measured two ways.** *At-home* VAS is what a participant reports about
their cycling generally, and it is the outcome in H1–H5. *Acute* VAS is measured
during the session itself, and its change across the protocol is the outcome in
H6. They are different variables — a result about one does not transfer to the
other.

**Cluster numbers are arbitrary.** They come from the initialisation and carry
no ordering. Cluster 0 in one analysis has no relationship to Cluster 0 in
another.
