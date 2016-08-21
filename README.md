
SNMP network monitoring template
===============================================

SNMP network monitoring
-------------------

Collect the performance information of network equipment using the SNMP statistics. The collected data is aggregated monitoring the server side and the graph registration.

**Notes**

1. SNMP statistics remote in a separate requirements of the Linux for the agent to be collected, and then running the Linux agent on the monitored server.
2. snmpwalk, in order to use the snmpget command, you will need the following additional net-snmp package to the agent.

```
sudo -E yum -y install net-snmp net-snmp-utils
```

File organization
-------

Necessary configuration files to the template is as follows.

|               Directory               |        file name         |             Applications            |
|---------------------------------------|--------------------------|-------------------------------------|
| lib/agent/SNMPNetwork/conf/           | ini file                 | agent collecting configuration file |
| lib/agent/SNMPNetwork/script/         | get snmp module          | agent collected script              |
| lib/Getperf/Command/Site/SNMPNetwork/ | pm file                  | data aggregation script             |
| lib/graph/SNMPNetwork/                | json file                | graph template registration rules   |
| lib/cacti/template/0.8.8g/            | xml file                 | Cacti template export file          |
| Script /                              | create_graph_template.sh | graph template registration script  |

Install
=====

Build SNMPNetwork template
-------------------

Clone the project from Git Hub

```
(Git clone to project replication)
```

Go to the project directory, Initialize the site with the template options.

```
cd t_SNMPNetwork
initsite --template .
```

Run the Cacti graph templates created scripts in order.

```
./script/create_graph_template.sh
```

Export the Cacti graph templates to file.

```
cacti-cli --export SNMPNetwork
```

Aggregate script, graph registration rules, and archive the export file set Cacti graph templates.

```
sumup --export=SNMPNetwork --archive=$GETPERF_HOME/var/template/archive/config-SNMPNetwork.tar.gz
```

Import of SNMPNetwork template
---------------------

Import the archive file that you created in the previous to the monitoring site

```
cd {monitoring site home}
sumup --import=SNMPNetwork --archive=$GETPERF_HOME/var/template/archive/config-SNMPNetwork.tar.gz
```

Import the Cacti graph templates. Please select a template in accordance with the monitored storage

```
cacti-cli --import SNMPNetwork
```

To reflect the imported aggregate script, and then restart the counting daemon

```
sumup restart
```

how to use
=============

Agent Setup
--------------------

The following agent collected script, and copy it to the bottom of the agent of the script directory (/ home/{OS user}/ptune/script /).

```
{Site home}/lib/agent/SNMPNetwork/script/
```

Similarly, the configuration file under the following conf, and copy it to the (/ home/{OS user}/ptune/conf /).

```
{Site home} /lib/agent/SNMPNetwork/conf/SNMPNetwork.ini
```

Check the configuration information of the network devices to be monitored. To copy the script check_snmp.pl to run with the -p option.

```
cd ~/ptune/script
./check_snmp.pl -p
```

Please enter the SNMP connection information of the monitored equipment. When you run, the following result files.

|    File name    |                               Definition                               |
|-----------------|------------------------------------------------------------------------|
| Check_snmp.yaml | of the monitored network configuration information                     |
| Check_snmp.cmd  | agent configuration command definition stationery file SNMPNetwork.ini |

The results of the check \ _snmp.cmd by reference to, please edit the SNMPNetwork.ini. check \ _snmp.cmd because you have all of the network port to monitor, please edit if necessary, such as by removing the unnecessary ports.

To reflect the settings, please restart the agent.

Customization of data aggregation
--------------------

After the agent setup, and data aggregation is performed, site home directory of the lib/Getperf/Command/Master/SNMPNetwork.pm file under is output.
This script is the master definition of monitored storage, please edit the SNMPNetwork.pm_sample under the same directory as an example.
Storage controller, LUN, describes the Raid group applications. Even without the editing of SNMPNetwork.pm, data is aggregated.

Graph registration
-----------------

After the agent setup, and data aggregation is performed, site node definition file under the node of the home directory will be generated.
Specify the generated directory and run the cacti-cli.

```
cacti-cli node/SNMPNetwork/{network node}/
```

AUTHOR
-----------

Minoru Furusawa <minoru.furusawa@toshiba.co.jp>

COPYRIGHT
-----------

Copyright 2014-2016, Minoru Furusawa, Toshiba corporation.

LICENSE
-----------

This program is released under [GNU General Public License, version 2] (http://www.gnu.org/licenses/gpl-2.0.html).