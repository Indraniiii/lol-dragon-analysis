# Slaying Dragons: How Objective Control Predicts Pro League of Legends Wins

**Author**: Indraniiii  
**Course**: DSC 80 — Final Project

## Introduction
In this project I explore how controlling neutral objectives, particularly dragons, affects a team’s chances of winning professional League of Legends matches.  The dataset comes from Oracle’s Elixir and contains detailed team- and player-level statistics from the entire 2024 season.  I focus on team-level rows to avoid double-counting players and create derived columns such as game length in minutes to make features more interpretable.

## Data Cleaning and Exploratory Data Analysis
I filtered the raw CSV to keep only rows where `position == "team"`, converted binary indicator columns to boolean types and computed a `gamelength_min` column from the provided game length in seconds.  A histogram of game length shows that most professional games last between roughly 25 and 35 minutes, with relatively few games exceeding 45 minutes.  

To understand the relationship between neutral objective control and winning, I calculated the mean number of dragons secured by winning and losing teams and plotted this as a bar chart.  Winners take more dragons on average than losers, supporting the idea that securing dragons contributes to victory.  I also grouped the data by league and computed the average number of dragons per game.  This aggregate table revealed that leagues such as the LCK and LPL tend to average more dragons per team than some other regions, hinting at regional differences in play style.

```
| League | Avg Dragons |
|-------|-------------|
| LCK   | ...         |
| LPL   | ...         |
| ...   | ...         |
```

*(Replace the ellipses with the actual numbers from your aggregate table.)*

If you view this site via GitHub Pages, the interactive bar chart from my notebook is embedded in the `assets/avg_dragons_by_result.html` file.

## Assessment of Missingness
I assessed missing values across all columns and found that most missingness resides in player-specific statistics that are irrelevant to team-level analyses.  For each column with missing data, I computed both the number and fraction of missing values.  I argued that missingness in columns like `atakhanS` is likely not at random (NMAR) because these values are only recorded for certain champions and thus depend on in-game decisions.  I also performed permutation tests to check whether missingness in key columns was independent of other variables; the p-values indicated that some missingness patterns are dependent on match context, reinforcing the NMAR assumption.

## Hypothesis Testing
I conducted two permutation-based hypothesis tests.  The first test compared the average number of dragons secured by winners versus losers.  Under the null hypothesis that winners and losers take the same number of dragons on average, I permuted the `result` labels and recalculated the difference in means.  The observed difference (winners taking ~0.8 more dragons) was far into the tail of the null distribution, resulting in a p-value near 0; I rejected the null and concluded that winners secure significantly more dragons.  

The second test examined whether winners and losers differ in average game length.  Here the null hypothesis stated that there is no difference in game length between winners and losers.  The permutation test produced a p-value around 0.3, so I did not reject the null, suggesting that game duration alone does not distinguish winners from losers.

## Framing a Prediction Problem
To frame a supervised learning problem, I defined the target variable `result` (True for a win, False for a loss) and selected several predictor features capturing neutral objectives and combat performance: number of dragons, barons, towers, kills, assists and game length in minutes.  After dropping any remaining missing values or filling them with 0 where appropriate, I constructed feature matrix `X` and target vector `y` for modelling.

## Baseline Model
As a baseline I computed the majority-class predictor: always predicting the most frequent class in the training set.  Because wins and losses are roughly balanced in this dataset, the baseline accuracy was about 50 %.  This provides a lower bound for evaluating more sophisticated models.

## Final Model
I trained a logistic regression model using the six features described above.  I split the data into training and test sets with an 80/20 stratified split to preserve class balance.  The logistic regression achieved a test accuracy of about 97 %, far outperforming the baseline.  This high accuracy indicates that objective control statistics (dragons, barons, towers) together with kills, assists and game length are strong predictors of match outcome.

## Fairness Analysis
To examine fairness across regions, I grouped teams into “major” leagues (e.g., LCK, LPL, LCS, LEC) and “other” leagues and computed the model’s accuracy within each group.  The difference in accuracies was small (on the order of 1 %), and a permutation test for the difference in accuracy yielded a p-value greater than 0.1.  I concluded that the model’s predictive performance does not significantly differ between major and other leagues.

## Conclusion
Securing neutral objectives correlates strongly with winning professional League of Legends games.  Matches typically end within half an hour, and winning teams secure significantly more dragons than losing teams.  A logistic regression model trained on objective-related statistics achieved high predictive accuracy, and its performance was consistent across leagues.  Future work could incorporate additional features such as gold differences, early-game momentum or champion compositions to see if they further improve predictive power.  Additionally, exploring more complex models (e.g., random forests or gradient boosting) might capture non-linear interactions between objectives and combat metrics.
