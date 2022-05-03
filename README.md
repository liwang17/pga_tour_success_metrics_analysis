 ---
title: "PGA Tour Success Metrics Analysis"
author: "Li Wang"
date: "`r Sys.Date()`"
output:
  html_document: github_document
  pdf_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
\

## Business Problem:
What metric is the best predictor of success on the modern PGA Tour? If I were a PGA Tour professional, what area of my game should I focus on in order to make the most money possible, while shooting the lowest scores possible?\
\

## Metrics:
The success metrics are defined as:\
\
**Earnings Per Event ($USD)**, a measure of how much money a player made over the timeline of the analysis per tournament, calculated by dividing the sum of their earnings by the number of events they played (the higher, the better). *Missed cuts and withdrawals, both medical and non-medical, automatically translate to $0 of earnings for the event.*\
\
**Scoring Average**, a measure of how many strokes it takes a player to complete a round, calculated by the total number of strokes a player makes divided by the total number of rounds they have played (the lower, the better).\
\
The input metrics are:\
\
**Driving Accuracy (%)**, a measure of how accurate a player’s tee shots are, calculated by dividing the total number of times a player’s tee shot finished in the fairway by the total number of tee shots they hit (the higher, the better);\
\
**Driving Distance (yards)**, a measure of how far a player’s average tee shot travels (the higher, the better);\
\
**Greens in Regulation (%)**, a measure of how accurate a player’s approach shots are, calculated by dividing the total number of times a player’s approach shot finishes on the putting green by the total number of approach shots they hit (the higher, the better);\
\
**Putts Per Round**, a measure of how many strokes a player takes to put the ball in the hole once reaching the putting green, calculated by dividing the total number of putts a player had by the total number of holes the player played (the lower, the better);\
\
**Scrambling (%)**, a measure of how often a player gets “up and down” (when a player puts the ball in the hole in two or fewer strokes after their approach shot missed the putting green), calculated by dividing the total instances of successfully getting up and down by the total number of times a player missed the putting green on their approach shot (the higher, the better);\
\

## Data Source:
The [Official PGA Tour website](https://www.pgatour.com/stats.html) keeps raw data of all of the above metrics, from 1980 to the present. However, I am only interested in the best predictor of success on the "modern" PGA Tour; due to two reasons, tournaments conducted prior to 2010 are not considered modern enough to be consistent with those conducted in the 12 years since. Firstly, recent advances in technology have allowed drivers to go further, irons to be more accurate, and putters to be more consistent, which has effectively made the game easier, and therefore the stats would heavily favor the post-2010 data. Secondly, the sizes of the purses from 1980 to 2010 dramatically increased over the years due to more sponsors and more fan attendance; while the rate of increase of modern-day purses since 2010 has been negligible in comparison, offering more consistency. As a result, the data for this analysis includes only the years 2010-2021.\
*Note also that the 2021-2022 season is only halfway complete, so data from 2022 will not be used.*\
\

## Processing the Data:
\
**Step 1:** Obtained the appropriate data from the official PGA Tour website, and used Excel to clean the data so that it is uniform and properly formatted for the next step. \
*	Found and deleted duplicates across all tables;\
*	Used business sense to find and replace null values with the correct values (i.e. in the “victories” field of the “money_made” table, if no record was found for that field, it meant that the player did not win any tournaments that year);\
* Appropriately renamed field headers across all tables, to make them consistent and more descriptive (i.e. renaming the “gir” field to “greens_in_regulation” to make it easier to understand);\
*	Created two matching keys across all data tables for the future joining of as many tables as necessary (the “player” and “year” fields);\
*	Assigned corresponding year values to each record across all tables;\
*	Exported each metric as an individual data table in respective .csv files.\
\
**Step 2:** Imported the data from the respective .csv files into Google Big Query and used the sandbox SQL platform to aggregate, filter, and sort the data for the next step. \
*	Created a new project titled “lw-capstone” inside of the SQL workspace;\
*	Created seven data tables by importing the seven .csv files created earlier;\
* Inside a nested query, joined all seven tables together using LEFT JOIN’s on a joint primary key created by “player” and “year” – the reason for joining is to combine the data from all tables into one table, the reason for using two keys jointly is to make sure all records match player names as well as the corresponding years they played, and the reason for using a LEFT JOIN rather than an INNER JOIN is that not all players were on the PGA Tour for all 12 years, but I still want to return their stats for the years in which they were active;\
* Within the same nested query, returned and renamed only the fields that I need;\
* In the outer query, aggregated the data by created and returning summarized values for all of the targeted metrics, grouped by player, filtered by players who have averaged more 10 or more events per year over the course of the 12 years (to ensure large enough of a sample size), and sorted the data from most earnings per event to least earnings per event;\
* Exported the data into a .csv file called “pga_aggregate_stats.csv” for further analysis.\
\

#### Here's the SQL query in its full form:
\
*You can also access the code on GitHub [HERE](https://github.com/liwang17/pga_tour_success_metrics_analysis/blob/main/sql_query)*\
\
![SQL Query](/Users/liwang/Desktop/Learning/Google Data Analytics Certificate/Capstone Project/full_sql_query.png) \
\

## Analyzing the Data:
\
**Step 1:** Launched R-Studio, installed and loaded the "tidyverse", "readr", "ggplot2", and "dplyr" packages used for data analysis, imported the data from the previously generated .csv file using the read.csv function, and saved the data as a dataframe called "pga_tour_stats": \
\
```{r prep1, include=FALSE}
install.packages("tidyverse", repos = "http://cran.us.r-project.org")
install.packages("readr", repos = "http://cran.us.r-project.org")
install.packages("ggplot2", repos = "http://cran.us.r-project.org")
install.packages("dplyr", repos = "http://cran.us.r-project.org")

library(tidyverse)
library(readr)
library(ggplot2)
library(dplyr)
```

```{r prep2, echo=TRUE, warning=FALSE}
setwd("~/Desktop/Learning/Google Data Analytics Certificate/Capstone Project")
pga_stats <- read.csv("pga_aggregate_stats.csv")
as_tibble(pga_stats)
```

\
**Step 2:** Calculated the correlation coefficients of all five input metrics (Driving Accuracy, Driving Distance, Greens in Regulation, Putts Per Round, and Scrambling) against the success metrics (Earnings Per Event and Scoring Average).\
\
The official definition of the correlation coefficient is: *"The correlation coefficient is a statistical measure of the strength of the relationship between the relative movements of two variables. The values range between -1.0 and 1.0. A calculated number greater than 1.0 or less than -1.0 means that there was an error in the correlation measurement." -[Investopedia](https://www.investopedia.com/terms/c/correlationcoefficient.asp#:~:text=The%20correlation%20coefficient%20is%20a,error%20in%20the%20correlation%20measurement.)*\
\
```{r analyze1, warning=FALSE, include=FALSE}
earnings_correlations <- pga_stats %>% 
  summarise(driving_accuracy_earnings = cor(driving_accuracy_percent,
                                            earnings_per_event,
                                            use = "complete.obs"),
              driving_distance_earnings = cor(avg_driving_distance,
                                            earnings_per_event,
                                            use = "complete.obs"),
              gir_earnings = cor(gir_percent, earnings_per_event,
                                 use = "complete.obs"),
              putting_earnings = cor(putts_per_round, earnings_per_event,
                                     use = "complete.obs"),
              scrambling_earnings = cor(scrambling_percent, earnings_per_event,
                                        use = "complete.obs")
              )


scoring_correlations <- pga_stats %>% 
  summarise(driving_accuracy_scoring= cor(driving_accuracy_percent,
                                            scoring_average,
                                            use = "complete.obs"),
              driving_distance_scoring = cor(avg_driving_distance,
                                            scoring_average,
                                            use = "complete.obs"),
              gir_scoring = cor(gir_percent, scoring_average,
                                 use = "complete.obs"),
              putting_scoring = cor(putts_per_round, scoring_average,
                                     use = "complete.obs"),
              scrambling_scoring = cor(scrambling_percent, scoring_average,
                                        use = "complete.obs")
              )
```

**Correlation Coefficients of Earnings Per Event vs. the Input Metrics:**\

```{r analyze2, echo=TRUE, warning=FALSE}
as_tibble(earnings_correlations)
```
As expected, all of the correlation coefficients were positive except for Putting, because in each of the other cases, a higher number is seen as better, but Putting is the only metric where less is better. It is also nice to see that the coefficients were all between -1.0 and +1.0, which means that there were no errors in calculation.\
\
Based on these results, **Driving Distance (0.3612) and Scrambling (0.3344) are the metrics most closely correlated with earning more money per event**, while Greens in Regulation (0.2990) and Putting (-0.2949) still had an impact but not as much, and Driving Accuracy (0.0096) did not make much of a difference at all. Moving onto the other success metric, Scoring Average, we get the following results:\
\
**Correlation Coefficients of Scoring Average vs. the Input Metrics:**\

```{r analyze3, echo=TRUE}
as_tibble(scoring_correlations)
```
Just like above, at first glance, the results were as expected; all of the correlation coefficients were negative except for Putting, because the success metric of Scoring Average is now defined as less-is-better. In addition, the coefficients were again all between -1.0 and +1.0, meaning there were no errors in calculation.\
\
Based on these results, **Scrambling (-0.6388) and Greens in Regulation (-0.6215) are the metrics most closely correlated with shooting lower scores**, while Putting (0.2842) and Driving Distance (-0.2786) still had an impact but not as much. Once again, Driving Accuracy (-0.2479) made the least difference, although its effect on Scoring Average was much more significant than on Earnings Per Event.\
\
\
**Step 3:** Calculated two correlation coefficients between metrics themselves (Driving Accuracy vs. Driving Distance, and Earnings Per Event vs. Scoring Average):\
\
```{r analyze4, include=FALSE}
other_correlations <- pga_stats %>% 
  summarise(driving_correlations= cor(driving_accuracy_percent,
                                            avg_driving_distance,
                                            use = "complete.obs"),
              earnings_scoring = cor(earnings_per_event,scoring_average,
                                     use = "complete.obs")
              )
```

```{r analyze5, echo=TRUE, warning=FALSE}
as_tibble(other_correlations)
```
\
Both correlation coefficients showed statistically significant relationships. **In the case of Driving Accuracy vs. Driving Distance (-0.5803), there was a noticeable negative correlation, which makes sense because as a player hits the ball further, they are naturally less accurate. Likewise, in the case of Earnings Per Event vs. Scoring Average (-0.6509), there was also a noticeable negative correlation, which also makes sense, because as a player scores lower in tournaments, they will naturally finish higher in the events and make more money.** But with both, the correlation was not close enough to -1.0 that they could be considered strictly correlated with each other, because there are other variables that impact them (namely, the other input metrics).\
\

## Data Visualizations:
Visually, the relationship between these variables can be plotted against each other like this:\
\

### Correlation Coefficients of Earnings Per Event vs. the Input Metrics:
```{r visualize1, echo=FALSE, warning = FALSE}
ggplot(data = pga_stats, mapping = aes(x = driving_accuracy_percent,
                                       y = earnings_per_event)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Earnings vs. Driving Accuracy (cor = 0.0096)",
       x = "Driving Accuracy (%)",
       y = "Earnings Per Event ($USD)")

ggplot(data = pga_stats, mapping = aes(x = avg_driving_distance,
                                       y = earnings_per_event)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Earnings vs. Driving Distance (cor = 0.3612)",
       x = "Driving Distance (yards)",
       y = "Earnings Per Event ($USD)")

ggplot(data = pga_stats, mapping = aes(x = gir_percent,
                                       y = earnings_per_event)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Earnings vs. Greens in Regulation (cor = 0.2990)",
       x = "Greens in Regulation (%)",
       y = "Earnings Per Event ($USD)")

ggplot(data = pga_stats, mapping = aes(x = putts_per_round,
                                       y = earnings_per_event)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Earnings vs. Putts Per Round (cor = -0.2949)",
       x = "Putts Per Round",
       y = "Earnings Per Event ($USD)")

ggplot(data = pga_stats, mapping = aes(x = scrambling_percent,
                                       y = earnings_per_event)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Earnings vs. Scrambling (cor = 0.3344)",
       x = "Scrambling Success Rate (%)",
       y = "Earnings Per Event ($USD)")
```
\
\
\
\
\
\

### Correlation Coefficients of Scoring Average vs. the Input Metrics:
```{r visualize2, echo=FALSE, warning = FALSE}
ggplot(data = pga_stats, mapping = aes(x = driving_accuracy_percent,
                                       y = scoring_average)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Scoring Average vs. Driving Accuracy (cor = -0.2479)",
       x = "Driving Accuracy (%)",
       y = "Strokes Per Round")

ggplot(data = pga_stats, mapping = aes(x = avg_driving_distance,
                                       y = scoring_average)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Scoring Average vs. Driving Distance (cor = -0.2786)",
       x = "Driving Distance (yards)",
       y = "Strokes Per Round")

ggplot(data = pga_stats, mapping = aes(x = gir_percent,
                                       y = scoring_average)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Scoring Average vs. Greens in Regulation (cor = -0.6215)",
       x = "Greens in Regulation (%)",
       y = "Strokes Per Round")

ggplot(data = pga_stats, mapping = aes(x = putts_per_round,
                                       y = scoring_average)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Scoring Average vs. Putts Per Round (cor = 0.2842)",
       x = "Putts Per Round",
       y = "Strokes Per Round")

ggplot(data = pga_stats, mapping = aes(x = scrambling_percent,
                                       y = scoring_average)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Scoring Average vs. Scrambling (cor = -0.6388)",
       x = "Scrambling Success Rate (%)",
       y = "Strokes Per Round")
```
\
\
\
\
\
\

### Correlation Coefficients Between Metrics Themselves:
```{r visualize3, echo=FALSE, warning = FALSE}
ggplot(data = pga_stats, mapping = aes(x = avg_driving_distance,
                                       y = driving_accuracy_percent)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Driving Distance vs. Driving Accuracy (cor = -0.5803)",
       x = "Driving Distance (yards)",
       y = "Driving Accuracy (%)")

ggplot(data = pga_stats, mapping = aes(x = avg_driving_distance,
                                       y = scoring_average)) +
  geom_point() +
  geom_smooth(formula = y ~ x, method = "lm") +
  labs(title = "Scoring Average vs. Earnings Per Event (cor = -0.6509)",
       x = "Earnings Per Event ($USD x 100)",
       y = "Strokes Per Round")
```
\
\

## Conclusions and Recommendations
Before tying these numbers back to the business problem at hand (what is the best predictor of success on the PGA Tour?), let's first recap the findings of the analysis:\
\
* Driving Distance (cor: 0.3612) and Scrambling (cor: 0.3344) are the metrics most closely correlated with earning more money per event, while Greens in Regulation (cor: 0.2990) and Putting (cor: -0.2949) still had an impact but not as much, and Driving Accuracy (cor: 0.0096) does not make much of a difference at all;\
* Scrambling (cor: -0.6388) and Greens in Regulation (cor: -0.6215) are the metrics most closely correlated with shooting lower scores, while Putting (0.2842) and Driving Distance (cor: -0.2786) still had an impact but not as much. Once again, Driving Accuracy (cor: -0.2479) made the least difference;\
* As Driving Distance increases, Driving Accuracy decreases (cor: -0.5803), and as Scoring Average decreases, Earnings Per Event increases (cor: -0.6509).\
\
Based on the above, I gave each of the input metrics a score of 1-5 for each category, with 5 being the best:\
* Earnings Per Event: (5) Driving Distance, (4) Scrambling, (3) Greens in Regulation, (2) Putting, (1) Driving Accuracy;\
* Scoring Average: (5) Scrambling, (4) Greens in Regulation, (3) Putting, (2) Driving Distance, (1) Driving Accuracy;\
\
It can be assumed that the average PGA Tour player cares far more about money than about his average score, because at the end of the day, they are competing for their livelihoods. However, we should not disregard the significant correlation between lower average scores and higher amounts of money earned, which means that Scoring Average contributes to overall success in other ways, and should still be factored into the end decision. Therefore, I will weight these scores in a 75%-25% split, in favor of Earnings Per Event.\
\
Taking into account the above weighting system, and combining it with the assigned input metric scores, I calculated the total scores for each input metric:\
\
```{r conclusion1, echo=TRUE, warning=FALSE}
earnings_coeff <- 0.75
scoring_coeff <- 0.25

input_scores <- data.frame(input_metrics = c("driving_distance", "scrambling", "greens in reg",
                             "putting", "driving_accuracy"),
           earnings_scores = c(5, 4, 3, 2, 1),
           scoring_scores = c(2, 5, 4, 3, 1))

input_scores <- input_scores %>% 
  mutate(combined_score = (earnings_coeff * earnings_scores) +
           (scoring_coeff * scoring_scores))

arrange(input_scores, desc(combined_score))
```

\
Once the input metrics have all been assigned scores, two of the metrics have distanced themselves from the others and are tied for most important: **Driving Distance and Scrambling**. Business intuition confirms that this indeed makes sense; the "modern" form of golf is widely regarded as favoring "bomb-and-gouge" players, or players that drive the ball a long ways (the "bomb"), and because distance is inversely correlated with accuracy, these same players will ultimately need to play more of their approach shots from the rough (the "gouge"), thus placing a high emphasis on their short games and their ability to scramble!\
\
**In conclusion, while golf is a multi-faceted sport with many variables at play, at the highest levels of golf on the modern PGA Tour, in order to have greater chance of success, a player should focus on increasing their Driving Distance, as well as their Scrambling Percentage, for the highest return on the investment of their training time.**\
\

## Future Considerations
\
In the future, I would love to expand on this current project, and have the following modifications / improvements in mind:\
\
* I would conduct a separate analysis on PGA metrics from the years 1980 to 2010, to see how the game has evolved over time;\
* I would dive deeper into Driving Distance and Scrambling to see if there are particular aspects of these metrics that matter more than others (i.e. "number of drives per season over 300 yards," or "bunker scrambling percentage" vs. "rough scrambling percentage");\
* I would look at other success metrics, such as the number of tournament victories a player has, although I hypothesize that the results would be similar, since the players with making the most money and having the lowest scoring averages are probably also the ones winning the most;\
* I would take a handful of the most successful golfers in the modern era (i.e. Tiger Woods, Phil Mickelson, Rory McIlroy, Dustin Johnson, etc.) and dive deeper into what made them so successful;\
* I would be interested in seeing how these results are similar or different on other major worldwide professional golf tours such as the DP World Tour (formerly the European Tour); the PGA Tour competes primarily in the United States, and maybe the courses that players play on elsewhere would not reward the "bomb-and-gouge" playing style in the same way;\
* I would be interested in diving deeper into any outliers found by this study (i.e. players who are low on Driving Distance, but have still made a lot of money per event and shoot low scores) to find out how and why;\
* I would be interested in seeing which input metrics correlate the most strongly with Scoring Average for the average, 15-handicap golfer.
