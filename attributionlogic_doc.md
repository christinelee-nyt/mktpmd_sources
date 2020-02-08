
## Overview of Paid Digital Marketing Attribution logic

We outline the data ingestion, preparation and attribution logic for the Paid Media Dashboard 2.0 (henceforth referred to as **PMD 2.0** in this post). 

**Sections:**

1. Data feeds: Digital media data sources & transformations

2. Attribution logic in _media_dashboard_v2.py_: mapping media delivery to validated conversions

3. Daily processing & ingestion in Paid Media Dashboard v2.0


## Data feeds: Digital media data sources

Existing ingested paid media data sources for PMD 2.0 can be grouped into the following categories: 

- delivery data by platform: `dcm_delivery.sql`, `aw_delivery.sql`, `fb_delivery.sql`, `tw_delivery.sql`
- conversions data by platform: `dcm_conversions.sql`, `fb_conversions.sql`
- NYT system of record (SOR) data: `sor_conversions.sql`
- agencies' taxonomy table: `kepler_master.sql`, `kepler_search_master.sql`

### Data source summary for PMD 2.0 

Raw media data is queried using BigQuery with the respective SQL scripts stored in `*.sql` files under the `sql_v2` folder. 

`dig-mkt/dags/media_dashboard/` repo:

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

The _media_dashboard_v2.py_ script ingests our raw media delivery & conversion data from various digital ad platforms and outputs aggregated **impression** and **validated conversions** data (validated by a join with our internally stamped SOR conversions, that also fall into select attributed windows) grouped by the key media dimensions of interest for our PMD 2.0, such as _site_. _campaign_, _placement_. 

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
### setup

Besides installing the relevant python libraries for this script, we also need to store a `hash_key` as an environment variable before we can process functions in the converstion validation section. 

```python
key = os.environ['HASH_KEY'].strip('\n')
```

The `key` variable storing the hash key in our environment variable is used in the `starts_validation(subs_df, sor_convs)` function to hash our unhashed `'sor_sub_id'`s from DSSOR records, to create `sor_conv['subscription_id']` used for a JOIN operation with unvalided media conversion data. 
```python
sor_conv['sor_sub_id'].apply(lambda x: create_signature(x, key))
```

### i. Functions to pull raw data

Text



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

#### Subtitle
Text

#### Subtitle
Text

#### Subtitle
Text

#### Subtitle
Text



