Elasticsearch URL Tokenizer and URL Token Filter
==============================

This plugin enables URL tokenization and token filtering by URL part.

[![Build Status](https://secure.travis-ci.org/jlinn/elasticsearch-analysis-url.png?branch=master)](http://travis-ci.org/jlinn/elasticsearch-analysis-url)

## Compatibility

| Elasticsearch Version | Plugin Version |
|-----------------------|----------------|
| 5.6.5 | 5.6.5.0 |
| 5.6.3 | 5.6.3.0 |
| 5.6.1 | 5.6.1.0 |
| 5.5.1 | 5.5.1.0 |
| 5.5.0 | 5.5.0.0 |
| 5.2.2 | 5.2.2.0 |
| 5.2.1 | 5.2.1.1 |
| 5.1.1 | 5.1.1.0 |
| 5.0.0 | 5.0.0.1 |
| 2.4.3 | 2.4.3.0 |
| 2.4.1 | 2.4.1.0 |
| 2.4.0 | 2.4.0.0 |
| 2.3.5 | 2.3.5.0 |
| 2.3.4 | 2.3.4.3 |
| 2.3.3 | 2.3.3.5 |
| 2.3.2 | 2.3.2.1 |
| 2.3.1 | 2.3.1.1 |
| 2.3.0 | 2.3.0.1 |
| 2.2.2 | 2.2.3 |
| 2.2.1 | 2.2.2.1 |
| 2.2.0 | 2.2.1 |
| 2.1.1 | 2.2.0 |
| 2.1.1 | 2.1.1 |
| 2.0.0 | 2.1.0 |
| 1.6.x, 1.7.x | 2.0.0 |
| 1.6.0 | 1.2.1 |
| 1.5.2 | 1.1.0 |
| 1.4.2 | 1.0.0 |

## Installation
### Elasticsearch v5
```bash
bin/elasticsearch-plugin install https://github.com/vkhomytskyi/elasticsearch-analysis-url/releases/download/v5.6.5.0/elasticsearch-analysis-url-5.6.5.0.zip
```

### Elasticsearch v2
```bash
bin/plugin install https://github.com/jlinn/elasticsearch-analysis-url/releases/download/v2.4.3.0/elasticsearch-analysis-url-2.4.3.0.zip
```

## Usage
### URL Tokenizer
#### Options:
* `part`: Defaults to `null`. If left `null`, all URL parts will be tokenized, and some additional tokens (`host:port` and `protocol://host`) will be included. Can be either a string (single URL part) or an array of multiple URL parts. Options are `whole`, `protocol`, `host`, `port`, `path`, `query`, and `ref`.
* `url_decode`: Defaults to `false`. If `true`, URL tokens will be URL decoded.
* `allow_malformed`: Defaults to `false`. If `true`, malformed URLs will not be rejected, but will be passed through without being tokenized.
* `tokenize_malformed`: Defaults to `false`. Has no effect if `allow_malformed` is `false`. If both are `true`, an attempt will be made to tokenize malformed URLs using regular expressions.
* `tokenize_host`: Defaults to `true`. If `true`, the host will be further tokenized using a [reverse path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `.`.
* `tokenize_path`: Defaults to `true`. If `true`, the path will be tokenized using a [path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `/`.
* `tokenize_query`: Defaults to `true`. If `true`, the query string will be split on `&`.

#### Example:
Index settings:
```json
{
	"settings": {
		"analysis": {
			"tokenizer": {
				"url_host": {
					"type": "url",
					"part": "host"
				}
			},
			"analyzer": {
				"url_host": {
					"tokenizer": "url_host"
				}
			}
		}
	}
}
```

Make an analysis request:
```bash
curl 'http://localhost:9200/index_name/_analyze?analyzer=url_host&pretty' -d 'https://foo.bar.com/baz.html'

{
  "tokens" : [ {
    "token" : "foo.bar.com",
    "start_offset" : 8,
    "end_offset" : 19,
    "type" : "host",
    "position" : 1
  }, {
    "token" : "bar.com",
    "start_offset" : 12,
    "end_offset" : 19,
    "type" : "host",
    "position" : 2
  }, {
    "token" : "com",
    "start_offset" : 16,
    "end_offset" : 19,
    "type" : "host",
    "position" : 3
  } ]
}
```

### URL Token Filter
#### Options:
* `part`: This option defaults to `whole`, which will cause the entire URL to be returned. In this case, the filter only serves to validate incoming URLs. Other possible values are:
`protocol`, `host`, `port`, `path`, `query`, and `ref`. Can be either a single URL part (string) or an array of URL parts.
* `url_decode`: Defaults to `false`. If `true`, the desired portion of the URL will be URL decoded.
* `allow_malformed`: Defaults to `false`. If `true`, documents containing malformed URLs will not be rejected, and an attempt will be made to parse the desired URL part from the malformed URL string.
If the desired part cannot be found, no value will be indexed for that field.
* `passthrough`: Defaults to `false`. If `true`, `allow_malformed` is implied, and any non-url tokens will be passed through the filter.  Valid URLs will be tokenized according to the filter's other settings.
* `tokenize_host`: Defaults to `true`. If `true`, the host will be further tokenized using a [reverse path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `.`.
* `tokenize_path`: Defaults to `true`. If `true`, the path will be tokenized using a [path hierarchy tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) with the delimiter set to `/`.
* `tokenize_query`: Defaults to `true`. If `true`, the query string will be split on `&`.

#### Example:
Set up your index like so:
```json
{
    "settings": {
        "analysis": {
            "filter": {
                "url_host": {
                    "type": "url",
                    "part": "host",
                    "url_decode": true,
                    "tokenize_host": false
                }
            },
            "analyzer": {
                "url_host": {
                    "filter": ["url_host"],
                    "tokenizer": "whitespace"
                }
            }
        }
    },
    "mappings": {
        "example_type": {
            "properties": {
                "url": {
                    "type": "multi_field",
                    "fields": {
                        "url": {"type": "string"},
                        "host": {"type": "string", "analyzer": "url_host"}
                    }
                }
            }
        }
    }
}
```

Make an analysis request:
```bash
curl 'http://localhost:9200/index_name/_analyze?analyzer=url_host&pretty' -d 'https://foo.bar.com/baz.html'

{
  "tokens" : [ {
    "token" : "foo.bar.com",
    "start_offset" : 0,
    "end_offset" : 32,
    "type" : "word",
    "position" : 1
  } ]
}
```
