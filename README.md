This repository provides instructions for setting up the Elastic Stack (Elasticsearch, Kibana, Logstash) and Filebeat on AWS. This setup allows you to collect, process, and visualize log data from various sources. Follow the steps below to get started.

Table of Contents
Prerequisites
Instance Setup
Elasticsearch
Kibana
Logstash
Apache 2 Installation
Filebeat
Post-Configuration
Verification
Log Flow
Prerequisites
Before you begin, make sure you have:

An AWS account with sufficient permissions to create and manage EC2 instances.
Basic knowledge of Linux commands.
Instance Setup
Create an EC2 instance on AWS with the following specifications:
8 CPU
32GB Memory
80GB HDD
Configure a Security Group to allow incoming traffic on ports 9200, 5601, and 5044.
Elasticsearch
Update and install required packages:
bash
Copy code
sudo apt update
sudo apt install curl wget gnupg2 openjdk-11-jdk apt-transport-https
Add Elasticsearch repository and update packages:
bash
Copy code
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch --no-check-certificate | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
Install and configure Elasticsearch:
bash
Copy code
sudo apt install elasticsearch
sudo nano /etc/elasticsearch/elasticsearch.yml
Edit the configuration file as instructed, then enable and start Elasticsearch:

bash
Copy code
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
Kibana
Install Kibana and configure:
bash
Copy code
sudo apt install kibana
sudo nano /etc/kibana/kibana.yml
Edit the configuration file as instructed, then enable and start Kibana:

bash
Copy code
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
Access Kibana by opening http://<ipaddress>:5601 in your browser.
Logstash
Install Logstash:
bash
Copy code
sudo apt install logstash
Create Logstash configuration files:
bash
Copy code
sudo nano /etc/logstash/conf.d/2-beats-input.conf
sudo nano /etc/logstash/conf.d/2-elasticsearch-output.conf
Edit the configuration files as instructed.

Enable and start Logstash:
bash
Copy code
sudo systemctl enable logstash
sudo systemctl start logstash
Apache 2 Installation
Install Apache 2 for generating log data:

bash
Copy code
sudo apt install apache2
Filebeat
Download and install Filebeat:
bash
Copy code
sudo curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.12-amd64.deb
sudo dpkg -i filebeat-7.17.12-amd64.deb
Configure Filebeat:
bash
Copy code
sudo nano /etc/filebeat/filebeat.yml
Edit the configuration file as instructed.

Enable the Filebeat module and load the index template:
bash
Copy code
sudo filebeat modules enable system
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
Enable and start Filebeat:
bash
Copy code
sudo systemctl enable filebeat
sudo systemctl start filebeat
Post-Configuration
In Kibana, go to Index Management and set the index pattern to filebeat-*.
Validate indexing by navigating to the Discover tab in Kibana.
Verification
Use the following commands to verify the setup:

Check Filebeat index:
bash
Copy code
curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
List indices:
bash
Copy code
curl -XGET 'http://localhost:9200/_cat/indices'
Check Filebeat logstash health:
bash
Copy code
curl http://localhost:9200/filebeat-*/_count?pretty
Log Flow
To view the log flow:

bash
Copy code
journalctl -u filebeat.service
Congratulations! You have successfully set up the ELK stack along with Filebeat on AWS. Your system is now ready to collect, process, and visualize log data.
