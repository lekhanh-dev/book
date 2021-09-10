## PART I. Getting Started

# Chap 1. You Know, for Search

Installing Elasticsearch

RESTful API with JSON over HTTP
- Có thể ử dụng curl để giao tiếp với ES
```curl -X<VERB> '<PROTOCOL>://<HOST>/<PATH>?<QUERY_STRING>' -d '<BODY>'```

    VERB : Http method : GET, POST,...

    PROTOCOL: Http or https

    HOST: localhost

    POST: 9200

    QUERY_STRING: Tham số VD: ?pretty

- ES sử dụng JSON để định dạng các document
- Có thể coi ES như là mội DB

  Relational DB ⇒ Databases ⇒ Tables ⇒   Rows
  ⇒ Columns

  Elasticsearch ⇒ Indices ⇒ Types ⇒    Documents ⇒ Fields
  
* Thực hành: Tạo index vào doc
- Tạo Indices: PUT /megacorp?pretty
- Tạo Doc: (Số 2 là id của doc nếu ko tạo, hệ thống sẽ ramdom, employee là type => đang dùng bản 7.4.0 thấy báo là type ko dùng nữa, mà chuyển thành /megacorp/_doc/2 (mặc định là _doc, chưa biết tại sao lại chuyển như này nhỉ? chả có nhẽ chỉ có 1 type)
    ```
    PUT /megacorp/employee/2
    {
    "first_name" : "Jane",
    "last_name" : "Smith",
    "age" : 32,
    "about" :"I like to collect rock albums",
    "interests": [ "music" ]
    }
    ```
- Search Doc vừa tạo
    ```GET /megacorp/employee/_search```=> search tất cả các doc trong index megacorp, type employee
    new syntax: ``` GET /megacorp/_doc/_search ```
- Search chính xác doc theo id hay một fields nào đấy
    ```GET /megacorp/employee/2```
    ```GET /megacorp/employee/_search?q=last_name:Smith```
- Search with Query DSL: một cách search mạnh mẽ hơn sử dụng JSON request body, VD dưới tương đương với GET /megacorp/employee/_search?q=last_name:Smith
   ``` GET /megacorp/employee/_search
    {
        "query" : {
            "match" : {
                "last_name" : "Smith"
            }
        }
    }
    ```
- More-Complicated Searches: Tìm kiếm với query phức tạp hơn, VD này tìm tất cả các nhân viên có tên sau là 'smith' và còn phải trên 30 tuổi ('gt' stands for greater than)
    ```GET /megacorp/employee/_search
    {
        "query" : {
            "filtered" : {
                "filter" : {
                    "range" : {
                        "age" : { "gt" : 30 }
                    }
                },
                "query" : {
                    "match" : {
                        "last_name" : "smith"
                    }
                }
            }
        }
    }
    ```
- Trong ES hỗ trợ Full Text Search, kết quả sẽ được xếp loại theo trường 'score', score càng cao thì độ tin cậy cho việc tìm kiếm chính xác càng cao VD:
    ```GET /megacorp/employee/_search
        {
            "query" : {
                "match" : {
                    "about" : "rock climbing"
                    }
                }
            }
    ```

- Phrase Search: Sử dụng 'match_phrase' khi bạn muốn tìm kiếm chính xác một từ nào đó trong một câu văn chẳng hạn VD:
    ```GET /megacorp/employee/_search
        {
            "query" : {
                "match_phrase" : {
                    "about" : "rock climbing"
                     }
                }
         }
    ```
- Highlighting Our Searches: Đơn giản chỉ là tạo hightlight cho kết quả tìm kiếm, đêm kết quả trong trường 'hightlight' vào HTML sẽ thấy sự khác biệt :)
    ```
        GET /megacorp/employee/_search
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
    ```
- Analytics: Một chức năng kiểu phân tích thống kê data (VD việc xuất ra kết quả thống kê sở thích của nhân viên in company), giống với GROUP BY bên SQL nhưng mạnh hơn VD:
    ```
        GET /megacorp/employee/_search
            {
                "aggs": {
                    "all_interests": {
                        "terms": { "field": "interests" }
                        }
                    }
            }
    ```
    
=> Hết chap 1 tìm hiểu những cách tìm kiếm kết quả đơn giản của ES

# Chap 2. Life inside a Cluster (cluster sẽ chưa node
- Mọi node in cluster đều biết được vị trí của doc, từ yêu càu tìm kiếm ở bất kỳ node nào các node sẽ giao tiếp với nhau để trả về kết quả
- Có 3 trạng thái của cluster để biết tình hình hoạt động của các cluster (GET /_cluster/health)
    + Green: Tất cả hoạt động ổn
    + Yellow: 1 một sô bản sao không hoạt động
    + Red: 1 số bản chính không hoạt động

