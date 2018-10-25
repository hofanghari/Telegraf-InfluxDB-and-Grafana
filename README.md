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
> sudo nano /etc/telegraf/telegraf.d/leaf1.conf

	[[inputs.snmp]]
	  agents = [ "192.168.1.101" ]
	  version = 2
	  community = "public"
	  interval = "60s"
	  timeout = "10s"
	  retries = 3

	  [[inputs.snmp.field]]
	    name = "hostname"
	    oid = "RFC1213-MIB::sysName.0"
	    is_tag = true

	  [[inputs.snmp.field]]
	    name = "uptime"
	    oid = "DISMAN-EXPRESSION-MIB::sysUpTimeInstance"

	  # IF-MIB::ifTable contains counters on input and output traffic as well as errors and discards.
	  [[inputs.snmp.table]]
	    name = "interface"
	    inherit_tags = [ "hostname" ]
	    oid = "IF-MIB::ifTable"

	    # Interface tag - used to identify interface in metrics database
	    [[inputs.snmp.table.field]]
	      name = "ifDescr"
	      oid = "IF-MIB::ifDescr"
	      is_tag = true

	  # IF-MIB::ifXTable contains newer High Capacity (HC) counters that do not overflow as fast for a few of the ifTable counters
	  [[inputs.snmp.table]]
	    name = "interface"
	    inherit_tags = [ "hostname" ]
	    oid = "IF-MIB::ifXTable"

	    # Interface tag - used to identify interface in metrics database
	    [[inputs.snmp.table.field]]
	      name = "ifDescr"
	      oid = "IF-MIB::ifDescr"
	      is_tag = true

	  # EtherLike-MIB::dot3StatsTable contains detailed ethernet-level information about what kind of errors have been logged on an interface (such as FCS error, frame too long, etc)
	  [[inputs.snmp.table]]
	    name = "interface"
	    inherit_tags = [ "hostname" ]
	    oid = "EtherLike-MIB::dot3StatsTable"

	    # Interface tag - used to identify interface in metrics database
	    [[inputs.snmp.table.field]]
	      name = "ifDescr"
	      oid = "IF-MIB::ifDescr"
	      is_tag = true
