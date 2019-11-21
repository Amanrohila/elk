# elk
Springboot elk example
Elasticsearch is a NoSQL database that is based on the Lucene search engine.
Logstash is a log pipeline tool that accepts inputs from various sources, executes different transformations, and exports the data to various targets. It is a dynamic data collection pipeline with an extensible plugin ecosystem and strong Elasticsearch synergy
Kibana is a visualization UI layer that works on top of Elasticsearch.

#1.Elasticsearch
Download athe latest version of elasticsearch from https://www.elastic.co/downloads/elasticsearch
Run bin\elasticsearch.bat from command prompt.
Accessed at http://localhost:9200

#2.Kibana
Download the latest version of kibana from https://www.elastic.co/downloads/kibana
Open config/kibana.yml in an editor and set elasticsearch.url to point at your Elasticsearch instance. In our case as we will use the local instance just uncomment  elasticsearch.hosts: ["http://localhost:9200"] or
elasticsearch.url: "http://localhost:9200"
Run bin\kibana.bat from command prompt.
Once started successfully, Kibana will start on default port 5601 and Kibana UI will be available at http://localhost:5601

#3.Logstash
Download the latest distribution from download page https://www.elastic.co/downloads/logstash
Create a configuration file named logstash.conf and write following code

input {
  file {
    type => "java"
	#for multiple file which are in folder of c:/elk
	path => "c:/elk/*.*"
	#for single file 
    #path => "C:/elk/spring-boot-elk.log" 
    codec => multiline {
      pattern => "^%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}.*"
      negate => "true"
      what => "previous"
    }
  }
}
 
filter {
  #If log line contains tab character followed by 'at' then we will tag that entry as stacktrace
  if [message] =~ "\tat" {
    grok {
      match => ["message", "^(\tat)"]
      add_tag => ["stacktrace"]
    }
  }
 
}
 
output {
   
  stdout {
    codec => rubydebug
  }
 
  # Sending properly parsed log events to elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}


#4.Springboot microservice Project 
Create  2 or more Spring Boot Project

define their logs file location in their corresponding properties file like below
logging.file=C:/elk/spring-boot-elk.log
logging.file=C:/elk/spring-boot-elk2.log 

write code to genrate log in your code like LOG.log(Level.INFO, response) 
run application and it would write logs in log files

#5.Kibana Portal Configuration
got to http://localhost:5601/app/kibana and create an index pattern logstash-* to see the indexed data
Management>ndex Patterns>logstash-*
now go to discover to see logs, now you can apply filter, search etc in it.



