# Premier League Match Outcome Prediction

> **Research project:** Conducted in collaboration with Professor Piotr Duch of Lodz University of Technology (TUL), and extending his research on English Premier League match outcome prediction with additional autoencoder experiments.

A research replication and extension of Wiechno, Bartosik, and Duch's time-series approach to English Premier League match prediction. This project rebuilds match-level inputs from Vaastav/Fantasy Premier League player statistics, compares conventional classifiers and a Siamese network, and adds two autoencoder-based experiments.

> **Research status:** This is an independent student project informed by the cited paper. It is not an official implementation or an exact replication. The reference study uses FBref match statistics and ClubElo; this project uses Vaastav/FPL data, rolling team aggregates, and a locally computed Elo rating.

## Highlights

- Three-class prediction: home win (`H`), draw (`D`), away win (`A`).
- Two historical-form representations: last five matches (`MatchForm-5`) and the last three home/away matches (`HomeAwayForm-3`).
- Classical baselines, an MLP, a Siamese neural network, and two autoencoder extensions.
- Elo on/off ablation, stratified 80/20 and final-season evaluation, and class-sensitive metrics.
- Leakage-aware historical features: a team's current fixture is shifted out before rolling statistics are computed; scaling is fitted on training data only.

## Main findings

The Siamese model reached 58.6% accuracy on `HomeAwayForm-3` with the 2023/24 season held out. In the reported with-Elo experiments, the best balanced result was 50.0% balanced accuracy and 49.3% macro F1 from the Decision Tree on `MatchForm-5` with the final season held out. Autoencoder 2 (shared autoencoder plus a latent-space classifier) reached 57.6% accuracy on the same `MatchForm-5` final-season setting, but predicted no draws. Autoencoder 1 (class-specific reconstruction-error classification) performed below the strongest baselines in the reported configurations.

Accuracy alone hides the main challenge: draw recall is low for most models. See [the research report](docs/research_report.md) for the protocol, full interpretation, limitations, and reference.

Scores differ from the paper because this project uses a different feature source and construction, a locally computed Elo rating, and different eligible match rows. For example, the Siamese score on MatchForm-5 with the 2023/24 season held out is 56.0% here versus 59.0% in the paper; the HomeAwayForm-3 comparison is 58.6% here versus 58.0%. These comparisons are descriptive rather than a controlled replication.

## Repository layout

```text
.
├── README.md
├── CITATION.cff
├── requirements.txt
├── data/
│   └── README.md
├── docs/
│   └── research_report.md
└── notebooks/
    └── match-outcome-autoencoders.ipynb
```

## Reproduce the notebook

1. Open `notebooks/match-outcome-autoencoders.ipynb` in Google Colab.
2. Run the setup and upload cells, then upload the four CSV exports described in [data/README.md](data/README.md). They are intentionally not committed because of file size and source licensing/distribution considerations.
3. Set the configuration cell to choose the form dataset, split, Elo setting, and class-weighting option, then run the remaining cells.

The full grid-search notebook can be computationally expensive. The reported values in the research report are transcribed from the supplied results document; they are not a claim that this repository's notebook has been rerun in the current environment.

## Evaluation

Report overall accuracy together with balanced accuracy, macro F1, class-wise precision/recall/F1, confusion matrices, and the number of predicted draws. The random stratified split is useful for comparison with the paper; the final-season split is a more time-aware check. Do not compare the project scores as if the underlying features, feature construction, and dataset rows were identical to those in the paper.

## Citation

If you use the reference method, cite the original paper as listed in [CITATION.cff](CITATION.cff) and in the research report. Cite this repository after the author and repository details are completed.
