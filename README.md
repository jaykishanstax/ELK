# Centralized Log management through Filebeat with Elastic and Kibana (ELK).

## Why centralized logging Needed?

I worked with micro services architecture which is looked like below. Each micro service is having its own log folder & file.

<h1 align="center">
  <br>
  <img src="https://github.com/jaykishanstax/ELK/blob/master/Image/micro_service_Arc.png">
</h1>

### Suppose we have multiple machine for each micro-service and each service is having multiple instance. 
###### For example we have 2 machine X and Y. each service (4 micro-service) is deployed on both machine and having their own log folder under each machine. So In total we have 8 folder & file which store log only for micro-service, other logs like nginx, application logs, system and access logs for each machine include so many logging which is kind of unstructured data. 

- If one of the request or something is fails from micro-services then to find out the cause of failure we need to Dig into both machine under each 8 folder/file which is very time & resource consuming. 

## Basic overview of centralized logging system.

Centralize logging is one of the solution with multiple implementation way, some of them are as below described.

#### 1.	Filebeat + Logstash + Elastic + Kibana
-	Filebeat will run on each machine and read log files and ship data. Its lightweight, fast and does not consume resources and memory.
-	Filebeat send data to Logstash and Elastic search where data indexed.
-	At last we can view data from Kibana with multiple type of filter based on our requirement and perform analytics.

#### 2.	Logstash + Elastic + Kibana
-	Logstash is log pipeline tool which take data from different sources and transform those data into pre-define format and send it to target. Grok filter will create JSON data from log.
-	Elastic will get data from logstash and add/save them with indices. Elasticsearch is a NoSQL database that is based on the Lucene search engine. log data will indexed on elastic
-	Kibana provide visualization for elasticsearch data through its analytics view and dashboard.

###### There are so many other ways around it like we can use Kafka, Elastic Ingest Node, JsonLogAppender etc. to improve centralize logging effectively.  


## Notes :: We can improve logs view and Tracing using below steps.  

1.	We can **add spring cloud sleuth** for span-id, trace-id and application name in log file.
2.	We can update **logback file** with identified pattern for LOG_LEVEL_PATTERN value. 
3.	We need to maintain logs to **centralized location** like,  /usr/logs/*.log or /usr/logs/archived/*.log 
4.	We need to **Structure Log** Data (JSON→ logstash)

###### Basic Requirement to Setup Filebeat + (Elastic + Kibana)same machine – with min 8 GB ram config.   





# Configuration for  ELK: 

## Centralized logging architecture look like below ::

<h1 align="center">
  <br>
  <img src="https://github.com/jaykishanstax/ELK/blob/master/Image/ELK_arc.png">
</h1>

## 1.	Elastic:  
- a.	Version 6.1.4
- b.	Instal from below link 
- - i.	https://www.elastic.co/guide/en/elasticsearch/reference/6.1/rpm.html
- c.	Below step to install ELASTIC 
- - i.	rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
- - ii.	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.4.rpm
- - iii.	sudo rpm --install elasticsearch-6.1.4.rpm
- - iv.	YML file under:  cd /etc/elasticsearch/
- - v.	sudo /bin/systemctl daemon-reload
- - vi.	sudo /bin/systemctl enable elasticsearch.service
- - vii.	sudo systemctl start elasticsearch.service
- - viii.	To check log : sudo journalctl --unit elasticsearch
- - ix.	To check elastic is up :: curl -i -XGET 'localhost:9200/'


## 2.	Kibana: 
- a.	Version 6.1.4
- b.	Install from below link :
- - i.	https://www.elastic.co/guide/en/kibana/6.1/rpm.html
- c.	Below Steps to install KIBANA
- - i.	 rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
- - ii.	wget https://artifacts.elastic.co/downloads/kibana/kibana-6.1.4-x86_64.rpm
- - iii.	sudo rpm --install kibana-6.1.4-x86_64.rpm
- - iv.	sudo /bin/systemctl daemon-reload
- - v.	sudo /bin/systemctl enable kibana.service
- - vi.	YML file under /etc/kibana/ folder
- - vii.	Start service :: sudo systemctl start kibana.service
- - viii.	To check it working :: curl -i -XGET 'localhost:5601/'
- - ix.	To check Logs :: journalctl -u kibana.service
- - x.	Kibana yml having configuration for log management with external path.
- - xi.	To give permission to kibana folder under log if it externally configured : “chmod 777 kibana/”


## 3.	Filebeat : 
- a.	Version 6.1.4
- b.	Install from below link : 
- - i.	https://www.elastic.co/guide/en/beats/filebeat/6.1/filebeat-installation.html
- c.	Steps to install filebeat :
- - i.	curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.1.4-x86_64.rpm
- - ii.	sudo rpm -vi filebeat-6.1.4-x86_64.rpm
- - iii.	YML file under :: cd /etc/filebeat/
- - iv.	Update yml file with kibana, elastic server name, log reading path etc. 
- - v.	Start filebeat service
- - vi.	filebeat setup --dashboards :: to load it on kibana


## Nginx config :: 
Change nginx config to support call from outside to elasticsearch and kibana can be access from outside. Once configured please restart. 

## Commands ::

### To check port used by service :
``` netstat -tlpn ```

### To start and stop service ::
``` sudo systemctl stop kibana.service
sudo systemctl start kibana.service
sudo systemctl stop elasticsearch.service
sudo systemctl start elasticsearch.service
service --status-all
```

### To remove all setup :
``` yum remove kibana
Clear all directory 
Rm -rf  /usr/share/kibana/
sudo rm -rf elasticsearch-6.4.2.tar.gz etc.
```
