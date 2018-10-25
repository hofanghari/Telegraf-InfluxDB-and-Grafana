# Telegraf-InfluxDB-and-Grafana
Telegraf, InfluxDB and Grafana monitor fo smartnet
## InfluxDB
	curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add - OK ubuntu@telegraf:~$ source /etc/lsb-release
	echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list  
	deb https://repos.influxdata.com/ubuntu xenial stable 
	sudo apt update 
	sudo apt install influxdb 
	sudo systemctl start influxdb

  

## Telegraf 

	wget https://dl.influxdata.com/telegraf/releases/telegraf_1.8.2-1_amd64.deb 
	sudo dpkg -I telegraf_1.8.2-1_amd64.deb 
	sudo systemctl start telegraf 

## snmp snmp-mibs-downloader
 
	sudo apt install snmp snmp-mibs-downloader 
	sudo nano /etc/snmp/snmp.conf 
Note that last step there - I commented out the mibs: line in snmp.conf.
