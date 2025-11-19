
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

-  The listings data comes back as a json response and they are limited by search pagination- one payload per page. I set it to iterate on each page and scraped the complete set of results.
- All the data gets compiled into one array, and I added some metadata fields for tracking the scraping source (zillow) and the ingestion timestamp (when the script ran).
- The data gets uploaded to my S3 bucket set by the search name and date (ex: `south_end_boston_ma_20250501`). The idea is to run the script once a day.
- I also make no transformations to the data, as I'll be doing that in dbt. I'm only concerned with getting the raw data into S3.

Once the data is in S3, the next step is to read it directly from a Snowflake external stage using a storage integration linked to my AWS creds. I'll be able to query the unstructured data directly in real time. If I wanted to reduce the time between updates from daily to hourly, I would only have to worry about getting it into S3 faster.

Here is an example of one home from the scraped payload:
```json
{
  "zpid": "457572532",
  "palsId": "12004_73445578",
  "id": "457572532",
  "rawHomeStatusCd": "ForSale",
  "marketingStatusSimplifiedCd": "For Sale by Agent",
  "imgSrc": "https://photos.zillowstatic.com/fp/0e2e3b0a1da804c68ac9f43f46832635-p_e.jpg",
  "hasImage": true,
  "detailUrl": "https://www.zillow.com/homedetails/1200-Washington-St-213A-Boston-MA-02118/457572532_zpid/",
  "statusType": "FOR_SALE",
  "statusText": "Condo for sale",
  "countryCurrency": "$",
  "price": "$634,500",
  "unformattedPrice": 634500,
  "address": "1200 Washington St #213A, Boston, MA 02118",
  "addressStreet": "1200 Washington St #213A",
  "addressCity": "Boston",
  "addressState": "MA",
  "addressZipcode": "02118",
  "isUndisclosedAddress": false,
  "beds": 1,
  "baths": 1,
  "area": 952,
  "latLong": {
      "latitude": 42.342796,
      "longitude": -71.06593
  },
  "isZillowOwned": false,
  "flexFieldText": "Private and quiet bedroom",
  "contentType": "homeInsight",
  "hdpData": {
      "homeInfo": {
          "zpid": 457572532,
          "streetAddress": "1200 Washington St #213A",
          "zipcode": "02118",
          "city": "Boston",
          "state": "MA",
          "latitude": 42.342796,
          "longitude": -71.06593,
          "price": 634500,
          "bathrooms": 1,
          "bedrooms": 1,
          "livingArea": 952,
          "homeType": "CONDO",
          "homeStatus": "FOR_SALE",
          "daysOnZillow": 30,
          "isFeatured": false,
          "shouldHighlight": false,
          "listing_sub_type": {"is_FSBA": true
      },
      "isUnmappable": false,
      "isPreforeclosureAuction": false,
      "homeStatusForHDP": "FOR_SALE",
      "priceForHDP": 634500,
      "timeOnZillow": 2649037000,
      "isNonOwnerOccupied": true,
      "isPremierBuilder": false,
      "isZillowOwned": false,
      "currency": "USD",
      "country": "USA",
      "unit": "# 213A",
      "lotAreaValue": 952,
      "lotAreaUnit": "sqft",
      "isShowcaseListing": false
  }
  },
  "isSaved": false,
  "isUserClaimingOwner": false,
  "isUserConfirmedClaim": false,
  "pgapt": "ForSale",
  "sgapt": "For Sale (Broker)",
  "shouldShowZestimateAsPrice": false,
  "has3DModel": false,
  "hasVideo": false,
  "isHomeRec": false,
  "hasAdditionalAttributions": true,
  "isFeaturedListing": false,
  "isShowcaseListing": false,
  "list": true,
  "relaxed": false,
  "info6String": "Danielle Bing Team",
  "brokerName": "Gibson Sotheby's International Realty",
  "carouselPhotosComposable": {
  "baseUrl": "https://photos.zillowstatic.com/fp/{photoKey}-p_e.jpg",
  "communityBaseUrl": null,
  "photoData": [
      {"photoKey": "0e2e3b0a1da804c68ac9f43f46832635"},
      {"photoKey": "7b993e1d82c66ce770fd078575f74be0"},
      {"photoKey": "70ec7db8044a2421d7616e9324842a81"},
      {"photoKey": "baa36608eb008c173b4b5e8316858789"},
      {"photoKey": "a330359d1eb7421800b59c7b29b52149"},
      {"photoKey": "06e675f697369de065173bbbf4593317"},
      {"photoKey": "2b26cd1ba3d575de7ce42ce55f87c3ba"},
      {"photoKey": "04b0d0a847ea8ab38fbb911054a32715"},
      {"photoKey": "466a66798cd61cfb2f6be3182246be47"},
      {"photoKey": "f5b7f144f1372dc2df001c59593a767c"},
      {"photoKey": "931dbe3bc282ec481a07f05126249a1b"},
      {"photoKey": "b04df402e5f06a27b1e64ad29f80e78a"},
      {"photoKey": "6a2b62b689b555230b9863607480c5b1"},
      {"photoKey": "4241a8aec6d7391e463d2afa3bfaefff"},
      {"photoKey": "cb8210595c46b1f91c6b9db72f804e92"},
      {"photoKey": "d72c9d8760a24952c039925240e9f233"},
      {"photoKey": "d1ed596cf2320dbde07b7f31ae2ec83a"},
      {"photoKey": "2479ec979e60c28a0f75b91ec13d8efb"},
      {"photoKey": "1419519c3f53dc40e4dde94130454e3f"},
      {"photoKey": "da8fc2582b36503254edd56e35bb9e33"},
      {"photoKey": "c7e41f4a13a0ec4ea108c263933d7653"},
      {"photoKey": "7939cf6fe63a35ebd4b9901e55b6dfb2"}
  ],
  "communityPhotoData": null,
  "isStaticUrls": false
  },
  "isPaidBuilderNewConstruction": false
}
```
