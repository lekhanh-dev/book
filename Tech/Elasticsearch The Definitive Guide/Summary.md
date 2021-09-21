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

# Chap 2. Life inside a Cluster (cluster sẽ chưa node)
- Mọi node in cluster đều biết được vị trí của doc, từ yêu càu tìm kiếm ở bất kỳ node nào các node sẽ giao tiếp với nhau để trả về kết quả
- Có 3 trạng thái của cluster để biết tình hình hoạt động của các cluster (GET /_cluster/health)
    + Green: Tất cả hoạt động ổn
    + Yellow: 1 một sô bản sao không hoạt động
    + Red: 1 số bản chính không hoạt động
- Với cơ chế mở rộng cluster theo chiều ngang, việc tạo ra những phân đoạn bản sao trên nhiều node giúp ES có thể tạo hồi phục lại phân đoạn chính nếu chúng bị mất đi do lỗi phần cứng => đảm bảo dữ liệu luôn được an toàn

# Chap 3. Data In, Data Out
- Việc biểu diễn các đối tượng trong cuộc sống theo kiểu bảng tính có nhiều bất cập => thay vào đó sử dụng JSON để biểu diễn flexible hơn
- Trong ES: Document đươc hiểu là đối tượng gốc cao nhất được biểu diễn dưới dạng JSON lưu trong ES với một ID duy nhất
- Một Doc không chỉ chứa data của riêng nó mà còn các thông tin đi liên quan (metadata) là: index, type, id
- Mọi Doc đều có số phiên bản (_version)  khi có sự thay đổi trong tài liệu thì nó được tăng lên
- Khi POST doc nếu ko chỉ định id thì ES tự thêm
- Mặc định thì GET thì ES sẽ lấy hết các trường của doc, nếu muốn lấy một phần ta use param '_source' VD
    ``` GET /website/blog/123?_source=title,text ```
- Sử dụng HEAD method để check doc có tồn tại trong ES ko
    ``` curl -i -XHEAD http://localhost:9200/website/blog/123 ``` 
        nếu status code = 200 thì tồn tại, 404 thì ko

- Quy trình mà ES update một doc
     ``` Get JSON từ doc cũ => Thay đổi nó => delete doc cũ => lập chỉ mục doc mới
- Delete 1 doc ```DELETE /website/blog/123```: ES sẽ ko xóa ngay mà đánh dấu là đã xóa
- ES là hệ phân tán, khi doc dc tạo, update, delete thì phiên bản mới của doc đươc sao chép sang các nút khác trong cụm, vì có thể xảy ra trường hợp doc cũ ghi đè doc mới nên ES sử dụng vesion đẻ đảm bảo các thay đổi được áp dụng đúng theo thứ tự

# Chap 4. Distributed Document Store
- Cách dữ liệu được lưu trong một hệ thống phân tán
- Mô tả cách create, index, delete một doc
    + Đầu tiên máy khách gửi yêu cầu đến một nút, tại nút đó dựa vào _id để xác định xem doc ở shards nào rồi gửi yêu cầu lại cho shard đó, tại shard chính nó sẽ gửi yêu cầu song song tới các shard bản sao, nếu thành công các shard bản sao gửi lại báo cáo thành công cho shard chính => shard chính gửi msg success cho shard ban đầu => rồi từ đó gửi thông báo thành công cho client
- Mặc định quy trình phản hồi từ clent > shard chính > shard sao là sync nhưng có thể thay đổi thành async cho nhanh hơn nhưng ES ko khuyến khích điều này
- Với yêu cầu đọc, nút yêu cầu sẽ vòng qua các shard bản sao, nếu tài liệu đó có trên shard chính nhưng chưa có trên shard bản sao => sau khi request index thành công thì doc sẽ có sẵn trên cả shard chính và shard bản sao.

# Chap 5. Searching - The basic tools
-The empty search: Trả về tất cả các doc và indices trong cluters
    ``` GET /_search ```
   VD result:
   ``` 
        {
            "hits" : {
                "total" : 14,
                "hits" : [
                   {
                        "_index":"us",
                        "_type":"tweet",
                        "_id":"7",
                        "_score":1,
                        "_source": {
                            "date":"2014-09-17",
                            "name":"John Smith",
                            "tweet":"The Query DSL is really powerful and flexible",
                            "user_id": 2
                        }
                    },
                ],
                "max_score" : 1
            },
            "took" : 4,
            "_shards" : {
                "failed" : 0,
                "successful" : 10,
                "total" : 10
            },
            "timed_out" : false
        }
   ```
    + Tìm kiếm tất cả nên score mặc định là 1
    +  Took: Thời gian thực thi query (milliseconds)
    +  shards: cho biết tổng số phân đoạn liên quan đến doc, số shard bị fail, success
    +  timeout: Nếu muốn trả những kết quả có thể trong một khoảng thời gian cho phép. 
            ``` GET /_search?timeout=10ms ```
- Các kiểu search
    ```/_search``` : Search all type in all indices
    ```/gb/_search``` : Search all type in gb indices
    ```/gb,us/_search``` : Search all type in gb and us indices
    ```/g*,u*/_search``` : Search all type in indices mà bắt đầu bằng kí tự g và u
    + Truy vấn phân trang
    ```GET /_search?size=5&from=10``` : Số lượng kết quả trả về là 5, và bắt đầu từ kết quả số 10

# Chap 6. Mapping and Analysis
- Cách xem cấu trúc tài liệu của một type (VD: type tweet of gb indices)
    ```GET /gb/_mapping/tweet```
- Data in ES dc chia thành 2 loại: Giá trị chính xác và full-text
- Giá trị full-text là một kiểu giá trị phi cấu trúc, thường muốn tìm những data trong doc có sự liên quan đến giá trị này
- ES sử dụng kỹ thuật `Inverted Index` để tìm kiếm kiểu full-text một cách nhanh chóng
    + ES sẽ tách các câu trên thành các từ, và đánh dấu xem những từ đó ở câu nào
    + Sau đó văn vản tìm kiếm cũng được tách thành các từ, rồi nó sẽ check về sự có mặt của từng tù trong các câu. Tổng hợp lại các từ được đánh dấu nhiều hơn trong câu nào thì câu đó gần với kết quả tìm kiếm
    + Nhưng có một số trường hợp đặc biệt VD như "Quick" và "quick" rõ ràng nghĩa giống nhau mà áp dụng cách trên thì ko hợp lý => ES đã cho về cùng chữ thường
    + Nếu la kiểu số nhiều hay số ít trong tiếng Anh chẳng hạn ('fox' and 'foxes') => ES cho số nhiều về số ít
    + Nếu các từ đồng nghĩa thì cho về một từ gốc 

- Analysis and Analyzers 
    + Phân tích gồm các việc sau:
        * Mã hóa văn bản thành các thuật ngữ riêng lể để sử dụng cho `Inverted Index`
        * Chuẩn hóa các thuật ngữ này về dạng chuẩn để tăng khả năng tìm kiếm (như 3 cái VD ở trên)
    + Built-in Analyzers: ES cung cấp các gói phân tích mà có thể sử dụng trực tiếp
        * Có câu mẫu sau : `Set the shape to semi-transparent by calling set_trans(5)`
        * Standard analyzer: Chuẩn mặc định của ES, tách các từ, bỏ dấu câu và lowercase các 
            set, the, shape, to, semi, transparent, by, calling, set_trans, 5
        * Simple analyzer: Tách từ giữa các ký tự ko phải là chữ cái
            set, the, shape, to, semi, transparent, by, calling, set, trans
        * Whitespace analyzer: Tách từ bằng khoảng trắng
            Set, the, shape, to, semi-transparent, by, calling, set_trans(5)
        * Language analyzers: Phân tích cho nhiều ngôn ngữ, VD tiếng Anh: nó bỏ đi các giới từ, mạo từ như 'the', 'to', 'or',...
            set, shape, semi, transpar, call, set_tran, 5
    + When Analyzers Are Used: Khi index một doc, các trường của full-text được phân tich dựa vào chuẩn đang đc sử dụng để tạo inverted index
        * Khi truy vân full-text thì nó sẽ áp dụng cùng một kiểu phân tích cho query string, còn truy vấn exact-value thì không
    + Testing Analyzers: ES cho xem cách một string được phân tích như thế nào bằng call API để xem kêt quả
        ``` GET /_analyze?analyzer=standard
            Text to analyze
        ```
        API đang thực hiện kêt quả string 'Text to analyze' vơi chuẩn là 'standard'

        Result

       ![Screenshot from 2021-09-19 00-03-52](https://user-images.githubusercontent.com/59026656/133896675-a3538b56-2174-4766-a7f0-3e2c670a476a.png)

    + Specifying Analyzers: Thường ES mặc định là ES dạng chuẩn, nếu muốn thay đổi chungs ta sẽ tự phải cấu hình bằng mapping

- Mapping
    + Core Simple Field Types: ES hỗ trợ các loại trường đơn giản sau
            String: string
            Whole number: byte , short , integer , long
            Floating-point: float , double
            Boolean: boolean
            Date: date
        * Khi index một doc mà xuất hiện một trường mới, ES sẽ map dynamic kiểu cho field mới đó 
    + Viewing the Mapping: Xem các field có kiểu dữ liệu gi
        ``` GET /gb/_mapping/tweet```
        
        ![c](https://user-images.githubusercontent.com/59026656/133897450-f5bbe137-72db-4778-b1fc-20b795d34dc0.png)

    + Customizing Field Mappings
        * Thuộc tính quan trọng nhất trong một field là type, đối với các trường khác string sẽ hiếm khi phải map
        * 2 thuộc tính quan trọng trong 1 field có type là string là: index và analyzer
        * Đối với index sẽ control cách string dc đánh chỉ mục, có 3 value:
            1. analyzed: Phân tích sting sau đó index => as fulltext
            2. not_analyzed: Lập chỉ mục chính xác giá trị đã được chỉ định, ko phân tích nó
            3. no: Không lập chỉ mục trường này, trường này sẽ không thể tìm kiếm được
            
            VD 
            ```
                {
                    "tag": {
                        "type": "string",
                        "index": "not_analyzed"
                    }
                }
            ```
        * Đối với analyzer: Thuộc tính này chỉ định máy phân tích nào được áp dụng khi tìm kiếm và lập chỉ mục, VD sử dụng máy phân tích tiếng anh
            ```
                {
                    "tweet": {
                        "type": "string",
                        "analyzer": "english"
                    }
                }

            ```
        * Updating a Mapping: Có thể thêm một mapping đến một new type hoặc update map của type đã tồn tại sử dụng endpoint /_mapping, nếu một trường đã mappinng
                              có thể data của field đõ đã đc index, nếu thay đổi field mapping, data dc index sẽ sai và ko thể tìm kiếm đúng.
                              Có thể cập nhập map để thêm new field, nhưng ko thể thay đổi field đã tồn tại từ analyzed to not_analyzed
                  VD update mapping
                    1. Xóa index db : `DELETE /gb`
                    2. Tạo index mới, chỉ định trường tweet sẽ use english analyzer

            ![Screenshot from 2021-09-20 00-51-26](https://user-images.githubusercontent.com/59026656/133937667-b4c948fc-3a51-4a17-8186-6a43b6a6be8a.png)
                    3. Thêm field 'tag' mới vào tweet

                    ```
                        PUT /gb/_mapping/tweet
                        {
                            "properties" : {
                                "tag" : {
                                "type" : "string",
                                "index": "not_analyzed"
                                }
                            }
                        }
                    ```


        * Testing the Mapping:
               ```
                    GET /gb/_analyze?field=tweet
                    Black-cats
               ```
    + Complex Core Field Types
        * Ngoài các kiểu dữ liệu đơn giản, JSON in ES còn hỗ trợ kiểu null, array, object
        * Multivalue Fields: Thay vì một chuỗi, chũng ta có thể index bằng một array
             ```  { "tag": [ "search", "nosql" ]} ```  
             1. Các giá trị trong mảng phải cùng kiểu dữ liệu
        * Empty Fields: Các gia trị dưới đây sẽ coi là trường trống và không được đánh chỉ mục
             ```        
                    "null_value": null,
                    "empty_array": [],
                    "array_with_null_value": [ null ]
             ```
        * Multilevel Objects: Kiểu đối tượng
            ``` 
                    {
                        "tweet": "Elasticsearch is very flexible",
                        "user": {
                                "id": "@johnsmith",
                                "gender": "male",
                                "age": 26,
                                "name": {
                                    "full": "John Smith",
                                    "first": "John",
                                    "last": "Smith"
                                }
                        }
                    }
            ```
        * Mapping for Inner Objects: ES tự phát hiện những trường đối tượng mới và tự động định kiểu type là object
                
            ![c](https://user-images.githubusercontent.com/59026656/134219190-b6ef161c-d897-4c41-b09b-5b610d081223.png)
           (1) Root object
           (2) Inner object
        * How Inner Objects are Indexed: Với data thêm vào ở mục `Multilevel Objects` dưới dây là cách mà ES index for one object
            ```
                    {
                        "tweet":[elasticsearch, flexible, very],
                        "user.id":[@johnsmith],
                        "user.gender":[male],
                        "user.age":[26],
                        "user.name.full":[john, smith],
                        "user.name.first": [john],
                        "user.name.last":[smith]
                    }
            ```
            Để phân biệt những trường cùng tên nó sẽ lây đường dẫn đầy đủ
        * Arrays of Inner Objects: Dưới dây là cách mà các obj in array đc index
            data:
            ```
                {
                     "followers": [
                        { "age": 35, "name": "Mary White"},
                        { "age": 26, "name": "Alex Jones"},
                        { "age": 19, "name": "Lisa Smith"}
                    ]
                }
            ```
            dc index: 
            ```
                {
                     "followers.age": [19, 26, 35],
                     "followers.name": [alex, jones, lisa, smith, mary, white]
                }
            ```
                Câu hỏi: Nếu bị index như này khi ta hỏi có ai 26 tuổi và tên là Alex Jones thì ÉS xử lý như nào ? Sẽ được trả lời trong chap 41
