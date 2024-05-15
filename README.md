# SNMP MANAGER COMMANDS

## Check snmpd service presence
```bash
sudo systemctl status snmpd
```

## Update package list
```bash
sudo apt update
```

## Install net-snmp
```bash
# snmp - net-snmp toolset
# snmpd - snmp agent service
# snmp-mibs-downloader - RFC MIB downloader script
# libsnmp-dev - development package, includes net-snmp-create-v3-user and net-snmp-config scripts
# ufw - frontend for iptables
sudo apt install snmp snmpd snmp-mibs-downloader libsnmp-dev ufw
```

## Check snmpd status
```bash
sudo systemctl status snmpd
```

## Start snmpd service
```bash
sudo systemctl start snmpd
```

## Enable snmpd service autostart after reboot
```bash
sudo systemctl enable snmpd
```

## Snmp agent configuration file edit
```bash
sudo cp /etc/snmp/snmpd.conf{,.backup}
sudo vim /etc/snmp/snmpd.conf
	## sample config
		### Interface to listen agents (self interface)
		#[EDIT]agentaddress udp:127.0.0.1:161,udp:[SNMP-MAN-IP]:161
		agentaddress udp:127.0.0.1:161,udp:192.168.1.3:161
		### system + hrSystem group, add :
		view all included .1
		view mib2 included .1.3.6.1.2.1
```

## Restart snmpd service after configuration change
```bash
sudo systemctl restart snmpd
```

## User specific MIB file location
```bash
mkdir ~/.snmp
mkdir ~/.snmp/mibs/
ls -l ~/.snmp/mibs/
```

## Show default MIB location
```bash
net-snmp-config --default-mibdirs
```

## Show snmp listening ports
```bash
sudo ss -nlpu | grep snmp
```

## Show firewall status and rules if firewall enabled
```bash
sudo ufw status
```

## Enable firewall if inactive
```bash
sudo ufw enable
```

## Allow inbound snmp udp/161 connection on firewall
```bash
#[EDIT] sudo ufw allow from [AGENT-IP] to [SNMP-MAN-IP] port 161 proto udp
sudo ufw allow from 192.168.1.2 to 192.168.1.3 port 161 proto udp
sudo ufw allow from 192.168.1.1 to 192.168.1.3 port 161 proto udp
```

## Show firewall status and rules
```bash
sudo ufw status
```

## Create SNMPv3 user
```bash
sudo systemctl stop snmpd
#[EDIT] sudo net-snmp-create-v3-user -A [yourAuthPassword] -a SHA -X [yourPrivPassword] -x AES [ADMIN-USERNAME]
sudo net-snmp-create-v3-user -A keyceadminsnmp@ -a SHA -X keyceadminsnmp@@ -x AES keyceadminsnmp
sudo systemctl start snmpd
```

## Query snmp agent (in this host) with authPriv configuration
```bash
#[EDIT] snmpget -v3 -a SHA -A [yourAuthPassword] -x AES -X [yourPrivPassword] -l authPriv -u [ADMIN-USERNAME] [IP] [OBJECT NAME]
snmpget -v3 -a SHA -A keyceadminsnmp@ -x AES -X keyceadminsnmp@@ -l authPriv -u keyceadminsnmp 127.0.0.1 SNMPv2-MIB::sysLocation.0
```

## Test agent from SNMP Manager
### Get all OID
```bash
#[EDIT] snmpwalk -v3 -u [agent-username] -l authPriv -a SHA -A [agentAuthPassword] -x AES -X [agentPrivPassword] [AGENT-IP]
snmpwalk -v3 -u router1agent -l authPriv -a SHA -A keycesnmpagent@ -x AES -X keycesnmpagent@@ 192.168.1.1
```
### Get message
```bash
#[EDIT] snmpget -v3 -a SHA -A [agentAuthPassword] -x AES -X [agentPrivPassword] -l authPriv -u [agent-username] 192.168.1.1 SNMPv2-MIB::sysLocation.0
snmpget -v3 -a SHA -A keycesnmpagent@ -x AES -X keycesnmpagent@@ -l authPriv -u router1agent 192.168.1.1 SNMPv2-MIB::sysLocation.0
```

# CISCO SNMP AGENT COMMANDS

## Enter the configuration mode
```bash
enable
configure terminal
```

## Enable SNMP v3
```bash
#[EDIT] snmp-server group [groupName] v3 priv
snmp-server group keycesnmp v3 priv
```

## Configure an SNMP v3 user
```bash
#[EDIT] snmp-server user [userName] [groupName] v3 auth sha [yourAuthPassword] priv aes 128 [yourPrivPassword]
snmp-server user router1agent keycesnmp v3 auth sha keycesnmpagent@ priv aes 128 keycesnmpagent@@
```

## Adjust the SNMP access permissions
```bash
#[EDIT] snmp-server host [SNMP-MAN-IP] traps version 3 auth [userName-SNMP-MAN]
snmp-server host 192.168.1.3 traps version 3 auth keyceadminsnmp
```

## Save config
```bash
end
copy running-config startup-config
```

