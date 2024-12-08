# League of Legends Map Control Statistical Analysis
Lynn Jegal
lynjeg@umich.edu

# Introduction
## General Introduction
League of Legends (LOL) is a popular 5v5 online game that has gained a massive esports following throughout the years. The dataset used in this project is a professional dataset developed by Oracle's Elixir that records match data from professional League of Legends matches in 2022.

In LOL, map control refers to the control a team has over a part of the map. A team with good map control is able to prevent the enemy from easily entering parts of the map. Some aspects that play into map control include vision control (placing and removing wards around the map) and taking objectives like turrets. These are actions that make it more difficult for the enemy to move throughout the map due to lack of information or fighting advantage.

Using this dataset, I will be investigating the significance of map control in determining the outcome of LOL matches.

## Introduction to the Data
The dataset has 150180 rows, corresponding to 12515 games, and has columns featuring various gameplay metrics and outcomes that will be useful to our investigation. Here's a brief introduction to the columns that will be relevant in our project:
- `gameid`: This column represents the identifier for every game played.
- `result`: This column represents the outcome of the game where 0 indicates that the team lost and 1 indicates that the team won.
- `gamelength`: This column represents how long the game took to finish.
- `complete`: This column represents the completeness of the game. If a game was played out completely, it has a value of 'complete', and if a game was stopped midway it is marked 'partial'.
- `position`: This column represents the position of the player in the game. It also allows for differentiation between rows that show player vs team statistics.
- `wardsplaced`: This column represents the number of wards placed by the player or team.
- `firsttower`: This column represents if the team was the first to destroy the first tower in the game.
- `firstmidtower`: This column represents if the team was the first to destroy the opponent's mid tower.
- `turretplates`: This column represents the number of turret plates that the team was able to get.

# Data Cleaning and Exploratory Data Analysis
## Data Cleaning
For data cleaning, I first filtered to keep only the complete games because the games marked 'partial' wouldn't be helpful in determining the outcome of the game. Next, I filtered to only keep the rows showing the team statistics because I was only interested in the overall team's map control. I also only kept the relevant columns: `gameid`, `result`, `wardsplaced`, `gamelength`, `firsttower`, `firstmidtower`, `firsttothreetowers`, and `turretplates`. I also created another column called `wardsplacedpermin` based on the `wardsplaced` and `gamelength` columns.

Below is part of the head of our cleaned dataframe (only displaying some of the columns for the sake of space).

| gameid                |   result |   wardsplaced |   gamelength |   firsttower |   turretplates |   wardsplacedpermin |
|:----------------------|---------:|--------------:|-------------:|-------------:|---------------:|--------------------:|
| ESPORTSTMNT01_2690210 |        0 |            74 |         1713 |            1 |              5 |             2.59194 |
| ESPORTSTMNT01_2690210 |        1 |            93 |         1713 |            0 |              0 |             3.25744 |
| ESPORTSTMNT01_2690219 |        0 |           119 |         2114 |            0 |              2 |             3.37748 |
| ESPORTSTMNT01_2690219 |        1 |           129 |         2114 |            1 |              3 |             3.66131 |
| ESPORTSTMNT01_2690227 |        1 |           119 |         1972 |            1 |              1 |             3.62069 |

## Univariate Analysis
I performed univariate analysis on the wards placed per minute per team. 

<iframe
  src="assets/univariate1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram shows that the distribution of wards placed per minute is basically normal, with most teams placing around 3 wards per minute

## Bivariate Analysis
I performed bivariate analysis on the wards placed per minute and result statistics in the dataset to visualize the difference in the wards placed per minute between winning teams and losing teams.

<iframe
  src="assets/bivariate1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

According to the plot, teams who won had more wards placed per minute compared to losing teams.

## Interesting Aggregates
Here are some interesting aggregates: 

| result       |   wardsplaced |   wardsplacedpermin |   firsttower |   firstmidtower |   firsttothreetowers |   turretplates |
|:-------------|--------------:|--------------------:|-------------:|----------------:|---------------------:|---------------:|
| loss         |       1009277 |             31356.5 |         3361 |            2889 |                 2350 |          40671 |
| win          |       1047418 |             32696.6 |         7261 |            7732 |                 8271 |          58584 |

I first groupby the result of the game then calculate the sum of the statistics. The winning team had better statistics all around with more vision placed and better first tower statistics.

## Imputation
The only imputation that had to be done was on the `firstmidtower` column in one game where there was no mid tower broken by either team. Both rows were filled with 0s to indicate that neither team was the first to break the mid tower.

# Framing a Prediction Problem

For the prediction problem, I will do a binary classification to predict the outcome of a League of Legends game (win or loss) based on a team's initial map control. I will only use statistics involving the first team to get certain tower objectives. I will not use vision-related statistics because the vision-related statistics in this dataset are only complete once the full game has been played out. Therefore, the vision-related statistics would not be known at the time of prediction. I am using accuracy to evaluate the model because the instances of my response classes of 'win' and 'loss' are balanced in the dataset.

# Baseline Model

For the baseline model, I used a Logistic Regression model with the features `firsttower` and `firstmidtower`, both of which are nominal features. Because they were already numerical features, I didn't have to perform any encodings.

After fitting the model, the accuracy score was 0.7385, which means that the model was able to correctly predict 73.85% of the data. This model seems like an okay predictor for the game outcome based on the accuracy score, but can likely improve with the addition of more features and better tuning.

# Final Model
In the final model, I added two more features of `turretplating`, an ordinal feature, and `firsttothreetowers`, a nominal feature. I used StandardScaler Transformer to transform `turretplating` into standard scale, but didn't need any additional encoding for `firsttothreetowers` because it was already in binary form. I added these two features because the statistics `turretplating` and `firsttothreetowers` are good indicators of how good a team's map control was toward the beginning of the game. 

The final model used a Random Forest Classifier with tuning on the number of estimators, max depth, and the minimum samples split. For the hyperparameters, I used grid search to find that the best number of estimators is 200, the best max depth is 5, and the best number of min samples split is 5.

The final result had an accuracy of 0.7790, an improvement over the baseline accuracy of 0.7385.