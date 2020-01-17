# Setup and configure an ELK Stack Server on AWS

## Scope:
Green Fox Academy, Project phase

## Purpose:
Providing instructions to set up an ELK-Stack server (using Dockerized images) on AWS, send data to it by installing applications on other servers to be monitored. It also includes instructions on creating a customized Kibana dashboard.

## References:
1.	[Amazon Web Services Login](https://aws.amazon.com/console/)
1.	[Chama-Malachite Guide To Terraform](https://github.com/green-fox-academy/chama-malachite/blob/master/GuideToTerra.md)
1.	[Dockerized ELK Stack](https://github.com/deviantony/docker-elk)
1.	[Getting Started with Filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-getting-started.html)
1.	[Getting Started with Metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-getting-started.html)
1.	[Getting Started with HeartBeat](https://www.elastic.co/guide/en/beats/heartbeat/current/configuring-howto-heartbeat.html)

## Description:

1. Set up an ELK Stack server in Amazon Web Services

* Follow the Chama-Malachite Terraform guide as a template to create the Terraform files.
*	Use at least a Medium sized AWS instance.
*	While creating the security groups, make sure that the ELK specific ports are opened:

```
ingress {
	from_port  			= 5000
	to_port    			= 5000
	protocol   			= "tcp"
	description			=	"Logstash TCP input"
}
ingress {
	from_port  			= 5601
	to_port    			= 5601
	protocol   			= "tcp"
	description			=	"Kibana"
}
ingress {
	from_port  			= 9200
	to_port    			= 9200
	protocol   			= "tcp"
	description			=	"Elasticsearch HTTP"
}
ingress {
	from_port  			= 9300
	to_port    			= 9300
	protocol   			= "tcp"
	description			=	"Elasticsearch TCP transport"
}
```
*	The Terraform setup should contain the commands to install Docker and Docker-Compose (may not be included in a regular Docker install)
*	Clone the ELK Stack git repository:

```
$	git clone https://github.com/deviantony/docker-elk.git
```
*	In order to limit the memory consumption of the stack, the docker.compose.yml has to be edited.
```
$	cd docker-elk
$	printf "\nlimits:\n  memory: 1500m" >> docker.compose.yml
```
*	Start the stack by giving the compose command.
```
docker-compose up -d
```
*	Go to the URL AWS assigns to your server, and open port 5601 in the browser. You should see the Kibana login screen. The default username and password can be found on the Docker-Elk github readme.

1. Install monitoring programs on the monitored server

*	Follow steps one (Installation) and two (configuration) for the Filebeat, Metricbeat and Heartbeat systems.
**	Make sure that 'localhost' is replaced with the IP address of the ELK-Stack server, but the ports remain in place.
**	Do not add new lines into the config files, always edit or uncomment the existing ones.
*	Verify that data is sent from the monitored server to Elasticsearch by logging into Kibana and going to the Dashboard that's for the metrics module sent to Elasticsearch.
