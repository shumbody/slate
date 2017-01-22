---
title: API Reference

<!-- language_tabs:
  - cURL -->



includes:
  - errors

search: true
---

# Introduction

The Thread Genius API offers fashion image recognition as a service. Use it to tag photos, perform visual search on your product catalog, or find products within your photos.

# Authentication

> To retrieve an access token:

```shell
curl "http://api.threadgenius.co/v0/token" \
  -d "api_key={api_key}" \
  -d "api_secret={api_secret}"
```

> The JSON response will include your <code>access_token</code>. Access tokens expire regularly and must be renewed on an ongoing basis. You can renew by just retrieving a new <code>access_token</code> as described above.

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "access_token": "PKmWwSGYF4szvyY7",
  "token_type": "bearer",
  "expires_in_seconds": 86399
}
```

Authentication to the API is handled using OAuth2 client credentials. Each user has a unique <code>api_key</code> and <code>api_secret</code> which you will use to exchange for an <code>access_token</code>. You then use this <code>access_token</code> to make authorized API calls.

This is achieved using the Authorization header:

`Authorization: Bearer PKmWwSGYF4szvyY7`


### HTTP Request

`GET http://api.threadgenius.co/v0/token/`

### Query Parameters

Parameter | Description
--------- | -----------
`api_key` | (Required) Your API Key
`api_secret`| (Required) Your API Secret


# Catalogs

## Overview

A catalog is a collection of image URLs, each associated with custom metadata you provide. When you perform visual search on a collection, we return images in a selected collection that are most similar to the query image.

### Catalog Fields

Parameter | Description
--------- | -----------
`catalog_id` | A 16-character ID we generate when you create a new catalog
`name` | A name you provide for the catalog on creation
`num_images_total` | The total number of images in your catalog
`num_images_indexed` | The number of images that have been indexed so far
`error_log` | If uploading or indexing results in an error, the output of the error is written here
`status` | One of `new`, `uploading`, `upload done`, `upload error`, `indexing`, `ready for query`, or `index error`
`date_created` | The date and time your catalog was created



## Create new catalog

```shell
curl "http://api.threadgenius.co/v0/catalog" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7" \
  -d "name=My Awesome Catalog"
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "catalog": {
    "catalog_id": "cdae7e788f1e4982",
    "date_created": "2016-12-12 09:24:54.301953+00:00",
    "name": "My Awesome Catalog",
    "status": "new"
  }
}
```

This endpoint creates a new empty catalog and returns the <code>catalog_id</code>.

### HTTP Request

`POST http://api.threadgenius.co/v0/catalog/`

### Query Parameters

Parameter | Description
--------- | -----------
`name`     | (Required) A name for your catalog


## Upload catalog file


```shell
curl "http://api.threadgenius.co/v0/catalog/cdae7e788f1e4982/upload" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7" \
  -F "catalog_file=@{/path/to/catalog_file}"
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "catalog": {
    "catalog_id": "cdae7e788f1e4982",
    "date_created": "2016-12-12 09:24:54.301953+00:00",
    "name": "My Awesome Catalog",
    "status": "uploading"
  }
}
```

Use this endpoint to populate your newly created endpoint with records. 

### HTTP Request

`POST http://api.threadgenius.co/v0/catalog/<catalog_id>/upload`

### URL Parameters

Parameter | Description
--------- | -----------
`catalog_id` | The ID of the catalog

### Query Parameters

Parameter | Description
--------- | -----------
`catalog_file` | A file listing image urls and associated IDs

### Catalog File Format
The <code>catalog_file</code> must be tab-delimited with the following entries.

Image ID (Required) | Image URL (Required) | JSON-readable metadata (Optional)
--------- | ----------- | -----------
`1` | `http://cdn.cool.images/url1.jpeg` | `{"price": 19.99, "SKU": 120984, "categories": ["sweatshirt", "red"]}`
`2` | `http://cdn.cool.images/url2.jpeg` | `{"price": 49.99, "SKU": 231235}`
... | ... | ...


## Index a catalog


```shell
curl "http://api.threadgenius.co/v0/catalog/cdae7e788f1e4982/index" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7"

```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "catalog": {
    "catalog_id": "cdae7e788f1e4982",
    "date_created": "2016-12-12 09:24:54.301953+00:00",
    "name": "My Awesome Catalog",
    "num_images_total": 50123,
    "status": "index in progress"
  }
}
```

In order to start searching against a catalog, we need to first build an index from the images. Use this endpoint to initiate an indexing job. During the indexing task, <code>num_images_indexed</code> updates dynamically.

### HTTP Request

`POST http://api.threadgenius.co/v0/catalog/<catalog_id>/index`

### URL Parameters

Parameter | Description
--------- | -----------
`catalog_id` | The ID of the catalog


## Get a catalog


```shell
curl "http://api.threadgenius.co/v0/catalog/cdae7e788f1e4982" \
  -H "Authorization: PKmWwSGYF4szvyY7"
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "catalog": {
    "catalog_id": "cdae7e788f1e4982",
    "date_created": "2016-12-12 09:24:54.301953+00:00",
    "name": "My Awesome Catalog",
    "num_images_indexed": 50123,
    "num_total_images": 50123,
    "status": "ready for query"
  }
}
```

Use this endpoint to get information about a catalog, including the status of uploading and indexing.

### HTTP Request

`GET http://api.threadgenius.co/v0/catalog/<catalog_id>`

### URL Parameters

Parameter | Description
--------- | -----------
`catalog_id` | The ID of the catalog


## Get all catalogs


```shell
curl "http://api.threadgenius.co/v0/catalog/" \
  -H "Authorization: PKmWwSGYF4szvyY7"
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "catalog": [
    {
      "catalog_id": "cdae7e788f1e4982",
      "date_created": "2016-12-12 09:24:54.301953+00:00",
      "name": "My Awesome Catalog",
      "num_images_indexed": 50123,
      "num_total_images": 50123,
      "status": "ready for query"
    },
    {
      "catalog_id": "004882361564b5ad",
      "date_created": "2016-12-11 07:03:43.222671+00:00",
      "name": "Test Catalog",
      "num_total_images": 100,
      "status": "upload done"
    }
  ]
}
```

Use this endpoint to list all your catalogs.

### HTTP Request

`GET http://api.threadgenius.co/v0/catalog/`


## Delete a catalog


```shell
curl "http://api.threadgenius.co/v0/catalog/cdae7e788f1e4982" \
  -X DELETE \
  -H "Authorization: PKmWwSGYF4szvyY7"
```

> Example Response:

```json
{
  "status": {
    "message": "Catalog deleted.",
    "code": 200
  },
  "catalog": {
    "catalog_id": "cdae7e788f1e4982"
  }
}
```

Use this endpoint to delete a specific catalog.

### HTTP Request

`DELETE http://api.threadgenius.co/v0/catalog/<catalog_id>`

### URL Parameters

Parameter | Description
--------- | -----------
`catalog_id` | The ID of the catalog


# Tagging

## Tag an Image

```shell
curl "http://api.threadgenius.co/v0/tag" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7" \
  -d "url=https://cdnc.lystit.com/photos/notre-shop/7807782918-Washed%20Black-b87a8c25-.jpeg"
```

> Example Response:

```json
{
  "status": {
      "message": "OK",
      "code": 200
    },
  "labels": [
    {
      "confidence": 0.7053355574607849,
      "type": "shape",
      "name": "chelsea boots"
    },
    {
      "confidence": 0.5393210053443909,
      "type": "color",
      "name": "color charcoal"
    }
  ]
}
```

> To query using an image file,

```shell
curl "http://api.threadgenius.co/v0/tag" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7" \
  -F "image=@{/path/to/image.jpeg}"
```


This endpoint automatically tags your image with attributes.

### HTTP Request

`POST http://api.threadgenius.co/v0/tag/`

### Query Parameters

Provide either a image URL or the image file itself.

Parameter | Description
--------- | -----------
`url` | (Either) The URL of the image
`image` | (Either) The file of the image

# Visual Search

## Find similar products

```shell
curl "http://api.threadgenius.co/v0/search" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7" \
  -d "url=https://cdnc.lystit.com/photos/notre-shop/7807782918-Washed%20Black-b87a8c25-.jpeg" \
  -d "catalog_id=cdae7e788f1e4982"
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "similars": [
    {
      "url": "https://cdn-images.farfetch.com/11/64/93/56/11649356_7801384_800.jpg",
      "distance": 0.47817564010620117,
      "iid": "1280619564",
      "metadata": {
        "categories": ["boots"],
        "price": 289.99
      }
    },
    {
      "url": "https://cdn-images.farfetch.com/11/14/62/38/11146238_5438398_800.jpg",
      "distance": 0.5194926857948303,
      "iid": "491260395",
      "metadata": {
        "categories": ["boots"],
        "price": 159.99
      }
    }
  ]
}


```

> To query using an image file,

```shell
curl "http://api.threadgenius.co/v0/search" \
  -X POST \
  -H "Authorization: PKmWwSGYF4szvyY7" \
  -F "image=@{/path/to/image.jpeg}" \
  -F "catalog_id=cdae7e788f1e4982"
```


Query for similar products with an image of a single item.

### HTTP Request

`POST http://api.threadgenius.co/v0/search/`

### Query Parameters

Provide either an image URL or the image file itself.

Parameter | Description
--------- | -----------
`url` | (Either) The URL of the image
`image` | (Either) The file of the image
`catalog_id` | (Required) The ID of the catalog which you are querying
`n_results` | (Optional) The number of results you want returned (default is 50)


# Item Detection

## Find clothing objects in an image

```shell
curl "http://api.threadgenius.co/v0/detect" \
  -X POST \
  -H 'Authorization: Bearer PKmWwSGYF4szvyY7' \
  -d 'url=https://s-media-cache-ak0.pinimg.com/564x/b5/6b/eb/b56beb741cba8e9667880178c3596139.jpg'
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "boxes": [
    {
      "category": "pants-long",
      "score": 0.6992457509040833,
      "bbox": [
        48,
        203,
        270,
        524
      ]
    },
    {
      "category": "footwear-regular",
      "score": 0.9771430492401123,
      "bbox": [
        143,
        455,
        216,
        536
      ]
    },
    {
      "category": "footwear-regular",
      "score": 0.7428352236747742,
      "bbox": [
        88,
        423,
        159,
        502
      ]
    },
    {
      "category": "outerwear-light",
      "score": 0.9962784647941589,
      "bbox": [
        66,
        55,
        243,
        293
      ]
    }
  ]
}
```

> To query using an image file,

```shell
curl "http://api.threadgenius.co/v0/detect" \
  -X POST \
  -H 'Authorization: Bearer PKmWwSGYF4szvyY7' \
  -F "image=@{/path/to/image.jpeg}"
```

This endpoint automatically finds bounding boxes around clothing objects within your image.

### HTTP Request

`POST http://api.threadgenius.co/v0/detect/`

### Query Parameters

Provide either an image URL or the image file itself.

Parameter | Description
--------- | -----------
`url` | (Either) The URL of the image
`image` | (Either) The file of the image


# Usage

## Retrieve your API usage summary

```shell
curl "http://api.threadgenius.co/v0/usage" \
  -X GET \
  -H 'Authorization: Bearer PKmWwSGYF4szvyY7' \
```

> Example Response:

```json
{
  "status": {
    "message": "OK",
    "code": 200
  },
  "usage": {
    "2017-1": {
      "catalog_index": {
        "num_calls": 2,
        "num_images": 1000
      },
      "search": {
        "num_calls": 11,
        "num_images": 11
      },
      "detect": {
        "num_calls": 1,
        "num_images": 1
      },
      "tag": {
        "num_calls": 7,
        "num_images": 7
      },
      "catalog_upload": {
        "num_calls": 8,
        "num_images": 4000
      }
    },
    "2016-12": {
      "catalog_index": {
        "num_calls": 10,
        "num_images": 518
      },
      "search": {
        "num_calls": 60,
        "num_images": 60
      },
      "tag": {
        "num_calls": 11,
        "num_images": 11
      },
      "detect": {
        "num_calls": 3,
        "num_images": 3
      },
      "catalog_upload": {
        "num_calls": 18,
        "num_images": 3023
      }
    }
  }
}
```

This endpoint returns your API usage summary

### HTTP Request

`POST http://api.threadgenius.co/v0/usage/`



