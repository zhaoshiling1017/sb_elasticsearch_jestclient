# sb_elasticsearch_jestclient
使用jestClient操作elasticsearch


全文搜索引擎 Elasticsearch 入门教程
http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html


springboot+jest 操作Elasticsearch 参考：https://www.cnblogs.com/youxiu326/p/elasticSearch-jestClient.html


创建mapper 参考：https://blog.csdn.net/tanga842428/article/details/76229235

中文doc: https://www.elastic.co/guide/cn/elasticsearch/guide/cn/query-time-boosting.html

```

1.创建一个索引

发送post请求:
http://192.168.79.129:9200/productindex

返回:
{
     "acknowledged": true
}

2.查看索引mapping内容

发送get请求：
http://192.168.79.129:9200/productindex/_mapping?pretty
返回:
{
  "productindex": {
    "mappings": {}
  }
}

3.给productindex索引加一个type，type name为product，并设置mapping

发送post请求:
http://192.168.79.129:9200/productindex/product/_mapping?pretty
参数:
{
    "product": {
            "properties": {
                "title": {
                    "type": "string",
                    "store": "yes"
                },
                "description": {
                    "type": "string",
                    "index": "not_analyzed"
                },
                "price": {
                    "type": "double"
                },
                "onSale": {
                    "type": "boolean"
                },
                "type": {
                    "type": "integer"
                },
                "createDate": {
                    "type": "date"
                }
            }
        }
  }
  
  返回:
  {
    "acknowledged": true
  }
  
  4.查看mapper是否创建成功
  
  发送get请求:
  http://192.168.79.129:9200/productindex/_mapping?pretty
  
  返回:
  {
    "productindex": {
      "mappings": {
        "product": {
          "properties": {
            "createDate": {
              "type": "date",
              "format": "strict_date_optional_time||epoch_millis"
            },
            "description": {
              "type": "string",
              "index": "not_analyzed"
            },
            "onSale": {
              "type": "boolean"
            },
            "price": {
              "type": "double"
            },
            "title": {
              "type": "string",
              "store": true
            },
            "type": {
              "type": "integer"
            }
          }
        }
      }
    }
  }
  
  5.修改mapping (product新增一个字段，那么需要修改mapping)
  
  发送post请求:
  http://192.168.79.129:9200/productindex/product/_mapping?pretty
  
  参数:
  {
   "product": {
              "properties": {
                   "amount":{
                      "type":"integer"
                 }
              }
          }
  }
  
  返回:
  {
    "acknowledged": true
  }
  
  
  
  
 

  6.创建索引
  
  
  {
    "product": {
        "properties": {
            "title": {
                "type": "keyword"
            },
            "description": {
                "type": "keyword"
            },
            "price": {
                "type": "double"
            },
            "onSale": {
                "type": "boolean"
            },
            "type": {
                "type": "integer"
            },
            "createDate": {
                "type": "date"
            },
            "colorDetails": {
                "properties":{
                    "image": {
                        "type": "keyword"
                    },
                    "code": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
}

```


 ---------------------------------------------------

```

计算集群中文档的数量，我们可以用这个
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}


创建一个索引:

curl -X PUT "localhost:9200/megacorp/employee/1" -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
'
curl -X PUT "localhost:9200/megacorp/employee/2" -H 'Content-Type: application/json' -d'
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
'
curl -X PUT "localhost:9200/megacorp/employee/3" -H 'Content-Type: application/json' -d'
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
'



查询索引
http://youxiu326.xin:9200/megacorp/employee/

{
  "query" : {}
}

curl -X GET "localhost:9200/megacorp/employee/1"

查询megacorp索引 employee类型 所有的文档
curl -X GET "localhost:9200/megacorp/employee/_search"



GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}

 同样搜索姓氏为 Smith 的雇员，但这次我们只需要年龄大于 30 的。查询需要稍作调整，使用过滤器 filter ，它支持高效地执行一个结构化查询
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith"
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 }
                }
            }
        }
    }
}


找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者短语 。 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
'


高亮查询
许多应用都倾向于在每个搜索结果中 高亮 部分文本片段，以便让用户知道为何该文档符合查询条件。在 Elasticsearch 中检索出高亮片段也很容易。
再次执行前面的查询，并增加一个新的 highlight 参数
curl -X GET "localhost:9200/megacorp/employee/_search" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
'

分析
终于到了最后一个业务需求：支持管理者对雇员目录做分析。 Elasticsearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 GROUP BY 类似但更强大。
举个例子，挖掘出雇员中最受欢迎的兴趣爱好：
get http://youxiu326.xin:9200/megacorp/employee/_search?pretty
{

	"aggs": {
	    "all_interests": {
	      "terms": { "field": "interests" }
	    }
	  }

}

{
	"query": {
	    "match": {
	      "last_name": "smith"
	    }
	  },
	"aggs": {
	    "all_interests": {
	      "terms": { "field": "interests" }
	    }
	  }

}

```