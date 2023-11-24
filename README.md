# ALERTING USING PROMETHEUS, ALERT-MANAGER AND NODE-EXPORTER

**REPO: https://github.com/muralialakuntla3/prometheus-alertmanager.git**

## STEPS:
    Setup Prometheus, Alertmanager and Node Exporter Servers
    Create service file for Prometheus, Alertmanager and Node Exporter
    Configure Prometheus with Node exporter server
    Configure Prometheus with Alertmanager server
    Real time Testing with our Nginx server
    Configure alertmanager to fire slack notification
## Step-1: Setup Prometheus, Alertmanager and Node Exporter Servers
    Launch 4 servers
    Capacity: 2 cpu & 4 gb
    Ebs: 10 gb
    Ports: 22, 80,443, 9090, 9100, 9093
### NGINX SERVER:
Login to nginx server and install nginx
Note: this server is for checking http status with ip addr..
    sudo apt update
    sudo apt install nginx -y
### PROMETHEUS SERVER:
Login to prometheus server and install prometheus
Visit: prometheus.io
    wget https://github.com/prometheus/prometheus/releases/download/v2.45.1/prometheus-2.45.1.linux-amd64.tar.gz
    tar -xvf prometheus-2.45.1.linux-amd64.tar.gz 
### NODE-EXPORTER SERVER:
Login to node server and install node-exporter
Visit: prometheus.io
    wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
    tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
### ALERT-MANAGER SERVER:
Login to alert-manager server and install alertmanager
Visit: prometheus.io
    wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz
    tar -xvf alertmanager-0.26.0.linux-amd64.tar.gz

## Step-2: Create service file for Prometheus, Alertmanager and Node Exporter

### PROMETHEUS SERVICE FILE:
    cd prometheus-2.45.1.linux-amd64/
    ./prometheus <!—-------for manually running prometheus ---->
Browse the prom-ip:9090
#### SERVICE FILE:
    sudo cp -rf prometheus-2.45.1.linux-amd64/* /usr/local/bin/prometheus
    sudo vi /etc/systemd/system/prometheus.service
**service file**
    [Unit]
    Description=Prometheus Service
    After=network.target
    [Service]
    Type=simple
    ExecStart=/usr/local/bin/prometheus/prometheus --config.file=/usr/local/bin/prometheus/prometheus.yml
    [Install]
    WantedBy=multi-user.target
**commands to run service file**
    sudo systemctl daemon-reload
    sudo service prometheus start
    sudo service prometheus status

### NODE-EXPORTER SERVICE FILE:
    cd node_exporter-1.7.0.linux-amd64/
    ./node_exporter <!—-------for manually running prometheus ---->
Browse the prom-ip:9100
SERVICE FILE:
sudo cp -rf node_exporter-1.7.0.linux-amd64/* /usr/local/bin/node-exporter
sudo vi /etc/systemd/system/node-exporter.service
[Unit]
Description=PrometheusNode Exporter  Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/node-exporter/node_exporter
[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo service node-exporter start
sudo service node-exporter status
ALERT-MANAGER SERVICE FILE:
cd alertmanager-0.26.0.linux-amd64/
./alertmanager —-------for manually running prometheus
Browse the prom-ip:9093
SERVICE FILE:
sudo cp -rf alertmanager-0.26.0.linux-amd64/* /usr/local/bin/alertmanager
sudo vi /etc/systemd/system/alertmanager.service
[Unit]
Description=Alertmanager  Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager/alertmanager --config.file=/usr/local/bin/alertmanager/alertmanager.yml
[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo service alertmanager start
sudo service alertmanager status
Step-3: Configure Prometheus with Node exporter server
Login to prometheus server
Add scrap_config in prometheus.yml under scrap config
Edit targets & node_exporter ip
cd prometheus-2.45.1.linux-amd64/
sudo vi prometheus.yml
scrape_configs:
   - job_name: ‘node_exporter’
     static_configs:
     - targets: ['node-exporter-ip:9100']

Step-4: Configure Prometheus with Alertmanager server

Login to prometheus server
Update alertmanager ip address
Specify rule files and configuration
cd prometheus-2.45.1.linux-amd64/
sudo vi prometheus.yml
# alertmanger configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 'alertmanager-ip:9093'
# alerts rulefile configuration
rule_files:
  - alert.rules.yml
Now create alert.rules.yml file in prometheus.yml file location
We are creating 4 alerts
vi alert.rules.yml
groups:
- name: alert.rules
  rules:

  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: "critical"
    annotations:
      summary: "Endpoint {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
  
  - alert: HostOutOfMemory
    expr: node_memory_MemAvailable / node_memory_MemTotal * 100 < 25
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host out of memory (instance {{ $labels.instance }})"
      description: "Node memory is filling up (< 25% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: HostOutOfDiskSpace
    expr: (node_filesystem_avail{mountpoint="/"}  * 100) / node_filesystem_size{mountpoint="/"} < 50
    for: 1s
    labels:
      severity: warning
    annotations:
      summary: "Host out of disk space (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 50% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

  - alert: HostHighCpuLoad
    expr: (sum by (instance) (irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m]))) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Host high CPU load (instance {{ $labels.instance }})"
      description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"


Step-5: Real time Testing with our Nginx server
Now goto prometheus browser
Status
Rules
Alert rules
Configuration
Here you will see complete prometheus.yml
Alerts
Here you will see alerts
Testing:
Down one server node-exporter/nginx server
You will get alert in prometheus
Step-6: Configure alertmanager to fire slack notification
Webhook integration in slack:
Webhook: Incoming-webhook
Select channel
Add webhook integration
Copy webhook url
Config alertmanager to get slack notification:
Login to alertmanager server
Edit alertmanager.yml
Configure receiver details to get slack notification
vi alertmanager.yml
global:
  resolve_timeout: 1m
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1m
  receiver: 'slack-notifications'
receivers:
- name: 'slack-notifications'
  slack_configs:
  - api_url: "put your slack webhook api here"
    channel: '#slack channel name here'
    send_resolved: true

    icon_url: https://avatars3.githubusercontent.com/u/3380462
    title: |-
      [{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .CommonLabels.alertname }} for {{ .CommonLabels.job }}
      {{- if gt (len .CommonLabels) (len .GroupLabels) -}}
        {{" "}}(
        {{- with .CommonLabels.Remove .GroupLabels.Names }}
          {{- range $index, $label := .SortedPairs -}}
            {{ if $index }}, {{ end }}
            {{- $label.Name }}="{{ $label.Value -}}"
          {{- end }}
        {{- end -}}
        )
      {{- end }}
    text: >-
      {{ with index .Alerts 0 -}}
        :chart_with_upwards_trend: *<{{ .GeneratorURL }}|Graph>*
        {{- if .Annotations.runbook }}   :notebook: *<{{ .Annotations.runbook }}|Runbook>*{{ end }}
      {{ end }}

      *Alert details*:

      {{ range .Alerts -}}
        *Alert:* {{ .Annotations.title }}{{ if .Labels.severity }} - `{{ .Labels.severity }}`{{ end }}
      *Description:* {{ .Annotations.description }}
      *Details:*
        {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
        {{ end }}
      {{ end }}

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']






