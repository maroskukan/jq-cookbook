# jq-cookbook
- [jq-cookbook](#jq-cookbook)
  - [Introduction](#introduction)
  - [Documentation](#documentation)


## Introduction

In a modern world of microservices where every application and service exposes a consumable API, there is a need for standard format for data represtation.

One of most popular format is undeniably JSON (JavaScript Object Notation). In its simple form, JSON object is composed of simple key/value pairs. For example:

```json
{
    "name": "Maros",
    "role": "Engineer" 
}
```

The `key` must be a text string wrapped around quotation marks. The `value` can be quoted text, boolean `true`, `false` or `null`. It can also be another JSON object `{}` or an array `[]`. 

For example, creating the following request toward Rijks Museum API.

Please note that most of API endpoints will involve some sort of authentication. In the case below I had to create an account and generate and API key which is required with every request.

```bash
curl https://www.rijksmuseum.nl/api/nl/collection?key=[api-key]&involvedMaker=Rembrandt+van+Rijn
```
Generates the following answer, which I have stripped for brevity.
```json
{
  "elapsedMilliseconds": 0,
  "count": 3540,
  "countFacets": {
    "hasimage": 2924,
    "ondisplay": 14
  },
  "artObjects": [
    {
      "links": {
        "self": "http://www.rijksmuseum.nl/api/nl/collection/SK-C-5",
        "web": "http://www.rijksmuseum.nl/nl/collectie/SK-C-5"
      },
      "id": "nl-SK-C-5",
      "objectNumber": "SK-C-5",
      "title": "De Nachtwacht",
      "hasImage": true,
      "principalOrFirstMaker": "Rembrandt van Rijn",
      "longTitle": "De Nachtwacht, Rembrandt van Rijn, 1642",
      "showImage": true,
      "permitDownload": true,
      "webImage": {
        "guid": "aa08df9c-0af9-4195-b31b-f578fbe0a4c9",
        "offsetPercentageX": 0,
        "offsetPercentageY": 1,
        "width": 2500,
        "height": 2034,
        "url": "https://lh3.googleusercontent.com/J-mxAE7CPu-DXIOx4QKBtb0GC4ud37da1QK7CzbTIDswmvZHXhLm4Tv2-1H3iBXJWAW_bHm7dMl3j5wv_XiWAg55VOM=s0"
      },
      "headerImage": {
        "guid": "29a2a516-f1d2-4713-9cbd-7a4458026057",
        "offsetPercentageX": 0,
        "offsetPercentageY": 0,
        "width": 1920,
        "height": 460,
        "url": "https://lh3.googleusercontent.com/O7ES8hCeygPDvHSob5Yl4bPIRGA58EoCM-ouQYN6CYBw5jlELVqk2tLkHF5C45JJj-5QBqF6cA6zUfS66PUhQamHAw=s0"
      },
      "productionPlaces": [
        "Amsterdam"
      ]
    },
```

Wouldn't it be great if there is a tool that is able to filter only the information you are interested in and transform this response for other formats for further processing? You are in luck, let me introduce to you jQuery.

## Documentation

- [jQuery Project](https://jquery.com/)
- [Json and Jq](https://programminghistorian.org/en/lessons/json-and-jq)
- [Rijks Museum Object medata APIs](https://data.rijksmuseum.nl/object-metadata/api/)

