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
    - [New JSON [] and {}](#new-json--and-)
    - [CSV Output @csv](#csv-output-csv)
  - [Advanced operations](#advanced-operations)
    - [JSON and JSON Lines](#json-and-json-lines)
    - [Relationships: One-to-Many](#relationships-one-to-many)
      - [One row per tweet](#one-row-per-tweet)
      - [One row per hashtag](#one-row-per-hashtag)


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
- [Conditionals and Comparisons](https://stedolan.github.io/jq/manual/#ConditionalsandComparisons)
- [Twarc](https://github.com/DocNow/twarc)
- [Json and Jq](https://programminghistorian.org/en/lessons/json-and-jq)
- [Rijks Museum Object medata APIs](https://data.rijksmuseum.nl/object-metadata/api/)
- [NYPL Collections](https://github.com/NYPL-publicdomain/data-and-utilities/tree/master/items)

## Environment

When it comes to learning environemnt, you have a choice of using our own local machine with required binaries or you can use remote web-based version of jQuery. For "production" use I recommend local environemnt as it can handle higher load and provides some level of confidentiality.

### Local environment

You can find jQuery included in most of the popular package managers. For Windows you can leverage [Chocolatey](https://chocolatey.org/)

```powershell
choco install jq
jq --version
jq-1.6
```

For Ubuntu, you can use apt.
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
```json
jq .count jq_rmk.json
359
```

If you want to access a nested object, you need to chain the filter. For example you would use the following structure to access the `self` key inside the first item in an rray that is inside `artObjects` object. The array `[]` operator will be explained in next section.
```json
jq '.artObjects[0].links.self' jq_rmk.json
"https://www.rijksmuseum.nl/api/nl/collection/SK-C-5"
```

### Array []

In the previous example, we encouterd an array object `.artObjects`. We used the element id to access the first object in this array `.artObjects[]`. If you want to access the second element, just increase the reference number.
```json
jq '.artObjects[1]'  jq_rmk.json
{
  "links": {
    "self": "https://www.rijksmuseum.nl/api/nl/collection/SK-A-1505",
    "web": "https://www.rijksmuseum.nl/nl/collectie/SK-A-1505"
  },
  "id": "nl-SK-A-1505",
  "objectNumber": "SK-A-1505"
/*Output omitted*/
```

### Pipe |

The pipe, you can combine several operators together. For example if you want to retrieve value of `id` key in each object inside `.artObject` array you would use the followin syntax.
```json
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
```json
jq '.artObjects[] | select(.productionPlaces | length >= 1) | .id' jq_rmk.json
"nl-SK-C-5"
"nl-SK-A-3924"
```

The select filter also support regular expressions, for example to display only those object which have `van` in their `.principalOrFirstMaker` key and then display `.id`, `title` and `principalOrFirstMaker` you can use the following expression:
```json
jq '.artObjects[] | select(.principalOrFirstMaker | test("van")) | {id: .id, title: .title, artist: .principalOrFirstMaker}' jq_rmk.json
{
  "id": "nl-SK-C-5",
  "title": "Schutters van wijk II onder leiding van kapitein Frans Banninck Cocq, bekend als de ???Nachtwacht???",
  "artist": "Rembrandt Harmensz. van Rijn"
}
{
  "id": "nl-SK-A-180",
  "title": "Een vrolijke vioolspeler",
  "artist": "Gerard van Honthorst"
}
{
  "id": "nl-SK-A-2205",
  "title": "Vanitas stilleven",
  "artist": "Gerrit van Vucht"
}
{
  "id": "nl-SK-A-1935",
  "title": "Landschap met stenen brug",
  "artist": "Rembrandt Harmensz. van Rijn"
}
{
  "id": "nl-SK-A-3246",
  "title": "De visvrouw",
  "artist": "Adriaen van Ostade"
}
```

### New JSON [] and {}

You may notice in above example, that by using {} in last filter, we created a new JSON structure. In next example, we iterate over all items in array and display `.id` and `.title`.
```json
jq '.artObjects[] | {id: .id, title: .title}' jq_rmk.json
{
  "id": "nl-SK-C-5",
  "title": "Schutters van wijk II onder leiding van kapitein Frans Banninck Cocq, bekend als de ???Nachtwacht???"
}
{
  "id": "nl-SK-A-1505",
  "title": "Een molen aan een poldervaart, bekend als ???In de maand juli???"
}
{
  "id": "nl-SK-A-180",
  "title": "Een vrolijke vioolspeler"
}
/*Output omitted*/
```

If you would like to output a list of values instread of arrays and keys, you can use `[]`.
```json
jq '.artObjects[0] | [ .id, .title ]' jq_rmk.json
[
  "nl-SK-C-5",
  "Schutters van wijk II onder leiding van kapitein Frans Banninck Cocq, bekend als de ???Nachtwacht???"
]
```

### CSV Output @csv

Sometimes you want to filter and transform the data to different format for further processing. One of the pupular choices is CSV. Lets start with some filter we know. The following query will return `.id`, `.title`, `.principalOrFirstMaker`, and `.webImage.url` in list format for each object inside `.artObjects`.
```json
jq '.artObjects[] | [.id, .title, .principalOrFirstMaker, .webImage.url]' jq_rmk.json
[
  "nl-SK-C-5",
  "Schutters van wijk II onder leiding van kapitein Frans Banninck Cocq, bekend als de ???Nachtwacht???",
  "Rembrandt Harmensz. van Rijn",
  "http://lh6.ggpht.com/ZYWwML8mVFonXzbmg2rQBulNuCSr3rAaf5ppNcUc2Id8qXqudDL1NSYxaqjEXyDLSbeNFzOHRu0H7rbIws0Js4d7s_M=s0"
]
[
  "nl-SK-A-1505",
  "Een molen aan een poldervaart, bekend als ???In de maand juli???",
  "Paul Joseph Constantin Gabri??l",
  "http://lh4.ggpht.com/PkQr-nNqzn0OVXVd4-hdJ6PPdWZ6-DQ_74WfBT3MZIV4LNYA-q8LUrtReXNstuzl9k6gKWkaBwG-LcFZ7zWU9Ch92g=s0"
]
/*Output omitted*/
```

To format the above output into CSV format, simple add `@csv` at the end of the chain.
```json
"\"nl-SK-C-5\",\"Schutters van wijk II onder leiding van kapitein Frans Banninck Cocq, bekend als de ???Nachtwacht???\",\"Rembrandt Harmensz. van Rijn\",\"http://lh6.ggpht.com/ZYWwML8mVFonXzbmg2rQBulNuCSr3rAaf5ppNcUc2Id8qXqudDL1NSYxaqjEXyDLSbeNFzOHRu0H7rbIws0Js4d7s_M=s0\""
"\"nl-SK-A-1505\",\"Een molen aan een poldervaart, bekend als ???In de maand juli???\",\"Paul Joseph Constantin Gabri??l\",\"http://lh4.ggpht.com/PkQr-nNqzn0OVXVd4-hdJ6PPdWZ6-DQ_74WfBT3MZIV4LNYA-q8LUrtReXNstuzl9k6gKWkaBwG-LcFZ7zWU9Ch92g=s0\""
"\"nl-SK-A-180\",\"Een vrolijke vioolspeler\",\"Gerard van Honthorst\",\"http://lh4.ggpht.com/iIwEcQp7nB5xmV6WctDIG-HRFWwMuegCw7j2On9lTksv9Mwl-nllpaQx_0BORbJiqyks4dR_E4K7AGnNur8hkoJ7WNkz=s0\""
"\"nl-SK-A-2205\",\"Vanitas stilleven\",\"Gerrit van Vucht\",\"http://lh3.ggpht.com/ltkNYBL4U__3FerjWvcmZlOWhZ5kCh5Bxtl0FlkCiTDjYGzO1G4AkyA_5OxLjXiGJJr1qDvvw5_uPGP7kiZYuLef9cA=s0\""
/*Output omitted*/
```


## Advanced operations

### JSON and JSON Lines

So far we have been working with JSON objects, which are known for having single JSON object that contains many smaller sub objects. 

Now we are going to explore [Twitter API](https://dev.twitter.com/overview/api), which in contrast returns JSON Lines which have multiple separate JSON objects each on one single line, not wrapped by `[]`.

For demostration, we will be using sample file `jq_twitter.json` provided by programminghistorian.org.

### Relationships: One-to-Many

In this example, we will create a table that maps a tweet to its hashtags. There are two general solutions available:
- One row per tweet with multiple hashtags in the same cell
- One row per hashtag/tweet combination, with tweet IDs and hashtags repeated as necessary

#### One row per tweet

Start by creating a filter that will reduce the Twitter JSON to display just `.id` and `hashtags`.
```json
jq '{id: .id, hashtags: .entities.hashtags}' jq_twitter.json
{
  "id": 501064141332029440,
  "hashtags": [
    {
      "indices": [
        41,
        50
      ],
      "text": "Ferguson"
    }
  ]
}
/*Output omitted*/
```

Chain another query to show just `text` value inside `hashtags` object.
```json
jq '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: .hashtags[].text}' jq_twitter.json
{
  "id": 501064141332029440,
  "hashtags": "Ferguson"
}
{
  "id": 501064171707170800,
  "hashtags": "Ferguson"
}
{
  "id": 501064180468682750,
  "hashtags": "Ferguson"
}
{
  "id": 501064194309906400,
  "hashtags": "USNews"
}
{
  "id": 501064196931330050,
  "hashtags": "Ferguson"
}
{
  "id": 501064196931330050,
  "hashtags": "MikeBrown"
}
/*Output omitted*/
```

THe hastag id `501064196931330050` id displayed twice as it has two different hashtags. We need to combine it and show hastags as array.
```json
jq '{id: .id, hashtags: .entities.hashtags} | {id: .id, hashtags: [.hashtags[].text]}' jq_twitter.json \
   | grep -B1 -A5 501064196931330050
{
  "id": 501064196931330050,
  "hashtags": [
    "Ferguson",
    "MikeBrown"
  ]
}
```

Before we can express this result in CSV format, we need to add delimiter to hashtags array.
```json
jq '{id: .id, hashtags: .entities.hashtags}
  | {id: .id, hashtags: [.hashtags[].text]}
  | {id: .id, hashtags: .hashtags | join(";")}' \
  jq_twitter.json
/* Output omitted*/
{
  "id": 501064201251880960,
  "hashtags": "Ferguson"
}
{
  "id": 501064201256071200,
  "hashtags": "Ferguson"
}
{
  "id": 501064201503113200,
  "hashtags": "MikeBrown;Ferguson"
}
/* Output omitted*/
```

And now convert to CSV.
```json
jq '{id: .id, hashtags: .entities.hashtags}
  | {id: .id, hashtags: [.hashtags[].text]}
  | {id: .id, hashtags: .hashtags | join(";")}
  | [.id, .hashtags]
  | @csv' \
  jq_twitter.json
/* Output omitted*/
"501064201251880960,\"Ferguson\""
"501064201256071200,\"Ferguson\""
"501064201503113200,\"MikeBrown;Ferguson\""
/* Output omitted*/
```

To remove the forward slashes, use `-r` option, which denotes raw output.
```json
jq -r '{id: .id, hashtags: .entities.hashtags}
  | {id: .id, hashtags: [.hashtags[].text]}
  | {id: .id, hashtags: .hashtags | join(";")}
  | [.id, .hashtags]
  | @csv' \
  jq_twitter.json
/* Output omitted*/
501064201251880960,"Ferguson"
501064201256071200,"Ferguson"
501064201503113200,"MikeBrown;Ferguson"
/* Output omitted*/
```

#### One row per hashtag

Lets tart with the previous filter which gave us tweet id and hashtags.
```json
jq '{id: .id, hashtags: .entities.hashtags} 
  | {id: .id, hashtags: .hashtags[].text}' \
  jq_twitter.json
```

The result is one id and one hashtag per object. Before you can convert it to CSV you need to create an array using another query `[.id, .hashtags]`. Finally, don't remember to add `-r` option for raw output.
```json
jq -r '{id: .id, hashtags: .entities.hashtags} 
  | {id: .id, hashtags: .hashtags[].text}
  | [.id, .hashtags]
  | @csv' \
  jq_twitter.json
/* Output omitted*/
501064196931330050,"MikeBrown"
501064197632167940,"Ferguson"
501064197632167940,"tcot"
501064197632167940,"uniteblue"
/* Output omitted*/
```

