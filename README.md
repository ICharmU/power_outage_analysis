# Major US Power Outages (2000-2016)
Supplementary project for DSC 80.

## Introduction
Over time, power outages have been increasingly impactful with the prevalence of digital technologies. Preventing major outages can be useful for keeping the US a productive state. When researching electrical grids I worked with [this](https://engineering.purdue.edu/LASCI/research-data/outages) power outage dataset. I focused on the economic and climate factors related to power outages.  

This outages dataset contains information from January 2000 to June 2016 for power outages occurring in the US (including non-continental states). The uncleaned dataset contains 1534 observations and 55 features. The most interesting columns were anomaly level and NERC region. Based on an oceanic temperature index, variations in ocean temperature at the time of the outage were reported. These temperatures were cyclical with an increasing amplitude toward 2016. This feature was useful for understanding the climate was like at the time of the outage (e.g. cold, hot). The NERC (North American Electric Reliability Corporation) regions differentiated the primary electrical power supplier for each state. During the mid 2000's there was a NERC merger that needed to be accounted.  

## Data Cleaning and Exploratory Data Analysis

When reading in the data from an Excel file, the column names were formatted with periods instead of underscores. I reformatted the column names to be in snake case with underscores. Additionally, many of the datatypes were read in as strings by default, so I converted most of these columns to floats (e.g. outage duration in minutes), along with a few datetime conversions (e.g. the outage start date and time).  

A notable property of the locations in the dataset was that Washington D.C. had major reported outages during this time period, but Rhode Island had none (and thus was not listed as a state for any observations). A major outage was defined as 50,000 impacted customers or a significant loss of power (300 megawatts), both of which have similar impacts, at least based on average household electricity consumption (~0.02 megawatts per day, although this can vary by location).  

More outages seemed to occur during the winter and summer months than in spring and fall. This makes sense as summer comes with extreme heat, and winter extreme cold, while spring and fall are more in the middle.  

<iframe
  src="assets/outages_by_month.html"
  width="600"
  height="400"
  frameborder="0"
>Missing observations by month across all years</iframe>

When cleaning the NERC regions I consolidated Alaska and Hawaii into non-continental, as NERC technically consists of only the continental states (Alaska and Hawaii have their own separate power grids). ECAR was one of three regions that [merged into the RFC region](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) (p. 102). ECAR was the only region of the three reported here. If you're curious about NERC regions, here is a [map](https://atlas.eia.gov/datasets/eia::nerc-regions/explore) I found from the Energy Information Administration that guided my inquiries for some of the oddly classified NERC regions (e.g. "FRCC, SERC" I classified under FRCC since the observation was associated with Florida for the state feature). Florida actually [split from SERC](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) in 1996 to be its own member in NERC (p. 171). This was before the data was collected so I disregarded FRCC's membership history. [Here](https://upload.wikimedia.org/wikipedia/commons/f/f4/NERC-map-en.svg) is a colored map that I found helpful for quickly differentiating the NERC regions.  

As mentioned in the introduction, there was an ocean temperature anomaly level that used an oceanic index to compare expected ocean temperatures to their actual temperatures. If you're curious, you can find the historical temperature differences [here](https://origin.cpc.ncep.noaa.gov/products/analysis_monitoring/ensostuff/ONI_v5.php). There are blue, red and grey colors temperatures on that NOAA website. Another feature in the dataset considered all blue temperatures to be cold (<-0.5°C), red temperatures to be warm (>0.5°C) and grey to be normal (within +/- 0.5°C). Below is a graph of the anomaly levels over time (by month, although not all months had a major outage reported).

<iframe
  src="assets/anomaly_level_over_time.html"
  width="600"
  height="400"
  frameborder="0"
>Observations to be imputed</iframe>

Each observation was associated with a cause (e.g. severe weather, intentional shutdown to prevent overload). There was also an optional detail for the cause associated with each observation (e.g. thunderstorm, winter storm). When cleaning the details I filled in missing values with the cause string as a good portion of the details were already duplicated like this. For the non-repeat values I grouped by common names (e.g. ice/rain -> winter storm, winter -> winter storm). One group I manufactured was called `intentional`, meaning that the grid was shutoff preemptively in order to prevent a larger shutdown. I also grouped faulty equipment together (anything related to failures). A domain specific grouping was `substations`, which essentially transfer power from a large generating station to individual households (there are a [few types of substations](https://www.nationalgrid.com/stories/energy-explained/what-is-a-substation), but this is the gist of what they do). For some general terms (storm, severe weather and fog), I created a grouping called `unknown origin` that would encapture their values as assigning the wrong storm to the wrong group (winter vs thunder) could bias later predictions more than leaving the two separate.  

I removed `demand loss` and `customers affected` as at least one of these criterions needed to be satisfied for the outage to be considered major. If one column was missing you could not predict the missing value using the known value as either the value was missing completely at random or the criterion was not met. Since this distinction could not be made, I found imputation to not be useful. Additionally, the demand loss criterion was reported in one of two ways (either peak demand loss or total demand loss), which had no distinction in reporting.  

Another subset of features was related to currency (e.g. electricity prices). I converted all percents to proportions here, as well as cents as integers to cents as floats. This allowed me to estimate total revenues for the state at its reported date. When comparing my estimates to the actual total revenue columns there were some small discrepancies that were likely attributable to a sector that is not residential, industrial or commercial (or there was a rounding error). Here I removed some features, such as total price, which averaged values across all three sectors. I found these aggregates to not be meaningful as the individual sector prices were available in the dataset. On average, across all years, residential electricity prices were higher than commercial prices, which were higher than industrial prices. `Total customers` had a similar issue with small discrepancies between the sum of sector customers and the total customer feature, so I decided to use the provided total customer feature instead of my estimate. I removed `gross state product` as a feature since electricity is not growth-driving industry and is typically considered stable as a utility.  

The subsection of features involved state information (e.g. population, percent of state as water/land, population density). I converted percents to proportions here as well. I found that larger states and states on the water (e.g. California, Texas) had the most number of outages over the time period. On the other hand, less densely populated states (e.g. North Dakota) had almost no major outages reported. This dataset differentiated `urban` (50,000+ people) from `urban clusters` (2,500-50,000 people). Wyoming had the highest percent of urban clusters, with about 40% of the state living within urban clusters, while most other states consisted of 10-20% urban clusters. Washington D.C. was considered 100% urban, so I filled in the missing urban cluster value with 0%. 

There were 9 observations with missing `anomaly levels`. I probabilistically imputed these missing values by conditioning on year and state. This fixed about half of the observations. For the rest I relaxed the condition to only be on year instead (only 1 observation contained Alaska, so state did not work). Due to the wave-like pattern of ocean anomaly levels this imputation was justified as each year's anomaly levels were relatively close together. In the graph below you can see the anomaly levels for non-missing states. With a lot of the missing observations coming from the early 2000s where no other state data was available, imputation conditional on year was preferred.  

<iframe
  src="assets/imputing_anomaly_data.html"
  width="600"
  height="400"
  frameborder="0"
>Observations to be imputed</iframe>

I assigned Alaska a climate region of cold based on its geographical location. For other missing climate regions I probabilistically imputed conditional on state by bootstrapping from other observations with the same climate regions. Looking back, this was unnecessary as I should have derived the climate region feature from the anomaly level directly.  

There were many missing outage durations from 2000 and 2016. Since the study started in 2000 and ended in 2016 I concluded these observations were missing due to data collection limitations. Since these were edge cases, imputing outage durations could bias my later predictions. I decided to remove these observations from my analysis. I found this acceptable as the missing observations came from many different states (see below). Note that some states may have more than one missing state in a year, but there are only ~50 missing outage durations total.  

<iframe
  src="assets/missing_states_by_year.html"
  width="600"
  height="400"
  frameborder="0"
>States with at least one missing outage duration by year</iframe>

`Hurricane name` was another column. Any outages that didn't occur during a hurricane were missing. I binarized this column so missing values were 0 (no hurricane) and non-missing values were 1 (hurricane occurred). This lost minimal information as each hurricane was associated with a month and year. The name is also arbitrary (ignoring the first letter).  

Looking at the average outage duration based on hurricane presence, it appears that hurricane outages have longer outages than non-hurricane outages, regardless of NERC region:

|  NERC Region  | No Hurricane |Hurricane |
|--------:|--------:|---------:|
|  FRCC   | 3344.28 |  5558.39 |
|  MRO    | 2933.59 |   nan    |
|  NPCC   | 2993.61 |  7928.38 |
|  RFC    | 3460.72 |  7668.17 |
|  SERC   | 1502.85 |  4680.43 |
|  SPP    | 2467.91 |  5268.6  |
|  TRE    | 2040.36 | 14463.2  |
|  WECC   | 1481.49 |   nan    |
|  AK/HI  |  845.4  |   nan    |

Lastly, there were two observations missing electricity prices, one from 2002 and the other from 2006. Since these observations seemed anomalous and all other columns had associated prices I removed them.

## Assessment of Missingness

### NMAR Analysis
As mentioned previously, `demand loss` and `customers affected` were removed. Meeting at least one of these was a criterion for being included in the dataset. If both criterions were NaN then interpolating one or both values could lead to incorrect expectations for a specific occurrence (i.e. you can't know which criteria caused the outage to be reported). If one criterion is missing, there is no way to distinguish whether this missing value didn't meet the criterion threshold or if the value is missing due to chance. If neither criterion is missing, then nothing needs to be done. However, a majority of points do not include both criterion as only 1 criterion is required to be reported. By this logic the missing values must be NMAR as you can't know their values with any other information available, except their actual values. 

In order to make these columns MAR I would need to know the number of criteria met when the observation was reported. I know at least one criterion was met, but there is ambiguity without the exact number.  

### Missingness Dependency

Since there weren't many missing values for some features I wanted to infer whether or not missingness depended on these features.  

I first compared outages by year (missing vs non-missing) to infer whether the distribution appeared different by chance.  

<iframe
  src="assets/non_missing_outages_over_time.html"
  width="600"
  height="400"
  frameborder="0"
>Non-missing outages over time</iframe>

<iframe
  src="assets/missing_outages_over_time.html"
  width="600"
  height="400"
  frameborder="0"
>Missing outages over time</iframe>

I performed a hypothesis test under the assumptions:  
H<sub>0</sub>: Missing and non-missing outage duration observations follow the same year distribution.  
H<sub>1</sub>: Missing and non-missing outage duration observations follow the same year distribution.  

Using the Kolmogorov Smirnov (KS) test for each permutation I found a distribution of p-values:
<iframe
  src="assets/ks_test_year_p_values.html"
  width="600"
  height="400"
  frameborder="0"
>KS test p-value distribution. alpha=0.05</iframe>

In ~99.998% of these trials (10,000 total), we would find the missing and non-missing outage durations to follow a different distributions of outages per year. This implies outage durations are likely missing at random (MAR), conditional on the year (p-value = 0.0002, alpha = 0.05).  

I performed a similar hypothesis test for hurricane presence, instead using difference in means. Hurricane presence was treated as a discrete variable.

H<sub>0</sub>: Missing and non-missing outage duration observations have the same mean hurricane presence.  
H<sub>1</sub>: Missing and non-missing outage duration observations do not have the same mean hurricane presence.  

From a quick glance, these distributions look quite similar:
<iframe
  src="assets/non_missing_hurricane.html"
  width="600"
  height="400"
  frameborder="0"
>Non-missing hurricane distribution</iframe>

<iframe
  src="assets/missing_hurricane.html"
  width="600"
  height="400"
  frameborder="0"
>Missing hurricane distribution</iframe>

Instead of finding the p-value distribution I found the distribution of mean differences. These differences can be interpreted as the proportion difference between the bootstrapped and expected hurricane presence.  

<iframe
  src="assets/mean_diff_results.html"
  width="600"
  height="400"
  frameborder="0"
>Mean difference distribution</iframe>

About 50% of non-missing bootstrapped mean differences are on either side of the actual mean difference. This indicates that knowing whether or not a hurricane occurred is not helpful in predicting missing values. The hurricane feature is likely MCAR.  

## Hypothesis Testing
When looking at all outage durations, I was curious whether an exponential distribution was followed since this is a time based feature. I also thought the empirical distribution looked somewhat similar to an exponential distribution.  

<iframe
  src="assets/expo_pdf.html"
  width="600"
  height="400"
  frameborder="0"
>Empirical exponential distribution</iframe>

<iframe
  src="assets/true_expo_pdf.html"
  width="600"
  height="400"
  frameborder="0"
>MLE exponential distribution</iframe>

H<sub>0</sub>: Outage duration follows an exponential distribution.  
H<sub>1</sub>: Outage duration does not follow an exponential distribution.    

I parameterized the null distribution with the maximum likelihood estimator beta = expectation(empirical distribution). This isn't a high stakes test, so I used a standard significance level of alpha = 0.05.   

Using a 2-sample KS test I can infer it is unlikely the outage duration follows an exponential distribution (p-value = 3.32*10<sup>-12</sup>). This isn't a huge surprise as the empirical distribution has a much larger concentration of points towards 0, compared to the MLE distribution.  

## Framing a Prediction Problem
My goal is to predict the duration of power outages to measure their severity.  

I will use a regressor model as time is continuous and the outages appear to follow some decaying distribution (some exponential-adjacent distribution). Since the outage duration PDF is non-linear I will use RMSE instead of R<sup>2</sup>. RMSE will also allow me to determine the average amount of time my predictions are off by.  

When making predictions the year will not be a useful predictor at the time of prediction as it is reliant on future months in the year for all observations that come from a given year. I also can't use the start and end dates as those are directly used to calculate the duration (and the end date is not available at the start of the outage). Although there is a revision period for ocean anomaly temperatures, this feature is still useful as it is produced on a historical basis. All other attributes can be known at the time of the event (usually), such as the type of weather. Or the information is (mostly) constant (e.g. NERC region, state name).  

## Baseline Model
As my baseline model I used linear regression with anomaly level and population as the features. Both anomaly level and population are continuous variables.  

I used an 80-20 train test split and shuffled the data as there may be a time element associated when the original input order.  

On the training set this model has a loss of ~6400 minutes or about 4.5 days. The testing set had a loss of ~3850 minutes or about 2.7 days.  

This model could be decent as it doesn't seem to overfit, but it may be underfitting (hard to tell). Being off by 2.7 days is good, but only for longer outages. When the outage is short I overestimate the outage time, on average.  

## Final Model
When looking at alternative models I tried a random forest approach, however this only ended up overfitting to the training data. The training loss was comparable to this baseline model, but I would worry about generalizing to years not included in the dataset.  

The model I ended up using was a linear regression model (again), but with the parameters:
- Month (discrete)
- NERC Region (nominal)
- Cause Category (nominal)
- Hurricane Occurred (nominal/discrete)
- Sector Proportions (continuous)
- Climate Category (ordinal)

I started off with `anomaly level` in my baseline model. I kept the general idea in my final model through `climate category` feature which classifies each anomaly level as cold, normal or warm.  

`Population` in the baseline model was really meant to capture location since it is different for each state. I thought this wasn't a good geographical comparison, which is what a state should represent. When I removed population, I included `NERC region` and `hurricane presence`. These describe the general region a state is located in and whether the state is near a large body of water. `Cause category` added some state-related features (e.g. weather), but this also compared the events themselves, regardless of location (e.g. crime based outages). I haven't mentioned `sector proportions` previously. This is a feature I engineered representing how close the residential, industrial and commercial sectors are to using similar proportions of electricity in a state. This is calculated as 27 * product(residential electricity use, industrial electricity use, commercial electricity use). The 27 comes from the fact that a perfectly equal energy use split will be close to 1 and a large imbalance will be close to 0. This provides information about the different energy usages in each state. Although it doesn't differentiate which sector used a specific amount of energy, my goal was to see if an imbalance in energy use was related to outages (possibly related to excessive energy use).  

My final model was trained on the same train test split as the baseline model. This new model had a training accuracy of 5900 minutes (4.1 days) and a testing accuracy of 3340 minutes (2.3 days). This is slightly better than my baseline model and it seems promising with a lower testing error than training error (less likely to be overfitting). However, it should still be noted that this training and testing set could be better for my final linear model and features. I didn't cross validate my linear regression model during training, so it's a bit unclear how well the model generalizes.

## Fairness Analysis
To check if my model had an underlying bias towards certain features I carried out a permutation test. Specifically, I tested if the RMSE was higher or lower, on average, when predicting certain years. This allowed me to retroactively check if my model was possibly biased.

<iframe
  src="assets/avg_rmse_by_year.html"
  width="600"
  height="400"
  frameborder="0"
>Average RMSE by year</iframe>

I will perform a permutation test under:  
H<sub>0</sub>: Average RMSE is the same for all years.  
H<sub>1</sub>: Average RMSE is not the same for all years.  
alpha = 0.05

When performing the permutation test I shuffled the RMSE and recalculated the mean RMSE for each year.  
As my test statistic I used total variation distance (TVD) by treating each year as an ordinal variable.

After performing 10,000 trials of permutation tests, I found the TVD to be at least 0.1 in 99.94% of the trials (p-value = 0.0006). A 10% difference between each year is quite lenient already. The fact that almost no permutations resulted in a smaller difference indicates the empirical number of outages distribution by year is almost certainly different for some years (at least based on this dataset). There is a lack of parity with respect to year. 