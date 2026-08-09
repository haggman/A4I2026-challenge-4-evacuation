# notebooks/

Two notebooks live here.

**`c4_01_load_explore.ipynb`** — start here. This is the one you run. It pulls the raw public data
live from its original sources, shows you the problems in it, fixes them in front of you, loads the
result into BigQuery in *your* project, and validates what it loaded. The explanations between the
code cells are the point: a reader who runs nothing and reads only the markdown should be able to
follow what is happening and why.

Import it into Colab Enterprise with **My Notebooks → Import → source: URL**, using the raw GitHub
URL from the README. No clone, no git, no auth dance.

**`c4_90_publish_snapshot.ipynb`** — ROI maintainers only. It regenerates the Cloud Storage snapshot
that `scripts/load.sh` reads from. Running it as a student throws a permissions error, because you
do not have write access to our bucket, and that is expected. You never need it.

The `c4_` prefix comes first because Colab Enterprise truncates notebook names from the right, so a
suffix-based scheme would leave you looking at five identically-labelled notebooks. `01` means start
here; `90` means maintainers only.
