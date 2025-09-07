# Major US Power Outages (2000-2016)
Supplementary project for DSC 80.

## Introduction
Over time, power outages have had an increasing impact with the increased use of digital technologies. Preventing these outages can be useful for keeping the US productive. When researching electrical grids working with [this](https://engineering.purdue.edu/LASCI/research-data/outages) power outage dataset, I focused on the economic and climate factors related to power outages.  

This outages dataset contains information from January 2000 to June 2016 for power outages occurring in US states (including non-continental states). The uncleaned dataset contains 1534 observations and 55 features. The most interesting columns were anomaly level and NERC region. Based on an oceanic temperature index, variations in ocean temperature at the time of the outage were reported. These temperatures were cyclical with an increasing amplitude toward 2016. This feature was useful when considering what the climate was like at the time of the outage (e.g. cold, hot) The NERC (North American Electric Reliability Corporation) regions differentiated the primary electrical power supplier for each state. During the mid 2000's there was a NERC merger that needed to be accounted.  

## Data Cleaning and Exploratory Data Analysis

When reading in the data from an Excel file, the column names were formatted with periods instead of underscores. I reformatted the column names to be in snake case with underscores. Additionally, many of the datatypes were read in as strings by default, so I converted most of these columns to floats (e.g. outage duration in minutes), along with a few datetimes (e.g. the outage start date and time).  

Some notable information about the locations where data was collected is that Washington D.C. had major reported outages during this time period, but Rhode Island had none (and thus was not listed as a state for any observations). A major outage was defined as 50,000 impacted customers or a significant loss of power (300 megawatts), both of which are similar in impact level, based on average household electricity consumption (~0.02 megawatts per day, although this can vary by location).  

When cleaning the NERC regions I consolidated Alaska and Hawaii into non-continental, as NERC technically consists of only the continental states (Alaska and Hawaii have their own separate power grids). ECAR was one of three regions that [merged into the RFC region](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) (p. 102). Although ECAR was the only region of the 3 that merged that was reported here. If you're curious about NERC regions, here is a [map](https://atlas.eia.gov/datasets/eia::nerc-regions/explore) I found from the Energy Information Administration that guided my inquiries for some of the oddly classified NERC regions (e.g. FRCC, SERC I classified under FRCC since that observation with associated with the state Florida). Florida actually [split from SERC](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) in 1996 to be its own member in NERC (p. 171). This was before the data was collected so I disregarded FRCC's membership history. [Here](https://upload.wikimedia.org/wikipedia/commons/f/f4/NERC-map-en.svg) is a colored map that I found helpful for quickly differentiating the NERC regions.  

As mentioned in the introduction, there was an ocean temperature anomaly level reporting that uses an oceanic index to compare expected ocean temperatures to their actual temperatures. If you're curious, you can find the temperature differences [here](https://origin.cpc.ncep.noaa.gov/products/analysis_monitoring/ensostuff/ONI_v5.php). There are blue, red and grey colors temperatures on that NOAA website. Another feature in the dataset considered all blue temperatures to be cold (<-0.5°C), red temperatures to be warm (>0.5°C) and grey to be normal (within +/- 0.5°C).  

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




