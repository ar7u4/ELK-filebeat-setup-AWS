# ELK-filebeat-setup-AWS

This repository provides instructions for setting up the Elastic Stack (Elasticsearch, Kibana, Logstash) and Filebeat on AWS. This setup allows you to collect, process, and visualize log data from various sources. Follow the steps below to get started.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Instance Setup](#instance-setup)
- [Elasticsearch](#elasticsearch)
- [Kibana](#kibana)
- [Logstash](#logstash)
- [Apache 2 Installation](#apache-2-installation)
- [Filebeat](#filebeat)
- [Post-Configuration](#post-configuration)
- [Verification](#verification)
- [Log Flow](#log-flow)

## Prerequisites

Before you begin, make sure you have:

- An AWS account with sufficient permissions to create and manage EC2 instances.
- Basic knowledge of Linux commands.

## Instance Setup

1. Create an EC2 instance on AWS with the following specifications:
    - 8 CPU
    - 32GB Memory
    - 30GB HDD
2. Configure a Security Group to allow incoming traffic on ports 9200, 5601, and 5044.

## Elasticsearch
Install Required package :
```
sudo apt update
sudo apt install curl 
sudo apt install wget 
sudo apt install gnupg2
sudo apt install openjdk-11-jdk
sudo apt install apt-transport-https 
```

Add Repository :
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch --no-check-certificate | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```
Update the Linux packages post adding the repository :
```
sudo apt update
```
  Elasticsearch
Install Elasticsearch package:

```
sudo apt install elasticsearch
```

Configure elasticsearch configuration file using #VIM editor

```
Path : sudo vi /etc/elasticsearch/elasticsearch.yml
```

```yml
Remove # tag before the line 
network.host:localhost
port:9200
```
Enable and Start the Elasticsearch:
```
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```
To verify elasticsearch:
```
curl -XGET 'http://localhost:9200'
curl -XGET 'localhost:9200/_cluster/health?pretty'
```
Kibana
Install Kibana package:
```
sudo apt install kibana
```
Configure kibana configuration file using #nano editor
```
Path : sudo nano /etc/kibana/kibana.yml
Remove # tag before the line 
ip.address:'0.0.0.0' # it will accept any IP address 
port:5601
elasticsearch "localhost:9200"
```

Enable and Start the kibana:
```
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
```
To verify kibana:
```
#Try to open the kibana url on the browser

http://<ipaddress:5601>
```

## Logstash
Install Logstash package:
```
sudo apt install logstash
```
#Create a configuration file called 2-beats-input.conf where you will set up your Filebeat input:
```
Path : sudo nano /etc/logstash/conf.d/2-beats-input.conf
#Insert below lines on above mentioned #path

input {
  beats {
    port => 5044
  }
}

output {
  stdout {}
}
```

#Next, create a configuration file called 2-elasticsearch-output.conf
```
Path : sudo nano /etc/logstash/conf.d/2-elasticsearch-output.conf
# Insert below lines on above mentioned #path. Essentially, this output configures Logstash to store the Beats data in Elasticsearch, which is running at localhost:9200, in an index named after the Beat used. The Beat used in this tutorial is Filebeat

output {
  if [@metadata][pipeline] {
    elasticsearch {
      hosts => ["localhost:9200"]
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      pipeline => "%{[@metadata][pipeline]}"
    }
  } else {
    elasticsearch {
      hosts => ["localhost:9200"]
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    }
  }
}
```
Enable and Start the logstash:
```
sudo systemctl enable logstash
sudo systemctl start logstash
```
Start the logstash with pipeline configuration:
```
sudo /usr/share/logstash/bin/logstash -f 2-elasticsearch-output.conf
```
Test your logstash configuration with below command:
```
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```

##Apache 2 ( Linux Web Server )

Install Apache 2 package:
```
sudo apt install apache2
```
##Filebeat
Download the filebeat:
```
sudo curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.12-amd64.deb
sudo dpkg -i filebeat-7.17.12-amd64.deb
```
Configure filebeat configuration file using #nano editor
```
Path : sudo nano /etc/filebeat/filebeat.yml

Remove # tag before the line 
output.logstash
hosts:['localhost:5044']
# Change the value to true to enable the input configuration
enabled: true
# Add the apache2 web server logs path 
- /var/log/apache2/access.log
# Set to True to enable the configuration reloading
reload.enabled:true
```
Enable the Filebeat module:
```
sudo filebeat modules enable system
```
Load the index Template using below command:
```
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```
Enable and Start the Filebeat:
```
sudo systemctl enable filebeat
sudo systemctl start filebeat
```
Post Configuration go to kibana and set the index
```
Go to Index Management and
Set the index filebeat-*
```
Validate the indexing by going to discover tab
To Verify :

Filebeat index :
```
curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
```
Indices
```
curl -XGET 'http://localhost:9200/_cat/indices'
```
Filebeat logstash health
```
curl http://localhost:9200/filebeat-*/_count?pretty
```
Log Flow
```
journalctl -u filebeat.service
```
