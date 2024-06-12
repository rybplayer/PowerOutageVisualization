



## US Coastal Power Outages
Author: Ryan Batubara

## Introduction

According to the United States [National Oceanic and Atmospheric Administration (NOAA)](https://coast.noaa.gov/states/fast-facts/economics-and-demographics.html), almost 40% of the U.S. population live on the coast, despite these areas being less than 10% of the U.S. land mass (excluding Alaska). As such, the reliability of power outages in such areas impact the stability of many people. That said, the coast is not without drawbacks, being prone to disasters such as hurricanes and floods that do not impact inland territories as much, as detailed by the NOAA [here](https://oceanservice.noaa.gov/facts/coastalthreat.html). Of course, this presents new challenges to the power infastructure in such areas.

Thus, the goal for this project is to explore the relationship of a state being coastal — which we define as having a coastline — or inland — which we define as having no coastline — on the power outages it experiences.

To do so we will explore a dataset of major power outages in the U.S. from January 2000 to July 2016 comprising of 1354 rows and 56 data columns. This data was collected by Purdue's Laboratory for Advancing Sustainable Critical Infastructure [here](https://engineering.purdue.edu/LASCI/research-data/outages), and whose complete data description can be found [here](https://www.sciencedirect.com/science/article/pii/S2352340918307182). 

This will serve as our backbone for answering the question **"Does being a costal state impact the frequency and severity of power outages compared to inland states?"**

Due to the large size of the dataset, and the specificity of our data science question, we will only be using a few columns for our analysis, which we break down into the following categories:

### Geographic Data

| Column           | Description                                                                               |
|------------------|-------------------------------------------------------------------------------------------|
| U.S._STATE       | Name of U.S. State of the outage                                                          |
| STATE.CODE       | Two-letter code of the state of the outage. This column was added in data cleaning.        |
| STATE.COASTAL    | `True` if state has coastline, `False` otherwise. This column was added in data cleaning.  |
| CLIMATE.REGION   | U.S. Climate Region, specified by the National Centers for Environmental Information       |
| CLIMATE.CATEGORY | Climate episode (Warm, Normal, Cold) of the region in that year when the outage occurred   |
| TOTAL.CUSTOMERS  | Total number of customers in that U.S. state                                               |

### Outage Data

| Column           | Description                            |
| --------------   | -------------------------------------- |
| OUTAGE.START     | Start time of the outage |
| OUTAGE.DURATION  | Length of the outage in minutes        |
| OUTAGE.RESTORATION | End time of the outage |
| CAUSE.CATEGORY   | Cause of the outage                    |
| HURRICANE.NAMES  | Name of the hurricane that caused the outage. `NaN` if not a hurricane |
| CUSTOMERS.AFFECTED | Number of customers affected by the outage |
| DEMAND.LOSS.MW   | Peak demand lost during outage in Megawatts, but in many cases the total demand is reported here |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

We begin by outlining how we will clean the data:

1. We drop all irrelevant columns and only keep the features outlined above.
2. We convert the `OUTAGE.START.DATE` and `OUTAGE.START.TIME` into one `pd.timestamp` column named `OUTAGE.START`, and similarly combine `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME` into `OUTAGE.RESTORATION`
3. We tidy up the remaining datatypes, casting `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, `TOTAL.CUSTOMERS`, and `DEMAND.LOSS.MW` to `float`. We need to use `float` datatypes rather than `int` to support `np.NaN` values.
3. We tidy up missing information by replacing them with `np.NaN`. In particular, `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, and `DEMAND.LOSS.MW` sometimes have a value of `0`, which does not make sense since an outage that lasted 0 minutes, affected 0 people, or caused 0 MW lost is not an outage. Thus we conclude this data is missing and explicilty make them `np.NaN`.
5. We then add additional columns that pertain to our specific analysis. In particular, we create the column `STATE.CODE` to contain the two-letter code of the state for convenience in visualization.  We also add the boolean column `STATE.COASTAL` which is `True` if the state has access the coast, and `False` otherwise. The costal states are, in order of decreasing length of coastline: Alaska, Florida, California, Hawaii, Louisiana, Texas, North Carolina, Oregon, Maine, Massachusetts, South Carolina, Washington, New Jersey, New York, Virginia, Georgia, Connecticut, Alabama, Mississippi, Rhode Island, Maryland, Delaware, and New Hampshire, taken from [this table by the NOAA](https://coast.noaa.gov/data/docs/states/shorelines.pdf).

### Data Cleaning Recap

The first few rows of the cleaned DataFrame are shown below:

|    | U.S._STATE   | STATE.CODE   | STATE.COASTAL   | CLIMATE.REGION     | CLIMATE.CATEGORY   |   TOTAL.CUSTOMERS |   CUSTOMERS.AFFECTED | CAUSE.CATEGORY     |   HURRICANE.NAMES | OUTAGE.START        | OUTAGE.RESTORATION   |   OUTAGE.DURATION |   DEMAND.LOSS.MW |
|---:|:-------------|:-------------|:----------------|:-------------------|:-------------------|------------------:|---------------------:|:-------------------|------------------:|:--------------------|:---------------------|------------------:|-----------------:|
|  0 | Minnesota    | MN           | False           | East North Central | normal             |       2.5957e+06  |                70000 | severe weather     |               nan | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |              3060 |              nan |
|  1 | Minnesota    | MN           | False           | East North Central | normal             |       2.64074e+06 |                  nan | intentional attack |               nan | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |                 1 |              nan |
|  2 | Minnesota    | MN           | False           | East North Central | cold               |       2.58690e+06 |                70000 | severe weather     |               nan | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |              3000 |              nan |
|  3 | Minnesota    | MN           | False           | East North Central | normal             |       2.60681e+06 |                68200 | severe weather     |               nan | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |              2550 |              nan |
|  4 | Minnesota    | MN           | False           | East North Central | warm               |       2.67353e+06 |               250000 | severe weather     |               nan | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |              1740 |              250 |

### Exploratory Data Analysis

Now that the data is cleaned up, let's explore some of the columns visually.

### Univariate Analysis

Recall from the introduction that we wanted to answer the question "Does being a costal state impact the frequency and severity of power outages compared to inland states?" This means we want to explore how columns related to differences of coastal and inland states impact power outages. Therefore, some natural univariate visualizations we can do are
1. A heatmap of the number of outages by U.S. State. We claim that the difference between coastal versus inland states is worth exploring; does this difference impact the number of recorded outages as well? If so, then it is worth exploring why these differenes occur.
2. A plot of the distribution of outage cause categories. We mentioned how coastal cities were prone to things such as hurricanes and floods that are less impactful in inland states. Does weather play a big role in causing outages? If so, this may guide our analysis on how coastal vs. inland weather causes outages.

<iframe
  src="assets/Univariate1.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

It appears that a staggeringly large number of recorded major outages occur in California, followed by Texas — both of which are coastal states. At a glance, states on the coast do appear to have more outages than inland states, but we will have to do a more rigorous test before drawing any more conclusions.

<iframe
  src="assets/Univariate2.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

It appears that weather and international attacks cause a very large number of major power outages in the U.S during this time period. This means that weather differences between coastal and inland states may play a significant role in differentiating between outages of these states, though once again we will need more testing to see whether each weather causes differ from coastal and inland states.

### Bivariate Analysis

By combining multiple columns of data, we can better model and visualize precisely how coastal or non coastal impacts power outages. We do three bivariate analyses:

1. A heatmap of US states and the number of outages each year. Though this does not directly relate with out coastal focused analysis, it may provide unexpected insight on how outages occur over time.

2. A scatterplot number of people affected by an outage, alongside the duration of the outage, colored by whether the state is coastal. Once again, the focus is on understanding how the number of customers affected by an outage scale with its duration. The coloring of coastal is just an afterthought to see if anything interesting comes up.

3. A scatterplot of state population versus number of outages, colored by whether the state is coastal. Once again, the focus is on the relationship between population and outages; perhaps a confound to our hypothesis is that population causes outages, moreso than being coastal.

<iframe
  src="assets/Bivariate1.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

Here, we see how the state to have the most power outages changes drastically year by year. At the same time, many of the states with most outages are coastal.

<iframe
  src="assets/Bivariate2.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

First, notice how we used log scaling for both the number of customers and outage duration axes. This is because some states have considerably larger population than others, and some ourages lasted many times longer than the others. Since we want to focus on general trends, a log scaled axis allows us to take these outliers into account while focusing on general trends.

Now we notice how, generally speaking, the number of customers affected in proportional to the ourage durataion, though this relationship is not very strong and has a lot of variance. Furthermore, there is no clear separation between coastal and inland states, but there is broadly more variance in both varaibles amongst inland states than coastal states.

<iframe
  src="assets/Bivariate3.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

There's a lot to unpack here. First, notice how each state is plotted multiple times. This is because each state has a different, typically increasing number of customers every year, which we plot with the same number of outages. This helps us visualize the range of total customers of each state has over the years.

Second, notice how coastal states vary more, and are generally more populous and have more outages than inalnd states, which is a great observation for our later models and hypothesis testing.

Third, notice how there is a roughly linear relationship between the number of outages and a state's population. This is an important discovery, because it may be the case that (if we are able to find statistically significant evidence for this through hypothesis testing) coastal states experience more outages than inland states, this may primarily be due to the larger average population of coastal states (particularly due to California and Texas).

### Aggregate Analysis

| STATE.COASTAL   |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:----------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| False           |           0.0318725 |               0.0239044 |             0.320717 |   0.0219124 |       0.0358566 |         0.515936 |                       0.0498008 |
| True            |           0.0426357 |               0.0377907 |             0.249031 |   0.0339147 |       0.0494186 |         0.488372 |                       0.0988372 |

There's a lot to unpack here: First, our guess that weather plays a larger role in coastal states in causing outages does not appear to be true. Furthermore, international attacks plays a significantly larger role in inland than coastal states. We remark that this is suprising considering how many of the U.S.' primary economic and administrative regions, such as New York or California, are coastal. We also notice how system operability disruption plays a much larger role in causing outages in coastal cities.

## Assessment of Missingness

### NMAR Analysis

We now shift our attention to the missingness of some columns in the dataset. Recall that a column is Not Missing at Random (NMAR) if the missingness is dependent on another column. In our dataset, some columns in particular stand out as potentially being NMAR:

1. `CLIMATE.REGION`: All of the missing values in this column belong to outages in the state of Hawaii. However, this means that the data is actually missing by design (MD), rather than NMAR on `U.S._STATE`. This is because `Climate Region` is only defined for the contiguous landmass U.S., and so we can know with certainty when the column has missing values, and thus is its primary missigness mechanism is _not_ NMAR.

2. `HURRICANE.NAME`: Once again, this column only has non `NaN` values if the cause of the outage is a hurricane. Thus, the column is primarily missing by design rather than being NMAR on the `CAUSE.CATEGORY`.

3. `OUTAGE.DURATION`, `DEMAND.LOSS.MW`, `CUSTOMERS.AFFECTED`. This is an interesting set of columns with missingness, since there's no clear reason why they would be missing. We believe this is NMAR since they can be said are dependant on other columns like `U.S._STATE` or `CAUSE.CATEGORY`. This is because the data was collected from a variety of sources, and it makes sense how certain locations or certain causes may hinder the collection of these pieces of data. For example, if the outage was caused by a large hurricane, it is difficult to precisely determine the number of customers, or if the outage was particularly remote, how many megawatts of power was really lost due to it. Thus we conclude that these three columns are NMAR. 

We remark that one may also argue that these columns are MAR, but the location or cause of outages explain the missingness of these columns much more than the values themselves. For example, it is reasonable to argue that a small, short outage in a heavily urban city would still be measured accurately, but an outage in a rural town may not have the equipment to make such precise measurements.

### Missingness Dependency

Let's explore the missingness of `DEMAND.LOSS.MW` a bit more through hypothesis testing. We will run a permutation test on the missingness of `DEMAND.LOSS.MW` compared to `CAUSE.CATEGORY`, which we claim its missingness is dependant on, and another compared to `OUTAGE.START`, which we claim has less of a correlation.

**Null Hypothesis:** The distribution of `CAUSE.CATEGORY` is the same when `DEMAND.LOSS.MW` is missing and not missing.

**Alternative Hypothesis:** The distribution of `CAUSE.CATEGORY` is different when `DEMAND.LOSS.MW` is missing versus not missing.

**Test Statistic:** Total Variation Distance (TVD).

Let's begin by drawing a plot of the observed missing vs. not missing distributions:

<iframe
  src="assets/Missingness1.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

It appears that there are noticeable differences between the distribution of `CAUSE.CATEGORY` when `DEMAND.LOSS.MW` is missing versus when it is not missing. In particular, `DEMAND.LOSS.MW` appears to be missing much more frequently when the cause is an international attack. Let's see if our hypothesis test says about this.

<iframe
  src="assets/Missingness2.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

With a p-value of almost 0 (i.e. outputted 0.0), and an observed tvd that is wildly different from the permutation test tvds, we can conclude that there is significant statistical evidence to reject the null hypothesis. That is, the missingness of `DEMAND.LOSS.MW` has some correlation with `CAUSE.CATEGORY`, and so we have additional evidence that `DEMAND.LOSS.MW` is NMAR on `CAUSE.CATEGORY`.

This makes sense given our initial barplot; For instance, it is much harder to determine the MegaWatts lost due to an attack compared to causes such as fuel supply emergency. As such, it makes sense that `DEMAND.LOSS.MW` is missing since the data collection process is difficult in such cases. 

## Hypothesis Testing

Now that we better understand what our data looks like, we can conduct some hypothesis tests related to our question "Does being a costal state impact the frequency and severity of power outages compared to inland states?"

Our first hypothesis test whether the number of customers affected by an outage is different on whether a state is coastal or inland. That is, our hypotheses are:

**Null Hypothesis:** On average, the number of customers affected is the same for outages in coastal states as outages in inland states.

**Alternative Hypothesis:** There is a statistically significant difference in the average number of customers between those in coastal to those in inland states.

**Test Statistic:** Absolute difference in means.

<iframe
  src="assets/Hypothesis1.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

Above is the Kernel Density Estimate (KDE) for `CUSTOMERS.AFFECTED` based on whether it occured in a coastal or inland state. We use a log-scaled x-axis as we wish to focus on the shape of the curve rather than the values right now. 

Interestingly, inland outages have a slightly higher probability of affecting fewer customers compared to coastal outages. There can be many reasons for this. For example, momre people live on the coast, and so fewer people are affected on average in inland outages. Alternatively, many coastal environments may lack land compared to their inland counterparts, causing people to live in denser neighborhoods and thus have more people on one power grid.

Regardless of the reasons, we see whether our hypothesis test claims this is something statistically siginicant:

With a p-value of 0.0108, we can say that there is something statistically significant regarding the difference in the average number of customers affected between coastal and inland outages. That means that our arguments above may be the reason; a hypothesis test does not confirm why these differences happen, but just provides evidence for us to reject the null hypothesis.

## Framing a Prediction Problem

We now shift our attention to trying to predict whether an outage occured in a coastal state or an inland state from other features. This will be a binary classification problem since there are only two possible outputs, coastal or inland.

We choose to predict whether an outage happens on the coast or inland as so far, we have only explored how costal versus inland impacts other columns, but never the other way around. So far, our analysis has shown that there are differences in the distribution of other columns between coastal and inland, and so our model will simply try to leverage those differences.

As such, we will use F1-score as our metric, as this is not a situation where precision wins over recall or vice versa. We also use r^2 score to supplement our analysis.

We will treat this problem as a post-analysis problem. That is, we will have access to information such as the customers affected and total megawatts lost. We choose to treat this as a post-event analysis since our goal is not to determine something that might be useful at the time of the outage — knowing where the outage is located immediately tells us whether it is on the coast or not. As such, we will not use any of the state columns in our model.

Rather, the intention is to create a model that highlights differences in coastal and inland outages and draws general trends about it. In particular, this advances our answer to the question "Does being a costal state impact the frequency and severity of power outages compared to inland states?" by shwowing that if a good performing model can be trained to identify coastal versus inland outages, then coastal versus inland plays a big role in determining the characteristics of power outages in the U.S.

## Baseline Model

Our baseline mode will attempt to classify outages as coastal or inland using `CUSTOMERS.AFFECTED`, which is a quantitative column, and `OUTAGE.DURATION`, which is also a quantitative column. We choose these two as features as our prior analysis has revealed strong relationships between these features and whether outages were coastal or inland.

We will take 50% of the original `outages` as our test set, and use the rest as our training set. We can be greedy about the size of our test set as there are very many rows in the dataset. We used Support Vector Classification (SVC) as after looping through many models with varying test sizes, we found it had the best results (see below). We standard scaled both features to better deal with the outliers in each.

|    | Classifier                  |   Pseudo R² |   F1 Score |
|---:|:----------------------------|------------:|-----------:|
|  0 | LogisticRegression          |    0.855422 |   0.902703 |
|  1 | LinearSVC                   |    0.855422 |   0.902703 |
|  2 | CalibratedClassifierCV      |    0.855422 |   0.902703 |
|  3 | RidgeClassifierCV           |    0.855422 |   0.902703 |
|  4 | RidgeClassifier             |    0.855422 |   0.902703 |
|  5 | LogisticRegressionCV        |    0.855422 |   0.902703 |
|  6 | SGDClassifier               |    0.853414 |   0.901218 |
|  7 | MLPClassifier               |    0.853414 |   0.900409 |
|  8 | AdaBoostClassifier          |    0.851406 |   0.899457 |
|  9 | SVC                         |    0.843373 |   0.895442 |
| 10 | NuSVC                       |    0.837349 |   0.891856 |
| 11 | RandomForestClassifier      |    0.843373 |   0.890449 |
| 12 | GradientBoostingClassifier  |    0.841365 |   0.890125 |
| 13 | DecisionTreeClassifier      |    0.835341 |   0.881503 |
| 14 | ExtraTreeClassifier         |    0.827309 |   0.876437 |
| 15 | ExtraTreesClassifier        |    0.821285 |   0.8734   |
| 16 | KNeighborsClassifier        |    0.809237 |   0.86676  |
| 17 | BaggingClassifier           |    0.813253 |   0.865412 |
| 18 | PassiveAggressiveClassifier |    0.783133 |   0.837349 |
| 19 | BernoulliNB                 |    0.714859 |   0.824691 |
| 20 | DummyClassifier             |    0.670683 |   0.802885 |
| 21 | Perceptron                  |    0.728916 |   0.791988 |
| 22 | NearestCentroid             |    0.60241  |   0.652632 |

The performance of this baseline model was acceptable, getting an r-squared of `0.6706827309236948` and a f1-score of `0.8028846153846154`. For two features, this is pretty good. This is consistent with our analysis thus far, as these features are clearly heavily correlated with whether the outages were coastal or inland.

## Final Model

The final model will incorporate `CUSTOMERS.AFFECTED`, which is a quantitative column, and `OUTAGE.DURATION`, which is also a quantitative column in the same way the base model did. However, we also add in `CLIMATE.REGION`, a nominal column, `CLIMATE.CATEGORY`, an ordinal column. We choose to add these three features in particular becaise `CLIMATE.REGION` and `CLIMATE.CATEGORY` logically allow us to guess the location of an outage. Our hope is that the model will improve considerably with these new features.

We will standard scale all the quantitative columns, and one-hot encode all others, dropping one resulting column to prevent multicollinearity. We will use SVC once again, as looping through many classifiers, it had one of the best r-squared and f1score (see below). This makes sense — SVC is known as a generally robust classification model compared to other approaches like LogisticRegression.

|    | Classifier                  |   Pseudo R² |   F1 Score |
|---:|:----------------------------|------------:|-----------:|
|  0 | LogisticRegression          |    0.855422 |   0.902703 |
|  1 | LinearSVC                   |    0.855422 |   0.902703 |
|  2 | SGDClassifier               |    0.855422 |   0.902703 |
|  3 | CalibratedClassifierCV      |    0.855422 |   0.902703 |
|  4 | RidgeClassifierCV           |    0.855422 |   0.902703 |
|  5 | RidgeClassifier             |    0.855422 |   0.902703 |
|  6 | LogisticRegressionCV        |    0.855422 |   0.902703 |
|  7 | MLPClassifier               |    0.853414 |   0.900137 |
|  8 | AdaBoostClassifier          |    0.851406 |   0.899457 |
|  9 | SVC                         |    0.843373 |   0.895442 |
| 10 | NuSVC                       |    0.837349 |   0.891856 |
| 11 | RandomForestClassifier      |    0.845382 |   0.891702 |
| 12 | GradientBoostingClassifier  |    0.843373 |   0.891667 |
| 13 | BaggingClassifier           |    0.837349 |   0.883453 |
| 14 | DecisionTreeClassifier      |    0.829317 |   0.87699  |
| 15 | ExtraTreesClassifier        |    0.819277 |   0.871429 |
| 16 | KNeighborsClassifier        |    0.809237 |   0.86676  |
| 17 | ExtraTreeClassifier         |    0.813253 |   0.866187 |
| 18 | BernoulliNB                 |    0.714859 |   0.824691 |
| 19 | PassiveAggressiveClassifier |    0.771084 |   0.824074 |
| 20 | DummyClassifier             |    0.670683 |   0.802885 |
| 21 | Perceptron                  |    0.728916 |   0.791988 |
| 22 | NearestCentroid             |    0.60241  |   0.652632 |

Our final model (prior to tuning) achieves a r-squared of `0.8433734939759037` and an f1-score of `0.8954423592493298` — a significant improvement to the base model's r-squared of `0.6706827309236948` and f1-score of `0.8028846153846154`. Once again, this makes sense. Being given a state's climate information logically will allow you to better differentiate coastal and inland states.

We now turn our attention to fine-tuning the model. We use GridSearchCV with hyperparamters

- `'classifier__C': [0.01, 0.1, 0.5, 1, 10, 100]`
- `'classifier__gamma': [1, 0.1, 0.01, 0.001]`
- `'classifier__kernel': ['linear', 'rbf']`

We get `C = 0.1`, `gamma=1`, and `kernel='linear'` to be the best hyperparamters in terms of f1-score and r2-squared. The tuned model had an r-squared of `0.8433734939759037` — compared to the untuned `0.8433734939759037`, no improvement — and a f1-score of `0.8962765957446809` — a slight improvement to the untuned `0.8954423592493298`.

Thus, the final model is a considerable improvement on the baseline model, both in terms of r-squared and f1-score. We summarize this by showing the final tuned confusion matrix on the test set:

<iframe
  src="assets/Model1.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

## Fairness Analysis

We had previously seen how population is strongly correlated witht whether a state is coastal or inland. Thus, it makes sense to analyze our fairness in terms of the number of customers affected by the outage. We define an outage to be _small_ if it impacted less than 50,000 people. Otherwise, we say the outage was _large_. We decided on these cutoffs as the mean number of customers affected was `143456`, but the median was `70135`.

Our evaluation metric will once again be f1-score, since the classes are imblanced in size. and f1-score takes this into consideration. We will conduct a permutation test on the f1-score with the following hypotheses:

**Null Hypothesis:** The model is fair. That is, the f1-scores for small and large outages are roughly the same, and any differences are due to chance.

**Alternative Hypothesis:** The model is unfait. The f1-score for larger outages is significantly different than for small outages.

**Test Statistic:** Absolute difference in F1-score.

<iframe
  src="assets/Fair1.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

We observed a difference of `0.04108241082410824`, and out permutatio test has a p-value of `0.3105`. This is not statistically significant enough for us to reject the null hypothesis, and so we have additional evidence that the model is fair to both small and large outages.

We now remark why fairness in models such as these even matter. Suppose this model was used to help determine whether more resources should be used to deal with coastal or inland outages. If the model had been unfair to small outages, which say are more common inland, then these communities would be greatly underserved — and coastal communities with extremely large populations may be overserved. Thus, by checking whether our model is fair to both small and large outages we help prevent such things from happening.

### Conclusions

Thank you for reading until the end of this project. We will now briefly recap the main findings in our analysis. First, we discovered how outages in coastal states have different characteristics than those in inland states. These differences allowed us to train a relatively accurate model to predict whether an outage occured in a coastal or inland state. 

Thus, to answer our original question **"Does being a costal state impact the frequency and severity of power outages compared to inland states?"**, the answer is leaning to yes, evidenced by the analysis done in this project.