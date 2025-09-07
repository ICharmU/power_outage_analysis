# Major US Power Outages (2000-2016)
Supplementary project for DSC 80.

## Introduction
Over time, power outages have had an increasing impact with the increased use of digital technologies. Preventing these outages can be useful for keeping the US productive. When researching electrical grids working with [this](https://engineering.purdue.edu/LASCI/research-data/outages) power outage dataset, I focused on the economic and climate factors related to power outages.  

This outages dataset contains information from January 2000 to June 2016 for power outages occurring in US states (including non-continental states). The uncleaned dataset contains 1534 observations and 55 features. The most interesting columns were anomaly level and NERC region. Based on an oceanic temperature index, variations in ocean temperature at the time of the outage were reported. These temperatures were cyclical with an increasing amplitude toward 2016. This feature was useful when considering what the climate was like at the time of the outage (e.g. cold, hot) The NERC (North American Electric Reliability Corporation) regions differentiated the primary electrical power supplier for each state. During the mid 2000's there was a NERC merger that needed to be accounted.  

## Data Cleaning and Exploratory Data Analysis
When reading in the data from an Excel file, the column names were formatted with periods instead of underscores. I reformatted the column names to be in snake case with underscores. Additionally, many of the datatypes were read in as strings by default, so I converted most of these columns to floats (e.g. outage duration in minutes), along with a few datetimes (e.g. the outage start date and time).  

Some notable information about the locations where data was collected is that Washington D.C. had major reported outages during this time period, but Rhode Island had none (and thus was not listed as a state for any observations). A major outage was defined as 50,000 impacted customers or a significant loss of power (300 megawatts), both of which are similar in impact level, based on average household electricity consumption (~0.02 megawatts per day, although this can vary by location).  

When cleaning the NERC regions I consolidated Alaska and Hawaii into non-continental, as NERC technically consists of only the continental states (Alaska and Hawaii have their own separate power grids). ECAR was one of three regions that [merged into the RFC region](https://www.nerc.com/news/Documents/NERCHistoryBook.pdf) (p. 102). Although ECAR was the only region of the 3 that merged that was reported here. If you're curious about NERC regions, here is a [map](https://atlas.eia.gov/datasets/eia::nerc-regions/explore) I found from the Energy Information Administration.



