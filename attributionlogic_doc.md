
## Overview of Paid Digital Marketing Attribution process

We outline the data ingestion, preparation and attribution logic for the Paid Media Dashboard 2.0 (henceforth referred to as **PMD 2.0** in this post). 

Please refer to the [Github repo here](https://github.com/nytm/dig-mkt/tree/develop/dags/media_dashboard). 


**Sections:**

1. Data source queries : Digital media data sources & transformations

2. Attribution logic: mapping media delivery to validated conversions

3. Daily processing & ingestion in Paid Media Dashboard v2.0


## Data feeds: Digital media data sources

Existing ingested paid media data sources for PMD 2.0 can be grouped into the following categories: 

- delivery data by platform: `dcm_delivery.sql`, `aw_delivery.sql`, `fb_delivery.sql`, `tw_delivery.sql`
- conversions data by platform: `dcm_conversions.sql`, `fb_conversions.sql`
- NYT system of record (SOR) data: `sor_conversions.sql`
- agencies' taxonomy table: `kepler_master.sql`, `kepler_search_master.sql`

### Data source summary for PMD 2.0 

Raw media data is queried using BigQuery with the respective SQL scripts stored in `*.sql` files under the `sql_v2` folder. 

`dig-mkt/dags/media_dashboard/` repo contains: 
```
/media_dashboard   
│   media_dashboard_v2.py
│   __init__.py
│
└───archive
└───sql_v2
│   │   aw_delivery.sql
│   │   dcm_conversions.sql
│   │   dcm_delivery.sql
│   │   dcm_media_conversions.sql
│   │   fb_conversions.sql
│   │   fb_delivery.sql
│   │   fb_delivery_octopus.sql
│   │   fb_media_conversions.sql
│   │   kepler_master.sql
│   │   kepler_search_master.sql
│   │   snap_delivery.sql
│   │   sor_conversions.sql
│   │   tw_delivery.sql
│   │   tw_delivery_octopus.sql
│   │   twitter_media_conversions.sql

```

## Media impression <-> user conversion attribution logic

The _media_dashboard_v2.py_ script ingests our raw media delivery & conversion data from various digital ad platforms and outputs aggregated **impression** and **validated conversions** data (validated by a join with our internally stamped SOR conversions, that also fall into select attributed window durations) that are grouped by key media dimensions of interest for our PMD 2.0 dashboard filters, such as _site_, _campaign_, _placement_ etc.. 

We provide an brief overview of the key logic & operations in this script. 

### the script's main functions

The main 5 functions are shown below (at the end of the script). 

```python
def run_all(project, key=key, **context):
# def run_all(start,end):
    start = datetime.strftime((context['execution_date'] -  timedelta(days=1)), '%Y-%m-%d')
    end = start
    start_nodash = start.replace('-','')

    imps_df,subs_df,media_subs_df,sor_convs,kepler_master = get_data_all(start,end)
    # regi_df = subs_df[subs_df['activity'] == 'Regi Conversion Pixel']
    subs_val_df = starts_validation(subs_df, sor_convs)
    subs_final = conversion_attr(subs_val_df,media_subs_df)
    media_data = merge_media_data(imps_df, subs_final)
    media_data_meta = parse_meta(media_data, kepler_master)
```
### Setup to process .py file

Required BQ table access needed - ask BQ Slack channel for user access:
```
nytdata.twitter_ads.*
nytdata.fb_insights.*
nyt-bigquery-beta-workspace.paid_media_data.*
nyt-octopus-prd.snapchat.*
```

Besides installing the relevant python libraries for this script, we also need to store a `hash_key` as an environment variable before we can process functions in the converstion validation section. 

```python
key = os.environ['HASH_KEY'].strip('\n')
```

The `key` variable storing the hash key in our environment variable is used in the `starts_validation(subs_df, sor_convs)` function to hash our unhashed `'sor_sub_id'`s from DSSOR records, to create `sor_conv['subscription_id']` used for a JOIN operation with unvalided media conversion data. 
```python
sor_conv['sor_sub_id'].apply(lambda x: create_signature(x, key))
```

### i. Function to pull and process raw media data

Impressions dataframe contains the following fields: 
```python 
imps_columns = ['data_source','date','account','account_id','site','campaign','campaign_id',
'placement','placement_id','_match','impressions','clicks','spend']
```
![alt text](image1.jpg)

Conversion dataframe contains the following fields: 
```python 
subs_columns = ['data_source','event_date','interaction_time','conv_window','account','account_id','site','campaign','campaign_id',
'placement','placement_id','_match','activity', 'interaction_type', 'agent_id', 'regi_id','subscription_id']
```
![alt text](image2.jpg)


Media conversion dataframe contains the following fields: 
```python 
media_subs_columns = ['_match','account','account_id','attr_window','campaign','campaign_id','data_source','date','placement','placement_id','site','sor_prod','view_conv','click_conv']
```

### ii. Functions to validate & dedup conversions 

Text

Text



### iii. Function to 

Text

Text


### iv. Functions to transform and merge imps <> attributed conversions

Text



### v. Functions to map media dimensions IDs to taxonomy meta data

Text



## Some questions for clarification by Marketing Analytics team

-[ ] Question: In section 1 under pull_data() function, `media_subs_df` was recently added in February 2020. How is this new media conversion dataframe different from the existing conversion dataframe `subs_df`? 

Ans:

-[ ] Question:

Ans:

-[ ] Question:

Ans:

-[ ] Question:

Ans:

-[ ] Question:

Ans:

-----------
