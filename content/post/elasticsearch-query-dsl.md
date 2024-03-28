+++
title = "理解 Elasticsearch Query DSL 中的 JSON 结构"
date = "2019-05-01T10:17:46+08:00"
categories = [ "编程",]
tags = [ "code", "elasticsearch",]
toc = "true"
+++


理解 ES 搜索中 JSON DSL 有助于自己写 JSON 查询，特别是手写复杂嵌套 json。

* [x] diffs in es 2.x and es 5.x
* [x] query dsl
* [x] aggr query

<!--more-->

# diffs in es 2.x and es 5.x

## 没有 string 类型，改为 text 和 keyword 2 个类型。text 字段可以指定 fields 来不分词。如下：city 字段被 ingest 为 city 和 city.raw 2 个字段。
   
```json5
{
    "mappings": {
        "_doc": {
            "properties": {
                "city": {
                    "type": "text",
                    "fields": {
                        "raw": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                }
            }
        }
    }
}
  
```
## default double -> float
## geo_point
```json5
//2.x
"location": {
    "type": "geo_point",
    "lat_lon": true,
    "geohash": true,
    "geohash_prefix": true,
    "geohash_precision": "1m"
}
//5.x
"location": {
    "type": "geo_point"
}
```

# query dsl
## basic query
就像砌房子的砖头，基本查询就是 ES 查询的砖头。基本查询是组合查询 (bool 查询等) 的单元。基本查询有：

```text
//basic query element
match,
multi_match,
common,
geoshape,
ids,
match_all,
query_string,
simple_query_string,
range,
prefix,
regexp,
span_term, 
term,
terms,
wildcard
```
其中`common, ids, prefix, span_term, term, terms, wildcard` 是不分析搜索 (即不能用于 text 字段) ，`match,	multi_match,	query_string,	simple_query_string`是全文检索，几乎可以确保可以返回结果。而`prefix,regexp,wildcard`是模式检索。这里分别给一些例子：

### multi_match
> multi_match 查询为能在多个字段上反复执行相同查询提供了一种便捷方式
既然是多字段查询，则有 3 中场景:best_fields、most_fields 和 cross_fields（最佳字段、多数字段、跨字段）
```json5
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields",  //默认的，可不填
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
}
```
等价于下面的形式：
```json
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}

```
还可以使用通配符指定字段，以及给某些字段添加权重。
```json
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] 
    }
}

```


### query_string 和  simple_query_string
非常灵活的一个查询方式：
```json5
// GET index_name/_search
{
    "query": {
        "query_string" : {
            "default_field" : "content",
            "query" : "(new york city) OR (big apple)" 
        }
    }
}

```
上面的 query 字段语法可以参考：[query_string_syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax)

`simple_query_string`不会抛出异常，而是直接忽略无效语句。

### term、terms 完全匹配
```json5
// 不要用于 text 字段
// GET /_search
{
    "query": {
        "term": {
            "user": {
                "value": "Kimchy",
                "boost": 1.0
            }
        }
    }
}
//terms 和 term 一样，不过可以指定多个值，"user" : ["kimchy", "elasticsearch"]// 返回 user 为 kimchy 或 elasticsearch 的文档
```

### prefix
```json5
// user 字段 (不分词字段) 中以"ki"开头的文档 类似 SQL 中的 like 'ki%'
{ "query": {
    "prefix" : { "user" : "ki" }
  }
}
```

## 组合查询

```text
bool, boosting, constant_score, dis_max, function_score, has_child, has_parent, indices, nested, span_first, span_multi,span_first, 
span_multi, span_near, span_not, span_or, span_term, top_children,
filtered(废弃，使用 bool 包含一个 must 和一个 filter 替代)
```
### bool
bool 查询的外框架结构为：
```json5
    {
        "query": {
            "bool": {
                "must": [
                    {}
                ],
                "should": [
                    {}
                ],
                "must_not": [
                    {}
                ],
                "filter": [
                    {}
                ]
            }
        }
    }
//some other parameter for bool:
//boost,minimum_should_match,disable_coord
```