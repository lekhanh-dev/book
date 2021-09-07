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
    GET /megacorp/employee/_search => search tất cả các doc trong index megacorp, type employee
    new: GET /megacorp/_doc/_search
- Search chính xác doc theo id hay một fields nào đấy
    GET /megacorp/employee/2
    GET /megacorp/employee/_search?q=last_name:Smith
- Search with Query DSL: một cách search mạnh mẽ hơn sử dụng JSON request body, VD dưới tương đương với GET /megacorp/employee/_search?q=last_name:Smith
    GET /megacorp/employee/_search
    {
        "query" : {
            "match" : {
                "last_name" : "Smith"
            }
        }
    }
- More-Complicated Searches: Tìm kiếm với query phức tạp hơn, VD này tìm tất cả các nhân viên có tên sau là 'smith' và còn phải trên 30 tuổi ('gt' stands for greater than)
    GET /megacorp/employee/_search
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
    
