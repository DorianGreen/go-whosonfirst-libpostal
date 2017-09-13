# The Mapzen Libpostal API

The Mapzen Libpostal API (application programming interface) is a REST-based interface to [the Libpostal C library](https://github.com/openvenues/libpostal) for parsing and normalizing street addresses in to data structures using [OpenStreetMap](http://www.openstreetmap.org/) data.

[Al Barrentine](https://twitter.com/albarrentine), who is Libpostal's author, wrote an [exhaustive blog post](https://mapzen.com/blog/inside-libpostal/) describing what Libpostal is and and how it works. It is aimed at a technical audience so we'll just excerpt the short version here:

> Libpostal uses machine learning and is informed by tens of millions of real-world addresses from OpenStreetMap. The entire pipeline for training the models is open source. Since OSM is a dynamic data set with thousands of contributors and the models are retrained periodically, improving them can be as easy as contributing addresses to OSM.

> Each country’s addressing system has its own set of conventions and peculiarities and libpostal is designed to deal with practically all of them. It currently supports normalizations in 60 languages and can parse addresses in more than 100 countries. Geocoding using libpostal as a preprocessing step becomes drastically simpler and more consistent internationally.

The Libpostal API has two endpoints that are exposed as HTTP `GET` requests, returning JSON-formatted results.

## GET /expand _?address=ADDRESS_

The `/expand` endpoint analyzes an address string and returns a list of normalized equivalent strings.

### Arguments

| Parameter | Sample Value | Required |
| :--- | :--- | :--- |
| `address` | "475 Sansome St San Francisco CA" | yes |

### Example

```
curl -s -X GET 'https://libpostal.mapzen.com/expand?address=475+Sansome+St+San+Francisco+CA' | python -mjson.tool
[
    "475 sansome saint san francisco california",
    "475 sansome saint san francisco ca",
    "475 sansome street san francisco california",
    "475 sansome street san francisco ca"
]
```

## GET /parse _?address=ADDRESS_

The `/parse` endpoint analyzes an address string and returns its component parts (street number, street name, city and so on).

### Arguments

| Parameter | Sample Value | Required |
| :--- | :--- | :--- |
| `address` | "475 Sansome St San Francisco CA" | yes |
| `format` | keys | no |

### Example

```
curl -s -X GET 'https://libpostal.mapzen.com/parse?address=475+Sansome+St+San+Francisco+CA' | python -mjson.tool
[
    {
        "label": "house_number",
        "value": "475"
    },
    {
        "label": "road",
        "value": "sansome st"
    },
    {
        "label": "city",
        "value": "san francisco"
    },
    {
        "label": "state",
        "value": "ca"
    }
]
```

By default both [Libpostal](https://github.com/openvenues/libpostal) and the Libpostal API return results a list of dictionaries, each containing a `label` and `value` key. This is because there are occasions when a given key may have multiple values, for example an address that contains a cross-street.

If you would prefer to have API results returned as a simple dictionary with labels as keys and values as lists of possible strings you should append the `format=keys` parameter.

### Example

```
curl -s -X GET 'https://libpostal.mapzen.com/parse?address=475+Sansome+St+San+Francisco+CA&format=keys' | python -mjson.tool
{
    "city": [
        "san francisco"
    ],
    "house_number": [
        "475"
    ],
    "road": [
        "sansome st"
    ],
    "state": [
        "ca"
    ]
}
```

The complete list of labels [defined in Libpostal](https://github.com/openvenues/libpostal/blob/master/src/address_parser.h) are:

* house
* house_number
* road
* suburb
* city_district
* city
* state_district
* state
* postalcode
* country

Depending on the address string you are parsing only some of those labels may be returned in the result set.

## Usage limits

The Libpostal service requires a Mapzen API key. In a request, you must append your own API key to the URL, following `api_key=`. See the [Mapzen developer overview](https://mapzen.com/documentation/overview/) for more on API keys and rate limits.
