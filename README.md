Log Service
=============

Get/Retrieve log and statistic email

- Registered port: 9297

API Reference
-------------

### Paging

Params: `?page=1`

### Get all messages

Request:

```bash
curl -X GET -H "Content-Type: application/json" "http://localhost:9297/"
```

Response success:

```json
{
  "meta": {
    "total_records":25,
    "page_size":null,
    "page":1,
    "page_count":null
  }.
  records:[{
    "timestamp":"2016-01-01 00:00:00",
    "status":"sent/deffered/bounced",
    "from":"test@example.com",
    "to":"test@example.com",
    "subject":""
  }]
}
```

### statistic code from postfix log

Request:

```bash
curl -X GET -H "Content-Type: application/json" "http://localhost:9297/statistic"
```

Response success:

```json
{
  {"meta":
    {
      "total_records":5,
      "page_size":null,
      "page":1,
      "page_count":null
    },
    "records":{250":7,"421":1,"553":2,"550":14}
  }
}
```

### Retrieve server errors log

Request:

```bash
curl http://localhost:9297/errors
```

Response success:

```json
{
  "meta": {
    "total_records":25,
    "page_size":null,
    "page":1,
    "page_count":null
  }.
  records:[{
    "email":null,
    "to":"test@example.com",
    "timestamp":"2016-01-01 00:00:00",
    "code":5xx,
    "status":"bounced",
    "subject":""}
  ]
}
```

KONG setup
----------

KONG registration information:

- name: 'ms-log-service'
- upstream_url: http://ms-log-service:9297
- request_path: /logs
- strip_request_path: true

ENV variables with default:

```ruby
$kong = {
  endpoint: (ENV['KONG_ENDPOINT'] || 'http://localhost:8001'),
  api: {
    name: (ENV['KONG_API_NAME'] || 'ms-log-service'),
    request_path: (ENV['KONG_API_REQUEST_PATH'] || '/logs'),
    strip_request_path: (ENV['KONG_API_STRIP_REQUES'] || true),
    upstream_url: (ENV['KONG_API_UPSTREAM_URL'] || 'http://ms-log-service:9297')
  }
}
```

Setup Elastichsearch , Logstash
- default'll localhost for elasticseach ( listen port at 9200 ) and logstash( listen port at 5000)
- if have external host you can set env for it:
  + For Elasticsearch: ```ELASTICSEARCH_HOST```
  + For Logstash: + LOGSTASH_HOST
                  + LOGSTASH_PORT
                  + LOGSTASH_PROTOCAL

Registration instruction:

Init API:

```bash
KONG_ENDPOINT=http://123.31.11.xx:8001 bundle exec rake kong:init
```

Un-register:

```bash
KONG_ENDPOINT=http://123.31.11.xx:8001 bundle exec rake kong:bye
```


Docker Run

```
docker pull docker-registry.vccloud.vn/lamdt/ms-log-service
docker run -d --name rabbitmq \
           -p 9297:9297 \
           --link elasticsearch:elasticsearch \
           docker-registry.vccloud.vn/lamdt/ms-log-service
```
