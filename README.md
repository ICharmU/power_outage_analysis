# Major US Power Outages (2000-2016)
Supplementary project for DSC 80.

## Introduction
Over time, power outages have had an increasing impact with the increased use of digital technologies. Preventing these outages can be useful for keeping the US productive. When researching electrical grids working with [this](https://engineering.purdue.edu/LASCI/research-data/outages) power outage dataset, I focused on the economic and climate factors related to power outages.  

This outages dataset contains information from January 2000 to June 2016 for power outages occurring in US states (including non-continental states). The uncleaned dataset contains 1534 observations and 55 features. The most interesting columns were anomaly level and NERC region. Based on an oceanic temperature index, variations in ocean temperature at the time of the outage were reported. These temperatures were cyclical with an increasing amplitude toward 2016. This feature was useful when considering what the climate was like at the time of the outage (e.g. cold, hot) The NERC (North American Electric Reliability Corporation) regions differentiated the primary electrical power supplier for each state. During the mid 2000's there was a NERC merger that needed to be accounted.  

## Data Cleaning and Exploratory Data Analysis

When reading in the data from an Excel file, the column names were formatted with periods instead of underscores. I reformatted the column names to be in snake case with underscores. Additionally, many of the datatypes were read in as strings by default, so I converted most of these columns to floats (e.g. outage duration in minutes), along with a few datetimes (e.g. the outage start date and time).  

Some notable information about the locations where data was collected is that Washington D.C. had major reported outages during this time period, but Rhode Island had none (and thus was not listed as a state for any observations). A major outage was defined as 50,000 impacted customers or a significant loss of power (300 megawatts), both of which are similar in impact level, based on average household electricity consumption (~0.02 megawatts per day, although this can vary by location).  

More outages seem to occur during the winter and summer months than in spring and fall. This makes sense as summer comes with extreme heat, and winter extreme cold, while spring and fall are more of middle grounds.  

<iframe
  src="assets/outages_by_month.html"
  width="600"
  height="400"
  frameborder="0"
>Missing observations by month across all years</iframe>

When cleaning the NERC regions I consolidated Alaska and Hawaii into non-continental, as NERC technically consists of only the continental states (Alaska and Hawaii have their own separate power grids). ECAR was one of three regions that [merged into the RFC region](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) (p. 102). Although ECAR was the only region of the 3 that merged that was reported here. If you're curious about NERC regions, here is a [map](https://atlas.eia.gov/datasets/eia::nerc-regions/explore) I found from the Energy Information Administration that guided my inquiries for some of the oddly classified NERC regions (e.g. FRCC, SERC I classified under FRCC since that observation with associated with the state Florida). Florida actually [split from SERC](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) in 1996 to be its own member in NERC (p. 171). This was before the data was collected so I disregarded FRCC's membership history. [Here](https://upload.wikimedia.org/wikipedia/commons/f/f4/NERC-map-en.svg) is a colored map that I found helpful for quickly differentiating the NERC regions.  

As mentioned in the introduction, there was an ocean temperature anomaly level reporting that uses an oceanic index to compare expected ocean temperatures to their actual temperatures. If you're curious, you can find the temperature differences [here](https://origin.cpc.ncep.noaa.gov/products/analysis_monitoring/ensostuff/ONI_v5.php). There are blue, red and grey colors temperatures on that NOAA website. Another feature in the dataset considered all blue temperatures to be cold (<-0.5°C), red temperatures to be warm (>0.5°C) and grey to be normal (within +/- 0.5°C). Below is a graph of the anomaly levels over time (by month).

<iframe
  src="assets/anomaly_level_over_time.html"
  width="600"
  height="400"
  frameborder="0"
>Observations to be imputed</iframe>

Each observation was associated with a cause (e.g. severe weather, intentional shutdown to prevent overload). There was also an optional detail for the cause associated with each observation (e.g. thunderstorm, winter storm). When cleaning the details I filled in missing values with the cause string as a good portion of the details were already duplicated like this. For the non-repeat values I grouped by common names (e.g. ice/rain -> winter storm, winter -> winter storm, etc.). Some of the largest groups I manufactured were intentional, meaning that the grid was shutoff preemptively in order to prevent a larger shutdown. I also grouped faulty equipment together (anything related to failures). A domain specific grouping was substations, which essentially transfer power from a large generating station to individual households (there are a [few types of substations](https://www.nationalgrid.com/stories/energy-explained/what-is-a-substation), but this is the gist of what they do). For some general terms (storm, severe weather and fog), I created a grouping called unknown origin that would encapture there values as assigning the wrong storm to the wrong group (winter vs thunder) could bias later predictions more than leaving the two separate.  

I removed the demand loss and customers affected columns as at least one of these criterions needed to be satisfied for the outage to be considered major. If one column was missing you could not predict the missing value using the known value as either the value was missing completely at random or the criterion was not met. Since this distinction could not be made, I found imputation to not be useful. Additionally, the demand loss criterion was reported in one of two ways (either peak demand loss or total demand loss), which had no distinction in reporting.  

There was also currency relate data (e.g. electricity prices). I converted all percents to proportions here, as well as cents as integers to centers as floats. This allowed me to estimate total revenues for the state. When comparing my estimates to the actual total revenue columns there were some small discrepancies that are likely attributable to a sector that is not residential, industrial or commercial (or there was a rounding error). Here I removed some features such as total price, which was the average cost of electricity across all sectors. I found this aggregate to not be meaningful as the individual sector prices were available in the dataset. On average, across all years, residential electricity prices were higher than commercial prices, which were higher than industrial prices. The total customers had a similar issue with small discrepancies between the sum of sector customers and the total customer feature, so I decided to use the provided total customer feature instead of my estimate. I removed state domestic product as a feature since electricity is not an industry that drives growth in a state and is typically considered stable as a utility.  

The last section of feature involved state information (e.g. population, percent of state as water/land, population density). I converted percents to proportions here as well. I found that larger states and states on the water (e.g. California, Texas) had the most number of outages over the time period. On the other hand, less densely populated states (e.g. North Dakota) had almost no major outages reported. This dataset differentiated urban areas (50,000+ people) from urban clusters (2,500-50,000 people) living in some local area. Wyoming had the highest statewide urban cluster percent, with about 40% of people living in urban clusters, while most states consisted of 10-20% urban clusters. Washington D.C. was considered 100% urban, so I filled in the missing urban cluster value with 0. 

There were 9 observations that were missing values for anomaly level, so I probabilistically imputed these missing values by conditioning on year and state. This fixed about half of the observations, so for the rest I relaxed the condition to only be on year instead (Alaska only had 1 observations, so state did not work). Due to the wave-like pattern of ocean anomaly levels this imputation was justified as each year will be relatively close together. In the graph below you can see the anomaly levels for non-missing states. Since a lot of the missing observations came from the early 2000s where no other state data is available, imputation conditional on year was ideal.  

<iframe
  src="assets/imputing_anomaly_data.html"
  width="600"
  height="400"
  frameborder="0"
>Observations to be imputed</iframe>

I assigned Alaska a climate region of cold based on its geographical location. For other missing climate regions I probabilistically imputed by state for other observations with the same climate regions. Looking back, this was unnecessary as I should have derived the climate region feature from the anomaly level directly.  

There were many missing outage durations from 2000 and 2016. Since the study started in 2000 and ended in 2016 I concluded that these observations were missing due to data collection limitations. Since these were edge cases and imputing these durations could greatly bias my later predictions I decided to remove these observations from the dataset. I found this to be acceptable as the missing observations came from many different states (see the plots below). Note that some states may have more than one missing state in a year, but there are only ~50 missing outage durations total.  

<iframe
  src="assets/missing_states_by_year.html"
  width="600"
  height="400"
  frameborder="0"
>States with at least one missing outage duration by year</iframe>

Hurricane name was another column. Any outages that didn't occur during a hurricane were missing. I binarized this column so all missing values were 0 (no hurricane) and 1 for observations with a hurricane name (hurricane occurred). This would lose minimal information as each hurricane would be associated with a month and year from the same observation and the name is mostly arbitrary (ignoring the first letter).  

Looking at the average outage duration when there was or was not a hurricane, it appears that hurricanes have longer outages on average, regardless of NERC region:

|  NERC   | No Hurricane |Hurricane |
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

There were two observations, one from 2002 and the other from 2006, that were missing electricity prices. Since these seemed anomalous as all other columns had associated prices I removed them (assuming MCAR).  

## Assessment of Missingness

### NMAR Analysis
As mentioned previously, demand loss and customers were removed. Meeting at least one of these was a criterion for being included in the dataset. If both criterions were NA then interpolating one or both values could lead to incorrect expectations for a specific occurrence (i.e. you can't know which criteria caused the outage to be reported). If one criterion is missing, there is no way to distinguish whether this missing value didn't meet the criterion threshold or if the value is missing due to chance. If neither criterion is missing, then nothing needs to be done. However, a majority of points do not include both criterion as only 1 criterion is required to be reported. By this logic the missing values must be NMAR as you can't know their values with any other information available, except their actual values. 

In order to make these columns MAR I would need to know the number of criteria met when the observation was reported. Right now I know at least one criterion was met, but without know the exact number there is still ambiguity whether one or both were met.  

### Missingness Dependency

Since there weren't many missing values in some cases, I wanted to infer whether or not missingness depended on certain features.  

I first compared the missingness by year to infer whether the distribution appeared different by chance or if there was actually a difference between the missing and non-missing distributions for each year.

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
$H_0$: Missing and non-missing outage duration observations follow the same year distribution.  
$H_1$: Missing and non-missing outage duration observations follow the same year distribution.  

Using the Kolmogorov Smirnov (KS) test for each permutation I found a distribution of p-values:
<iframe
  src="assets/ks_test_year_p_values.html"
  width="600"
  height="400"
  frameborder="0"
>KS test p-value distribution</iframe>

In ~99.998% of these trials (10,000 total), we would find the missing and non-missing outage durations to follow a different distributions of outages per year. This implies that outage durations are missing at random (MAR), conditional on the year.  

I also performed a similar test for hurricane prevalence (whether or not a hurricane occurred). Here I performed a difference in means test by treating hurricane and no hurricane as discrete.  

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

Instead of finding the p-value distribution I found the distribution of mean difference. These differences can be interpreted as the proportion difference between the bootstrapped and expected hurricane prevalences.  

<iframe
  src="assets/mean_diff_results.html"
  width="600"
  height="400"
  frameborder="0"
>Mean difference distribution</iframe>

About 50% (p-value = 0.497) of non-missing bootstrapped mean differences are on either side of the actual mean difference. This indicates that knowing whether or not a hurricane occurred is not helpful in predicting missing values. The hurricane feature is likely MCAR.  

## Hypothesis Testing
When looking at all outage durations, I was curious whether an exponential distribution was followed since this is a time based feature. I also thought the empirical distribution looked kind of similar to an exponential distribution.  

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

$H_0$: Outage duration follows an exponential distribution.  
$H_1$: Outage duration does not follow an exponential distribution.    

I parameterized the null distribution with the maximum likelihood estimator $\beta = mean$ from the empirical distribution. This isn't a high stakes test, so I used a standard significance level of $\alpha = 0.05$.   

Using a 2-sample KS test I concluded that it was unlikely the outage duration follows an exponential distribution (p-value = $3.32*10^{-12}$). This wasn't a huge surprise as the empirical distribution has a much larger concentration of points towards 0, compared to the MLE distribution.  


