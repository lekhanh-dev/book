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
