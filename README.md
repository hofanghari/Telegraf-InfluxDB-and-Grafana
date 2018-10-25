# Telegraf-InfluxDB-and-Grafana
Telegraf, InfluxDB and Grafana monitor fo smartnet

https://lkhill.com/telegraf-influx-grafana-network-stats/
## InfluxDB
	curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add - OK ubuntu@telegraf:~$ source /etc/lsb-release
	echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list  
	deb https://repos.influxdata.com/ubuntu xenial stable 
	sudo apt update 
	sudo apt install influxdb 
	sudo systemctl start influxdb
	sudo apt install influxdb-client


  

## Telegraf 

	wget https://dl.influxdata.com/telegraf/releases/telegraf_1.8.2-1_amd64.deb 
	sudo dpkg -I telegraf_1.8.2-1_amd64.deb 
	sudo systemctl start telegraf 

## snmp snmp-mibs-downloader
 
	sudo apt install snmp snmp-mibs-downloader 
	sudo nano /etc/snmp/snmp.conf 
Note that last step there - I commented out the mibs: line in snmp.conf.

My switch already has SNMP configured, but let’s quickly check it is behaving

> snmpwalk -v 2c -c public 192.168.1.101 system

	SNMPv2-MIB::sysDescr.0 = STRING: AOS-W Version 6.4.2.6-4.1.1.12
	SNMPv2-MIB::sysObjectID.0 = OID: SNMPv2-SMI::enterprises.6486.800.1.1.2.2.2.1.2.52
	DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (181035051) 20 days, 22:52:30.51
	SNMPv2-MIB::sysContact.0 = STRING: 
	SNMPv2-MIB::sysName.0 = STRING: 9c:1c:12:cd:b4:a8
	SNMPv2-MIB::sysLocation.0 = STRING: 
	SNMPv2-MIB::sysServices.0 = INTEGER: 72
	
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
You can use the --test option to get Telegraf to grab one cycle of metrics, and publish them to stdout. This tells you that your configuration is sane, and capable of collecting data:

> telegraf --test --config /etc/telegraf/telegraf.d/leaf1.conf

	> snmp,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8 uptime=181872084i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=eth0,ifIndex=1 ifAdminStatus=1i,ifInDiscards=194i,ifInErrors=0i,ifInNUcastPkts=1717402i,ifInOctets=778255575i,ifInUcastPkts=15979946i,ifInUnknownProtos=0i,ifLastChange=0i,ifMtu=1500i,ifOperStatus=1i,ifOutDiscards=0i,ifOutErrors=0i,ifOutNUcastPkts=5638112i,ifOutOctets=2095379020i,ifOutUcastPkts=4142798i,ifPhysAddress="9c:1c:12:cd:b4:a8",ifSpeed=1000000000i,ifType=117i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=radio0_ssid_id0,ifIndex=50 ifAdminStatus=1i,ifInDiscards=0i,ifInErrors=0i,ifInNUcastPkts=5272177i,ifInOctets=2154070637i,ifInUcastPkts=60917395i,ifInUnknownProtos=0i,ifLastChange=0i,ifMtu=1500i,ifOperStatus=1i,ifOutDiscards=259767i,ifOutErrors=79i,ifOutNUcastPkts=476520i,ifOutOctets=1580578118i,ifOutUcastPkts=99785712i,ifPhysAddress="9c:1c:12:5b:4a:90",ifSpeed=0i,ifType=188i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=radio1_ssid_id0,ifIndex=70 ifAdminStatus=1i,ifInDiscards=0i,ifInErrors=0i,ifInNUcastPkts=1640025i,ifInOctets=2353863859i,ifInUcastPkts=10442330i,ifInUnknownProtos=0i,ifLastChange=0i,ifMtu=1500i,ifOperStatus=1i,ifOutDiscards=228429i,ifOutErrors=8i,ifOutNUcastPkts=476515i,ifOutOctets=1789217497i,ifOutUcastPkts=17963438i,ifPhysAddress="9c:1c:12:5b:4a:80",ifSpeed=0i,ifType=188i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=gre0,ifIndex=90 ifAdminStatus=2i,ifInDiscards=0i,ifInErrors=0i,ifInNUcastPkts=0i,ifInOctets=0i,ifInUcastPkts=0i,ifInUnknownProtos=0i,ifLastChange=0i,ifMtu=1500i,ifOperStatus=2i,ifOutDiscards=0i,ifOutErrors=0i,ifOutNUcastPkts=0i,ifOutOctets=0i,ifOutUcastPkts=0i,ifPhysAddress="00:00:00:00:00:00",ifSpeed=0i,ifType=131i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=BR0,ifIndex=500 ifAdminStatus=1i,ifInDiscards=0i,ifInErrors=0i,ifInNUcastPkts=0i,ifInOctets=0i,ifInUcastPkts=0i,ifInUnknownProtos=0i,ifLastChange=0i,ifMtu=1300i,ifOperStatus=1i,ifOutDiscards=2i,ifOutErrors=0i,ifOutNUcastPkts=0i,ifOutOctets=0i,ifOutUcastPkts=0i,ifPhysAddress="9c:1c:12:cd:b4:a8",ifSpeed=0i,ifType=1i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=gre0 ifConnectorPresent=1i,ifHCInBroadcastPkts=0i,ifHCInMulticastPkts=0i,ifHCInOctets=0i,ifHCInUcastPkts=0i,ifHCOutBroadcastPkts=0i,ifHCOutMulticastPkts=0i,ifHCOutOctets=0i,ifHCOutUcastPkts=0i,ifHighSpeed=0i,ifInBroadcastPkts=0i,ifInMulticastPkts=0i,ifLinkUpDownTrapEnable=2i,ifName="gre0",ifOutBroadcastPkts=0i,ifOutMulticastPkts=0i,ifPromiscuousMode=2i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=BR0 ifConnectorPresent=1i,ifHCInBroadcastPkts=0i,ifHCInMulticastPkts=0i,ifHCInOctets=0i,ifHCInUcastPkts=0i,ifHCOutBroadcastPkts=0i,ifHCOutMulticastPkts=0i,ifHCOutOctets=0i,ifHCOutUcastPkts=0i,ifHighSpeed=0i,ifInBroadcastPkts=0i,ifInMulticastPkts=0i,ifLinkUpDownTrapEnable=2i,ifName="BR0",ifOutBroadcastPkts=0i,ifOutMulticastPkts=0i,ifPromiscuousMode=2i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=eth0 ifConnectorPresent=1i,ifHCInBroadcastPkts=1717402i,ifHCInMulticastPkts=1717402i,ifHCInOctets=778260587i,ifHCInUcastPkts=15979974i,ifHCOutBroadcastPkts=5638112i,ifHCOutMulticastPkts=5638112i,ifHCOutOctets=2095384203i,ifHCOutUcastPkts=4142821i,ifHighSpeed=0i,ifInBroadcastPkts=1717402i,ifInMulticastPkts=1717402i,ifLinkUpDownTrapEnable=2i,ifName="eth0",ifOutBroadcastPkts=5638112i,ifOutMulticastPkts=5638112i,ifPromiscuousMode=2i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=radio0_ssid_id0 ifConnectorPresent=1i,ifHCInBroadcastPkts=5272177i,ifHCInMulticastPkts=5272177i,ifHCInOctets=2154070901i,ifHCInUcastPkts=60917399i,ifHCOutBroadcastPkts=476520i,ifHCOutMulticastPkts=476520i,ifHCOutOctets=1580580974i,ifHCOutUcastPkts=99785716i,ifHighSpeed=0i,ifInBroadcastPkts=5272177i,ifInMulticastPkts=5272177i,ifLinkUpDownTrapEnable=2i,ifName="radio0_ssid_id0",ifOutBroadcastPkts=476520i,ifOutMulticastPkts=476520i,ifPromiscuousMode=2i 1540451775000000000
	> interface,agent_host=192.168.1.101,host=localhost.localdomain,hostname=9c:1c:12:cd:b4:a8,ifDescr=radio1_ssid_id0 ifConnectorPresent=1i,ifHCInBroadcastPkts=1640025i,ifHCInMulticastPkts=1640025i,ifHCInOctets=2353863967i,ifHCInUcastPkts=10442332i,ifHCOutBroadcastPkts=476515i,ifHCOutMulticastPkts=476515i,ifHCOutOctets=1789217583i,ifHCOutUcastPkts=17963438i,ifHighSpeed=0i,ifInBroadcastPkts=1640025i,ifInMulticastPkts=1640025i,ifLinkUpDownTrapEnable=2i,ifName="radio1_ssid_id0",ifOutBroadcastPkts=476515i,ifOutMulticastPkts=476515i,ifPromiscuousMode=2i 1540451775000000000

This may produce a lot of output, if your device has many interfaces.

Now we just need to tell Telegraf to pick up the new configuration:

> sudo systemctl reload telegraf
That will now begin collecting data and storing it in InfluxDB.

After a couple of minutes, check to see if InfluxDB is storing results:
> influx

	Connected to http://localhost:8086 version 1.6.1
	InfluxDB shell version: 1.6.1
	>exit

http://docs.grafana.org/installation/debian/

> echo "deb https://packagecloud.io/grafana/stable/debian/ jessie main" | sudo tee /etc/apt/sources.list.d/grafana.list
> curl https://packagecloud.io/gpg.key | sudo apt-key add -
> sudo apt install grafana
