# Premier League Match Outcome Prediction

## A Vaastav/FPL-based adaptation with autoencoder extensions

**Research project conducted in collaboration with Professor Piotr Duch, Lodz University of Technology (TUL)**  
**Author:** Mikołaj Żyszkiewicz  
**Task:** Predict English Premier League home win, draw, or away win  
**Reference:** Wiechno, Bartosik, and Duch (2025)

## Abstract

This project adapts the time-series and Siamese-network framework proposed by Wiechno, Bartosik, and Duch for English Premier League (EPL) match outcome prediction. It covers the same seven seasons, 2017/18–2023/24, and the same 2,660-fixture raw EPL sample described in the reference study, but constructs predictors from Vaastav/Fantasy Premier League (FPL) player statistics rather than FBref match statistics. Player rows are aggregated to team-match records; rolling form features are built from prior fixtures; and an Elo rating is computed sequentially from the project's match outcomes. Classical classifiers, a multilayer perceptron, and a Siamese network are compared with two additional autoencoder approaches: class-specific reconstruction-error classification and classification in a shared autoencoder's latent space. Across the reported experiments, overall accuracy is generally in the 46–59% range, while draws remain difficult to identify. Autoencoder 2 attained 57.6% accuracy in the `MatchForm-5` final-season evaluation with Elo, but detected none of the test-set draws. These results are an adapted comparison, not an exact replication of the published experiments.

## 1. Introduction

Three-way football outcome prediction is affected by temporal form, class imbalance, and uncertainty in match-level performance. The reference study proposes a Siamese neural network that compares historical home- and away-team information and evaluates it alongside classical models, an MLP, and recurrent models. It reports 2,660 EPL fixtures from 2017/18–2023/24, using FBref statistics and ClubElo.

This project follows the broad research question and selected evaluation settings, then changes the feature source and representation. It also adds autoencoder-based experiments to ask whether unsupervised compression or class-specific reconstruction patterns provide useful representations for match outcome classification.

### Contributions

1. Reconstruct team-level historical form features from player-level Vaastav/FPL data.
2. Compare `MatchForm-5` and `HomeAwayForm-3` representations, with and without an internally computed Elo feature.
3. Evaluate a reconstruction-error autoencoder and a latent-space classifier in addition to the baseline model families.
4. Report class-sensitive metrics to make the weak draw performance visible.

## 2. Data and feature construction

### 2.1 Scope and target

The raw fixture sample spans EPL seasons 2017/18–2023/24 and contains 2,660 matches: 1,192 home wins, 602 draws, and 866 away wins. The target is derived from the final score and encoded as `H`, `D`, or `A`. Match and competition metadata come from `match.csv`, `league.csv`, and `league_season.csv`; player-level FPL statistics come from `vaastav_stats.csv`. The league lookup explicitly filters the fixture seasons to the Premier League.

The reference paper's raw fixture count and class counts are matched, but that does not mean the project uses the paper's feature table or identical modeling rows. Rolling-history requirements exclude fixtures without sufficient prior form, so the model-ready samples are smaller.

### 2.2 Team-match aggregation

Player records are grouped by fixture and side. Additive event/statistic columns (including assists, own goals, penalty events, cards, saves, influence, creativity, and threat) are summed. Goals scored and conceded are taken from the match score to avoid summing repeated player-level score fields. Clean sheets are derived from the official conceded-goals value. Missing or non-numeric additive inputs are set to zero in the current notebook; this assumes a missing record means no recorded event and should be checked when using a different export.

Thirteen team-level variables are retained for historical form: goals scored, assists, clean sheets, goals conceded, own goals, penalties saved, penalties missed, yellow cards, red cards, saves, influence, creativity, and threat. Current fixture scores are used to define the label and historical measurements, but are excluded from predictors for that same fixture.

### 2.3 Historical form

Each match becomes a home-side and away-side team record. Records are ordered chronologically per team and `shift(1)` is applied before rolling calculations, so the current fixture cannot contribute to its own predictors.

- **MatchForm-5:** mean of each variable across each team's previous five EPL matches, regardless of venue.
- **HomeAwayForm-3:** mean across the home team's previous three home fixtures and the away team's previous three away fixtures.

A fixture is retained only when both sides have sufficient history. The supplied report records 2,560 usable fixtures for MatchForm-5 and 2,538 for HomeAwayForm-3.

### 2.4 Elo feature

When enabled, Elo is computed in date order from a starting rating of 1,500, K-factor 20, and a home-advantage adjustment of 65 points in expected-score calculation. Each fixture receives the two ratings as they stood before that fixture's result is applied. Thus Elo contributes two features; together with 26 rolling home/away features, the models receive 28 inputs. The no-Elo ablation uses the same rows and 26 rolling inputs.

## 3. Experimental design

### 3.1 Models

The reported model families include Random Forest, Gaussian Naive Bayes, SVM, Decision Tree, KNN, Bagging, XGBoost, MLP, Siamese neural network, and two project-specific autoencoder experiments. The Siamese model follows the reference paper's paired home/away representation at a high level; data and feature construction differ.

**AE1 — class-specific reconstruction error.** A separate autoencoder is trained for each outcome class. A test example is assigned to the class whose autoencoder yields the lowest reconstruction error. This is a nearest-class reconstruction rule, not a supervised class-probability model.

**AE2 — latent-space classifier.** A single autoencoder is trained on all training examples. Its encoder maps examples into a lower-dimensional representation, and a separate classifier is trained on those latent vectors. The supplied notebook uses a neural classifier for this second stage.

### 3.2 Splits and preprocessing

The notebook supports stratified random 80/20, chronological 80/20, and final-season holdout (2023/24) settings. The report's primary comparison includes stratified 80/20 and final-season results. Features are standardized with a scaler fitted only on the training subset. Hyperparameter searches use training data and five-fold stratified cross-validation in the reported settings. The random split is the closest counterpart to the paper's random 80/20 setup; the final-season setting tests generalization to a later season. In the final-season evaluation, form and Elo are updated chronologically using observed results as the season progresses. This represents sequential, in-season prediction with newly observed match data; it is not a setting where every 2023/24 prediction is made before that season begins.

Unless noted otherwise, the reported reference-comparison runs use no class weights, five-fold grid search optimized for accuracy, and seed 42. The Elo ablation repeats the configurations without the two rating inputs.

### 3.3 Metrics

Overall accuracy is reported together with balanced accuracy, macro F1, per-class metrics, confusion matrices, and the number of predicted draws. For a three-class problem with fewer draws, accuracy can be high even when a model rarely predicts a draw, so overall accuracy should not be read alone.

## 4. Results

### 4.1 Primary with-Elo results

Across the reported with-Elo settings, the strongest final-season overall accuracy was 58.6% for the Siamese network on HomeAwayForm-3. The strongest balanced result was the Decision Tree on MatchForm-5 in the final-season test: 50.0% balanced accuracy and 49.3% macro F1. These results show that the leading model depends on the metric and form representation.

The autoencoder outcomes illustrate the class-imbalance issue:

| Setting | Autoencoder | Accuracy | Balanced accuracy | Macro F1 | Draw recall |
|---|---|---:|---:|---:|---:|
| HomeAwayForm-3, stratified 80/20, Elo | AE1: reconstruction error | 46.3% | 39.1% | 35.8% | 3.5% |
| HomeAwayForm-3, stratified 80/20, Elo | AE2: latent classifier | 51.8% | 43.1% | 37.9% | 0.0% |
| HomeAwayForm-3, final season, Elo | AE1: reconstruction error | 48.1% | 37.4% | 32.4% | 2.5% |
| HomeAwayForm-3, final season, Elo | AE2: latent classifier | 55.6% | 46.4% | 40.8% | 0.0% |
| MatchForm-5, stratified 80/20, Elo | AE1: reconstruction error | 46.3% | 38.1% | 34.2% | 2.6% |
| MatchForm-5, stratified 80/20, Elo | AE2: latent classifier | 54.3% | 45.5% | 40.0% | 0.0% |
| MatchForm-5, final season, Elo | AE1: reconstruction error | 50.7% | 41.1% | 37.2% | 2.5% |
| MatchForm-5, final season, Elo | AE2: latent classifier | 57.6% | 47.3% | 41.8% | 0.0% |

AE2 was more accurate than AE1 in each listed with-Elo configuration, but AE2 predicted zero draws in every one of those configurations. AE1 predicted few draws and still had low draw recall. These experiments therefore do not establish a useful autoencoder improvement for balanced three-way prediction; they provide a baseline for further representation-learning work.

### 4.2 Elo ablation

The supplied report states that adding Elo improved overall accuracy in 39 of 44 paired model/configuration comparisons, tied in three, and reduced it in two. Reported mean changes were approximately +2.7 percentage points in accuracy, +2.9 in balanced accuracy, and +3.0 in macro F1. This is descriptive evidence from the recorded runs, not a guarantee that Elo helps in other splits or datasets.

Without Elo, AE2 and the Siamese model both reached 56.0% accuracy on MatchForm-5 with the final season held out. AE2 predicted no draws; Gaussian Naive Bayes had lower accuracy (52.0%) but the highest reported balanced accuracy (47.2%), macro F1 (46.8%), and draw F1 (24.7%) in that configuration.

### 4.3 Interpretation

The experiments reinforce three observations. First, recent team form and Elo contain useful signal, but overall accuracy is moderate. Second, performance changes with the form representation and test design. Third, draw classification remains the central weakness. AE2's comparatively strong overall accuracy alongside zero draw recall is a clear example of why accuracy alone is insufficient.

### 4.4 Why the scores differ from the reference study

The project scores should be expected to differ because the project changes several parts of the experiment at once. It is therefore not possible to attribute a score gap to one cause from this comparison alone.

1. **Different predictor data and definitions.** The paper uses FBref match statistics and ClubElo. This project aggregates player-level Vaastav/FPL records into 13 team variables, averages those over past matches, and calculates a new Elo sequence (initial rating 1,500, K=20, home adjustment 65). FPL measures such as influence, creativity, and threat are not substitutes for the paper's match-event statistics, and this project's Elo values are not the published ClubElo ratings.
2. **Different model-ready samples.** The paper reports 2,307 MatchForm-5 matches and 2,228 HomeAwayForm-3 matches after its history requirements. This project reports 2,560 and 2,538 respectively. The raw fixture totals match, but the retained rows and feature construction do not. Models are consequently trained and evaluated on different examples.
3. **Different realizations and model selection.** A stratified random 80/20 split can select different fixtures, especially when eligible samples differ. The final-season test is more comparable in time but still uses different inputs and retained rows. The notebook also uses its own hyperparameter grids and neural-network implementation. The supplied comparison does not include repeated splits or confidence intervals, so some of the observed gap may be split or training variation.

For example, in MatchForm-5 with the final season held out, the paper reports 59.0% accuracy for the Siamese model and this project reports 56.0%, a 3.0 percentage-point difference. In HomeAwayForm-3 with the final season held out, the corresponding values are 58.0% and 58.6%. In the two stratified 80/20 settings, the project Siamese result is 3.5 points lower for HomeAwayForm-3 (53.0% vs. 56.5%) and 4.7 points lower for MatchForm-5 (52.9% vs. 57.6%). The direction is not uniform, which is consistent with the comparisons being affected by several differences rather than a single model change. These are descriptive comparisons, not controlled estimates of the effect of changing the data source.

### 4.5 What the autoencoder results mean

AE2 is the stronger of the two autoencoder variants on overall accuracy in all four with-Elo settings: its scores range from 51.8% to 57.6%, versus 46.3% to 50.7% for AE1. In the MatchForm-5 final-season experiment, AE2 reaches 57.6% accuracy, close to the project's best conventional model result of 58.1% from Bagging and equal to the Decision Tree's 57.6%. However, AE2 has zero draw recall in all four with-Elo settings. It attains its accuracy by classifying home and away wins reasonably often while entirely missing the draw class. Its 47.3% balanced accuracy and 41.8% macro F1 on MatchForm-5 final-season data reflect that weakness.

AE1 performs worse overall and also detects few draws: its draw recall is 2.5–3.5% across the four with-Elo settings. The classifier chooses whichever class-specific autoencoder has the smallest mean squared reconstruction error. This assumes that errors from separately trained models are directly comparable and that each class has a distinctive, compact feature pattern. Those assumptions may not hold: match outcomes can have overlapping form profiles, and draw matches can be especially heterogeneous. The draw-specific autoencoder also sees fewer training examples than the home-win autoencoder, which can make its reconstruction estimates less reliable. The current method does not calibrate class priors or decision thresholds.

AE2 learns a 16-dimensional representation by minimizing reconstruction loss over all training examples, without using outcome labels in that representation-learning stage. The later classifier is supervised, but it uses ordinary cross-entropy with no class weighting in the reported runs, and predicts by maximum class probability. A representation that preserves variation useful for reconstructing team statistics is not necessarily one that separates draws from wins. With fewer draws and accuracy-oriented decision behavior, a classifier can maximize overall correctness by rarely or never predicting a draw. This matches the observed zero draw recalls, although the current experiments do not isolate which mechanism is responsible.

The appropriate conclusion is that AE2 is a promising accuracy baseline, not evidence that autoencoders improve three-way prediction. The tests show no improvement in balanced performance over the strongest reported conventional model, and they expose a failure on draws. AE1 does not outperform AE2 or the strongest baselines in the reported settings. Because each variant uses one fixed architecture and training schedule, and the report supplies no repeated-seed or uncertainty analysis, these findings describe the tested configurations rather than the full potential of autoencoders.

## 5. Limitations and reproducibility notes

- **Not an exact replication:** project predictors use Vaastav/FPL aggregates and locally computed Elo; the paper uses FBref features and ClubElo, and its time-series representation retains different historical information.
- **Potential source-semantic assumptions:** missing additive statistics are replaced with zero; FPL fantasy measures such as influence, creativity, and threat are not equivalent to match-event variables in the reference study.
- **Different sample counts after feature construction:** the raw 2,660-fixture set is reduced by rolling-history availability.
- **Class imbalance:** the small draw class and tendency of several models to predict few or no draws limit practical interpretation.
- **Split variance and test-time information:** the stratified random split is not strictly future-only. In the final-season experiment, later test fixtures use earlier test-season outcomes/statistics in their rolling form and Elo inputs. This is reasonable for a sequential in-season prediction setting where each result becomes known before the next fixture, but would be optimistic for a batch forecast of the entire season. The final-season split also covers only one test season.
- **Reported results provenance:** values in this document are transcribed from the supplied results report. The notebook has not been rerun as part of preparing this repository; package versions, hardware, and deterministic behavior may affect regenerated scores.
- **Model selection and reporting:** grid search optimizes accuracy in the reference-comparison settings. This objective can favor majority classes. Future work should tune for macro F1 or balanced accuracy and report repeated temporal holdouts.

## 6. Conclusion

This study reproduces the broad question and selected model comparisons of Wiechno, Bartosik, and Duch, while testing them on a different Vaastav/FPL feature pipeline. The project does not exactly reproduce the article: it uses different feature definitions, locally computed Elo, different model-ready rows, and independently implemented models. The resulting accuracy differences are modest but configuration-dependent: the project is lower in three of the four reported Siamese comparisons and slightly higher in the HomeAwayForm-3 final-season case. The results should therefore be read as an adaptation and comparison, not as evidence that the original method has been independently matched or contradicted.

For this dataset and protocol, historical form and Elo support moderate three-way prediction, but draw recognition remains the main unresolved problem. AE2 is competitive on overall accuracy and reaches 57.6% in the MatchForm-5 final-season test, yet its zero draw recall across all with-Elo runs means it does not deliver balanced outcome prediction. AE1 is weaker and likewise rarely recognizes draws. Thus, the additional autoencoder tests do not show a benefit over the strongest conventional/Siamese models for the full three-class task. They instead demonstrate that unsupervised reconstruction objectives can produce useful general representations without learning the minority-class boundaries needed for draw prediction.

The next experiments should keep the data and split fixed while changing one component at a time: compare the autoencoder latent classifier with and without class-weighted or focal loss; tune the draw decision threshold on a validation set rather than the test set; compare against a supervised bottleneck MLP and a denoising autoencoder; and repeat final-season evaluation across multiple seasons or rolling-origin folds. Report macro F1, balanced accuracy, draw precision/recall, and uncertainty across seeds alongside accuracy. Those tests would show whether the current draw failure is due mainly to class imbalance, the reconstruction objective, or the available features.

## Reference

Wiechno, W., Bartosik, B., & Duch, P. (2025). *Time Series and Deep Learning Approaches for Predicting English Premier League Match Outcomes*. Proceedings of the 17th International Conference on Agents and Artificial Intelligence (ICAART 2025). https://doi.org/10.5220/0013182500003890
