# Forecasting the Beautiful Game

**Bayesian Inference and Graphical Models for World Cup 2026 Prediction**

LASR 2026 Workshop, School of Mathematics, University of Leeds

## Overview

This repository contains the code, data pipeline, and visualisations behind a poster presented at the LASR 2026 Workshop. The project fits a Dixon-Coles model to international football results and uses it to simulate the 2026 FIFA World Cup, producing calibrated win probabilities for every contending team rather than single point predictions.

The central idea is simple: most football forecasters report a single predicted winner. This project instead treats team strength as a probability distribution, updates that distribution using Bayesian inference as match data arrives, and checks whether the resulting forecasts are actually trustworthy by testing them against held out matches.

## Contents

| File | Description |
|------|-------------|
| `WorldCup2026_Poster_Visuals.ipynb` | Full Google Colab notebook. Downloads data, fits the model, runs simulations, generates all poster charts, and finishes with a dedicated Argentina win probability estimate |
| `Poster.pptx` | Final poster, PowerPoint format |

## Dataset

International football results (1872 to present), compiled by Mart Jurisoo.

- Source: https://github.com/martj42/international_results
- Direct CSV: https://raw.githubusercontent.com/martj42/international_results/master/results.csv
- Free, public domain, no registration required
- The notebook downloads this automatically; no manual download needed

The pipeline filters to matches from 2010 onward involving 20 likely World Cup 2026 contenders, across major tournaments and international friendlies.

## Method

### 1. The Poisson goal model

Goals scored by team *i* against team *j* are modelled as independent Poisson random variables:

```
G_ij ~ Poisson(lambda_ij)
lambda_ij = alpha_i * delta_j * gamma
```

where `alpha_i` is attack strength, `delta_j` is defensive weakness, and `gamma` is home advantage.

### 2. Dixon-Coles correction

A correction term `rho` adjusts the joint probability for low scoring results (0-0, 1-0, 0-1, 1-1), where the plain Poisson independence assumption tends to break down. Parameters are fitted by maximum likelihood using bounded SLSQP optimisation.

### 3. Bayesian framing

The model is expressed as a probabilistic graphical model, with team strength parameters as latent nodes updated via Bayes' theorem as data arrives:

```
P(theta | data) is proportional to P(data | theta) * P(theta)
```

This produces full posterior distributions over team strength rather than fixed point estimates, which is what allows the tournament simulation to express genuine uncertainty rather than a single predicted bracket.

### 4. Tournament simulation

The fitted model is used to simulate a full knockout tournament 50,000 times, sampling goals from the fitted Poisson rates at each stage. Draws are resolved with a 50 percent penalty shootout. The proportion of simulations won by each team gives its estimated win probability.

### 5. Calibration check

Predictions are tested on a held out set of matches not used during fitting. If the model says a team has a 70 percent chance of winning, that result should occur close to 70 percent of the time. This is what distinguishes a genuinely calibrated forecast from an arbitrarily confident one.

### 6. Team-specific case study: Argentina

The notebook closes with a dedicated query computing Argentina's tournament win probability, including a bootstrap 95 percent confidence interval and a comparison of Argentina's fitted attack and defence ranks against the rest of the field. The same pattern can be reused for any team by changing a single variable.

## Running the code

### Option A: Google Colab (recommended)

1. Download `WorldCup2026_Poster_Visuals.ipynb`
2. Open https://colab.research.google.com and upload the notebook
3. Run all cells in order

No manual setup needed. The first cell installs all required packages.

### Option B: Local Python

```bash
pip install scipy pandas numpy matplotlib
python worldcup_poster_visuals.py
```

### Optional: full Bayesian MCMC

A commented out section in the notebook uses PyMC to run full MCMC sampling instead of maximum likelihood point estimates, producing genuine posterior distributions over team parameters. This is slower (around 5 to 10 minutes on Colab) and requires:

```bash
pip install pymc arviz
```

## Outputs

Running the full pipeline produces six charts, all used directly in the poster, plus a printed team-specific win probability summary:

1. Poisson goal distribution for a sample match
2. Attack and defence parameter rankings across all teams
3. Tournament win probabilities from the simulation
4. Bayesian prior to posterior update illustration
5. Plate diagram of the graphical model structure
6. Calibration plot on held out matches
7. Argentina win probability estimate with bootstrap confidence interval (printed output, end of notebook)

## Key references

- Dixon, M.J. and Coles, S.G. (1997). Modelling Association Football Scores and Inefficiencies in the Football Betting Market. *Applied Statistics*, 46(2), 265 to 280.
- Maher, M.J. (1982). Modelling Association Football Scores. *Statistician*, 31(1), 68 to 72.
- Karlis, D. and Ntzoufras, I. (2003). Analysis of Sports Data by Using Bivariate Poisson Models. *The Statistician*, 52(3), 381 to 393.
- Groll, A. et al. (2018). Prediction of the FIFA World Cup 2018, A Random Forest Approach. *Statistical Modelling*, 19(5).
- Gelman, A. et al. *Bayesian Data Analysis*, 3rd edition. CRC Press.

## Acknowledgements

Thanks to the LASR 2026 Organising Committee, Dr Luisa Cutillo, Dr Sofya Titarenko, and Professor Kanti Mardia, for the opportunity to present this work. This project used the publicly available results dataset compiled by Mart Jurisoo (martj42) on GitHub, and was built using Python, SciPy, PyMC, and Matplotlib.

## Author

Krithik Sharan Suresh Alagianayagi,
MSc Data Science and Analytics, School of Mathematics, University of Leeds
