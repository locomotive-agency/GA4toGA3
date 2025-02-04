# GA3toGA4
A Python tool for formatting GA4 data to match and be backfilled with historical GA3 data in BigQuery.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1X7bWFcxsoe2WJ892vbbpsERNurv28k7S?usp=sharing)


![Logo for GA3toGA4](https://github.com/locomotive-agency/GA3toGA4/blob/7bae90e97605ea0de97e649eb6a45541ebfe5b31/ga3toga4-2.png)

<p align="center"> 
  <a href="https://datastudio.google.com/u/0/reporting/58d63144-52a0-424e-a30e-fc46d0edc72f/page/I7luC" target="_blank">View the Dashboard</a>
</p>


<br/><br/>

## About

### Welcome to GA3 to GA4 tool

:warning: **Warning**: This tool should be considered alpha software and likely to break.  It has been tested on a handful of sites and is shared as a proof-of-concept.  Use at your own risk.  While we will not support this open-source tool, please submit issues so we can make it better.

GA3 to GA4 is a tool for creating and filling a Bigquery table with available GA4 data and formatting it to be compatible with historical data pulled from the Google Analytics Reporting API v4 for GA3 data. This tool has been designed to run in google colab and to automate a majority of the process.

This notebook aims to provide 3 main functions
1. Create table with select GA4 data formatted to GA3 data style.
2. Set up a scheduled query for a daily recurring pull of new GA4 data.
3. Backfill formatted GA4 data with historical GA3 data

<br/><br/>

#### Pulled GA3 & GA4 Data

| GA3                   | GA4                             | 
| ----------------------|---------------------------------|
| ga:date               | date                            | 
| ga:hostname           | landing_page                    | 
| ga:landingPagePath    | landing_page                    | 
| ga:country            | country                         |
| ga:region             | region                          |
| ga:city               | city                            |
| ga:source             | utm_source                      |
| ga:medium             | utm_medium                      |
| ga:campaign           | utm_campaign                    |
| ga:users              | users                           |
| ga:newUsers           | new_users                       |
| ga:entrances          | entrances                       |
| ga:sessions           | sessions                        |
| ga:uniquePageviews    | unique_page_views               |
| ga:timeOnPage         | engagment_time_sec_per_session  |
| *goal_metric*         | *conversions*                   |
| ga:transactionRevenue | ecommerce_revenue               |
| ga:transactions       | ecommerce_transactions          |

The table above shows what GA3 data is matched with what GA4 data by default. You can edit these values by changing ga4-to-ga3-query.sql and process_ga3.py. 

**Note**: goal_metric is only a placeholder for whatever GA3 conversion event you wish to pull. Defaults to ga:goalCompletionsAll.

<br/><br/><br/>

## How To Use

### Google Cloud project set-up

This tool assumes that you have a Google Cloud project created already. This project will contain your from dataset (raw GA4 data from a daily export) and your to dataset (select GA4 data formatted to GA3 style) that will be created by this tool.

Ensure that BigQuery Daily Export is set up for your chosen GA4 account to your desired Google Cloud project and that you have a table with your raw GA4 data. If you don’t, follow the sets [here](https://support.google.com/analytics/answer/9358801?hl=en) to set it up. This should be set up **without** Streaming Export, and should just be a Daily Export.

### Part 1: GA3 to GA4 SQL Workflow

First, open the notebook we have shared using Google Colab here: https://colab.research.google.com/drive/1X7bWFcxsoe2WJ892vbbpsERNurv28k7S?usp=sharing

You can create your own editable copy of the notebook by going to File > Save a copy...

#### Install BigQuery Data Transfer
This is needed to set up the reoccuring queries. 

You will need to restart the runtime of this notebook after running this cell.

This can be done by going to Runtime > Restart runtime or pressing CTRL + M + .

![Restart runtime location](https://user-images.githubusercontent.com/89162068/171264419-eadd41e1-34ab-4896-99fe-b66f6eba9c8f.png)

#### Load needed libraries

Run cell to load in the libraries needed for this tool.

#### Sign in and Authenticate

Run the cells and input the project name of your Google Cloud Project. You will be asked to sign in. Choose the account with your Google Cloud project on, which should be the same account you are using in colab.

These cells give access to the Google Drive of the account you sign into, as well as setting up a service account and enabling the needed api permissions within Google Cloud. 

You will have to add the created service account with read access to your GA3 data.

You do this by going to Google Analytics, selecting the GA3 view that you want to use, clicking Admin in the bottom right, looking in the property settings and selecting property access management. Once there, select the blue button on the top right to add another user. There, you add the service account email with viewer permissions.

![Adding service account to GA3 property visual](https://user-images.githubusercontent.com/89162068/171265347-3b0dd2cf-b9ab-4f5b-a06f-caac99a9e170.png)

#### Specify configuration parameters for the Source and Target Datasets

For the BigQuery From Table (Source Dataset) you need to input the following:

* `from_dataset_id` : name of the dataset in your project with your GA4 data.
* `from_table_prefix` : prefix of the tables in source dataset. Most of the time will be “events_”.
* `from_conversion_event` : Optional, set conversion event from GA4 you want to pull.
* `from_initial_pull_prior_days` : How many days back from initial pull do you want.

For the BigQuery To Table (Target Dataset) you will need to input the following:
* `to_dataset_id` : Name of the dataset you want created/to use in your project to hold modified GA4 data.
* `to_table_name` : Name of the table you want created in the target dataset.

#### Authenticate BigQuery

Run cell to authenticate BigQuery Client.

#### Built Initial Table
Run to pull GA4 data and create the target dataset and table, will error if table is already created under specified name and location.

#### Set Up Daily Reoccuring Pull
Run to set up the daily scheduled query in Google Cloud. If it errors, check that you have restarted your runtime after [installing BigQuery Data Transfer](https://github.com/locomotive-agency/ga4toga3/edit/main/README.md#install-bigquery-data-transfer).

#### Review Data
Run to pull data from the target dataset for review.

### Part 2: Backfil GA3 data

#### Specify configuration parameters for the GA3 View

You need to input the following:

* `ga3_view_id` : The GA3 view ID that you want to use to backfill your GA4 data with.
* `pull_start_date` : The beginning of the timeframe of GA3 data that you want to pull.
* `goal_metric` : GA3 goal you want to pull.

#### Backfill GA3 Data
Run to backfil GA3 data to your GA4 data. 

**Note**: If you want to add a conversion event, you will need to add it to the metrics list and to the order dictionary in in lib/process_ga3.py.

#### Review Data
Run to pull data from the target dataset for review.

<br/><br/><br/>

## Acknowledgements
* Thank you to [JR](https://github.com/jroakes), [Joe](https://github.com/joejoinerr), and [Savannah](https://github.com/SavannahCasto) for the work on the creation of this tool.
* Thanks to Derek Perkins of [Nozzle](https://nozzle.io/) for SQL code review.
* This tool was created and released open for the SEO community by [LOCOMOTIVE.agency](https://locomotive.agency/).

<br/><br/><br/>

## Feedback
If any errors, concerns, or areas for improvement are found, please open an issue for it on this repo.
If you want to change something, please make a pull request.
We would love to feature dashboards users have made with the data.  Please let us know if you have any to share!

<br/><br/><br/>


