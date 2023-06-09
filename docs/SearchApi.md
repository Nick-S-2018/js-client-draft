# Manticoresearch.SearchApi

All URIs are relative to *http://127.0.0.1:9308*

Method | HTTP request | Description
------------- | ------------- | -------------
[**percolate**](SearchApi.md#percolate) | **POST** /json/pq/{index}/search | Perform reverse search on a percolate index
[**search**](SearchApi.md#search) | **POST** /json/search | Performs a search



## percolate

> SearchResponse percolate(index, percolateRequest)

Perform reverse search on a percolate index

Performs a percolate search. 
This method must be used only on percolate indexes.

Expects two parameters: the index name and an object with array of documents to be tested.
An example of the documents object:

  ```
  {"query":{"percolate":{"document":{"title":"What a nice bag"}}}}
  ```

Responds with an object with matched stored queries: 

  ```
  {"took":0,"timed_out":false,"hits":{"total":1,"hits":[{"_index":"products","_type":"doc","_id":"2811045522851233808","_score":"1","_source":{"query":{"ql":"@title bag"}},"fields":{"_percolator_document_slot":[1]}}]}}
  ```


### Example

```javascript
var Manticoresearch = require('manticoresearch');

var searchApi = new Manticoresearch.SearchApi();
async function() {
    res = await searchApi.percolate('products',{"query":{"percolate":{"document":{"title":"What a nice bag"}}}});
}

```

### Parameters



Name | Type | Description  | Notes
------------- | ------------- | ------------- | -------------
 **index** | **String**| Name of the percolate index | 
 **percolateRequest** | [**PercolateRequest**](PercolateRequest.md)|  | 

### Return type

[**SearchResponse**](SearchResponse.md)

### Authorization

No authorization required

### HTTP request headers

- **Content-Type**: application/json
- **Accept**: application/json


## search

> SearchResponse search(searchRequest)

Performs a search


Expects an object with mandatory properties:
* the index name
* the match query object

Example :

  ```
  {"index":"myindex","query":{"query_string":"@title \"find me fast \"/2"}}
  ```

It responds with an object with:
- time of execution
- if the query timed out
- an array with hits (matched documents)
- additional, if profiling is enabled, an array with profiling information is attached


  ```
  {"hits":{"hits":[{"_id":"1","_score":1,"_source":{"title":"first find me fast","gid":11}},{"_id":"2","_score":1,"_source":{"title":"second find me fast","gid":12}}],"total":2},"profile":None,"timed_out":False,"took":0}
  ```

Alternatively, you can use auxiliary query objects to build your search queries as it's shown in the example below.
For more information about the match query syntax and additional parameters that can be added to  request and response, please check: https://manual.manticoresearch.com/Searching/Full_text_matching/Basic_usage#HTTP.



### Example
```javascript
var Manticoresearch = require('manticoresearch');
var client = new Manticoresearch.ApiClient();
client.basePath="http://localhost:9308";
var searchApi = new Manticoresearch.SearchApi(client);
```

### SearchRequest

[[Detailed information on search options]](https://manual.manticoresearch.com/Searching/Options#Search-options)
```javascript

var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};
	
searchRequest.limit = 10;
searchRequest.track_scores = true;
searchRequest.options = {'cutoff': 5};
searchRequest.options['ranker'] = 'bm25';
searchRequest.source = 'title';

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### SourceByRules

[[SourceByRules]](SourceByRules.md)

[[Detailed information on the `source` property]](https://manual.manticoresearch.com/Searching/Search_results#Source-selection)
```javascript
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

searchRequest.source = new Manticoresearch.SourceByRules();
searchRequest.source.includes = ['title', 'year'];
searchRequest.source.excludes = ['code'];

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### Sort
```javascript
//Setting the `sort` property:
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

searchRequest.sort = ['year']

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### SortOrder
### SortMVA

[[SortOrder]](SortOrder.md)
[[SortMVA]](SortMVA.md)

[[Detailed information on sorting]](https://manual.manticoresearch.com/Searching/Sorting_and_ranking#HTTP)
```javascript
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

var sort2 = new Manticoresearch.SortOrder('rating', 'asc');
var sort3 = new Manticoresearch.SortMVA('code', 'desc', 'max');
searchRequest.sort.push(...[sort2,sort3]);

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### Expressions

[[Detailed information on expressions]](https://manual.manticoresearch.com/Searching/Expressions#Expressions-in-HTTP-JSON)
```javascript    
//Setting the `expressions` property:
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

var expr = {'expr': 'min(year,2900)'};
searchRequest.expressions = [expr];
searchRequest.expressions.push({'expr2': 'max(year,2100)'});
searchRequest.source.includes.push('expr2');

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### Aggregation

[[Aggregation]](Aggregation.md)

[[Detailed information on aggregations](https://manual.manticoresearch.com/Searching/Faceted_search#Aggregations)
```javascript    
//Setting the `aggs` property with an auxiliary object:
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

var agg1 = new Manticoresearch.Aggregation('agg1', 'year');
Manticoresearch.Aggregation.constructFromObject({size:10}, agg1);
searchRequest.aggs = [agg1];
searchRequest.aggs.push(new Manticoresearch.Aggregation('agg2', 'rating'));

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### Highlight

[[Highlight]](Highlight.md)

[[Detailed information on highlighting]](https://manual.manticoresearch.com/Searching/Highlighting#Highlighting)
```javascript
//Settting the `highlight` property with an auxiliary object:
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

var highlight = new Manticoresearch.Highlight();
highlight.fieldnames = ['title'];
highlight.post_tags = '</post_tag>';
highlight.encoder = 'default';
highlight.snippet_boundary = 'sentence';
searchRequest.highlight = highlight;

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

#### HighlightField

[[HighlightField]](HighlightField.md)

[[Detailed information on highlighting]](https://manual.manticoresearch.com/Searching/Highlighting#Highlighting)
```javascript
// settting `highlight.fields` property with an auxiliary HighlightField object
var searchRequest = new Manticoresearch.SearchRequest();
searchRequest.index = "movies";
searchRequest.query = {"match_all": {}};

var highlight = new Manticoresearch.Highlight();
var highlightField = new Manticoresearch.HighlightField('title');
var highlightField2 = new Manticoresearch.HighlightField('plot');
Manticoresearch.HighlightField.constructFromObject({limit:5}, highlightField2);
Manticoresearch.HighlightField.constructFromObject({limit_words:10}, highlightField2);
highlight.fields = [highlightField, highlightField2];
searchRequest.highlight = highlight;

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

### FulltextFilter
#### QueryFilter

[[Detailed information on fulltext filters]](https://manual.manticoresearch.com/Searching/Full_text_matching/Basic_usage#HTTP)

[[QueryFilter]](QueryFilter.md)
```javascript    
//Setting the `fulltext_filter` property using different fulltext filter objects:

//Using a QueryFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

searchRequest.fulltext_filter = new Manticoresearch.QueryFilter('Star Trek 2');

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

#### MatchFilter

[[MatchFilter]](MatchFilter.md)
```javascript    
//Using a MatchFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

searchRequest.fulltext_filter = new Manticoresearch.MatchFilter('Nemesis', 'title');

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

#### MatchPhraseFilter

[[MatchPhraseFilter]](MatchPhraseFilter.md)
```javascript    
//Using a MatchPhraseFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

searchRequest.fulltext_filter = new Manticoresearch.MatchPhraseFilter('Star Trek 2', 'title');

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

#### MatchOpFilter

[[MatchOpFilter]](MatchOpFilter.md)
```javascript
//Using a MatchOpFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

searchRequest.fulltext_filter = new Manticoresearch.MatchOpFilter('Enterprise test', 'title,plot', 'or');

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```    

### AttrFilter
#### EqualsFilter

[[EqualsFilter]](EqualsFilter.md)

[[Detailed information on equality filters]](https://manual.manticoresearch.com/Searching/Filters#Equality-filters)
```javascript
//Setting the `attr_filter` property using different attribute filter objects:

//Using EqualsFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

searchRequest.attr_filter = new Manticoresearch.EqualsFilter('year', 2003);

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

#### InFilter

[[InFilter]](InFilter.md)

[[Detailed information on set filters]](https://manual.manticoresearch.com/Searching/Filters#Set-filters)
```javascript
//Using InFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

var inFilter = new Manticoresearch.InFilter('year', [2001, 2002]);
inFilter.values = inFilter.values.concat([10,11]);
searchRequest.attr_filter = inFilter;

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```
			
#### RangeFilter

[[RangeFilter]](RangeFilter.md)

[[Detailed information on range filters]](https://manual.manticoresearch.com/Searching/Filters#Range-filters)
```javascript
//Using RangeFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

var rangeFilter = new Manticoresearch.RangeFilter('year');
Manticoresearch.RangeFilter.constructFromObject({lte: 2002}, rangeFilter);
Manticoresearch.RangeFilter.constructFromObject({gte: 1000}, rangeFilter);
searchRequest.attr_filter = rangeFilter;

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}	
```

#### GeoDistanceFilter

[[GeoDistanceFilter]](GeoDistanceFilter.md)

[[Detailed information on geo distance filters]](https://manual.manticoresearch.com/Searching/Filters#Geo-distance-filters)
```javascript
//Using GeoDistanceFilter object
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "geo";

var geoFilter = new Manticoresearch.GeoDistanceFilter();
var locAnchor = {'location_anchor': {'lat':10,'lon':20}};
Manticoresearch.GeoDistanceFilter.constructFromObject(locAnchor, geoFilter);
Manticoresearch.GeoDistanceFilter.constructFromObject({'location_source':'field3,field4'}, geoFilter);
Manticoresearch.GeoDistanceFilter.constructFromObject({'distance_type': 'adaptive', 'distance': '100km'}, geoFilter);
searchRequest.attr_filter = geoFilter;

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```

#### BoolFilter

[[BoolFilter]](BoolFilter.md)

[[Detailed information on Bool queries]](https://manual.manticoresearch.com/Searching/Filters#bool-query)
```javascript
//Setting the `attr_filter` property using a bool filter object:
var searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";

var boolFilter = new Manticoresearch.BoolFilter();
boolFilter.must = [ new Manticoresearch.EqualsFilter('year', 2001) ];
rangeFilter = new Manticoresearch.RangeFilter('rating');
Manticoresearch.RangeFilter.constructFromObject({lte: 20}, rangeFilter);
boolFilter.must.push(rangeFilter);
searchRequest.attr_filter = boolFilter;

async function(){
	var res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}

boolFilter.must_not = [ new Manticoresearch.EqualsFilter('year', 2001) ];
searchRequest.attr_filter = boolFilter;

async function(){
	res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}

//Using nested bool filters
searchRequest = manticoresearch.model.SearchRequest();
searchRequest.index = "movies";
	
var fulltextFilter = new Manticoresearch.MatchFilter('Star', 'title');
var nestedBoolFilter = new Manticoresearch.BoolFilter();
nestedBoolFilter.should = [ new Manticoresearch.EqualsFilter('rating', 6.5), fulltextFilter];
boolFilter.must = [nestedBoolFilter];
searchRequest.attr_filter = boolFilter;

async function(){
	res = await searchApi.search(searchRequest);
	console.log(JSON.stringify(res, null, 4));  
}
```
			
### Example of how to build search requests using the alternative way with a single literal object 
```javascript

var searchRequest = {"index":"myindex","query":{"query_string":"@title \"find me fast \"/2"}};
async function(){
    var res = await searchApi.search(searchRequest);
    console.log(JSON.stringify(res, null, 4));
}
```

### Parameters

Name | Type | Description  | Notes
------------- | ------------- | ------------- | -------------
 **searchRequest** | [**SearchRequest**](SearchRequest.md)|  | 

### Return type

[**SearchResponse**](SearchResponse.md)

### Authorization

No authorization required

### HTTP request headers

- **Content-Type**: application/json
- **Accept**: application/json

