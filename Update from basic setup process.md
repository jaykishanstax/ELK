# After setting up Filebeat and ELK

## Improve Indexing through INGEST NODE and Grok Filter processing.

 ## Basic use of FILEBEAT
 
 #### 1. Configure filebeat input with type of log and path
 ``` 
 filebeat.inputs:
    - type: log
      paths:
         - /var/log/messages
         - /var/log/*.log 
 ```
               
#### 2. We can exclude or include line with regex in configuration
```
exclude_lines: ['^DBG']
include_lines: ['timestamp']
```

#### 3. We can read multiple line message while exception thrown using below configuration. 
```
multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
multiline.negate: true
multiline.match: after
```

- filebeat match all line start with above exp. which is for timestamp. if time stamp is not their than it will combine all other line until timestamp comes

#### 4. Multiline pattern should be common so that it can be used into Grok processor to genrate index in elastic.



 ## Basic use of ELASTIC
 
- it will create index pattern based on incoming log from filebeat.

- We can add INGEST NODE and Grok Filter processing to manage log and indexing easily .

- Below command will use to generate grok filter

```
PUT /_ingest/pipeline/logpipeline
{
  "description" : "Pipeline for logs from filebeat",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["(?m)%{TIMESTAMP_ISO8601:logtime}%{SPACE}%{LOGLEVEL:loglevel}%{SPACE}%{THREAD:thread}%{SPACE}%{USER:user}%{SPACE}---%{SPACE}%{GREEDYDATA:class}%{SPACE}:%{GREEDYDATA:message}"],
        "pattern_definitions": {
            "THREAD" : "[[a-zA-Z0-9-]]+",
            "USER" : "[[a-zA-Z0-9:]]+"
          }
      }
    }
  ]
}
```

OR 

```
PUT /_ingest/pipeline/log_pipeline1
{
  "description": "Parsing the application logs",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": [
          "%{TIMESTAMP_ISO8601:timestamp} %{GREEDYMULTILINE:stackTrace}"
        ],
        "pattern_definitions": {
          "GREEDYMULTILINE": "(.|\n)*"
        },
        "ignore_missing": true
      }
    }
  ]
}
```


## Basic use of KIBANA 

##### 1.  check all Index from Console view using below command :
 - ```GET /_cat/indices?v&s=index:desc```
 
 ##### 2.  Delete Particular/all Index from Console view using below command :
 - ```DELETE /filebeat-6.1.4-2019.01.27```
 
