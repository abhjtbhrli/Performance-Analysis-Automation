# Valuing Passes
### _Advanced Analytics & Machine Learning_

**PRELUDE**

In football, aggregate stats like pass completion percentage (or pass completion rate) offer very little detail and insight. 

For example, a centre-back who plays 90% of his passes sideways to his partner centre-back as a means to keep possession and aid gradual build-up play is always going to rank high in the pass completion percentage metric. 

Now, compare the centre-back to an attacking midfielder who is always looking to feed through passes and final balls to the striker. Due to the nature of the attacking midfielder's passes, it is much more difficult to complete them (_because a completed pass means a scoring opportunity, stopping which is the opposition team's main objective_). Therefore, going by pass completion percentage, players like the aforementioned centre-back will always rank higher than the aforementioned attacking midfielder, although the attacking midfielder has the potential to add more value to his team via passing.

Therefore, basic pass completion percentage doesn't quantify the value added by a passer with his passing. We want to know which player adds more value through passing rather than which player completes most sideways passes (which, inadvertently, is a big driver of pass completion percentages).

We can add weights to every pass that happens on the pitch by its difficulty and then subsequently rank players based on their ability to complete passes, easy and difficult. 

For example, a particular pass (_**pass X**_) is historically successfully completed only 50% of the time (_50% completion percent_). Therefore, a player that plays **pass X** and completes it 70% of the time can be said to be a better passer of **pass X** because he completes a difficult pass (_a 50% completion rate makes **pass X** a difficult pass to complete_) at a higher rate than average.

Another example. **Pass Y** is a type of pass that historically has been successfuly played 90% of the time (_which makes **pass Y** an easy pass, as it has been successfully completed 90% of the time_). However, if a player plays **pass Y** and completes it only 70% of the time, it means that he is a worse passer of **pass Y** than the average player.

This entire scheme can be added to every pass that happens on a football pitch and we can subsequently separate good passers from the average ones. With the help of a huge sample of data and some machine learning techniques, we can easily find out the best effective passers in a game of football.

Football is highly variable, hence true passing rates tend to converge only after a considerable sample. For example, a player might misplace all 5 of his first 5 passes in a match. That doesn't mean he is the worst passer ever (_0% completion rate_). He will eventually get his passes correct (passing, afterall, is the very foundation of football and no player will misplace all his passes) and his rate is bound to go up. Therefore, small samples don't offer a clear picture of analyses like this. We use bigger samples (which is easier because one football match can generate in the range of 500-1000 passes) to perform passing value analysis.

**____________________**

Carrying out machine learning tasks in the context of performance analysis is often a labour of love for the analyst. Automating tasks in performance analysis is still at a nascent stage, which is understandable since performance analysts don't usually have coding background and programming is an acquired skill rather than a prerequisite skill.

**____________________**

We use the `tidyverse` and `tidymodels` packages to do almost all our tasks for this project. The `tidyverse` package is a nice all-encompassing package that contains functions and libraries for data cleaning, data wrangling and data visualisation. The `tidymodels` package is a one-stop shop for all our machine learning needs. `tidymodels` is said to have a steep learning curve, but it is super intuitive and helps in easily building our *_Expected Passing_* model.

First, we load the libraries we need.

```
library(tidyverse) 
library(tidymodels)
library(ggsoccer)
library(extrafont)
library(tidyr)
library(readr)
library(magrittr)
```
Next step is importing the datasets. There are two datasets in my locale, _Indian Super League Master.csv_ and _I-League Master.csv_. The first one contains event data from 575 ISL matches since the league's inception in 2014. The second dataset contains event data from the last two I-League seasons.




