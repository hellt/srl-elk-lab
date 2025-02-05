# Intro

In old ages reading log files was normal exercise for sysadmin guru with assistance of old school and robust tools like grep/egrep, awk, sed...
Today's infrastructure requirements go well beyond just looking for the root cause why your application is crashing, incorrectly configured or ... just to realize that port occupied by another application.
For sure topics related to ML, close-loop automation, intrusion detection, security analysis and just keeping logs in structured form and programmatically accessible way are becoming a norm for system design and architecture.

This lab provides you with SR Linux / ELK playground to collect, handle and manage logs from your network devices. 
It comes with predefined pipelines and configurations of ELK stack to let playing w/o hassle with SR Linux, as well as developing log management applications with ready to use infrastructure.

# Lab Topology

This lab was created to allow easily enter this domain with your SR Linux based fabric.
clab topology represents 2-tier fabric topology and with 2 containers sitting within one L2 domain.

![ELK lab topology][topology]


Naming conventions are very simple:
* srl-1-* - leafs,
* srl-2-* - spines.
cl11 connectivity is using one interface attached to leaf1 (srl-1-1).
cl12 is connected as A/S to leaf3 (srl-1-3) and leaf4 (srl-1-4) with standby link signalling using LACP.
spine1 (srl-2-1) and spine2 (srl-2-2) are acting as BGP RR.
This setup is more than enough to demonstrate the way to integrate fabric with ELK stack.

# Quick start 

In order to bring up your lab follow the next simple steps:

1. Clone repo

```sh
git clone https://github.com/azyablov/srl-elk-lab.git
cd srl-elk-lab
```

2. Deploy the lab

```sh
cd <lab folder>
sudo clab deploy -t srl-elk.clab.yml
```

3. Create index template (to avoid automatic template creation by elastic)

```sh
curl -X PUT "localhost:9200/_index_template/fabric?pretty" -H 'Content-Type: application/json' -d @elk/logstash/index-template.json 
```

4. Import Kibana templates as described in [Kibana](#kibana) section. Kibana should available via [http://localhost:5601](http://localhost:5601)

5. Delete index created initially since it does not follow mappings and could not be adjusted any longer.

![Kibana delete index][index_deletion]

5. Run simulation to quickly ingest data into elasticsearch as described in [Simulation](#simulation)


# Simulation

In order to help quickly enrich ELK stack with logs ```outage_simulation.sh``` script could be executed with the following parameters:

```-S``` - to replace configuration for logstash remote server under ```/system/logging/remote-server[host=$LOGSTASHIP]"``` with new one.

```<WAITTIMER>``` - to adjust time interval between destructive actions applied (10 sec by default).

Basic configuration can found [here](./sys_log_logstash.json.tmpl), which represent default lab configuration, and can be adjusted per your needs and requirements.

```sh
./outage_simulation.sh -S
```

By default configuration for remote server using UDP:

```json
    {
      "host": "172.22.22.11",
      "remote-port": 1514,
      "subsystem": [
        {
          "priority": {
            "match-above": "informational"
          },
          "subsystem-name": "aaa"
        },
        {
          "priority": {
            "match-above": "informational"
          },
          "subsystem-name": "acl"
        },
<...output omitted for brevity...>
    }
```
In case TLS is a requirement, you can consider to put rsyslog in front, simple docker image with self-signed and custom certificate can be found on [github.com/azyablov/rsyslogbase](https://github.com/azyablov/rsyslogbase)


To run simulation just execute ```./outage_simulation.sh``` or ```./outage_simulation.sh 15``` in case machine is a bit slow or you have another labs running on the same compute.

![Outage Simulation][outage_simulation]

# Kibana

For the fast and convenient start of demo dashboard and discover search configuration [objects](./elk/kibana/kibana-dashboard.ndjson) are provided as part of this lab.
It could be added in few clicks using Kibana import under Stack Management.

![kibana import][kibaba_stask_mgmt]

Then you can go to to Discovery and Dashboard under Analytics and see simple dashboard.

![kibana discovery][kibaba_dashboard]

![kibana dashboard][kibaba_dashboard_2]


[topology]: ./pic/topology.png "Lab topology"
[kibaba_stask_mgmt]: ./pic/kibana_import.png "Stack Management"
[kibaba_dashboard]: ./pic/kibana_dashboard.png "Kibana dashboard #1"
[kibaba_dashboard_2]: ./pic/kibana_dashboard_2.png "Kibana dashboard #2"
[index_deletion]: ./pic/delete_index.png "Kibana delete index"
[outage_simulation]: ./pic/outage_smulation.gif "Simulation"
