# Voi Network
Voi a layer-1 blockchain solution that starts with Algorand's technology


# Hardware Requirement [^1][^2]
[^1]: [Voi Node Running Guide](https://docs.google.com/document/d/16n0yEivEUjFtSypiZ4TRaZQmRi_emMtFPrgAarsSNSE/edit)
[^2]: [Algorand Developer Portal](https://developer.algorand.org/docs/run-a-node/setup/install/)

- CPU : 8 vCPU
- RAM : 16 GB
- Storage : 100 GB NVMe SSD or equivalent
- Network :
  - Minimum : 100 Mbps
  - Recommend : 1 Gbps connection with low latency

> [!TIP]
> `CPU`, `Network` and `Disk I/O` become a potential bottleneck during High TPS. [^3]  
> If could, it is better to choose higher spec server than requirement.
[^3]: [Discord Voi Network chris.voi](https://discord.com/channels/1055863853633785857/1128672413048115250/1189180858205229066)


# Install Algorand node
Currently in production.  
Please use [D13 guide](https://d13.co/posts/set-up-voi-participation-node/#install-algorand-node-software) and [VNBnode guide](https://github.com/vnbnode/VNBnode-Guides/blob/main/VOI/voi.en.md) as a reference.

# Check the node’s status
Run this command:
``` bash
goal node status
```

Check that the node status is the fine.
1. `Genesis ID:` must be `voitest-v1`
2. Check that the node is synced/caught up.  
   The `Sync Time:` will display `Sync Time: 0.0s` when the node is fully caught up.  
   Comparing this `Last committed block: ` number to what is shown using an [Voi Explorer](https://voi.observer/explorer/home).

**----- Result -----**
```
Last committed block: 2899408    <===== CHECK HERE
Time since last block: 1.3s
Sync Time: 0.0s    <=================== CHECK HERE
Last consensus protocol: https://github.com/algorandfoundation/specs/tree/abd3d4823c6f77349fc04c3af7b1e99fe4df699f
Next consensus protocol: https://github.com/algorandfoundation/specs/tree/abd3d4823c6f77349fc04c3af7b1e99fe4df699f
Round for next consensus protocol: 2899409
Next consensus protocol supported: true
Last Catchpoint: 
Genesis ID: voitest-v1   <============= CHECK HERE
Genesis hash: IXnoWtviVVJW5LGivNFc0Dq14V3kqaXuK2u5OQrdVZo=
```

> [!Note]
> If your node status is not correct, please re-run node by following community guides[^4][^5] step by step.  
> If the problem is not resolved, please ask at [Voi Network Discord #node-runner-help](https://discord.com/channels/1055863853633785857/1128672413048115250)
[^4]: [D13 Set Up Voi Participation Node on Ubuntu 22.04](https://d13.co/posts/set-up-voi-participation-node/#optional-enable-telemetry)
[^5]: [VNBnode Guide](https://github.com/vnbnode/VNBnode-Guides/blob/main/VOI/voi.en.md)

# Set up monitoring system: Prometheus + Grafana
> [!TIP]
> **_Purpose_**  
> The purpose of system monitoring is to detect potential issues or anomalies in the system, so that they can be addressed proactively before they cause significant problems on network.
> 
> **_Prometheus_**  
> An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
>   
> **_Grafana_**  
> The open and composable observability and data visualization platform. Visualize metrics, logs, and traces from multiple sources like Prometheus and Loki.

> [!CAUTION]
> This guide is supported for `Ubuntu22.04 LTS`.

## Install Prometheus

Run this command:
``` bash
sudo apt-get install -y prometheus prometheus-node-exporter
```

Check if `prometheus` and `prometheus-node-exporter` has been installed:
``` bash
sudo dpkg -l prometheus prometheus-node-exporter
```

**----- Result -----**
```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                     Version                     Architecture Description
+++-========================-===========================-============-==========================================
ii  prometheus               2.31.2+ds1-1ubuntu1.22.04.2 amd64        monitoring system and time series database
ii  prometheus-node-exporter 1.3.1-1ubuntu0.22.04.2      amd64        Prometheus exporter for machine metrics
```

> [!TIP]
> The command install both `Prometheus` and `Prometheus Node Exporter`.  
> The `Prometheus Node Exporter` exposes a wide variety of `hardware-` and `kernel-related` metrics. [^6]
[^6]: [Prometheus: MONITORING LINUX HOST METRICS WITH THE NODE EXPORTER](https://prometheus.io/docs/guides/node-exporter/)

## Install Grafana
Run this command:
``` bash
sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo echo "deb https://packages.grafana.com/oss/deb stable main" > grafana.list
sudo mv grafana.list /etc/apt/sources.list.d/grafana.list

sudo apt-get update && sudo apt-get install -y grafana
```

Check if `grafana` has been installed:
``` bash
sudo dpkg -l grafana
```

**----- Result -----**
```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version      Architecture Description
+++-==============-============-============-=================================
ii  grafana        10.2.3       amd64        Grafana
```

## Start Prometheus and Grafana
Run this command:
``` bash
sudo systemctl enable grafana-server.service prometheus.service prometheus-node-exporter.service
sudo systemctl start grafana-server.service prometheus.service prometheus-node-exporter.service
```

1. Check if `Grafana` status is `active (running)`:
``` bash
sudo systemctl status grafana-server.service --no-pager -l
```

**----- Result -----**
```
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: http://docs.grafana.org
   Main PID: 69210 (grafana)
      Tasks: 21 (limit: 76910)
     Memory: 50.5M
        CPU: 2min 1.834s
     CGroup: /system.slice/grafana-server.service
             └─69210 /usr/share/grafana/bin/grafana server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:default.paths.logs=/var/l…

Dec 27 11:05:26 voi-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:05:26.769563461+01:00 level=info msg="Update check succeeded" duration=7.663637ms
Dec 27 11:05:26 voi-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:05:26.808206779+01:00 level=info msg="Completed cleanup jobs" duration=64.861523ms
Dec 27 11:05:26 voi-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:05:26.874721247+01:00 level=info msg="Update check succeeded" duration=70.901991ms
Dec 27 11:15:26 voi-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:15:26.769039867+01:00 level=info msg="Update check succeeded" duration=6.935579ms
Dec 27 11:15:26 voi-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:15:26.810085388+01:00 level=info msg="Completed cleanup jobs" duration=66.294104ms
Dec 27 11:15:26 voi-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:15:26.875407205+01:00 level=info msg="Update check succeeded" duration=72.228362ms
Dec 27 11:16:42 voi-node-testnet grafana[69210]: logger=infra.usagestats t=2023-12-27T11:16:42.755259475+01:00 level=info msg="Usage stats are ready to report"
Dec 27 11:25:26 voi-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:25:26.768865417+01:00 level=info msg="Update check succeeded" duration=6.73339ms
Dec 27 11:25:26 voi-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:25:26.809052346+01:00 level=info msg="Completed cleanup jobs" duration=65.2849ms
Dec 27 11:25:26 voi-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:25:26.860347684+01:00 level=info msg="Update check succeeded" duration=56.999845ms
```

2. Check if `Prometheus` status is `active (running)`:
``` bash
sudo systemctl status prometheus.service --no-pager -l
```

**----- Result -----**
```
● prometheus.service - Monitoring system and time series database
     Loaded: loaded (/lib/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:19 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://prometheus.io/docs/introduction/overview/
             man:prometheus(1)
   Main PID: 69220 (prometheus)
      Tasks: 18 (limit: 76910)
     Memory: 130.9M
        CPU: 4min 19.124s
     CGroup: /system.slice/prometheus.service
             └─69220 /usr/bin/prometheus

Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.716Z caller=compact.go:459 level=info component=tsdb msg="compact blocks" count=3 mint=1703613603258 maxt=1703635200000 ulid=01HJMT9WENTPNRFKS8V7AV04KN sources="[01HJKYTZ67785VT052NA9T7P1T 01HJM5PHHZPQ3NF1QEQFQ5PGAF 01HJMCJ8T01V9EDKPT5QVACB1S]" duration=183.420025ms
Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.725Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJKYTZ67785VT052NA9T7P1T
Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.731Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJM5PHHZPQ3NF1QEQFQ5PGAF
Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.740Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJMCJ8T01V9EDKPT5QVACB1S
Dec 27 08:00:03 voi-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.400Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703649603257 maxt=1703656800000 ulid=01HJN15EHZ5AMEWV02TT0K5SM0 duration=136.587432ms
Dec 27 08:00:03 voi-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.402Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.514546ms
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.421Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703656803257 maxt=1703664000000 ulid=01HJN81AP7G92NC8F4CHQP4X60 duration=157.185812ms
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.423Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.538992ms
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.452Z caller=checkpoint.go:97 level=info component=tsdb msg="Creating checkpoint" from_segment=12 to_segment=13 mint=1703664000000
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.541Z caller=head.go:974 level=info component=tsdb msg="WAL checkpoint complete" first=12 last=13 duration=88.867604ms
```


3. Check if `Prometheus Node Exporter` status is `active (running)`:
``` bash
sudo systemctl status prometheus-node-exporter.service --no-pager -l
```

**----- Result -----**
```
● prometheus-node-exporter.service - Prometheus exporter for machine metrics
     Loaded: loaded (/lib/systemd/system/prometheus-node-exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://github.com/prometheus/node_exporter
   Main PID: 69211 (prometheus-node)
      Tasks: 29 (limit: 76910)
     Memory: 13.7M
        CPU: 11min 42.505s
     CGroup: /system.slice/prometheus-node-exporter.service
             └─69211 /usr/bin/prometheus-node-exporter

Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=thermal_zone
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=time
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=timex
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=udp_queues
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=uname
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=vmstat
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=xfs
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=zfs
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```

## Add Voi metrics target to Prometheus config file
Run this command:
```
sudo sh -c "cat << EOT >> /etc/prometheus/prometheus.yml

  - job_name: voi
    static_configs:
      - targets: ['localhost:8080']
EOT"
```
> [!TIP]
> The port `8080` is REST API port.  
> You can get accounts, assets and block info by using REST API. [^7]
> 
> **Command to check account**  
> `curl 127.0.0.1:8080/v2/accounts/RWNOVLS5ZJM5GHM5WXIRMHG7NHXCYWLWYB5E4DTPILHQ7IIVNM5CVR4PVE -H "X-Algo-API-Token: $(cat /var/lib/algorand/algod.token)" | jq`  

[^7]: [Alorand REST API](https://developer.algorand.org/docs/rest-apis/algod/)


Check `Prometheus config file`:
``` bash
sudo cat /etc/prometheus/prometheus.yml
```
**----- Result -----**
``` yaml
# Sample config for Prometheus.

global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 5s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

  - job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100']

  - job_name: voi       # <======================== CHECK HERE
    static_configs:     # <======================== CHECK HERE
      - targets: ['localhost:8080']      # <======= CHECK HERE
```


## Reload Prometheus
Run this command:
``` bash
sudo systemctl reload prometheus.service
```

## Restart Prometheus and Grafana
Run this command:
``` bash
sudo systemctl restart grafana-server.service prometheus.service prometheus-node-exporter.service
```

1. Check if `Grafana` status is `active (running)`:
``` bash
sudo systemctl status grafana-server.service --no-pager -l
```

**----- Result -----**
```
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: http://docs.grafana.org
   Main PID: 69210 (grafana)
      Tasks: 21 (limit: 76910)
     Memory: 50.5M
        CPU: 2min 1.834s
     CGroup: /system.slice/grafana-server.service
             └─69210 /usr/share/grafana/bin/grafana server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:default.paths.logs=/var/l…

Dec 27 11:05:26 voi-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:05:26.769563461+01:00 level=info msg="Update check succeeded" duration=7.663637ms
Dec 27 11:05:26 voi-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:05:26.808206779+01:00 level=info msg="Completed cleanup jobs" duration=64.861523ms
Dec 27 11:05:26 voi-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:05:26.874721247+01:00 level=info msg="Update check succeeded" duration=70.901991ms
Dec 27 11:15:26 voi-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:15:26.769039867+01:00 level=info msg="Update check succeeded" duration=6.935579ms
Dec 27 11:15:26 voi-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:15:26.810085388+01:00 level=info msg="Completed cleanup jobs" duration=66.294104ms
Dec 27 11:15:26 voi-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:15:26.875407205+01:00 level=info msg="Update check succeeded" duration=72.228362ms
Dec 27 11:16:42 voi-node-testnet grafana[69210]: logger=infra.usagestats t=2023-12-27T11:16:42.755259475+01:00 level=info msg="Usage stats are ready to report"
Dec 27 11:25:26 voi-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:25:26.768865417+01:00 level=info msg="Update check succeeded" duration=6.73339ms
Dec 27 11:25:26 voi-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:25:26.809052346+01:00 level=info msg="Completed cleanup jobs" duration=65.2849ms
Dec 27 11:25:26 voi-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:25:26.860347684+01:00 level=info msg="Update check succeeded" duration=56.999845ms
```

2. Check if `Prometheus` status is `active (running)`:
``` bash
sudo systemctl status prometheus.service --no-pager -l
```

**----- Result -----**
```
● prometheus.service - Monitoring system and time series database
     Loaded: loaded (/lib/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:19 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://prometheus.io/docs/introduction/overview/
             man:prometheus(1)
   Main PID: 69220 (prometheus)
      Tasks: 18 (limit: 76910)
     Memory: 130.9M
        CPU: 4min 19.124s
     CGroup: /system.slice/prometheus.service
             └─69220 /usr/bin/prometheus

Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.716Z caller=compact.go:459 level=info component=tsdb msg="compact blocks" count=3 mint=1703613603258 maxt=1703635200000 ulid=01HJMT9WENTPNRFKS8V7AV04KN sources="[01HJKYTZ67785VT052NA9T7P1T 01HJM5PHHZPQ3NF1QEQFQ5PGAF 01HJMCJ8T01V9EDKPT5QVACB1S]" duration=183.420025ms
Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.725Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJKYTZ67785VT052NA9T7P1T
Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.731Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJM5PHHZPQ3NF1QEQFQ5PGAF
Dec 27 06:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.740Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJMCJ8T01V9EDKPT5QVACB1S
Dec 27 08:00:03 voi-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.400Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703649603257 maxt=1703656800000 ulid=01HJN15EHZ5AMEWV02TT0K5SM0 duration=136.587432ms
Dec 27 08:00:03 voi-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.402Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.514546ms
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.421Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703656803257 maxt=1703664000000 ulid=01HJN81AP7G92NC8F4CHQP4X60 duration=157.185812ms
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.423Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.538992ms
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.452Z caller=checkpoint.go:97 level=info component=tsdb msg="Creating checkpoint" from_segment=12 to_segment=13 mint=1703664000000
Dec 27 10:00:08 voi-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.541Z caller=head.go:974 level=info component=tsdb msg="WAL checkpoint complete" first=12 last=13 duration=88.867604ms
```


3. Check if `Prometheus Node Exporter` status is `active (running)`:
``` bash
sudo systemctl status prometheus-node-exporter.service --no-pager -l
```

**----- Result -----**
```
● prometheus-node-exporter.service - Prometheus exporter for machine metrics
     Loaded: loaded (/lib/systemd/system/prometheus-node-exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://github.com/prometheus/node_exporter
   Main PID: 69211 (prometheus-node)
      Tasks: 29 (limit: 76910)
     Memory: 13.7M
        CPU: 11min 42.505s
     CGroup: /system.slice/prometheus-node-exporter.service
             └─69211 /usr/bin/prometheus-node-exporter

Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=thermal_zone
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=time
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=timex
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=udp_queues
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=uname
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=vmstat
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=xfs
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=zfs
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Dec 26 04:15:18 voi-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```



# Set up Grafana Dashboard
> [!WARNING]
> If you set `ufw` at your server, You should open Grafana port (default: `3000`).  
> `ufw allow 3000/tcp`

> [!IMPORTANT]
> The following steps should be executed from your **Client, NOT server**.  


## Access your Grafana
Launch browser and go to `http://<YOUR-SERVER-GLOBAL-IP>:3000/`

> [!TIP]
> Run this command to check your server global ip.  
> `curl inet-ip.info`
<img width="1680" alt="grafana-01" src="https://github.com/qyeah98/voi/assets/99518712/123d7194-bd1c-45a6-9870-bbd331c91683">


## Login to Grafana
> [!TIP]
> Default username / password is `admin` / `admin`
<img width="1680" alt="grafana-02" src="https://github.com/qyeah98/voi/assets/99518712/c8d5a30e-cb02-41f9-8fd7-964e8367be91">

## Go to Connections -> Data Sources
<img width="1680" alt="grafana-04" src="https://github.com/qyeah98/voi/assets/99518712/4b8d5f57-9d46-4385-8596-9a84dacf7d65">

## Click Add new data source and select Prometheus
<img width="1680" alt="grafana-05" src="https://github.com/qyeah98/voi/assets/99518712/6f0a32c7-e952-4090-bf0e-90e9988165ef">
<img width="1680" alt="grafana-06" src="https://github.com/qyeah98/voi/assets/99518712/a0bb03fe-cdc4-4a19-9cf5-ebe0a2989960">

## Enter http://localhost:9090 into Prometheus server URL
<img width="1680" alt="grafana-07" src="https://github.com/qyeah98/voi/assets/99518712/b98ff99e-08cd-4262-b49f-206f8cc0f135">

## Scroll to the bottom and click Save & test
<img width="1680" alt="grafana-08" src="https://github.com/qyeah98/voi/assets/99518712/f473c4ad-7eae-48c1-b8d1-e1c682d90a5a">

## Go to Dashboards to create dashboard
<img width="1680" alt="grafana-10" src="https://github.com/qyeah98/voi/assets/99518712/a167cf36-7df2-41d8-987f-65b9020fdc0f">
<img width="1680" alt="grafana-11" src="https://github.com/qyeah98/voi/assets/99518712/3276da80-3d04-4ee8-8d89-eddaa02711f9">

## Click New > Import
<img width="1680" alt="grafana-12" src="https://github.com/qyeah98/voi/assets/99518712/5be3761c-b4fa-4b7a-aa8a-9ccd048dbe48">

## Click Upload JSON file
Please download the [voi grafana json file](https://github.com/qyeah98/voi/blob/main/grafana/voi-node.json) and upload it.
<img width="1680" alt="grafana-13" src="https://github.com/qyeah98/voi/assets/99518712/45d6e169-4fba-4468-b2de-b6b523312fbb">

## Select prometheus datasource
<img width="1680" alt="grafana-15" src="https://github.com/qyeah98/voi/assets/99518712/877795fb-ef44-4c65-a510-379db89c5734">
<img width="1680" alt="grafana-16" src="https://github.com/qyeah98/voi/assets/99518712/be5cfd78-46f2-4935-bbe8-cd5b37e1a06e">

## Complete
<img width="1680" alt="grafana-17" src="https://github.com/qyeah98/voi/assets/99518712/2714276f-54e9-43cf-bdcd-b4a79dda0c5a">

# (Optional) Set up notification system: Grafana + PagerDuty
> [!TIP]
> **_PagerDuty_**  
> PagerDuty can automatically group related alerts into a single incident to minimize noise while centralizing relevant context.
> Incident notifications are automatically sent using any preferred combination of phone calls, SMS, push notifications and emails.

Currently in production.
