# VM Monitoring via Prometheus(DB & Logger) and Grafana(visualization)



## Table of Contents

- [About](#about)
- [Getting Started](#getting_started)
- [Usage](#usage)
- [Contributing](../CONTRIBUTING.md)

## About <a name = "about"></a>

Write about 1-2 paragraphs describing the purpose of your project.

## Getting Started <a name = "getting_started"></a>

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See [deployment](#deployment) for notes on how to deploy the project on a live system.

### Prerequisites

What things you need to install the software and how to install them.

```
Give examples
```

### Installing

A step by step series of examples that tell you how to get a development env running.

Say what the step will be

```
Give the example
```

Check the URL 

http://localhost:9090/targets?search=


![Prometheus output](image.png)

## NodePorter <a name = "NodePorter"></a>


Firstly create a system user for Nodeporte
```
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter

```

Download the latest version from
https://prometheus.io/download/

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz

```

![alt text](image-1.png)


Extract node exporter from the archive.
```
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
```

Move the binary 

```
sudo mv \
  node_exporter-1.7.0.linux-amd64/node_exporter \
  /usr/local/bin/

```

Cleanup the zip and folder 

```
rm -rf node_exporter*
```

Check the version

```
node_exporter --version
```
![alt text](image-2.png)


Now create the systemd service
```
sudo nano /etc/systemd/system/node_exporter.service

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind

[Install]
WantedBy=multi-user.target


```
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
journalctl -u node_exporter -f --no-pager

![alt text](image-3.png)

we need to add node_exporter in prometheus


```
sudo nano /etc/prometheus/prometheus.yml

ADD THIS LINES init


  - job_name: node_export
    static_configs:
      - targets: ["localhost:9100"]
![alt text](image-4.png)


check the  syntax via

```


promtool check config /etc/prometheus/prometheus.yml
Then, you can use a POST request to reload the config.
```
curl -X POST http://localhost:9090/-/reload
```
or just restart the service via 

``
sudo systemctl restart prometheus.service 
```

and check the prometheus

![alt text](image-13.png)
![alt text](image-14.png)


click on graph under prometheus screen and enter "scrape_duration_seconds" under query and execute

![alt text](image-15.png)
check node port working by 
http://localhost:9100/metrics




![alt text](image-8.png)
```
click on metrics
```
![alt text](image-9.png)

```


Check the targets section http://<ip>:9090/targets

```

## INSTALL GRAFANA


First, let's make sure that all the dependencies are installed.

sudo apt-get install -y apt-transport-https software-properties-common

Next, add GPG key.

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -


echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update


sudo apt-get -y install grafana

To automatically start the Grafana after reboot, enable the service.


sudo systemctl enable grafana-server
Then start the Grafana.

sudo systemctl start grafana-server
To check the status of Grafana, run the following command:

sudo systemctl status grafana-server

![alt text](image-7.png)



go to url :
http://localhost:3000/login


![alt text](image-5.png)

type admin , admin as usernm and passwd


now you will see the grafana login 

![alt text](image-6.png)


To visualize metrics, you need to add a data source first. 
* Click Add data source and select Prometheus. For the URL, enter http://localhost:9090 . You can see Data source is working.


![alt text](image-10.png)
and click Save and test

![save and test](image-11.png)


* WITHOUT UI adding datasource from code : 

Usually, in production environments, you would store all the configurations in Git. Let me show you another way to add a data source as a code. Let's remove the data source from UI.
Create a new datasources.yaml file.


```

sudo nano /etc/grafana/provisioning/datasources/datasources.yaml

```


```
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    url: http://localhost:9090
    isDefault: true


RESTART after saving this file

sudo systemctl restart grafana-server


```

you should now see datasource added by default 

![alt text](image-12.png)

   

    Click on graph under prometheus screen and enter "scrape_duration_seconds" under query and execute

![alt text](image-15.png)


    This metric indicates the duration of the scrape of each Prometheus target. 

    Start Grafana and create a new dashboard/create dashboard >> then add Visualization. 

    select data source as "Prometheus" that comes by default>>

    Title the panel "Scrape Duration" and paste the scrape_duration_seconds metric. 

    Adjust the time interval to 1 hour for better visualization. 

![alt text](image-16.png)

    Use the job label for the legend and set the unit to seconds. 

 ![alt text](image-17.png)

![alt text](image-18.png)

    Click apply and save the dashboard as "Prometheus". 

    here is what it will look like

![alt text](image-19.png)

    Import an open-source dashboard for Node Exporter metrics from the Grafana website. 
https://grafana.com/grafana/dashboards/1860-node-exporter-full/

    Copy dashboard ID 1860 to the clipboard. 

    In Grafana, click Import, paste the ID, and load the dashboard. 
![alt text](image-20.png)

    Select Prometheus as the data source and import the dashboard. 

you will see something like this
![alt text](image-26.png)



## INSTALL PUSHGATEWAY

Create a dedicated user first.

```
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false pushgateway


INSTALL

wget https://github.com/prometheus/pushgateway/releases/download/v1.7.0/pushgateway-1.7.0.linux-amd64.tar.gz

tar -xvf pushgateway-1.7.0.linux-amd64.tar.gz

sudo mv pushgateway-1.7.0.linux-amd64/pushgateway /usr/local/bin/

pushgateway --version

sudo nano /etc/systemd/system/pushgateway.service

Enable the service.

sudo systemctl enable pushgateway
Start Pushgateway.

sudo systemctl start pushgateway
Check the status.

sudo systemctl status pushgateway


sudo nano /etc/prometheus/prometheus.yml

 - job_name: pushgateway
    honor_labels: true
    static_configs:
      - targets: ["localhost:9091"]

Check Prometheus configuration. If it's valid, reload the config.

promtool check config /etc/prometheus/prometheus.yml
curl -X POST http://localhost:9090/-/reload
```

To send metrics to the Pushgateway, you just need to send a POST request to the following endpoint http://localhost:9091/metrics/job/backup. Where backup is an arbitrary name that will show up as a label.


![alt text](image-21.png)

check in prometheus url as well
![alt text](image-22.png)

Use curl and pipe the string with echo to Pushgateway. Let's imagine that the Jenkins job that we named backup took almost 16 seconds to complete.

```
echo "jenkins_job_duration_seconds 15.98" | curl --data-binary @- http://localhost:9091/metrics/job/backup
```

You can find this metric in Prometheus. Refresh the page and start typing jenkins_job_duration_seconds.

![alt text](image-23.png)


Since prometheus is Open i.e without authentication it can be accesssed by anyone
you can add a auth with nginx at front 

* BUT Prometheus comes with a builtin auth feature
* Prometheus will not store your passwords; it will compute the hash and compare it with the existing one for the given user.

```
sudo apt-get -y install python3-bcrypt

nano generate_password.py


import getpass
import bcrypt

password = getpass.getpass("password: ")
hashed_password = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
print(hashed_password.decode())

```

Run the script and enter Devops@123 for the password.

python3 generate_password.py

Copy this hash and create an additional Prometheus configuration file.

sudo nano /etc/prometheus/web.yml
web.yml

---
basic_auth_users:
    admin: $2b$12$CVcceMyfix1Qa7Kupisfe.JVHXG.U4PWFUculUnGlxPrTlBxfNGRe
Now, we need to provide this config to the Prometheus. Let's update the systemd service definition.

sudo nano /etc/systemd/system/prometheus.service
prometheus.service

...
ExecStart=/usr/local/bin/prometheus \
  ...
  --web.config.file=/etc/prometheus/web.yml
Every time you update the systemd service, you need to reload it.

sudo systemctl daemon-reload
You also need to restart Prometheus.

sudo systemctl restart prometheus
And check the status in case of an error.

sudo systemctl status prometheus
Now, we can test basic authentication. Go to Prometheus and reload the page.
We also need to update the Grafana datasource to provide a username and password.

sudo nano /etc/grafana/provisioning/datasources/datasources.yaml
datasources.yaml

---
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    url: http://localhost:9090
    isDefault: true
    basicAuth: true
    basicAuthUser: admin
    secureJsonData:
      basicAuthPassword: Devops@123
Restart grafana.

sudo systemctl restart grafana-server

# Install Alertmanager

First, let's create a system user for Alertmanager.

sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false alertmanager

wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz

tar -xvf alertmanager-0.26.0.linux-amd64.tar.gz


For Alertmanager, we need storage. It is mandatory (it defaults to "data/") and is used to store Alertmanager's notification states and silences. Without this state (or if you wipe it), Alertmanager would not know across restarts what silences were created or what notifications were already sent.


sudo mkdir -p /alertmanager-data /etc/alertmanager

sudo mv alertmanager-0.26.0.linux-amd64/alertmanager /usr/local/bin/
sudo mv alertmanager-0.26.0.linux-amd64/alertmanager.yml /etc/alertmanager/

check:
alertmanager --version

sudo nano /etc/systemd/system/alertmanager.service

[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=alertmanager
Group=alertmanager
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/alertmanager \
  --storage.path=/alertmanager-data \
  --config.file=/etc/alertmanager/alertmanager.yml

[Install]
WantedBy=multi-user.target
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
sudo systemctl status alertmanager


Alertmanager will be exposed on port 9093 http://<ip>:9093.

It's time to create a simple alert. In almost all Prometheus setups, you have an alert that is always active. It is used to validate the monitoring system itself. For example, it can be integrated with the deadmanssnitch service. If something goes wrong with the Prometheus or Alertmanager and, you will get an emergency notification that your monitoring system is down. It's a very useful service, especially in production environments.

Let's create alert but without integration with DeadMansSnitch.


sudo nano /etc/prometheus/dead-mans-snitch-rule.yml

---
groups:
- name: dead-mans-snitch
  rules:
  - alert: DeadMansSnitch
    annotations:
      message: This alert is integrated with DeadMansSnitch.
    expr: vector(1)


You also need to update the Prometheus config to specify the location of Alertmanager and specify the path to the new rule.


sudo nano /etc/prometheus/prometheus.yml
...
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093
rule_files:
  - dead-mans-snitch-rule.yml


promtool check config /etc/prometheus/prometheus.yml
sudo systemctl restart prometheus
sudo systemctl status prometheus


![alt text](image-24.png)



## Alert Manager - Slack Integration

Alertmanager can be configured to send emails, can be integrated with PagerDuty and many other services. For this demo, I will integrate Alertmanager with Slack. We're going to create a slack channel where all the alerts will be sent.

Let's create alerts Slack channel.

Create a new Slack app from scratch. Give it a name Prometheus and select a workspace.
You can modify the app from the basic information. Let's upload the Prometheus icon.
Next, we need to enable incoming webhooks. Then add webhook to the workspace.
The last thing, we need to copy Webhook URL and use it in Alertmanager config.
Now, update alertmanager.yml config to include a new route to send alerts to the Slack.


sudo nano /etc/alertmanager/alertmanager.yml

cat /etc/alertmanager/alertmanager.yml 
```
---
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 1m
  repeat_interval: 10m
  receiver: 'web.hook'
  routes:
  - receiver: slack-notifications
    match:
      severity: warning
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
- name: slack-notifications
  slack_configs:
  - channel: "#monitoringdevops"
    send_resolved: true
    api_url: "https://hooks.slack.com/services/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"
    title: "{{ .GroupLabels.alertname }}"
    text: "{{ range .Alerts }}{{ .Annotations.message }}\n{{ end }}"
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']

```



![FINALL OUTPUT](image-25.png)


## Since you can see your grafana dashboard but you want to automate the process of alerting so for that we will create a batch-job-rule.yml file

```
sudo nano batch-job-rules.yml

groups:
- name: batch-job-rules
  rules:
  - alert: Ignore!
    annotations:
      message: Testing Server Alerting by Maheboob.
    expr: jenkins_job_duration_seconds{job="backup"} > 30
    for: 5s
    labels:
      severity: warning

  - alert: HighCpuUsage
    expr: (100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)) > 5
    for: 5s
    labels:
      severity: warning
    annotations:
      summary: "High CPU Usage Detected"
      message: "CPU usage is more than 5% for 5 seconds."

  - alert: HighRamUsage
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 50
    for: 5s
    labels:
      severity: warning
    annotations:
      summary: "High RAM Usage Detected"
      message: "RAM usage is more than 50% for 5 seconds."
```

and edit the prometheus.yml as well

```

# my global config
global:
  scrape_interval: 10s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 10s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - localhost:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - dead-mans-snitch-rule.yml
  - batch-job-rules.yml

  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    basic_auth:
      username: admin
      password: Devops@123
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_export"
    static_configs:
      - targets: ["localhost:9100"]

  - job_name: pushgateway
    honor_labels: true
    static_configs:
      - targets: ["localhost:9091"]
```

now just run these commands
```
promtool check config /etc/prometheus/prometheus.yml

curl -X POST -u admin:Devops@123 http://localhost:9090/-/reload

sudo systemctl daemon-reload


```


and you can see notification in the configured channel whenever there is high cpu or ram utilization:
![alt text](image-27.png)







 