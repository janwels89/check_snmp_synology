# synology-nagios-plugin
A simple Nagios Plugin which checks the health of a Synology NAS

It supports checking the following items:
* System status
* Power status
* Fan status
* Disks status
* RAID (Volumes) status
* DSM version and update status
* System temperature
* UPS information (maybe)

## Based on
This plugin is a modified version of a plugin by deegan199. It can be found [here.](https://exchange.nagios.org/directory/Plugins/Network-and-Systems-Management/Others/Synology-status/details)

### Changes From Original
This version supports volume usage on DSM 6.2. Also, it checks readings by type,
rather then checking everything at once. This allows for splitting checks into
different services in Nagios. Look below for more information. As a part of this
change, the verbose option was removed and it is now always verbose. Also, the
UPS option was removed and added as a check type instead (but it hasn't been tested with a UPS).

- This Version supports divergin authentication and encryption passwords


## Requirements
snmpwalk and snmpget need to be installed. Also, the SNMP agent on the NAS has to be activated.

## Usage
```
usage: ./check_snmp_synology [OPTIONS] -u [user] -p [pass] -h [hostname]
options:
            -u [snmp username]          Username for SNMPv3
            -A [snmp privacy-password]          Password for SNMPv3
            -X [snmp authentification-password]         Password for SNMPv3
                  -l [level]            Set security level (noAuthNoPriv|authNoPriv|authPriv) (default AuthPriv)
                  -a [protocol]         Set authentication protocol (MD5|SHA) (default SHA)
                      -x [protocol]             Set privacy protocol (DES|AES) (default AES)

            -2 [community name]         Use SNMPv2 (no need user/password) & define community name (ex: public)

            -h [hostname or IP](:port)  Hostname or IP. You can also define a different port

            -W [warning temp]           Warning temperature (for disks & synology) (default 50)
            -C [critical temp]          Critical temperature (for disks & synology) (default 60)

            -w [warning %]              Warning storage usage percentage (default 80)
            -c [critical %]             Critical storage usage percentage (default 95)

            -t [check type]             The type of check to perform, must be one of the following: system version temperature power fan disk raid ups

            -i                          Ignore DSM updates

examples:       ./check_snmp_synology -u admin -p 1234 -h nas.intranet -t temperature
                ./check_snmp_synology -u admin -p 1234 -h nas.intranet -v -t system
                ./check_snmp_synology -2 public -h nas.intranet -t power
                ./check_snmp_synology -2 public -h nas.intranet:10161 -t ups
```

## Icinga2 Configuration Examples
### Command Definition
```
object CheckCommand "synology_snmp" {
  import "plugin-check-command"
  command = [ "<path to check_snmp_synology>" ]
  arguments = {
		"-u" = {
			order = -1
			value = "$snmpv3_user$"
					}
		"-A" = {
			order = -1
			value = "$snmpv3_auth_key$"
					}
		"-X" = {
			order = -1
			value = "$snmpv3_priv_key$"
					}
		"-h" = {
			order = -1
			value = "$address$"
					}
		"-t" = {
			value = "$synology_check_type$"
		}
  }
}
```

### Service Definition Expample
```
apply Service "temperature"{
  import "by-snmp"
  check_command = "synology_snmp"
  vars.synology_check_type = "temperature"
  assign where host.vars.synology
}

```
template Service "by-snmp" {
	import "global-service"
	check_command = "check_snmp"
	vars.snmpv3_user = "<your username>"
	vars.snmpv3_auth_key = "<your password>"
	vars.snmpv3_priv_key = "<your password>"
}
```
