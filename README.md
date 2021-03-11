# jq-cookbook
- [jq-cookbook](#jq-cookbook)
  - [Introduction](#introduction)
  - [Documentation](#documentation)
  - [Environment](#environment)
    - [Local environment](#local-environment)
    - [Remote environment](#remote-environment)
  - [Filters](#filters)
    - [dot .](#dot-)
    - [Array []](#array-)
    - [Pipe |](#pipe-)
    - [Filter select()](#filter-select)


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

Generates the following answer, which I have stripped for brevity and formated it for easier interpretation.
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

I have found the following websites very useful when learning to use jQeury.

- [jQuery Project](https://jquery.com/)
- [Json and Jq](https://programminghistorian.org/en/lessons/json-and-jq)
- [Rijks Museum Object medata APIs](https://data.rijksmuseum.nl/object-metadata/api/)

## Environment

When it comes to learning environemnt, you have a choice of using our own local machine with required binaries or you can use remote web-based version of jQuery. For "production" use I recommend local environemnt as it can handle higher load and provides some level of confidentiality.

### Local environment

You can find jQuery included in most of the popular package managers. For Windows I am using [Chocolatey](https://chocolatey.org/)

```powershell
choco install jq
jq --version
jq-1.6
```

For Ubuntu, you can use apt
```bash
apt-get install jq
jq --version
jq-1.6
```

### Remote environment

The remote environment is available at [jQPlay](https://jqplay.org/) website. It provides a cheatsheet which may be handy while getting familiar with the tool.

## Filters

Filtering is one of the core capability of jQeury. Filters dictate how you want to transform the input data. We will be using sample JSON file `jq_rkm.json` available at `dataset` folder.

### dot .

The dot `.` without any paramters leaves output unmodified. However if you add name of the key to it, it will filter return value of that key.
```bash
jq .count jq_rmk.json
359
```

If you want to access a nested object, you need to chain the filter. For example you would use the following structure to access the `self` key inside the first item in an rray that is inside `artObjects` object. The array `[]` operator will be explained in next section.
```bash
jq '.artObjects[0].links.self' jq_rmk.json
"https://www.rijksmuseum.nl/api/nl/collection/SK-C-5"
```

### Array []

In the previous example, we encouterd an array object `.artObjects`. We used the element id to access the first object in this array `.artObjects[]`. If you want to access the second element, just increase the reference number.
```bash
jq '.artObjects[1]'  jq_rmk.json
{
  "links": {
    "self": "https://www.rijksmuseum.nl/api/nl/collection/SK-A-1505",
    "web": "https://www.rijksmuseum.nl/nl/collectie/SK-A-1505"
  },
  "id": "nl-SK-A-1505",
  "objectNumber": "SK-A-1505"
...
# Omited for brevity
...
```

### Pipe |

The pipe, you can combine several operators together. For example if you want to retrieve value of `id` key in each object inside `.artObject` array you would use the followin syntax.
```bash
jq '.artObjects[] | .id jq_rmk.json
"nl-SK-C-5"
"nl-SK-A-1505"
"nl-SK-A-180"
"nl-SK-A-2205"
"nl-SK-A-1923"
"nl-SK-A-1935"
"nl-SK-A-690"
"nl-SK-A-2983"
"nl-SK-A-3924"
"nl-SK-A-3246"
```

### Filter select()

This filter is very useful when you need to filter output based on some condition. For example, if you need to only select object `.id` from ones that have atleast one value in `productionPlaces` array.
```bash
jq '.artObjects[] | select(.productionPlaces | length >= 1) | .id' jq_rmk.json
"nl-SK-C-5"
"nl-SK-A-3924"
```

