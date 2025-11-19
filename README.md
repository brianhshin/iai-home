
# iai-home

tldr; some python to scrape zillow results and save them to a s3 bucket


## Description

```bash
.
├── config
│   ├── zillow_search_config.yaml
├── iai_utils.py
├── zillow_ingest.py
└── README.md
```


`zillow_ingest.py` takes Zillow search parameters from `zillow_search_config.yaml` and scrapes listings using bs4 in a function from `iai_utils.py`.
The listings get uploaded to my S3 bucket as a json payload using another function from `iai_utils.py`.


An example of one of the searches to scrape from the Zillow search config:
```yaml
- name: south-end-boston-ma
  data:
    neighborhood: South End
    city: Boston
    state: MA
    region_id: "275429"
    price_min: 400000
    price_max: 700000
```

The listings data comes back as a json response and they are limited by search pagination- one payload per page. I set it to iterate on each page and scraped the complete set of results. All the data gets compiled into one array, and I added some metadata fields for tracking the scraping source (zillow) and the ingestion timestamp (when the script ran). The data gets uploaded to my S3 bucket set by the search name and date (ex: `south_end_boston_ma_20250501`). I idea is to run the script once a day. I also make no transformations to the data, as I'll be doing that in dbt. I'm only concerned with getting the raw data into S3.

Once the data is in S3, the next step is to read it directly from a Snowflake external stage using a storage integration linked to my AWS creds. I'll be able to query the unstructured data directly in real time. If I wanted to reduce the time between updates from daily to hourly, I would only have to worry about getting it into S3 faster.
